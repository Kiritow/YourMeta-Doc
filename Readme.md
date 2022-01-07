# YourMeta 玩家开发者文档

在YourMeta中通过自己的代码创建机器人以实现你的想法

玩家代码会在每个游戏刻(Tick)被执行, 可以通过调用API来提交这个Tick希望进行的动作. 每个Tick内可以调用多次API来提交动作, 但只有最后提交的一个动作会生效.

目前游戏支持的语言是Lua.

示例代码如下:

```lua
local api = require('api')
local x, y = api.getpos()
api.print(string.format('bot at %d %d', x, y))
if x < 50 then
    api.move(1, 0)
elseif y < 50 then
    api.move(0, 1)
end
```

这段代码展示了如何使用基本API来进行游戏刻操作. 首先通过`api.getpos()`获取到当前机器人所在的坐标, 接下使用`api.print(...)`打印机器人当前的坐标信息到本地的窗口, 接下来判断如果x坐标<50则调用`api.move`向右移动一格, 如果x坐标>=50, 那么如果y坐标<50则调用`api.move`向上移动一格.

这里要补充介绍下, 游戏逻辑基于平面直角坐标系, 规定y轴方向为右, x轴方向为上. 目前所有机器人和物品都只存在于第一象限.

游戏内置了LiteOS, 提供了相对基本API更丰富和易用的接口. 使用LiteOS实现刚刚的功能, 代码如下:

```lua
local bot = require('bot')
require('liteos').entry = function()
    local x, y = bot.getpos()
    print('bot at', x, y)
    bot.moveTo(x < 50 and 50 or x, y < 50 and 50 or y)
end
```

这段代码首先引用了LiteOS提供的BotAPI, 接下来引入LiteOS包并设置入口函数(entry). 函数中首先获取当前bot的位置, 接下来通过`bot.moveTo`方法将机器人的x和y坐标调整到至少50以上.

需要注意的是, 在LiteOS中, 用户应当将所有逻辑封装在一个主函数中, 并将其设为LiteOS的入口函数. 此时玩家不再需要考虑游戏刻问题, LiteOS将在用户提交动作的时候相对用户代码阻塞的执行操作. 例如 `api.move()` 会立刻返回, 但直到全部逻辑完成时才会生效. `bot.up()`在当前游戏刻直接执行移动操作, 当`bot.up()`返回时已经进入下一刻且机器人已完成向上移动的操作.
