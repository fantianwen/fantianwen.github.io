---
layout:     post
title:      "怎样让期货交易变成 Semi-Information Game"
subtitle:   "通过多层次信息获取减少信息不对称，提升交易优势"
date:       2025-12-26
author:     fantianwen
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - trading
    - orderbook
    - information-asymmetry
    - market-microstructure
---

## 引言

在之前的文章中，我们讨论了加密货币市场是一个**不完全信息博弈**：你永远无法知道做市商的真实意图、交易所的内部数据、或项目方的真实计划。

但我们可以通过**系统化地收集和分析多层次信息**，将交易从"不完全信息博弈"转变为"**半信息博弈 (Semi-Information Game)**"。

本文将从三个层次探讨如何获取和利用这些信息：
1. **K线指标层**：统计性指标，反映一般价格趋势
2. **Orderbook Flow 层**：微观结构指标，精准反映价格移动的根本
3. **场外指标层**：宏观和链上数据，提供市场背景

---

## 1. K线指标层：统计性趋势信号

### 1.1 K线指标的本质

**K线指标是统计性指标，反映了一般价格趋势。**

这些指标不是"预测未来"，而是**对历史价格行为的数学描述**。它们之所以有效，是因为：
- 市场具有**记忆性**：过去的行为模式往往会在未来重复
- **趋势延续性**：一旦趋势形成，往往会持续一段时间
- **均值回归性**：极端偏离后往往会回归

但要注意：**K线指标是滞后指标**，它们告诉你"现在发生了什么"，而不是"未来会发生什么"。

### 1.2 比较有用的统计性指标

#### EMA21, EMA55, EMA200

**指数移动平均线 (Exponential Moving Average)** 是趋势跟踪的核心工具。

| 周期 | 用途 | 市场含义 |
|------|------|---------|
| **EMA21** | 短期趋势 | 约1个月（21个交易日），捕捉短期动量 |
| **EMA55** | 中期趋势 | 约1个季度，过滤月度噪音 |
| **EMA200** | 长期趋势 | 约1年，判断牛熊分界线 |

**实战用法：**
- **EMA21 > EMA55 > EMA200**：多头排列，看涨
- **价格在 EMA200 上方**：长期牛市
- **价格回踩 EMA55**：中期趋势中的回调买入点

```python
import pandas as pd
import numpy as np

def calculate_ema_signals(df):
    """计算 EMA 信号"""
    df['ema21'] = df['close'].ewm(span=21, adjust=False).mean()
    df['ema55'] = df['close'].ewm(span=55, adjust=False).mean()
    df['ema200'] = df['close'].ewm(span=200, adjust=False).mean()
    
    # 趋势信号
    df['trend'] = np.where(
        (df['ema21'] > df['ema55']) & (df['ema55'] > df['ema200']), 
        'BULLISH',
        np.where(
            (df['ema21'] < df['ema55']) & (df['ema55'] < df['ema200']),
            'BEARISH',
            'NEUTRAL'
        )
    )
    
    # 回踩信号
    df['pullback_buy'] = (df['close'] > df['ema200']) & \
                         (df['close'] < df['ema55']) & \
                         (df['close'].shift(1) >= df['ema55'].shift(1))
    
    return df
```

#### VWAP (Volume Weighted Average Price)

**成交量加权平均价格**是机构交易者最看重的指标之一。

**为什么重要？**
- **机构成本线**：大单通常以 VWAP 为基准执行
- **日内支撑/阻力**：价格偏离 VWAP 过远会回归
- **真实平均价**：比简单均价更能反映市场真实成本

**实战用法：**
- 价格在 VWAP 上方：多头占优
- 价格回踩 VWAP：买入机会（如果趋势向上）
- 价格远离 VWAP > 2%：可能回调

```python
def calculate_vwap(df):
    """计算 VWAP"""
    # VWAP = Σ(Price × Volume) / Σ(Volume)
    df['typical_price'] = (df['high'] + df['low'] + df['close']) / 3
    df['vwap'] = (df['typical_price'] * df['volume']).cumsum() / df['volume'].cumsum()
    
    # 偏离度
    df['vwap_deviation'] = (df['close'] - df['vwap']) / df['vwap'] * 100
    
    return df
```

#### Volume (成交量)

**成交量是趋势的"燃料"**。

**核心洞察：**
- **价涨量增**：健康上涨，趋势可持续
- **价涨量缩**：虚假突破，可能反转
- **价跌量增**：恐慌性抛售，可能见底
- **价跌量缩**：正常回调，趋势未变

