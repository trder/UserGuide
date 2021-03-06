# AI纪元量化平台编程指南

通过简单编程，创建你的第一个量化交易系统。

## 交易系统

无论是程序化自动交易，还是凭借经验进行的主观交易。在开始之前，都需要定义我们的交易系统。

AI纪元量化平台采用主流的Python3语言来定义交易系统，文章的后半部分会详细说明交易系统定义的过程。

如果你没有编程基础也没有关系，AI纪元量化平台会给出示例代码。经典的海龟交易系统会作为它的第一个示例，而AI交易系统会作为第二个。
示例交易系统是可以直接运行的，你可以使用其它人在AI纪元量化平台编写好的交易系统来运行，或者Fork别人的交易系统改进成自己的系统。
回测系统

除非你想要亏光你的钱，或者只是纯赌博，否则在开始之前我们应当对交易系统进行检验。

就算无法精确计算出未来的收益，也至少应该知道系统在未来的预期收益是否为正。
回测系统被定义为一种能够使用历史数据对量化交易系统的收益和风险进行量化评估的系统。
回测系统是AI纪元量化平台的一部分，它以定义好的交易系统为输入，调用历史K线信息，输出回测报告。
通过回测报告，我们能够对量化交易系统进行评分。对于遗传算法来说，评分的准确性，直接决定了量化交易系统能否向正确的方向演化。

## 量化平台

以下内容使用“本平台”代指“AI纪元量化平台”。
通过高度抽象，我们将忽略交易的底层细节（数据库交互，底层交易接口的实现，ATR等指标的计算逻辑），仅通过少量的参数与高度抽象的脚本代码，来定义完整的量化交易系统。
定义好的交易系统可以既可以加载到模拟回测系统中，也可以加载到实盘交易系统中。
为了降低脚本编写的门槛，本平台使用目前主流的Python语言（版本3.10以上）来编写脚本。

## 自动交易框架

让我们回顾一下自动交易的执行过程，分析一下哪些过程是固定的，而哪些又是变化的？

### 信号/入市

首先是对市场和头寸的监控。所有量化交易系统都需要实时监控市场和头寸的状态，出现入市信号就应该入市，满足退出/止损条件就应该立即退出，满仓时要忽略入市信号，不能无限加仓。
对于不同的系统来说，要监控的指标是不一样的。
对于海龟交易系统来说，要监控的指标是10日/20日/55日通道的突破（下单/退出），以及通道突破后以0.5ATR为单位的二次突破（建仓/止损）。同一时间发生多个突破，选择信号最强的市场来操作（调用入市信号的计算函数来计算）。
对AI交易系统来说，要监控的指标是入市信号和退出信号强弱（由AI实时计算，入市信号大于0.5时入市）的变化（下单/建仓/退出）和头寸盈亏的变化（止损）。如果多个市场的均满足入市条件，选择入市信号最强的市场进行操作。

### 退出/止损

退出和止损是两种不同意义上的操作，尽管它们产生的效果都是头寸的退出。

止损操作是系统的最终安全屏障。
无论是哪种交易系统，对于每笔交易来说，我们都不应该让亏损无限放大，因为这会导致爆仓风险（对于带有杠杆的双向交易系统来说）。
我们应该在下单之前就设定好它的亏损上限。根据亏损上限，计算出相应的止损点。这个止损点可以是根据入市价格计算出的固定止损点，也可以是根据最大盈利点计算出的动态止损点。
由于止损是最终完全屏障，为了它能够100%的执行，应当使用立即执行的市价单，或先挂出限价单（相比市价单，具有成交价格的优势，但不会立即成交），如果无法在限定时间内交易完成，再转为市价单。
退出操作是依据市场的退出信号而发起的操作。一般在止损操作发生之前，市场就会转为恶劣，并触发退出操作。
对于海龟交易系统来说，当市场向亏损方向突破10日或20日极值时，触发退出操作；对于AI交易系统来说，当市场退出信号大于0.5时触发退出操作。

### 头寸单位

当决定入市的，需要一个公式来计算要入市的头寸单位的大小。当头寸单位达到上线时，应该输出0，表示忽略本次入市信号。
头寸的计算应该根据余额，ATR，风险系数，市场价格，以及当前已经持仓的头寸单位的个数。

## 输入/输出

系统的输入包括一个以trade_xxx命名的文件夹，其中xxx表示交易策略的具体名称，使用大小写英文字符或数字来命令，长度在1到20之间。

