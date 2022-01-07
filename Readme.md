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

## API列表

### LiteOS API

#### LiteOS

```lua
local liteos = require('liteos')
```

`liteos.entry` 由用户设置, 可以是任何可调用的对象. 此函数将作为用户逻辑的入口. 当此函数返回时机器人将停止运行.

`print` 此函数会被liteos替换为能向客户端发送消息的函数. 若没有引入`LiteOS`则print的内容不能在客户端看见.

#### Bot

```lua
local bot = require('bot')
```

`bot.up()` 机器人向上移动1m

`bot.down()` 机器人向下移动1m

`bot.left()` 机器人向左移动1m

`bot.right()` 机器人向右移动1m

`bot.die()` 机器人停止运行, 此函数不会返回.

`bot.broadcast(rad: number, msg: string)` 以机器人当前位置为中心, 向半径`rad`内的机器人发送广播, 内容为`msg`. 半径内的机器人将在下一个Tick完整的收到这个内容. 广播操作将消耗大量能量.

`bot.sendto(x, y, rad: number, msg: string)` 机器人向目标点`(x, y)`发送一道波束, 目标点附近半径`rad`内的机器人将收到完整的广播内容`msg`. 波束的传输需要时间, 但消耗能量较低.

`bot.read(): string|nil` 读取最后一条未收取的消息内容. 机器人默认只能保存1条未收取的消息内容. 如果当前没有未收取的消息则返回`nil`. 收取消息不额外消耗能量.

`bot.wait(): string` 阻塞机器人, 直到收到任何消息.

`bot.getpos(): number, number` 返回机器人当前的坐标 (x, y)

`bot.gethp(): number` 返回机器人当前的生命值

`bot.getpower(): number` 返回机器人当前的能量值. 当能量耗尽时机器人将停止运行.

`bot.sleep(ntick: number|nil)` 机器人等待`ntick`个游戏刻. 当不传入参数时, 默认等待1游戏刻.

`bot.moveTo(tx, ty: number, max_step: number|nil): number` 要求机器人向 (tx, ty) 方向移动, 且最多移动`max_step`格. 当不传入`max_step`时, 默认无最大步数限制. 此方法返回整个过程中一共移动的步数.

#### Strings

```lua
local strings = require('strings')
```

`strings.split(raw, sep: string): table` 将字符串`raw`以`sep`为分隔符进行分割, 返回一个数组.

#### Util

```lua
local util = require('util')
```

`util.checkArg(n: number, value: any, type1: string, [type2: string, ...])` 检查`value`的类型是否为`type1`, `type2`等. 如果都不满足则抛出一个异常: `bad argument #n (type1 expected, got type(value))`

### 基本API

```lua
local api = require('api')
```

`api.move(dx, dy: number)` 向x方向移动`dx`格, 向y方向移动`dy`格. 在同一个tick内完成. 根据机器人型号不同可能并不支持此操作.

`api.sendto(x, y, rad: number, msg: string)` 参见`bot.sendto`

`api.broadcast(rad: number, msg: string)` 参见`bot.broadcast`

`api.getpos(): number, number` 参见`bot.getpos`

`api.gethp(): number` 参见`bot.gethp`

`api.getmsgcache(): string|nil` 参见`bot.read`

`api.print(msg: string)` 打印`msg`到客户端窗口中.
