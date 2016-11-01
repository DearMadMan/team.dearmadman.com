title: 算法系列 —— 队列、栈
date: 2016-10-21 14:58:48
tags: [php, javascript]
---

## 队列

> 队列（queue）在计算机科学中，是一种先进先出的线性表。

> 它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。队列中没有元素时，称为空队列。


### 普通队列

#### 实现

##### PHP

``` php
$arr = ['PHP', 'JavaScript'];

// 入队
array_push($arr, 'Swift');

// 出队
array_shift($arr);
```

##### JavaScript

``` js
let arr = ['PHP', 'JavaScript']

// 入队
arr.push('Swift')

// 出队
arr.shift()
```

### 双端队列

> 双端队列（deque，全名double-ended queue）是一种具有队列和栈性质的数据结构。双端队列中的元素可以从两端弹出，插入和删除操作限定在队列的两边进行。
> 双端队列可以在队列任意一端入队和出队。此外，经常还会有一个查看（Peek）操作，返回该端的数据而不将其出队。

#### 实现

##### PHP

``` php
class DoubleEndedQueue 
{
    /**
     * The queue list.
     *
     * @var array
     */
    protected $queue;

    /**
     * Push an item at the end of the list.
     *
     * @param  mixed  $item
     * @return void
     */
    public function push($item)
    {
        array_push($this->queue, $item);
    }

    /**
     * Push an item at the beginning of the list.
     *
     * @param  mixed  $item
     * @return void
     */
    public function inject($item)
    {
        array_unshift($this->queue, $item);
    }

    /**
     * Pop the item at the end of the list.
     *
     * @param  mixed $item
     * @return void
     */
    public function pop()
    {
        array_pop($this->queue);
    }
    
    /**
     * Shift an item at the beginning of the list.
     *
     * @return void
     */
    public function shift()
    {
        array_shift($this->queue);
    }

    /**
     * Get an item at the beginning of the list.
     *
     * @return mixed
     */
    public function first() 
    {
        return reset($this->queue);
    }

    /**
     * Get an item at the end of the list.
     *
     * @return void
     */
    public function last()
    {
        return end($this->queue);
    }

    /**
     * Get the length of the list.
     *
     * @return void
     */
    public function getLength()
    {
        return count($this->queue);
    }
}
```

##### JavaScript

``` js
let arr = ['PHP', 'JavaScript'];

arr.push('Swift');

arr.unshift('Phython')

arr.pop()

arr.shift()

arr.length
```

### 优先队列

> 普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 (largest-in，first-out)的行为特征。

#### 实现

##### PHP

``` php
<?php

class PriorityQueue extends SplPriorityQueue
{
    public function compare($current, $next)
    {
        if ($current === $next) 
        {
            return 0;
        }

        return $current > $next ? 1 : -1;
    }
}

$queue = new PriorityQueue();

$queue->insert('PHP', 100);
$queue->insert('Swift', 80);
$queue->insert('JavaScript', 90);

$queue->setExtractFlags(PriorityQueue::EXTR_BOTH);

$queue->top();

while($queue->valid()) {
   print_r($queue->current());
   $queue->next(); 
}
```

##### JavaScript

``` js
class PriorityQueue
{
    constructor()
    {
        this.queue = []
    }

    insert(data, priority)
    {
        this.queue.push({data, priority})
    }

    next()
    {
       if (!this.queue.length) return false 

       let start = this.queue[0].priority
       let max = this.queue.reduce((priority, item) => priority > item.priority ? priority : item.priority, start)

       return this.queue.splice(
            this.queue.findIndex(item => item.priority === max ),
            1
       )
    } 
}

let queue = new PriorityQueue()

queue.insert('PHP', 100)
queue.insert('Swift', 80)
queue.insert('JavaScript', 90)

queue.next()
```

## 栈

> 堆栈（英语：stack），也可直接称栈（港澳台作堆叠），在计算机科学中，是一种特殊的串列形式的数据结构，它的特殊之处在于只能允许在链接串列或阵列的一端（称为堆叠顶端指标，英语：top）进行加入数据（英语：push）和输出数据（英语：pop）的运算。另外栈也可以用一维数组或连结串列的形式来完成。堆叠的另外一个相对的操作方式称为伫列。
由于堆叠数据结构只允许在一端进行操作，因而按照后进先出（LIFO, Last In First Out）的原理运作。

双端队列去掉首端插入、删除操作就是栈了。