文件夹中至少包含：`trading.py`和`config.config`两个文件。

其中：`trading.py`是使用Python3编写的代码 。包含了交易系统运行的基本逻辑（信号判定，入市，头寸规模，止损，退出…）。

`config.config`是json格式的文件，包含了交易系统运行的相关参数（最小订单金额，风险系数等等）。

回测引擎读取两个输入文件后，调用已采集的K线数据，执行回测运算，并输出回测报告（初始余额、最终余额，交易时长，交易次数、胜率、年化收益，最大亏损、最长衰落期，MAR比率、余额波动标准差、夏普比率等）。

## 元交易系统

元交易系统是高度抽象的量化交易系统，当我描述元系统的时候，实际上我描述了所有被**AI纪元量化平台**所支持的交易系统。

`注意：为了保持元交易系统的简洁性，它将不支持一次创建大量订单的网格交易系统，它的每个入市信号最多只对应单个订单。`

元交易系统的具体执行过程如下：

循环调用`entry_signal(exchange,symbol)`获取交易所`exchange`中市场`symbol`的当前入市策略`strategy`。

`strategy`中包含信号`sign`（介于[0,1]之间，超过0.5表示需要执行交易）方向`side`（做多buy或做空sell）和头寸大小`pos`（以USD为单位）。

当`strategy.sign`大于0.5且`strategy.pos`大于最小订单金额`min_order`时(`min_order`默认为100 USD，可在`config.json`中设置)，调用`execute_entry(exchange,symbol,strategy)`创建订单`order`，并将`order`添加到`order_queue`中。

```Python3
#strategy策略对象
strategy = {
"sign":0.5, #信号强度
"side":"buy" #方向：做多buy或做空sell
"pos":500.0 #头寸大小：（以USD为单位）
}
```

```Python3
#order订单对象
order = {
"exchange":"bitfinex", #交易所
"symbol":"BTC/USDT", #币种
"side":"buy", #方向：做多buy或做空sell
"order_id":"xxxxxxxxxxxxx", #订单编号
"entry_price":50000.0, #平均成交价格
"best_price":50010.0, #盈利最大价格
"stop_price":49010.0, #止损价格(对于动态止损策略，stop_price会根据best_price动态变化)
"total_amount":"0.1", #数量
"executed_amount":"0.04", #已执行数量
"unexecuted_amount":"0.06", #未执行数量
"status":1, #0未执行;1部分执行;2全部执行
"timestamp":1650176916.000, #订单创建时间（秒）
"fees":2.0, #已产生的手续费（美元）
"ATR": 2500.0, #ATR
"ATRP": 5.0 #ATR%
}
```

`注意：order_queue中的订单并不会立即执行，而是会在接近市价的位置挂单，并根据市场价格的波动实时调整价格，使订单价格始终保持在最容易成交的位置，一直监控订单的状态order.status，直到全部执行为止。`

枚举`order_queue`中的`order`，调用`exit_signal(order)`，检查订单`order`的退出信号`exit_sign`（介于[0,1]之间）和退出类型`etype`（etype分为0信号退出或1止损退出）。

当退出信号大于0.5时调用`execute_exit(order,etype)`执行退出操作，并将`order`从`order_queue`中删除，添加到`order_history`中。

- 对于`order.status`为未执行的订单，直接取消订单。

- 对于`order.status`为全部执行或部分执行的订单，先取消未执行的部分，对于已经执行的部分：

- 对于`etype`为"止损退出"的订单，会在前30秒内使用动态调整的限价单执行，30秒后将剩余部分转为市价单全部执行。

- 对于`etype`为"信号退出"的订单，会一直使用动态调整的限价单执行。

## 输入模版