**实战用法：**
- 突破关键阻力位时，必须有**放量确认**
- 成交量萎缩 + 价格横盘 = 变盘在即
- **成交量异常放大**（如 3倍平均量）= 重大事件，密切关注

```python
def analyze_volume(df):
    """分析成交量"""
    df['volume_ma'] = df['volume'].rolling(window=20).mean()
    df['volume_ratio'] = df['volume'] / df['volume_ma']
    
    # 成交量信号
    df['volume_signal'] = np.where(
        (df['close'] > df['close'].shift(1)) & (df['volume_ratio'] > 1.5),
        'BULLISH_BREAKOUT',
        np.where(
            (df['close'] < df['close'].shift(1)) & (df['volume_ratio'] > 1.5),
            'BEARISH_BREAKDOWN',
            'NORMAL'
        )
    )
    
    return df
```

---

## 2. Orderbook Flow 层：价格移动的根本

### 2.1 Orderbook Flow 的本质

**Orderbook Flow 才是精准反映价格移动的根本。**

为什么？

1. **价格是由订单簿决定的**：订单簿的买卖盘不平衡直接导致价格偏移
2. **做市商 (MM) 的报价系统**：做市商根据订单簿的净流量调整报价
3. **真实意图**：大单隐藏在订单簿中，比 K线更早暴露意图

**关键洞察：**
> 订单簿的**减法 (Subtraction)** 会直接影响做市商的报价系统，导致价格偏移。

当大单吃掉订单簿某一侧的流动性时：
- 做市商需要**重新平衡库存**
- 报价会向**流动性缺失的方向移动**
- 这创造了**可预测的价格移动**

### 2.2 Orderbook 相关指标

#### CVD (Cumulative Volume Delta)

**累计成交量差** = 主动买入量 - 主动卖出量

**核心逻辑：**
- **CVD 上升**：买盘压力 > 卖盘压力，价格倾向于上涨
- **CVD 下降**：卖盘压力 > 买盘压力，价格倾向于下跌
- **CVD 背离**：价格创新高但 CVD 下降 = 虚假突破

**实战用法：**
```python
def calculate_cvd(trades_data):
    """
    trades_data: 包含 price, volume, side (buy/sell) 的交易数据
    """
    # 计算每笔交易的 delta
    trades_data['delta'] = np.where(
        trades_data['side'] == 'buy',
        trades_data['volume'],
        -trades_data['volume']
    )
    
    # 累计 delta
    trades_data['cvd'] = trades_data['delta'].cumsum()
    
    # CVD 变化率
    trades_data['cvd_change'] = trades_data['cvd'].diff()
    
    return trades_data
```

**信号生成：**
- CVD 连续上升 + 价格横盘 = **蓄势待发，即将突破**
- CVD 创新高 + 价格未创新高 = **看涨背离**
- CVD 快速下降 = **大单卖出，警惕下跌**

#### Orderbook Footprint (订单簿足迹)

**订单簿足迹**显示在**每个价格水平上的买卖订单分布**。

**关键信息：**
- **大单位置**：哪些价格有大量挂单（支撑/阻力）
- **订单消耗**：哪些价格的大单被快速吃掉
- **不平衡程度**：买卖盘的不对称程度

**实战用法：**
```python
def analyze_orderbook_footprint(orderbook_snapshots):
    """
    orderbook_snapshots: 订单簿快照序列
    每个快照包含: bids[{price, size}], asks[{price, size}]
    """
    footprint = []
    
    for snapshot in orderbook_snapshots:
        # 计算买卖盘总量
        total_bid = sum([bid['size'] for bid in snapshot['bids'][:10]])  # 前10档
        total_ask = sum([ask['size'] for ask in snapshot['asks'][:10]])
        
        # 不平衡比率
        imbalance = (total_bid - total_ask) / (total_bid + total_ask)
        
        # 大单识别（超过平均值的订单）
        avg_size = (total_bid + total_ask) / 20
        large_bids = [b for b in snapshot['bids'] if b['size'] > avg_size * 2]
        large_asks = [a for a in snapshot['asks'] if a['size'] > avg_size * 2]
        
        footprint.append({
            'imbalance': imbalance,
            'large_bids': large_bids,
            'large_asks': large_asks,
            'support_level': snapshot['bids'][0]['price'] if snapshot['bids'] else None,
            'resistance_level': snapshot['asks'][0]['price'] if snapshot['asks'] else None,
        })
    
    return footprint
```

