# Entitas

C#版的ECS框架，有Unity版本。

## 特点

- 使用Feather控制System的执行顺序。

- 拥有一个独特的ReactiveSystem，从而使部分System不必每帧执行。

## Entity

## Component

## System

所有System实现接口`ISystem`。

#### 1. `public interface ISystem {}`

#### 2. `public interface IInitializeSystem : ISystem { void Initialize(); }`

实例化时执行

#### 3. `public interface IExecuteSystem : ISystem { void Execute(); }`

帧逻辑

#### 4. `public interface ICleanupSystem : ISystem { void Cleanup(); }`

清除之前执行

#### 5. `public interface ITearDownSystem : ISystem { void TearDown(); }`

拆毁时执行

**统一管理器**：

`public class Systems : IInitializeSystem, IExecuteSystem, ICleanupSystem, ITearDownSystem { /*逻辑*/ }`

该类中包含四个List，分别对应`IInitializeSystem`、`IExecuteSystem`、`ICleanupSystem`、`ITearDownSystem`。

通过这四个List控制整个ECS系统中所有System的生命周期。

#### 6. `public interface IReactiveSystem : IExecuteSystem { void Activate(); void Deactivate(); void Clear(); }`

反应式System

由于继承自`IExecuteSystem`，所以生命周期由`Systems`的`List<IExecuteSystem>`来管理，在被调用的`Execute()`中，通过`Filter`对`Entity`进行组件类型筛选过滤，符合条件的`Entity`加入缓存字段`_buffer`，最终通过`_buffer`中的Entity数量是否为0，判断是否执行逻辑。

#### 7. `public abstract class JobSystem<TEntity> : IExecuteSystem where TEntity : class, IEntity { /*逻辑*/ }`

Entitas中的JobSystem，开发者注释提示不能在该System中进行组件的Add和Replace，感觉没什么用。

#### 8. `public abstract class MultiReactiveSystem<TEntity, TContexts> : IReactiveSystem where TEntity : class, IEntity where TContexts : class, IContexts`


