title: laravel 基础教程 —— 集合
date: 2016-06-03 07:01:48
tags: [php, laravel]
---

# 集合

## 简介

`Illuminate\Support\Collection` 类提供了对数组数据流利操作的封装。比如，看下面的延时代码。我们会使用 `collect` 帮助方法从一个数组中创建一个新的集合实例，然后运行 `strtoupper` 方法转为大写再剔除集合中的空元素：

```php
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
  return strtoupper($name); 
})
->reject(function ($name) {
  return empty($name);
});
```

就如你所看到的，`Collection` 类允许你链式的调用它的方法，就是这样提供了一种流利的映射的执行能力的同时缩小了底层的数组。通常情况下，所有的 `Collection` 方法都会返回一个全新的 `Collection` 实例。

## 创建集合

就如上面的代码，`collect` 帮助方法会根据给定的数组返回一个新的 `Illuminate\Support\Collection` 实例。所以，创建一个集合是非常的简单：

```php
$collection = collect([1, 2, 3]);
```

默认的，Eloquent 模型集合总是返回 `Collection` 的实例。事实上，你可以在你的应用中任意位置使用 `Collection` 类。

## 可用的方法

在文档的余下部分，我们将探讨 `Collection` 类中所有可用的方法。你应该记住，所有的这些方法都可以链式调用来流利的操作底层的数组。另外，几乎每一个方法都会返回一个新的 `Collection` 实例，这允许你可以在必要时保存原始的集合。

## 方法名单

`all()`

`all` 方法会简单的返回集合中所包含的底层数组:

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

`avg()`

`avg` 方法会返回集合中所有项的平均值：

```php
collect([1, 2, 3, 4, 5])->avg();

// 3
```

如果集合中包含的是嵌套的数组或者对象，你应该传递 key 来表明所需要计算的值的平均值：

```php
$collection = collect([
  ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
  ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->avg('pages');

// 636
```

`chunk()`

`chunk` 方法会根据给定的大小来将集合分割成多个小的集合：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->toArray();

// [[1, 2, 3, 4], [5, 6, 7]]
```

该方法在视图中使用类似于 Bootstrap 的网格系统时特别有用。想象一下你有一个关于 Eloquent 模型的集合想要在网格中显示：

```php
@foreach ($products->chunk(3) as $chunk)
  <div class="row">
    @foreach ($chunk as $product)
      <div class="col-xs-4">{{ $product->name }}</div>
    @endforeach
  </div>
@endforeach
```

`collapse()`

`collapse` 方法会将集合中的数组从多维坍塌到一维：

```php
$collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

`combine()`

`combine` 方法会将一个集合中的 keys 和另外一个集合或数组中的 values 相结合：

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

`contains()`

`contains` 方法来判断集合中是否含有给定的项：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

你也可以传递键值对到 `contains` 方法，这将会用来判断集合中是否含有给定的键值对项:

```php
$collection = collect([
  ['product' => 'Desk', 'price' => 200],
  ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

最后，你也可以传递一个匿名函数到 `contains` 方法来提供自己的真值判断逻辑：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function ($key, $value) {
  return $value > 5; 
}); 

// false
```

`count()`

`count` 方法会返回集合中项目的总数：

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

`diff()`

`diff` 方法用来比较集合中存在而其他集合或者原生 PHP 的数组不存在的值：

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

`diffKeys()`

`diffKeys` 方法用来比较集合中存在而其他集合或原生 PHP 数组不存在的键：

```php
$collection = collect([
  'one' => 10,
  'two' => 20,
  'three' => 30,
  'four' => 40,
  'five' => 50,
]);

$diff = $collection->diffKeys([
  'two' => 2,
  'four' => 4,
  'six' => 6,
  'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

`each()`

`each` 方法会迭代集合中的每一项，并将该项传递给所给定的回调：

```php
$collection = $collection->each(function ($item, $key) {
  // 
});

在回调中返回 `false` 将会中断迭代：

```php
$collection = $collection->each(function ($item, $key) {
  if (/* some condition */) {
    return false;
  } 
}); 
```

`every()`

`every` 方法会创建一个由每第 N 个元素（包含起始位）所组成的集合：

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->every(4);

// ['a', 'e']
```

你可以传递第二个参数来设置位移：

```php
$collection->every(4, 1);

// ['b', 'f']
```

`except()`

`except` 方法返回集合中的项，并在项目中剔除选定的键：

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

`filter()`

`filter` 方法会根据给定的回调的迭代结果进行过滤，如果回调返回的是真值，该项将会被保留：

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function ($value, $key) {
  return $value > 2; 
});

$filtered->all();

// [3, 4]
```
`rejct` 方法与 `filter` 方法相反。

