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

## ECS和传统OOP区别

- OOP是面向对象，对象包含了属性、方法，并且通常进行了继承并且重写，从而形成一个独立的对象。

- ECS中是面向数据，`Entity`（实体）包含了其拥有的`Components`（组件数据），并不具有任何行为逻辑。为了实现逻辑就添加一个`System`（系统），=系统会对符合`Component`组过滤条件的`Entity`进行逻辑处理，通常结果都是增删改`Component`数据。

## 思想

思想是ECS模式的核心，从框架到前端开发再到策划。

- 改变传统的开发流程，需要将OOP模式下的对象进行更深层次的细化，将各种数据抽离为Component的形式，将各种Manager的逻辑抽离为多个System。
- 低耦合，例如在处理玩家角色的逻辑时，往往特别复杂，在ECS框架下可以将玩家角色的行为细化为多个System，让多人并行开发，每个System的作用大多都是让增删改玩家角色Entity身上的Component数据。而且每个System关心的Component通常并不相同，所以如果逻辑出了Bug可以很容易定位到问题代码段。
- 在后期需要修改逻辑时，根据功能点细化的粒度找到处理对应逻辑的System进行修改即可。

## Entitas与Entities的区别

`Entitas`和`Entities`都是Unity中可以使用的ECS框架。

#### Entitas

框架精炼且完善，作为一款第三方插件可以实现代码的移植。

#### Entities

作为官方出品的ECS框架拥有很多Entitas做不到的优化，性能直接得到了提升，但却造成了与引擎的高耦合。

1. 内存连续

Unity的ECS框架实现的内存连续，使用基于Chunk迭代的NativeArray数组装Component，是一种对内存友好的数据结构，在使用的时候由于粒度很细直接提升了***Cache Hit***，从而提升性能。

2. 多线程

Unity的ECS框架实现的JobSystem，实现了基于ECS的多线程，按照JobSystem的代码规范进行编程，能很方便、安全的使用多线程。