**信号生成：**
- **Imbalance > 0.3**：买盘明显强于卖盘，看涨
- **大单在关键价位**：这些是真实的支撑/阻力
- **大单快速消失**：被吃掉 = 真实需求/供给

#### Heatmap (热力图)

**订单簿热力图**可视化订单簿的**动态变化**。

**关键观察：**
- **热区**：大量订单聚集的价格区域（强支撑/阻力）
- **冷区**：订单稀少的区域（容易突破）
- **移动方向**：热区的移动方向 = 价格可能移动的方向

**实战用法：**
- 价格接近**热区上沿**：可能突破
- 价格在**热区内**：震荡，等待方向
- **热区快速移动**：大资金在调仓，跟随

#### Volume Profile (成交量分布)

**成交量分布**显示在**每个价格水平上成交了多少量**。

**核心价值：**
- **POC (Point of Control)**：成交量最大的价格 = 市场共识价格
- **Value Area**：70% 成交量集中的价格区间 = 合理价值区间
- **低成交量节点 (Low Volume Node)**：价格容易快速通过

**实战用法：**
```python
def calculate_volume_profile(ohlcv_data, bins=50):
    """
    计算成交量分布
    """
    # 将价格区间分成 bins 个区间
    price_min = ohlcv_data['low'].min()
    price_max = ohlcv_data['high'].max()
    price_bins = np.linspace(price_min, price_max, bins)
    
    # 计算每个价格区间的成交量
    volume_profile = []
    for i in range(len(price_bins) - 1):
        mask = (ohlcv_data['low'] >= price_bins[i]) & (ohlcv_data['high'] <= price_bins[i+1])
        volume_in_range = ohlcv_data.loc[mask, 'volume'].sum()
        volume_profile.append({
            'price_range': (price_bins[i], price_bins[i+1]),
            'volume': volume_in_range
        })
    
    volume_profile = pd.DataFrame(volume_profile)
    
    # POC: 成交量最大的价格区间
    poc_idx = volume_profile['volume'].idxmax()
    poc_range = volume_profile.loc[poc_idx, 'price_range']
    
    # Value Area: 累计成交量达到 70% 的价格区间
    volume_profile = volume_profile.sort_values('volume', ascending=False)
    cumulative_volume = volume_profile['volume'].cumsum()
    total_volume = volume_profile['volume'].sum()
    value_area = volume_profile[cumulative_volume <= total_volume * 0.7]
    
    return {
        'poc': poc_range,
        'value_area': value_area,
        'volume_profile': volume_profile
    }
```

**信号生成：**
- 价格在 **Value Area 下方**：被低估，可能反弹
- 价格突破 **POC**：趋势确认
- 价格在 **Low Volume Node**：快速移动，不要追

---

## 3. 场外指标层：宏观背景

场外指标提供**市场的大背景**，帮助判断：
- **当前处于什么市场环境**（牛市/熊市/震荡）
- **资金流向**（流入/流出）
- **市场情绪**（贪婪/恐惧）

### 3.1 链上指标

- **交易所净流入/流出**：资金流向交易所 = 可能抛售
- **稳定币市值变化**：稳定币增发 = 新资金入场
- **大户持仓变化**：鲸鱼在增持/减持

### 3.2 衍生品指标

- **资金费率**：市场情绪和拥挤度
- **持仓量 (OI)**：市场参与度
- **清算数据**：潜在的连锁反应

### 3.3 期权市场指标：GEX 和 Skew

期权市场对现货/期货价格有**结构性影响**，这是很多交易者忽略的重要信息源。

#### GEX (Gamma Exposure)：决定波动性

**Gamma** 是期权 Delta 对价格变化的敏感度。**GEX** 是所有未平仓期权的 Gamma 加权总和。

**核心机制：**
- 做市商为了保持 Delta 中性，需要在价格变动时**对冲**
- 当 GEX 为正时，做市商是**净卖出期权**（Short Gamma）
- 当 GEX 为负时，做市商是**净买入期权**（Long Gamma）

**GEX 对价格的影响：**

| GEX 状态 | 做市商行为 | 价格影响 | 市场特征 |
|---------|-----------|---------|---------|
| **正 GEX** | 价格涨 → 买入对冲<br>价格跌 → 卖出对冲 | **阻尼效应**（抑制波动） | 价格被"粘住"，难以突破 |
| **负 GEX** | 价格涨 → 卖出对冲<br>价格跌 → 买入对冲 | **加速效应**（放大波动） | 价格容易快速移动 |

