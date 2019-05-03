# 面向对象（Object - Oriented）编程

```lua
function Account.Func(self, value)
end
function Account:Func(value)
end
```

`self`参数是实现面向对象的核心点。

Lua语言使用*冒号操作符（colon operator）*隐藏该参数。



## 类（Class）

要让B成为A的一个原型

```lua
setmetatable(A, {__index = B})
```

---

#### 示例1：

```lua
local mt = {__index = Account}

function Account.new(o)
    o = o or {}
    setmetatable(o, mt)
    return o
end

function Account:Test(value)
    return value
end
```

使用：

```lua
a = Account.new{balance = 0}
local value = a:Test(10)
```

---

#### 示例1改进：

```lua
funtion Account:new(o)
    o = o or {}
    self.__index = self
    setmetatable(o, self)
    return o
end
```

当调用时，隐藏的参数self得到的实参是`Account`，`Account.__index  = Account`

## 继承（Inheritance）

```lua
SpecialAccount = Account:new()
s = SpecialAccount:new{limit = 1000.00}
```

`s`调用的是`SpecialAccount`的`new`。

`__index`也是从 *s -> SpecialAccount -> Account*的顺序来查找，所以可以重写函数。

## 多重继承（Multiple Inheritance）

关键在于把一个函数用作`__index`元方法。同时应该定义一个独立的函数`createClass`来创建子类。

```lua
-- 在表'plist'的列表中查找'k'
local function search(k, plist)
    for i = 1, #plist do
        local v = plist[i][k]    --尝试第'i'个超类
        if v then return v end
    end
end

funtion creatClass(...)
    local c = {}                --新类
    local parents = {...}        --父类列表
    
    -- 在父类列表中查找类缺失的方法
    setmetatable(c, {__index = function(t, k)
        return search(k, parents)
    end})
    
    --将'c'作为其实例的元表
    c.__index = c
    
    -- 为新类定义一个新的构造函数
    function c:new(o)
        o = o or {}
        setmetatable(o, c)
        return o
    end
    
    return c                -- 返回新类
end
```

测试：

```lua
Named = {}

function Named:getname()
    return self.name
end

function Named:setname(n)
    self.name = n
end

NameAccount = createClass(Account, Named)

account = NamedAccount:new{name = "Paul"}
print(account:getname())        --> Paul
```

#### 私有性（Privacy）

*私有性（信息隐藏，information hiding）*

每个对象的状态都应该由它自己控制