> trading.py
```Python3
class trading:
    def entry_signal(exchange:string,symbol:string) -> dict:
        #strategy策略对象
        strategy = {
        "sign":0.5, #信号强度
        "side":"buy" #方向：做多buy或做空sell
        "pos":500.0 #头寸大小：（以USD为单位）
        }
        return strategy
        
    def execute_entry(exchange:string,symbol:string,strategy:"strategy") -> dict:
        #order订单对象
        order = {
        "exchange":"bitfinex", #交易所
        "symbol":"BTC/USDT", #币种
        "side":"buy", #方向：做多buy或做空sell
        "order_id":"xxxxxxxxxxxxx", #订单编号
        "entry_price":50000.0, #平均成交价格
        "best_price":50010.0, #盈利最大价格
        "stop_price":49010.0, #止损价格(对于动态止损策略，stop_price会根据best_price动态变化)
        "total_amount":"0.1", #数量
        "executed_amount":"0.04", #已执行数量
        "unexecuted_amount":"0.06", #未执行数量
        "status":1, #0未执行;1部分执行;2全部执行
        "timestamp":1650176916.000, #订单创建时间（秒）
        "fees":2.0, #已产生的手续费（美元）
        "ATR": 2500.0, #ATR
        "ATRP": 5.0 #ATR%
        }
        return order
        
    def exit_signal(order:"order") -> tuple:
        exit_sign = 0.5 #退出信号强度（介于[0,1]之间）
        etype = 0 #退出类型：0信号退出 1止损退出
        return exit_sign, etype
        
    def execute_exit(order:"order",etype:int) -> None:
        return
```
> config.json
```json
{
    "min_order":100.0,
    "risk_factor":1.0
    "exchange":"bitstamp"
    "markets": ["BTC/EUR"]
}
```

## 样例一：海龟交易系统

### 说明

海龟交易系统的执行流程，可以用文字表述为：
1. 检查价格是否突破了20日或55日的唐奇安通道；
    1. 如果没有发生突破，记为状态S0；
    2. 如果突破了20日的唐奇安通道；
        1. 如果是向上突破；
            1. 如果刚刚突破通道，记为状态S10；
            2. 如果突破了通道+0.5ATR，记为状态S11；
            3. 如果突破了通道+1.0ATR，记为状态S12；
            4. 如果突破了通道+1.5ATR，记为状态S13；
        3. 如果是向下突破；
            1. 如果刚刚突破通道，记为状态S20；
            2. 如果突破了通道-0.5ATR，记为状态S21；
            3. 如果突破了通道-1.0ATR，记为状态S22；
            4. 如果突破了通道-1.5ATR，记为状态S23；
    4. 如果突破了55日的唐奇安通道：
        1. 如果是向上突破；
            1. 如果刚刚突破通道，记为状态S30；
            2. 如果突破了通道+0.5ATR，记为状态S31；
            3. 如果突破了通道+1.0ATR，记为状态S32；
            4. 如果突破了通道+1.5ATR，记为状态S33；
        3. 如果是向下突破；
            1. 如果刚刚突破通道，记为状态S40；
            2. 如果突破了通道-0.5ATR，记为状态S41；
            3. 如果突破了通道-1.0ATR，记为状态S42；
            4. 如果突破了通道-1.5ATR，记为状态S43；
2. 检查上一次20日突破是否为亏损型突破（在退出前执行了止损）？
    1. 如果上一次是亏损型突破，则可在状态S10/S30时入市做多，在状态S20/S40时入市做空。
    2. 如果上一次是盈利型突破，则可在状态S30时入市做多，在状态S40时入市做空。
3. 逐步建仓：
    1. 在S10、S20、S30、S40创建第一个头寸；
    2. 在S11、S21、S31、S41创建第二个头寸；
    3. 在S12、S22、S32、S42创建第三个头寸；
    4. 在S13、S23、S33、S43创建第四个头寸；
    5. 单个市场最多持有4个头寸。
4. 止损与重建：
    1. 当价格跌破入市价-0.5ATR时，执行止损；
    2. 在一个头寸单位止损退出后，交易者将在价格恢复到最初的入市价时重新建立这个单位(双重损失策略)。
5. 退出：
    1. 对于20日突破建立的头寸，在跌破10日唐奇安通道时退出；
    2. 对于55日突破建立的头寸，在跌破20日唐奇安通道时退出。
6. 多个市场
    1. 如果同一时间多个市场发出入市信号，选择信号最强的市场交易；
    2. 如果多个市场存在高度关联，则最多持有6个头寸单位；
    3. 如果多个市场存在松散关联，则最多持有10个头寸单位；
    4. 单个方向最多持有12个头寸单位。

> 头寸规模单位=账户的1%/市场的绝对波动幅度

### 代码

见：[API参考文档](https://github.com/trder/APIReference)。

## 参考
原文地址：[AI纪元量化平台编程指南](
https://github.com/trder/UserGuide/blob/main/AI%E7%BA%AA%E5%85%83%E9%87%8F%E5%8C%96%E5%B9%B3%E5%8F%B0%E7%BC%96%E7%A8%8B%E6%8C%87%E5%8D%97.md)。