`first()`

`first` 方法会返回集合中第一个回调返回真值的项:

```php
collect([1, 2, 3, 4])->first(function ($key, $value) {
  return $value > 2; 
});

// 3
```

你也可以调用无参数的 `first` 方法，该方法会返回集合中的第一个元素。如果集合是空的，则会返回 `null`：

```php
collect([1, 2, 3, 4])->first();

// 1
```

`flatMap()`

`flatMap` 方法会迭代处理所有元素，并将处理后的集合进行扁平化处理（从多维降为一维）：

```php
$collection = collect(
  ['name' => 'Sally'],
  ['school' => 'Arkansas'],
  ['age' => 28]
);

$flattened = $collection->flatMap(function ($values) {
  return strtoupper($values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => 28];
```

`flatten()`

`flatten` 方法将多规格的集合转换为单一格式的集合：

```php
$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

$flattened = $collection->flatten();

$flattened->all();

// ['taylor', 'php', 'javascript'];
```

你也可以传递一个“深度”参数到方法：

```php
$collection = collect([
  'Apple' => [
    ['name' => 'iPhone 6S', 'brand' => 'Apple'],
  ],
  'Samsung' => [
    ['name' => 'Galaxy S7', 'brand' => 'Samsung']
  ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
  [
    ['name' => 'iPhone 6S', 'brand' => 'Apple'],
    ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
  ]
 */
```

这里，调用不提供深度的 `flatten` 方法也会扁平化数组，这将导致返回 `['iPhone 6S', 'Apple', 'GalaxyS7', 'Samsung']`。提供深度可以使你限制拉平嵌套数组的层级。

`flip()`

`flip` 方法将会反转键值对：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['taylor' => 'name', 'laravel' => 'framework']
```

`forget()`

`forget` 方法根据所提供的键剔除集合中相应的项：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all();

// ['framework' => 'laravel']
```

> 注意： 不像其他的集合方法，`forget` 方法不会返回一个新的修改后的集合，它会直接修改当前的集合。

`forPage()`

`forPage` 方法将返回给定当前页所包含的项的集合：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

该方法需要传递当前页的数值及每页所包含的数目。

`get()`

`get` 方法将根据给定的键从集合中返回项，如果给定键不存在，则返回 `null`:

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('name');

// taylor
```

你可以传递第二个参数来作为默认值：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('foo', 'default-value');

// default-value
```

你也可以传递一个回调作为默认值。回调的结果将会被作为未找到键时的默认值：

```php
$collection->get('email', function () {
  return 'default-value'; 
});

// default-value
```


`groupBy()`

`groupBy` 方法将会根据给定的键将集合进行分组：

```php
$collection = collect([
  ['account_id' => 'account-x10', 'product' => 'Chair'],
  ['account_id' => 'account-x10', 'product' => 'Bookcase'],
  ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->toArray();

/*
  [
    'account-x10' => [
      ['account_id' => 'account-x10', 'product' => 'Chair'],
      ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ],
    'account-x11' => [
      ['account_id' => 'account-x11', 'prodcut' => 'Desk'],
    ],
  ]
 */
```

除了传递一个字符串 `key` 之外，你也可以传递一个回调函数。回调函数应该返回你所期望进行分组的键的值：

```php
$grouped = $collection->groupBy(function ($item, $key) {
  return substr($item['account_id'], -3); 
});

$grouped->toArray();

/*
  [
    'x10' => [
      ['account_id' => 'account-x10', 'product' => 'Chair'],
      ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ],
    'x11' => [
      ['account_id' => 'account-x11', 'product' => 'Desk'],
    ],
  ]
 */
```

`has()`

`has` 方法用来判断集合中是否存在给定的键:

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk']);

$collection->has('email');

// false
```

`implode()`

`implode` 方法会将集合中的项连接成字符串。其参数取决于集合中的项目类型。

如果集合中包含的是键值对数组或对象，你就应该传递一个键到方法来表明你想要连接的值，然后紧跟着一个胶连字符串参数：

```php
$collection = collect([
  ['account_id' => 1, 'product' => 'Desk'],
  ['account_id' => 2, 'product' => 'Chair'],
]);

$collection->implode('product', ', ');

// Desk, Chair
```

如果集合只是包含了简单的字符串或者数组类型的值，你可以直接传递胶连字符串参数到方法：

```php
collect([1, 2, 3, 4, 5])->implode('-');