**实战应用：**
```python
def analyze_gex(option_data):
    """
    option_data: 包含所有未平仓期权的数据
    每个期权: strike, expiry, type (call/put), open_interest, gamma
    """
    total_gex = 0
    
    for option in option_data:
        # Gamma Exposure = Gamma × Open Interest × 100 (每手100股)
        gex = option['gamma'] * option['open_interest'] * 100
        
        # Call 的 GEX 为正，Put 的 GEX 为负（通常）
        if option['type'] == 'call':
            total_gex += gex
        else:
            total_gex -= gex
    
    return {
        'total_gex': total_gex,
        'effect': 'DAMPENING' if total_gex > 0 else 'ACCELERATING',
        'volatility_expectation': 'LOW' if total_gex > 0 else 'HIGH'
    }
```

#### Skew：决定情绪方向

**Skew** 衡量**看跌期权 (Put)** 和**看涨期权 (Call)** 的隐含波动率差异。

**计算方法：**
- **Put-Call Skew** = IV(Put) - IV(Call)
- 或使用 **25 Delta Skew** = IV(25Δ Put) - IV(25Δ Call)

**Skew 的含义：**

| Skew 状态 | 市场情绪 | 解释 |
|----------|---------|------|
| **正 Skew** | **看跌情绪** | Put 的 IV 高于 Call，市场担心下跌 |
| **负 Skew** | **看涨情绪** | Call 的 IV 高于 Put，市场预期上涨 |

**实战应用：**
```python
def calculate_skew(option_chain):
    """
    option_chain: 期权链数据
    """
    # 计算相同到期日、相同 Delta 的 Put 和 Call 的 IV
    puts_iv = [opt['iv'] for opt in option_chain if opt['type'] == 'put' and opt['delta'] == -0.25]
    calls_iv = [opt['iv'] for opt in option_chain if opt['type'] == 'call' and opt['delta'] == 0.25]
    
    if puts_iv and calls_iv:
        skew = np.mean(puts_iv) - np.mean(calls_iv)
        return {
            'skew': skew,
            'sentiment': 'BEARISH' if skew > 0 else 'BULLISH',
            'fear_level': abs(skew)  # Skew 绝对值越大，情绪越极端
        }
    return None
```

#### GEX × Skew 四象限模型

**组合使用 GEX 和 Skew 可以预测价格行为：**

```
        Skew (情绪方向)
            ↑
            |
  看跌情绪  |  看涨情绪
            |
    ────────┼────────→
            |        GEX (波动性)
            |      (阻尼/加速)
            |
```

**四个象限：**

| 象限 | GEX | Skew | 价格行为 | 交易策略 |
|------|-----|------|---------|---------|
| **象限 1** | 正（阻尼） | 正（看跌） | **横盘或受阻** | 等待突破，或做空 |
| **象限 2** | 正（阻尼） | 负（看涨） | **横盘或缓慢上涨** | 逢低买入，耐心持有 |
| **象限 3** | 负（加速） | 正（看跌） | **快速下跌（危险）** | 减仓/做空，设置止损 |
| **象限 4** | 负（加速） | 负（看涨） | **快速上涨（强势）** | 追涨或持有，但注意回调 |

**实战代码：**
```python
def gex_skew_quadrant(gex_value, skew_value):
    """
    判断当前市场处于哪个象限
    """
    if gex_value > 0 and skew_value > 0:
        return {
            'quadrant': 1,
            'description': '阻尼 + 看跌 = 横盘或受阻',
            'action': 'WAIT_OR_SHORT',
            'risk': 'MEDIUM'
        }
    elif gex_value > 0 and skew_value < 0:
        return {
            'quadrant': 2,
            'description': '阻尼 + 看涨 = 横盘或缓慢上涨',
            'action': 'BUY_ON_DIP',
            'risk': 'LOW'
        }
    elif gex_value < 0 and skew_value > 0:
        return {
            'quadrant': 3,
            'description': '加速 + 看跌 = 快速下跌（危险）',
            'action': 'REDUCE_POSITION_OR_SHORT',
            'risk': 'HIGH'
        }
    else:  # gex < 0 and skew < 0
        return {
            'quadrant': 4,
            'description': '加速 + 看涨 = 快速上涨（强势）',
            'action': 'RIDE_TREND_BUT_WATCH_REVERSAL',
            'risk': 'MEDIUM_HIGH'
        }
```

