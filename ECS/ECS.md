# ECS

## 简介

ECS即为Entity Component System的简称，是区别于传统OOP开发的一种模式。

传统的OOP开发模式由于大量的封装、继承多态，造成了代码逻辑的高耦合，面对修改逻辑等问题的时候往往会牵一发而动全身。

低耦合、可并发是ECS的特点。

- Entity：实体，通常为一个Int值。

- Component：组件，挂载在Entity上，记录数据，不包含任何逻辑。

- System：纯逻辑，通过Group的概念对所有Entity根据Entity所挂载的Component情况进行筛选分组，拿到一组Entity进行纯逻辑的运算，运算的结果往往是在该Entity上增删改Component中所记录的数据。

ECS系统的逻辑部分不存在继承的概念。

## 特点

#### 1. 思想

思想是ECS模式的核心，从框架到前端开发再到策划。

- 改变传统的开发流程，需要将OOP模式下的对象进行更深层次的细化，将各种数据抽离为多个Component，将各种逻辑抽离为多个System。



#### 2. 内存连续

Unity的ECS框架实现的内存连续，使用基于Chunk迭代的NativeArray数组装Component，在使用的时候由于粒度很细可以使Cache 命中率提升，从而提升性能。



#### 3. 多线程

Unity的ECS框架实现的JobSystem的调度系统，实现了一套Unity环境下的ECS多线程功能，按照JobSystem的代码规范进行编程，能很方便、安全的使用多线程。其中多线程需要设置Schedule的数量，一般为64。
