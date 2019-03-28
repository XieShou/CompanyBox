# ECS

## 简介

ECS即为Entity Component System的简称，是区别于传统OOP开发的一种模式。

传统的OOP开发模式由于大量的封装、继承多态，造成了代码逻辑的高耦合，面对修改逻辑等问题的时候往往会牵一发而动全身。

低耦合、可并发是ECS的特点。

- Entity：实体，为一个Int值，作为Component的索引。

- Component：组件，挂载在Entity上，记录数据，不包含任何逻辑。

- System：纯逻辑，可以有一些缓存变量。

  1. System写明自己处理拥有哪几种Component的Entity，从而拿到一组Group命名为：Entities，对Entities逐个进行遍历。

  2. System根据Entity身上的各个Component包含的数据进行逻辑运算，运算的结果通常是在该Entity上Add、Replace、Remove Compoent数据。

ECS系统的逻辑部分不存在继承的概念。

## 特点

#### 1. 思想

思想是ECS模式的核心，从框架到前端开发再到策划。

- 改变传统的开发流程，需要将OOP模式下的对象进行更深层次的细化，将各种数据抽离为多个Component，将各种逻辑抽离为多个System。
- 低耦合，例如在处理玩家角色的逻辑时，往往特别复杂，在ECS框架下可以将玩家角色的行为细化为多个System，让多人并行开发，每个System的作用大多都是让增删改玩家角色Entity身上的Component数据。而且每个System关心的Component通常并不相同，所以如果逻辑出了Bug可以很容易定位到问题代码段。
- 在后期需要修改逻辑时，根据功能点细化的粒度找到处理对应逻辑的System进行修改即可。

#### 2. 内存连续

Unity的ECS框架实现的内存连续，使用基于Chunk迭代的NativeArray数组装Component，在使用的时候由于粒度很细可以使Cache 命中率提升，从而提升性能。

从操作系统批量申请内存，提前为Component申请额外的空间。

#### 3. 多线程

Unity的ECS框架实现的JobSystem的调度系统，实现了一套Unity环境下的ECS多线程功能，按照JobSystem的代码规范进行编程，能很方便、安全的使用多线程。其中多线程需要设置Schedule的数量，一般为64。
