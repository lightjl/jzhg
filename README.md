https://www.joinquant.com/post/1899?f=study&m=algorithm
导语：我们经常在抓反弹时感觉像在抓一只刺猬，不知道该怎么下手。左侧交易会不会买在半山腰？右侧交易会不会进场太晚？什么时候买入真是让人头疼。切莫苦恼，你完全可以通过历史数据统计判断反弹的时机。本文就将抛砖引玉地介绍一种简单的统计方法，它不能保证抓反弹次次成功，但可以让你对多错少，累积起来就是真实的收益！

作者：肖睿
编辑：宏观经济算命师

本文由JoinQuant量化课堂推出，难度为进阶（上），深度为 level-0。

本文是量化课堂的《均值回归入门》的进阶版，并且用到《指标效果的统计分析》一文中的统计方法。
动机

我们先回顾一下均值回归的基本概要：

股票的价格会以它的均线为中心进行波动。也就是说，当标的价格由于波动而偏离移动均线时，它将调整并重新归于均线。那么如果我们如果能捕捉偏离股价的回归，就可以从此获利。

在入门篇文章里我们尝试了一个非常简单的均值回归策略，也发现了相应的问题：那就是无法准确判断何时买入，也缺乏买在半山腰之后的处置机制。本篇要讲的是，统计历史数据中价格偏离所对应的反弹幅度，选择胜率最高的偏离度作为入场信号。
波动大小的相对性

在入门篇文章中我们粗略地以价格和均线的偏差作为买入的评估标准，但它有一个弊端，那就是忽略了股价波动的相对性。假如股票 A 在一段时间里每天振幅 3%，股票 B 在同样一段时间里每日振幅 0.5%，今天我们发现 A 和 B 的股价都和均线偏离了 2%，我们认为那支的偏离更显著呢？是 B 对吧。

举实例来说。下图是某支股票的 60 分钟线，中间紫色曲线为股价的 5 日均线。可以看出在蓝色线段标明的时间里，股价波动平缓。在两个粉色箭头处，股价比均线分别高 4.1% 和低 5.9%。这些波动较前段时间更为剧烈，可以告诉我们股价发生了偏离。
1.png

再看下图，这是同一支股票在另一段时间的 60 分钟线。这段时间里价格波动幅度大，在粉色箭头的位置，价格和均线的差距分别为5.3% 和 5.1%，但我们不将其视为显著的价格偏离。
2.png

那么我们该如何测量近期波动幅度的大小呢？这就要涉及到统计学中的标准差概念。
标准差

标准差（standard deviation），通常用小写希腊字母 σσ（sigma，读“西格玛”）表示。通俗地讲，一组数据的标准差就是这组数据离均值的普遍差距。

假设 x1,x2,…,xnx1,x2,…,xn 是我们观测到的一组数据，这组数据的均值是 μ=(x1+x2+⋯+xn)/nμ=(x1+x2+⋯+xn)/n
标准差的计算公式为
σ=(x1−μ)2+(x2−μ)2+⋯+(xn−μ)2)n−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−√
σ=(x1−μ)2+(x2−μ)2+⋯+(xn−μ)2)n

这里 xt−μxt−μ 计算的是第 tt 项数据和均值的差值。将这 nn 项差值的平方取了平均，再算出平方根，我们得到的是这组数与均值偏差的一个标准。如果这组数据的波动较大，那么 σσ 相应也会较大；相反的，如果这组数据波动小，那么 σσ 会更接近零。

举例来说，假设我们有一组数据 A=(1,0,0,0,0−1,0,0,0,0)A=(1,0,0,0,0−1,0,0,0,0)，如下
3.png

它的平均值是 00，标准差是
σ=(1−0)2+(−1−0)2)10−−−−−−−−−−−−−−−−−−√=1/5−−−√≈0.45
σ=(1−0)2+(−1−0)2)10=1/5≈0.45

这时如果下一项出现的数据是 x11=1x11=1，可以算出它和均值的差是 (1−0)/σ≈2.23(1−0)/σ≈2.23 倍标准差，可以被视为是一个较大的波动。

在另一种情况里，假设我们的数据是 B=(1,−1,1,−1,1,−1,1,−1,1,−1)B=(1,−1,1,−1,1,−1,1,−1,1,−1)，如下图
4.png

可以直观地看出波动较大。这组数据同样是均值等于 00，它的标准差是
σ=(1−0)2+(−1−0)2+⋯+(−1−0)2)10−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−√=1√=1
σ=(1−0)2+(−1−0)2+⋯+(−1−0)2)10=1=1

明显比数据 AA 的标准差更大。这时如果出现新的数据 x11=1x11=1，由于它和均值的差距等同于标准，我们不认为它是一个很大的波动。

有了这个概念，我们可以构造一种利用标准差来测量当日股价波动幅度的方法。取过去 NN 天收盘价，并取其标准差 σσ，设今日股价为 PP 并且 NN 日均线值为 MANMAN，计算
ρ=P−MANσ
ρ=P−MANσ

这个值得含义是，相较过去 NN 天的价格，今日股价与均值的偏差为 ρρ 倍的标准差。如果绝对值 |ρ||ρ| 比较大，我们判定今日价格波动比历史波动更为突出；反之，如果 |ρ||ρ| 比较小，我们判定今日的波动很普通。嗯，这个 ρρ 嘛，为了方便起见，作者就自行起一个名字吧，就叫它“NN 日偏离倍数好了”。

