title: 众筹系统中抽奖模块一种简单设计方案
date: 2016-07-22 09:09:32
tags: [php, laravel]
---

# 抽奖模块

众筹系统的奖励回报中有一种抽奖回报，这种回报和一元云购的回报模式是及其相似的，在这种需求中，用户可以选择进行奖励回报，并在支付完成之后获得抽奖号码，如果众筹成功，那么根据相应的抽奖规则来匹配抽出中奖号码。目前有多种抽奖规则，有根据彩票结果进行相关计算的，有根据股票进行计算的。这里我们根据需求以股票为相关计算基础。

## 抽奖规则

> 获得抽奖号
> 购买抽奖回报项时按购买顺序产生抽奖号。
> 开奖日期
> 项目成功结束后的下一个周一开奖。如遇周一为非工作日，则延至周一后的第一个工作日开奖。
> 中奖号计算方法：
> *开奖日收盘时的上证指数 6 位数字 & 收盘时的深证成指 7 位数字，两组数字排列得到的 13 位数；
> *将此 13 位数整除以项目结束时总抽奖号个数，得到的余数加 1。即得到该项目的中奖号码。
> *如果项目开多个奖，则以项目公示“每 X 个参与者抽取 1 名中奖用户”中的 X 作为基数，第 N 个中奖号码 = 第一个中奖号码 +（N-1）* X
> *如果加值后抽奖号码超过了总抽奖号个数，则将抽奖号码减去抽奖号个数得到新的中奖号码。
> *每个抽奖号只有一次中奖机会，如果抽奖号重复中奖，则将该抽奖号码 +1 得到新的中奖号码。
> *如遇周末节假日等股市休市情况，收盘指数以上一个交易日数据为准。

## 需求分析

根据流程进行需求分析如下：
- 用户选择抽奖回报并支付
- 支付成功生成抽奖号
- 众筹成功，根据规则进行抽奖，并记录中奖号码
- 奖品分配

## 功能分析
- 需要一个生成抽奖号的功能
- 需要一个根据规则生成中奖号码的功能
- 生成的抽奖号需要入库
- 生成的中奖号需要匹配入库

## 数据库设计

```
`lotteries`:
  - integer `id` increments
  - integer `user_id` 
  - integer `project_id`
  - integer `order_id`
  - integer `number`
  - boolean `is_winner` default:false
```

## 职责分析
分析后得出只需以下职责：
- 生成抽奖号
- 抽奖号入库
- 中奖算法的封装
- 获取中奖号码
- 根据中奖号码匹配库中抽奖号及设置抽奖号状态

## 编码设计

lottery 类的设计：

```php
class Lottery extends Model {
  protected $guarded = ['id'];

  public function generateNumber($projectId) {
    return $this->where('project_id', $projectId)->count() + 1;
  }

  public function setWinner ($projectId, $number) {
    $lottery = $this->where(['project_id' => $projectId, 'number' => $number])->first();
    $lottery->is_winner = true;
    $lottery->save();
  }
  public function getCount ($projectId) {
    return $this->where('project_id', $projectId)->count();
  }
}
```

我们构造一个 LotteryWorker 类用来处理中奖相关的责任：

```php
use GuzzleHttp\Client;
class LotteryWorker {
  protected $lottery;
  protected $shenZhenIndex;
  protected $shangHaiIndex;
  public function __construct (Lottery $lottery) {
    $this->lottery = $lottery;
  }

  public function create ($attributes) {
    $attributes['number'] = $this->lottery->generateNumber($attributes['project_id']);
    return $this->lottery->create($attributes);
  }

  public function setWinner ($projectId, $number) {
    return $this->lottery->setWinner($projectId, $number);
  }

  public function getLuckyNumbers () {

  }

  public function counter ($projectId) {
    $shangHaiIndex = $this->getShangHaiCompositeIndex();
    $shenZhenIndex = $this->getShenZhenCompositeIndex();

    $number = join('', [$shangHaiIndex, $shenZhenIndex]);
    $count = $this->lottery->getCount($projectId);
    
    $winner = $number % $count + 1;

    $winners = [];
    array_push($winners, $winner);
    if ($prize->isMulti()) {
      for ($i = 1; $i< $prize->count; $i++) {
        $nextWinner = $winner + ($i - 1) * $prize->limit; 
        if ($nextWinner > $count) {
          $nextWinner -= $count;
        }
        while (array_search($nextWinner, $winners) !== false) {
          $nextWinner += 1;
          if ($nextWinner > $count) {
            $nextWinner -= $count;
          }
        }
        array_push($winners, $nextWinner);
      }
    }

    return $winners;
  }

  public function getShangHaiCompositeIndex () {
    if ($this->shangHaiIndex) return $this->shangHaiIndex;

    $client = new Client(['base_uri' => 'http://hq.sinajs.cn/']); 
    $content = $client->get('', ['query' => 'list=s_sh000001'])
        ->getBody()
        ->getContents();
    $content = iconv('gbk', 'utf-8', $content);

    preg_match('/"([\S]+)"/', $content, $mathes);
    if (count($mathes) !== 2) {
      throw new Exception('Date In Wrong Format!');
    }
    if (strpos($mathes[1], '上证指数,') === false) {
      throw new Exception('Data In Wrong Format!');
    }
    $index = explode(',', $mathes[1])[1];
    return $this->numberFormat($index, 6);
  }

  public function getShenZhenCompositeIndex () {
    if ($this->shenZhenIndex) return $this->shenZhenIndex;

    $client = new Client(['base_uri' => 'http://hq.sinajs.cn/']);
    $content = $client->get('', ['query' => 'list=s_sz399001'])
        ->getBody()
        ->getContents();
    $content = iconv('gbk', 'utf-8', $content);

    preg_math('/"([\S]+)"/', $content, $mathes);
    if (count($matches) !== 2) {
      throw new Exception('Data In Wrong Format!');
    }
    if (strpos($mathes[1], '深证成指,') === false) {
      throw new Exception('Data In Wrong Format!');
    }
    $index = explode(',', $mathes[1])[1];
    return $this->numberFormat($index, 7);
  }
  
  public function numberFormat ($number, $count) {
    $number = str_replace('.', '', $number);
    return substr($number, 0, $count);
  }

}
```

