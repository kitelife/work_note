# Lua

- 布尔类型只有`nil`和`false`是false，其他的如数字`0`、`''`空字符串（`'\0'`）都是true。
- Lua中的变量如果没有特殊说明，全是全局变量，那怕是语句块或是函数里。变量前加local关键字的是局部变量。
- Lua没有++或是+=这样的操作
- Lua中不等于是`~=`，而不是`!=`
- 字符串的拼接操作符是`..`
- 条件表达式中的与或非为分是：`and`, `or`, `not`关键字。
- Lua中数组（也是table）的下标是从1开始的！
- 若变量arr是数组，那么`#arr`即数组的长度
- Lua中的变量，如果没有`local`关键字，全都是全局变量，Lua也是用table来管理全局变量的，Lua把这些全局变量放在了一个叫`_G`的table里。
- 遍历table `T`：

```lua
for k, v in pairs(T) do
    print(k, v)
```

- metatable 和 metamethod；setmetatable、getmetatable；Lua内建约定的metamethod有：
    - `__add(a, b)`     对应表达式 `a + b`
    - `__sub(a, b)`     对应表达式 `a - b`
    - `__mul(a, b)`     对应表达式 `a * b`
    - `__div(a, b)`     对应表达式 `a / b`
    - `__mod(a, b)`     对应表达式 `a % b`
    - `__pow(a, b)`     对应表达式 `a ^ b`
    - `__unm(a)`        对应表达式 `-a`
    - `__concat(a, b)`  对应表达式 `a .. b`
    - `__len(a)`        对应表达式 `#a`
    - `__eq(a, b)`      对应表达式 `a == b`
    - `__lt(a, b)`      对应表达式 `a < b`
    - `__le(a, b)`      对应表达式 `a <= b`
    - `__index(a, b)`   对应表达式 `a.b`
    - `__newindex(a, b, c)` 对应表达式 `a.b = c`
    - `__call(a, ...)`  对应表达式 `a(...)`

- `require`函数，载入同样的lua文件时，只有第一次的时候会去执行，后面的相同的都不执行了（说明模块加载有缓存）；如果要让每一次文件都会执行，可以使用`dofile("module_name")`函数；如果想载入后不执行，等需要的时候再执行，可以使用`loadfile("module_name")`函数。

## 推荐资源

- [Lua 5.3参考手册](http://cloudwu.github.io/lua53doc/)
- [X分钟速成Lua](http://learnxinyminutes.com/docs/zh-cn/lua-cn/)
- [Lua简明教程](http://coolshell.cn/articles/10739.html)
- [Lua教程](http://wiki.jikexueyuan.com/project/lua/)