好，可以合理地判定波动大小了，那么应该如何利用这些信息判断何时入场呢？
偏离倍数和赢输的统计

我们没法凭空预测偏离倍数在多大的情况下价格会反弹，也不能盲目地认为偏离倍数越高越好。我们干脆穷举，对历史上所有的偏离倍数（范围），统计其后后股价的走势，然后将胜率最高的偏离倍数视为入场信号，在其出现时买入。

接下来的统计方法请参阅量化课堂文章《指标效果的统计分析》。

按照文中的思路，我们要计量的指标是上一节中介绍的 NN 日偏离倍数 ρρ。另外，我们只记录 ρ<−1ρ<−1 的情况，因为如果 ρ≥−1ρ≥−1，那说明价格偏离不大，则不构成回归信号。

我们将要统计的结果进行如下判定：设定观测输赢的天数 TT，止盈的 σσ 倍数 uu，止损的 σσ 倍数 dd。如果在未来的 TT 天里，股价先上涨 σ⋅uσ⋅u，则记作“赢”；先下跌 σ⋅dσ⋅d，则记作“输”；如果 TT 天股价都没离开这个区间，则记作“平”。

以某股票从 2006 年到 2016 年的数据为例，统计结果如下：
5.png

通过直接看图可估测出 ρρ 在 2.32.3 到 3.03.0 的区间里胜率很高。我们再通过文章中取最佳指标区间的算法获取一个胜率最高的偏离倍数区间。这里，我们想选定一个区间宽度 ww，并在以上统计中计算一个宽度为 ww 的横轴区间（ww 个连续竖条），需要它的胜率（绿色区域总和除以红色区域总和）是最高的。当然，还需要设置一个最小数据比 θθ，区间内的数据量必须高于总数据量的 θθ 倍，不然区间内的统计缺乏可信性。

我们设置参数：区间宽度 w=5w=5，最小数据比 θ=0.05θ=0.05。通过计算，得到区间 [−2.6,3.1][−2.6,3.1]，区间中的数据如下：
6.png
其中赢 (5,7,4,3,8)(5,7,4,3,8) 次，输 (1,4,2,3,2)(1,4,2,3,2)。计算出输赢率为
5+7+4+3+81+4+2+3+2=2712
5+7+4+3+81+4+2+3+2=2712

也就是说，根据历史统计，如果当偏离倍数在 −3.1−3.1 和 −2.6−2.6 之间时买入该股票，输赢的预期是 2727 赢比 1212 输，当然，还有 22 次平局。

策略和回测

其实策略的主要部分已经完成，只要加上对资金和仓位的管理，就可以构成一个可运行的策略。这里我们就构建一个很简单的方案。

设定如上一节所述每的参数：均线天数 NN，统计输赢天数 TT，止盈倍数 uu，止损倍数 dd，区间宽度 ww，和最小数据比 θθ。每交易日执行以下操作：
一、全仓卖出应当止盈或止损的股票，或持有超过 TT 天的股票；
二、执行上一节的统计方法来更新每一支股票的最佳偏离倍数区间；
三、如果空仓，选定任意一支偏离倍数在最佳区间内的股票并全仓买入，如果所有股票都不符合则空仓。买入时记录止盈线 uu 倍 σσ 和止损线 dd 倍 σσ，并设置如果 TT 日内未触碰止盈或止损线，也全部卖出。

来看一看回测结果。以下回测使用的股票池只包含 1 或 2 支标的股，这样便于看出策略对于个股发出的信号。文章最后的策略代码只适用于小规模的股票池；对于更大的股票池，信号会更加密集，因此需要调整每支股票的持仓量并且分别止盈止损，读者可以自行进行修改，或等待量化课堂未来的文章。另外，我们的回测从 2011 年开始，这样保证回测开始时就有一定的历史数据储备。

首先是股票 002013 的回测，使用参数 N=30N=30，T=30T=30，u=d=1u=d=1，w=4w=4，θ=0.05θ=0.05，回测结果如下。
7.png
策略在震荡市中产生比较稳定的收益，并保持比较小的回撤。在牛市中完全不产生信号，因此没有半点反应，倒是在股灾之后稳稳地抓好了反弹，在短期内获取大量收益。

还使用同样的参数，我们看 002230 的回测。
8.png
同样是在震荡市中表现良好，但在股灾中抓反弹一度失利，不过后期在弥补损失之余获得了更高的收益。

观看回测最下方的“每日买卖”图可看出这个策略产生的信号和交易实则不多，在股票池里放入多支股票的话可以轮回抓取反弹，产生叠加的收益。比如，下图是将上面两支股票的作为标的回测，对比的指数是沪深300。
9.png
结语

嗯，我们看看啊…总结就是三点。首先是均值回归核心思路：

价格偏离均线太多就会弹回来

然后是，我们怎么判断偏离多少呢？

价格减去均线除以历史波动的标准差可以丈量偏离的程度

判断入场的话…

利用历史统计的偏离倍数对应的输赢率判断入场时机

最后还要指出，本篇文章旨为抛砖引玉，展示一种思路。文中的统计方法比较粗糙，策略也比较简单。想要在实战中应用的话，还需要进一步拓展、打磨并结合其他的方法，才能构造出一套成熟的交易策略。量化课堂未来的文章会在此策略上升级改进，结合凯利公式进行动态的仓位调整，敬请期待。