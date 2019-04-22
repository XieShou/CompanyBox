# 迭代器和泛型 *for*

## 迭代器和闭包

*迭代器（iterator）*是一种可以让我们遍历一个集合中所有元素的代码结构。

一个*闭包*结构通常涉及两个函数：闭包本身和一个用于创建该闭包及其封装变量的*工厂（factory）*。

```lua
function values(t)
    local i = 0
    return function() i = i + 1; return t[i] end
end
```

## 泛型 *for* 的语法

```lua
for var-list in exp-list do
    body
end
```

例：

```lua
for k,v in pairs(t) do print(k,v) end
```

1. ***var-list***是由一个或多个变量组成的列表，以逗号分隔。

   列表的第一个（或唯一的）变量成为*控制变量（control variable）*，其值在循环过程中永远不会为`nil`，因为当其值为`nil`时循环就结束了。

2. ***exp-list***是一个或多个表达式组成的列表，以逗号分隔。

   `for`会对*exp-list*表达式求值，这个表达式应该返回三个值供*for*保存：

   *迭代函数*、*不可变状态*、*控制变量*

   只有最后一个（或唯一的）表达式能够产生不止一个值。表达式列表的结果只会保留三个，多余的值会被丢弃，不足则以`nil`补齐。

## 无状态迭代器

***无状态迭代器（stateless iterator）***就是一种自身不保存任何状态的迭代器。因此，可以在多个循环中使用同一个迭代器，从而避免创建新闭包的开销。

`ipairs`（工厂）和迭代器都非常简单，迭代的状态由正在被遍历的表（一个不可变状态，它不会在循环中改变）及当前的索引值（控制变量）组成。

```lua
local function iter(t, i)
 i = i + 1
 local v = t[i]
 if v then
     return i, v
 end
end

function ipairs(t)
    return iter, t, 0
end
```

*pairs*的迭代函数是Lua语言中的一个基本函数`next`：

```lua
function pairs(t)
    return next, t, nil
end

function next(t, k)
    body
end
```

在调用`next(t, k)`时，k是表t的一个键，该函数会以随机次序返回表中的下一个键及k对应的值（作为第二个返回值）。

```lua
for k, v in next, t do
    loop body
end
```

## 迭代器的真实含义

*for*只不过为每次的迭代提供连续的值。

更确切的说，迭代器接收一个函数作为参数，这个函数在循环的内部被调用，这种迭代器就被称为真正的迭代器（true iterator）。