// '1-2-3-4-5'
```

`intersect()`

`intersect` 方法返回给定数组与集合的交集：

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

就如你所看到的，结果集合将保留原始集合的键。

`isEmpty()`

`isEmpty` 方法用来判断集合是否为空，如果集合为空，则返回 `true`，否则返回 `false`：

```php
collect([])->isEmpty();

// true
```

`keyBy()`

键值化给定键的集合：

```php
$collection = collect([
  ['product_id' => 'prod-100', 'name' => 'desk'],
  ['product_id' => 'prod-200', 'name' => 'chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
  [
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
  ]
 */
```

如果多个项目含有相同的键，那么只有最后一个项目会出现在新的集合中。

你也可以传递你自己的回调来返回集合键的值：

```php
$keyed = $collection->keyBy(function ($item) {
  return strtoupper($imte['product_id']) ;
}); 

$keyed->all();

/*
  [
    'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
  ]
 */
```

`keys()`

`keys` 方法返回集合中所有的键：

```php
$collection = collect([
  'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
  'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

`last()`

`last` 方法返回回调中最后一个返回真值的项:

```php
collect([1, 2, 3, 4])->last(function ($key, $value) {
  return $value < 3; 
});

// 2
```

你也可以调用无参数的 `last` 方法，它将返回集合中最后一个元素，如果集合为空，则返回 `null`:

```php
collect([1, 2, 3, 4])->last();

// 4
```

`map()`

`map` 方法会使用给定的回调来进行迭代集合中的所有项，在回调函数中可以自由的修改该项并进行返回，这样新的集合将包含修改后的值：

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function ($item, $key) {
  return $item * 2; 
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

> 注意：就像其他的集合方法一样，`map` 返回一个新的集合实例。它并不会突变原集合。如果你想要在原有集合中进行改变，你应该使用 `transform` 方法。

`max()`

`max` 方法返回给定键中最大的值：

```php
$max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

// 20

$max = collect([1, 2, 3, 4, 5])->max();

// 5
```

`merge()`

`merge` 方法会合并给定的数组到集合。数组中的值将会覆盖集合中相同键的值：

```php
$colletion = collect(['product_id' => 1, 'name' => 'Desk']);

$merged = $collection->merge(['price' => 100, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]
```

如果给定的数组的键是数值，则它的值将会被追加在集合的末尾：

```php
$collection = collect(['Desk', 'Chair']);

$merged = $collection->merge(['Bookcase', 'Door']);

$merged->all();

// ['Desk', 'Chair', 'Bookcase', 'Door']
```

`min()`

`min`方法会返回集合中给定键的最小值：

```php
$min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

// 10

$min = collect([1, 2, 3, 4, 5])->min();

// 1
```

`only()`

`only` 方法返回集合中的项，并修改项目中的键值只包含选定的键：

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

`pluck()`

`pluck` 方法将根据给定的键检索集合中所有项的值：

```php
$collection = collect([
  ['product_id' => 'prod-100', 'name' => 'Desk'],
  ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']
```

你也可以指定将结果如何键化：

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

`pop()`

`pop` 方法将从集合中剔除最后一个元素：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

`prepend()`

`prepend` 方法将在集合的起始端添加项目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4, 5]
```

你也可以传递第二个参数作为前置项目的键：

```php
$collection = collect(['one' => 1, 'two' => 2]);

$collection->prepend(0, 'zero');

$collection->all();

// ['zero' => 0, 'one' => 1, 'two' => 2]
```

`pull()`

`pull` 方法从集合中返回给定键的同时将其从集合中剔除：

```php
$collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

$collection->pull('name');

// 'Desk'

$collection->all();

// ['product_id' => 'prod-100']
```

`push()`

`push` 方法将向集合中添加给定元素到末尾：

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5);

$collection->all();

// [1, 2, 3, 4, 5]
```

`put()`

`put` 方法在集合中设置给定的键和值：

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk']);

$collection->put('price', 100);

$collection->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
```

`random()`

`random` 方法随机的从集合中返回元素：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (ertrieved randomly)
```

你也可以传递一个整型值到 `random`。如果整型值大于 `1`。则相应个数的随机项将会被返回：

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (retrieved randomly)
```

`reduce()`

`reduce` 方法将会缩小集合收集单个值。它会通过迭代的方式将其值传递给随后的迭代器：

```php
$collection = collect([1, 2, 3]);

$total = $collection->reduce(function ($carry, $item) {
  return $carry + $item; 
});

// 6
```

 上面的 `$carry` 在第一个迭代器中将会是 `null`；你也可以指定一个起始值到第二个参数：

 ```php
 $collection->reduce(function ($carry, $item) {
   return $carry + $item;
 }, 4);

 // 10
 ```

 `reject()`

 `reject` 方法使用给定的回调从集合中返回过滤的值。你需要在回调中返回 `true` 来让其从结果中移除：

 ```php
 $collection = collect([1, 2, 3, 4]);

 $filtered = $collection=>reject(function ($value, $key) {
   return $value > 2;
 });

 $filtered->all();

 // [1, 2]
 ```

 `reverse()`

 `reverse` 方法会逆序排列集合中的项目：

 ```php
 $collection = collect([1, 2, 3, 4, 5]);

 $reversed = $collection->reverse();

 $reversed->all();

 // [5, 4, 3, 2, 1]
 ```

 `search()`

 `search` 方法根据给定的值搜索集合中的项，如果找到则返回该项的键，如果未找到则返回 `false`:

 ``` php
 $collection = collect([2, 4, 6, 8]);

 $collection->search(4);

 // 1
 ```

搜索使用的是疏松的比较，如果需要进行严格比较，你应该传递 `true` 作为第二个参数：

```php
$collection->search('4', true);

// false
```

另外，你也可以传递一个回调作为搜索的结果判断依据，在回调中首个返回真值的项目将会被检索到：

```php
$collection->search(function ($item, $key) {
  return $item > 5; 
});

// 2
```

`shift()`

`shift` 方法会从集合中返回首个元素的同时从集合中剔除：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift();

// 1

$collection->all();

// [2, 3, 4, 5]
```

`shuffle()`

`shuffle` 方法随机打乱集合中项目的顺序：

```php
$collect = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] // (generated randomly)
```

`slice()`

`slice` 方法根据给定的索引返回集合中的一小片：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$slice = $collection->slice(4);

$slice->all();

// [5, 6, 7, 8, 9, 10]
```

如果你想要限制返回的切片的大小，你可以传递期望的大小到第二个参数：

```php
$slice = $collection->slice(4, 2);

$slice->all();

// [5, 6]
```

返回的切片将会有一个新的数字索引键。如果你想要保留原数组的键，你需要传递 `true` 到第三个参数。

`sort()`

`sort` 方法将对集合进行排序：

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3 ,4 ,5]
```

被排序的集合会保留原始的数组键。上面的例子中我们使用了 `values` 方法来重置了键到连续的数字索引。

对于嵌套的数组或者对象的排序，请参照 `sortBy` 和 `sortByDesc` 方法。

如果你的排序需要更多的逻辑支持，你可以传递一个回调到 `sort` 方法。你可以参考 PHP 文档的 [usort](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), 集合的 `sort` 方法就是在基于该方法的。

`sortBy()`

`sortBy` 方法根据给定的键进行集合排序：

```php
$collection = collect([
  ['name' => 'Desk', 'price' => 200],
  ['name' => 'Chair', 'price' => 100],
  ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortBy('price');

$sorted->values()->all();

/*
  [
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
    ['name' => 'Desk', 'price' => 200],
  ]
 */
```

被排序的集合会保留原始的数组键。上面的例子中我们使用了 `values` 方法来重置了键到连续的数字索引。

你也可以传递你自己的回调作为排序的逻辑依靠：

```php
$collection = collect([
  ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
  ['name' => 'Chair', 'colors' => ['Black']],
  ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$sorted = $collection->sortBy(function ($product, $key) {
  return count($product['colors']);
});

$sorted->values()->all();

/*
  [
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
  ]
 */
```

`sortByDesc()`

该方法与 `sortBy` 方法有相同的运作方式，但是它是以集合中相反的顺序进行排序。

`splice()`

`splice` 方法根据指定的索引来从集合中剔除一片:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2);

$chunk->all();

// [3, 4, 5]

$collection->all();

// [1, 2]
```

你也可以传递第二个参数到方法来限制切片的大小：

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 4, 5]
```

另外，你可以传递第三个参数来作为从原集合中剔除元素的补偿：

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1, [10, 11]);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 10, 11, 4, 5]
```

`sum()`

`sum` 方法返回集合中所有项的和：

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

如果集合中包含的是嵌套的数组或者对象。你可以传递键来决定需要求和的值：

```php
$collection = collect([
  ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
  ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->sum('pages');

// 1272
```

另外，你也可以传递一个回调来决定你所需要进行求和的值：

```php
$collection = collect([
  ['name' => 'Chair', 'colors' => ['Black']],
  ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
  ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$collection->sum(function ($product) {
  return count($product('colors')) 
});
```

`take()`

`take` 方法从集合中返回给定数值的项目：

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(3);

$chunk->all();

// [0, 1, 2]
```

你也可以传递一个负值到方法，它将从集合的结尾开始返回：

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(-2);

$chunk->all();

// [4, 5]
```

`toArray()`

`toArray` 方法会将集合退化为原生的 PHP 数组。如果集合的值列是 Eloquent 模型。模型也会被转化为数组：

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toArray();

/*
  [
    ['name' => 'Desk', 'price' => 200],
  ]
 */
```

> 注意：`toArray` 也会转化所有的嵌套的对象到数组。如果你希望得到底层的原始数组，你可以使用 `all` 方法。

`toJson()` 

`toJson` 方法转化集合到 JSON：

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toJson();

// '{"name":"Desk","price":200}'
```

`transform()`

`transform` 通过一个回调函数迭代处理集合中的项目。集合中的项目会在被迭代的结果所替换：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function ($item, $key) {
  return $item * 2; 
});

$collection->all();

// [2, 4, 6, 8, 10]
```
> 注意：不像其他的集合方法，`transform` 会修改原始的集合自身。如果你希望创建一个新的集合来替代，请使用 `map` 方法。

`union()`

`union` 方法添加给定的数组到集合。如果给定的数组中包含的键在集合中已经存在，那么集合中的键将会被保留：

```php
$collection = collect([1 => ['a'], 2 => ['b']]);

$union = $collection->union([3 => ['c'], 1 => ['b']]);

$union->all();

// [1 => ['a'], 2 => ['b'], 3 => ['c']]
```

`unique()`

`unique` 方法将返回所有在集合中具有独特性（去重）的项目：

```php
$collection = collect([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();

$unique->values()->all();

// [1, 2, 3, 4]
```

返回的集合中保留了原始的数组键，上面的例子用使用了 `values` 方法来重置键到连续的数组索引。

当对嵌套的数组或者对象进行运算时，你应该制定一个键来进行唯一性的判定：

```php
$collection = collect([
  ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
  ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
  ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
  ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
  ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
]);

$unique = $collection->unique('brand');

$unique->values()->all();

/*
  [
    ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone']
  ]
 */
```

你也可以传递自己的回调到方法来进行判定：

```php
$unique = $collection->unique(function ($item) {
  return $item['brand'].$item('type'); 
});

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]
 */
```

`values()`

`values` 方法返回一个键经过重置成连续整型索引的新集合：

```php
$collection = collect([
  10 => ['product' => 'Desk', 'price' => 200],
  11 => ['product' => 'Desl', 'price' => 200],
]);

$values = $collection->values();

$values->all();

/*
  [
    0 => ['product' => 'Desk', 'price' => 200],
    1 => ['product' => 'Desk', 'price' => 200],
  ]
 */
```

`where()`

`where` 方法根据指定的键值对来进行集合的过滤：

```php
$collection = collect([
  ['product' => 'Desk', 'price' => 200],
  ['product' => 'Chair', 'price' => 100],
  ['product' => 'Bookcase', 'price' => 150],
  ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->where('price', 100);

$filtered->all();

/*
  [
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Door', 'price' => 100],
  ]
 */
```

`where` 方法会使用严格的比较模式。你可以使用 `whereLoose` 方法来使用疏松的比较模式。

`whereLoose()`

`whereLoose`

该方法与 `where` 方法的运作相同，只是其比较值的方式是疏松模式。

`whereIn()`

`whereIn` 方法根据给定的键值对中的值组来对集合中的项目进行过滤：

```php
$collection = collect([
  ['product' => 'Desk', 'price' => 200],
  ['product' => 'Chair', 'price' => 100],
  ['product' => 'Bookcase', 'price' => 150],
  ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereIn('price', [150, 200]);

$filtered->all();

/*
  [
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Desk', 'price' => 200],
  ]
 */
```

`whereIn` 方法使用的是严格的比较模式，如果你需要使用疏松的比较模式请使用 `whereInLoose` 方法。

`whereInLoose()` 

该方法与 `whereIn` 方法的运作相同，只是其使用的是疏松的比较模式。

`zip()`

`zip` 方法会根据相应的索引将给定的数组中的值与集合中项目的值进行合并：

```php
$collection = collect(['Chair', 'Desk']);

$zipped = $collection->zip([100, 200]);

$zipped->all();

// [['Chair', 100], ['Desk', 200]]
```