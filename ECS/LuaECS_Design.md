# *`Lua ECS`*

- Unity的热更新方案大部分都是由Lua实现

- Lua的轻量级

---

### 1. Entity

- 一个number类型的数值。作为唯一标识

- 重载 `+`、`-`操作符对应*Component*操作,返回一个*Match*。

- *IEntity*

```lua
IEntity = { index = nil }
```

---

### 2. Component

- 其中包含必要的属性字段
- 每个*Component*对应一个唯一的`int`值
- *IComponent*

```lua
IComponent = { index = nil }
```

---

### 3. System

```lua
local ISystem = { index = nil,
ecs_system_type = ECS_SYSTEM_TYPE.InitializeSystem or ECS_SYSTEM_TYPE.ExecuteSystem or ECS_SYSTEM_TYPE.CleanupSystem or ECS_SYSTEM_TYPE.TearDownSystem}
local IInitializeSystem : ISystem { void Initialize(); }
local IExecuteSystem : ISystem { void Execute(); }
local ICleanupSystem : ISystem { void Cleanup(); }
local ITearDownSystem : ISystem { void TearDown(); }
```

---

### 4. Match

- 对*Component*的匹配器、筛选器
  
  ```lua
  local match = component_1 + component_2
  
  Match = { components_array = {
  component_1.index = component_1, 
  component_2.index = component_2} }
  ```

---

### 5. Group

符合某个*Match*规则的*Entity*的集合

---

### 6. Archetype

一组Group，作为预制体。

---

### 7. Feature

一组*System*的集合体

---

### 8. World

```lua
local ECS_SYSTEM_BUFFER = {}
local ECS_SYSTEM_FILE_PATH = { --需要加载的System的路径 }

function WORLD:ADD(system)
    if system and system.ecs_system_type then
        local type = system.ecs_system_type
        ECS_SYSTEM_BUFFER[type][system.index] = system
    end
end

function WORLD:Initialize()
    --创建system集合
    for i,v in pairs(ECS_SYSTEM_TYPE) do
        ecs_system_buffer[v] = {}
    end
    --根据路径require加载system
    for i,v in pairs(ECS_SYSTEM_FILE_PATH) do
        local system = require(v)
        self:ADD(system)
    end
    --调用Initialize的System
    for index,system in pairs(ECS_SYSTEM_BUFFER[ECS_SYSTEM_TYPE.Initialize_SYSTEM]) do
        system:Initialize()
    end
end

function WORLD:Execute()
    for index,system in pairs(executeSystem_list) do
        system:Execute()
    end
end

function WORLD:Cleanup()
    for index,system in pairs(cleanupSystem_list) do
        system:Cleanup()
    end
end

function WORLD:TearDown()
    for index,system in pairs(tearDownSystem_list) do
        system:TearDown()
    end
end

```

---

# 编辑器工具

由编辑器工具负责创建`Component`和`System`,*Component*通过*Json*配置，*System*完全并行化，通过Json生成初始版本，读取另一个文件夹中的进行逻辑控制？

---