**关键洞察：**

1. **GEX 决定波动性**
   - 正 GEX = 阻尼（抑制波动）
   - 负 GEX = 加速（放大波动）

2. **Skew 决定情绪方向**
   - 正 Skew = 看跌情绪
   - 负 Skew = 看涨情绪

3. **组合决定价格行为**
   - 象限 1：阻尼 + 看跌 = 横盘或受阻
   - 象限 2：阻尼 + 看涨 = 横盘或缓慢上涨
   - 象限 3：加速 + 看跌 = 快速下跌（危险）
   - 象限 4：加速 + 看涨 = 快速上涨（强势）

**数据来源：**
- **Deribit**：BTC/ETH 期权数据最全
- **Laevitas**：提供 GEX 和 Skew 的可视化
- **Greeks.live**：实时 Gamma 和 Skew 数据

### 3.4 宏观指标

- **美元指数 (DXY)**：风险资产的反向指标
- **利率预期**：影响资金成本
- **监管消息**：政策风险

---

## 4. 系统化应用：从信息到优势

### 4.1 信息层级整合

**三层信息的优先级：**

```
Orderbook Flow (微观) > K线指标 (中观) > 场外指标 (宏观)
     ↓                    ↓                    ↓
  即时信号              趋势确认              背景判断
```

**实战流程：**

1. **场外指标**：判断大环境（牛市/熊市/震荡）
2. **K线指标**：确认趋势方向（EMA, VWAP）
3. **Orderbook Flow**：寻找精确入场点（CVD, Footprint）

### 4.2 信号过滤系统

```python
class SemiInformationTradingSystem:
    def __init__(self):
        self.signals = {
            'macro': None,      # 场外指标
            'trend': None,      # K线指标
            'micro': None,      # Orderbook Flow
        }
    
    def generate_signal(self, macro_data, kline_data, orderbook_data):
        # 1. 宏观背景
        macro_signal = self.analyze_macro(macro_data)
        if macro_signal == 'BEAR_MARKET':
            return 'NO_TRADE'  # 熊市不交易
        
        # 2. 趋势确认
        trend_signal = self.analyze_trend(kline_data)
        if trend_signal == 'NEUTRAL':
            return 'WAIT'
        
        # 3. 微观入场
        micro_signal = self.analyze_orderbook(orderbook_data)
        
        # 4. 综合判断
        if trend_signal == 'BULLISH' and micro_signal == 'BUY_PRESSURE':
            return {
                'action': 'LONG',
                'confidence': 'HIGH',
                'entry': orderbook_data['best_bid'],
                'stop_loss': kline_data['ema55'],
            }
        
        return 'WAIT'
```

### 4.3 从"不完全信息"到"半信息博弈"

**关键转变：**

| 维度 | 不完全信息博弈 | 半信息博弈 |
|------|---------------|-----------|
| **信息源** | 只看价格 | 价格 + 订单簿 + 链上数据 |
| **决策依据** | 猜测 | 数据驱动的信号 |
| **优势来源** | 运气 | 信息收集和分析能力 |
| **胜率** | ~50% | 55-60% (通过信息优势) |

**核心原则：**

1. **系统化收集**：不要依赖单一信息源
2. **实时监控**：Orderbook Flow 变化很快，需要实时跟踪
3. **多层验证**：微观、中观、宏观信号要一致
4. **持续优化**：市场在变化，系统需要适应

---

## 总结

**将期货交易变成 Semi-Information Game 的路径：**

1. **K线指标层**：EMA21/55/200, VWAP, Volume - 提供趋势方向
2. **Orderbook Flow 层**：CVD, Footprint, Heatmap, Volume Profile - 提供精确入场点
3. **场外指标层**：
   - 链上数据：交易所流入流出、稳定币变化
   - 衍生品数据：资金费率、持仓量、清算数据
   - **期权市场数据**：GEX（波动性）、Skew（情绪方向）- 预测价格行为
   - 宏观数据：DXY、利率、监管消息

**关键洞察：**

- **Orderbook Flow 是价格移动的根本**，比 K线更早反映市场意图
- **多层信息验证**可以显著提高信号质量
- **系统化应用**这些信息，可以将胜率从 50% 提升到 55-60%

**记住：你永远无法获得完全信息，但通过系统化收集和分析，你可以获得相对优势。**

