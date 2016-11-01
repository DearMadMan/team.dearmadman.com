title: Laravel 源码分析系列 —— 单一职责
date: 2016-07-19 13:29:22
tags: [php, laravel]
---

# 面向对象设计原则

在正式分析 Laravel 源码之前，我们应该先来回顾一下面向对象的基本设计原则，这个由 罗伯特·C·马丁 所提及的 SOLID 原则可以帮助我们很好的应对代码的扩展与维护，这也是 Laravel 源码如此优雅的关键。

你一定听说过“代码的坏味道”，没错，使用这些设计原则，你可以很好的扫清代码中的坏味道，那么接下来，我们就来聊一聊这五个设计原则：
- Single Reponsibility Principle 单一职责原则
- Open Closed Principle 开放封闭原则
- Liskov Substitution Principle 里氏替换原则
- Interface Segregation Principle 接口隔离原则
- Dependency Invertion Principle 依赖反转原则

## 单一职责

> 它规定一个类，只能有一个引起变化的原因。

你可能不太理解这个变化具体指的是什么，我们可以理解为这个变化就指这个类的职责，它的意思就是一个类，应该只有一个职责，只有这个职责变化时才能引起它的变化，也就是只有职能改变的时候，你才能修改这个类。

一个类的耦合度，是判断一个类设计好坏的标准。如果说一个类它的职责高度集中，那么我们就说它是一个高内聚的类，而如果一个类包含了太多的职责，那么它就是一个具有耦合性的类。通常我们希望一个类它是高内聚低耦合的。

> 如果一个类承担的职责过多，就等于把这些职责耦合在一起了。一个职责的变化可能会削弱或者抑制这个类完成其他职责的能力。这种耦合会导致脆弱的设计，当发生变化时，设计会遭受到意想不到的破坏。而如果想要避免这种现象的发生，就要尽可能的遵守单一职责原则。此原则的核心就是解耦和增强内聚性。


那么为什么要遵循单一职责呢？

我们都知道，软件开发的过程中大多数时候都是类与类之间的交互，那么在理想情况下，我们一定会希望当一个类改变时，不会引起其它类的连锁变化。如果说类 A 具有职能 A，类 B 具有职能 B，那么理想情况下，如果类 A 根据需求的变化而进行了修改，类 B 是不应该也同时被修改的。如果说 A 修改了，我们也必须要修改 B，那么说明这个类 B 肯定具有太多的职责，而导致它违背了单一职责原则。

所以你应该能看的出，单一职责其实是要求一个类的功能边界和它的职责应该是非常狭窄和紧凑的，它应该只做它该做的事情，并且它不应该被它所依赖的类的任何变化所影响。如果它的依赖变化时它也必须要跟着变化，那说明它知道的太多了。

> **你知道的太多了**
> 
> 一只猫拿着把枪追杀一只老鼠 
> 老鼠求它别杀它
> 猫说 可以 但你要回答我一个问题 1加1等于几
> 等于二 老鼠说
> 猫一枪把老鼠打死了 
> 老鼠垂死挣扎问道 我答对了为什么还要杀我 
> 猫回答道 你知道的太多了

## 实战

为了更好的理解这个原则，我们来举个例子看一下，考察一下下面的类：

