# Entitas 1.13.0

C#版的ECS框架，有Unity版本。

## 特点

- 使用Feather控制System的执行顺序。

- 拥有一个独特的ReactiveSystem，从而使部分System不必每帧执行。

## Entity

所有Entity实现接口IEntity。

---

`public interface IEntity : IAERC{ /*逻辑*/ }`

```C#
public interface IAERC{
    int retainCount{ get; }
    void Retain(object owner);
    void Release(object owner);
}
```

---

 Entity  : `public class Entity : IEntity`

包含三个事件对应组件的`Add`、`Removed`和`Replaced`行为。

`public int creationIndex` : 每个Entity的专属ID

组件操作：

```C#
1. public void AddComponent(int index, IComponent component) { /*逻辑*/ }
2. public void RemoveComponent(int index) { /*逻辑*/ }
3. public void ReplaceComponent(int index, IComponent component) { /*逻辑*/ }
4. public IComponent GetComponent(int index) { /*逻辑*/ }
```



在添加、移除、替换和获取组件的时候，都使用了一个`index`字段，在Entitas中这是每个Component的唯一标识。同时会向对应事件添加此次操作行为。

## Component

所有Component实现接口`Icomponent`。

`public interface IComponent{}`

## System

所有System实现接口`ISystem`。

#### 1. `public interface ISystem {}`

#### 2. `public interface IInitializeSystem : ISystem { void Initialize(); }`

实例化时执行

#### 3. `public interface IExecuteSystem : ISystem { void Execute(); }`

帧逻辑

#### 4. `public interface ICleanupSystem : ISystem { void Cleanup(); }`

清除之前执行

#### 5. `public interface ITearDownSystem : ISystem { void TearDown(); }`

拆毁时执行

---

**统一管理器**：

```C#
public class Systems : IInitializeSystem, IExecuteSystem, ICleanupSystem, ITearDownSystem { /*逻辑*/ }
```

该类中包含四个List，分别对应`IInitializeSystem`、`IExecuteSystem`、`ICleanupSystem`、`ITearDownSystem`。

通过这四个List控制整个ECS系统中所有System的生命周期。

---

#### 6. `public interface IReactiveSystem : IExecuteSystem { void Activate(); void Deactivate(); void Clear(); }`

反应式System

由于继承自`IExecuteSystem`，所以生命周期由`Systems`的`List<IExecuteSystem>`来管理，在被调用的`Execute()`中，通过`Filter`对`Entity`进行组件类型筛选过滤，符合条件的`Entity`加入缓存字段`_buffer`，最终通过`_buffer`中的Entity数量是否为0，判断是否执行逻辑。

#### 7. `public abstract class JobSystem<TEntity> : IExecuteSystem where TEntity : class, IEntity { /*逻辑*/ }`

JobSystem使用实体的子集调用`Execute(entities)`，并将工作负载分配给指定数量的线程。

在Entitas中编写多线程代码时，不要使用生成的方法，如`AddXyz()`和`ReplaceXyz()`。

#### 8. `public abstract class MultiReactiveSystem<TEntity, TContexts> : IReactiveSystem where TEntity : class, IEntity where TContexts : class, IContexts`

后续添加，大致为满足多种条件才触发的ReactiveSystem。

如果存在基于指定收集器的更改，则reactiveSystem将调用`Execute(entities)`，并且只传入更改的实体。一个常见的用例是对变化做出反应，例如改变实体的位置以更新相关游戏对象的gameobject.transform.position。

## Matcher

**匹配器**，作为一种分组工具。

成员函数：`AllOf()`、`AnyOf()`、`NoneOf()`

通过匹配器的这三种成员函数的多种组合，可以获得不同的`Components`的组合方式。

## Group

Group即为能通过某个`Matcher`的`Entity`的集合。

其中也包含了三种针对组中Entity的操作事件。

```C#
1. public event GroupChanged<TEntity> OnEntityAdded;
2. public event GroupChanged<TEntity> OnEntityRemoved;

3. public event GroupUpdated<TEntity> OnEntityUpdated
```

PS：个人感觉这里跟`ReactiveSystem`的响应有关系。






