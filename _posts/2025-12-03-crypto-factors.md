---
layout:     post
title:      "Crypto Quant: From Pure CTA to Hybrid Factor Strategies"
subtitle:   "Deep dive into Trend Following and Risk Premia in BTC Futures"
date:       2025-12-03
author:     fantianwen
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - quantitative-trading
    - cta
    - btc
    - factors
---

## 引言 (Introduction)

在加密货币量化交易中，**CTA (Commodity Trading Advisors)** 和 **因子投资 (Factor Investing)** 是两种主流的策略范式。

- **CTA 策略**：通常指趋势跟踪（Trend Following），核心在于捕捉资产价格的动量（Momentum），即“追涨杀跌”。
- **因子策略**：源于多因子模型（如 Fama-French），旨在捕获特定的风险溢价（Risk Premia），如携带（Carry）、价值（Value）、波动率（Volatility）等。

本文将探讨如何在 BTC 期货上构建纯 CTA 策略，以及如何融合因子构建更稳健的混合策略。

---

## 1. Pure CTA Strategy: The Trend Follower
> "Cut losses short, let profits run."

纯 CTA 策略的核心假设是：**市场价格具有序列相关性，趋势一旦形成，往往会持续一段时间。** BTC 由于其高波动性和情绪驱动特性，是趋势策略的绝佳标的。

### 1.1 核心逻辑：时间序列动量 (Time-Series Momentum, TSMOM)

最经典的 CTA 策略不是预测未来，而是跟随现在。

**策略示例：双均线交叉 (Dual Moving Average Crossover)**

*   **信号生成**：
    *   $SMA_{fast}$ (e.g., 20 days) > $SMA_{slow}$ (e.g., 60 days) $\rightarrow$ **Long**
    *   $SMA_{fast}$ < $SMA_{slow}$ $\rightarrow$ **Short** or **Close**
*   **核心优势**：捕捉长尾的大波动（Fat Tails）。
*   **核心劣势**：震荡市场中反复磨损（Whipsaw）。

### 1.2 波动率目标调整 (Volatility Targeting)

CTA 的关键不在于信号，而在于**风控**。BTC 的波动率是变化的，因此仓位大小应该是动态的。

$$ Position Size = \frac{Target Risk \times Capital}{\sigma_{daily}} $$

*   当市场平静时（$\sigma$ 低），放大仓位。
*   当市场狂暴时（$\sigma$ 高），降低仓位。
*   这保证了无论市场怎么变，你承担的风险是恒定的。

---

## 2. Crypto Factors: Beyond Price Trends
因子策略关注的是**结构性的超额收益来源**。在 BTC 期货市场，最重要的几个因子是：

### 2.1 携带因子 (Carry Factor) ⭐⭐⭐
在加密市场，Carry 主要体现为**资金费率 (Funding Rate)** 和 **基差 (Basis)**。

*   **定义**：持有资产所获得的无风险（或低风险）收益。
*   **逻辑**：如果资金费率为正（多头付给空头），做空期货具有“正 Carry”。
*   **策略**：
    *   **Cash & Carry Arbitrage**：买入现货，做空期货（当基差 > 0）。
    *   **Directional Carry**：作为过滤器。如果我要做多，最好资金费率是负的（我能收到钱）；或者至少不高。

### 2.2 动量因子 (Momentum Factor)
这与 CTA 重叠，但在多因子框架下，它通常指**横截面动量 (Cross-Sectional Momentum)**。
*   **逻辑**：买入过去一段时间表现最好的币，做空表现最差的币（BTC vs ETH vs SOL）。

### 2.3 波动率因子 (Volatility Factor)

*   **逻辑：均值回归 (Mean Reversion)**。波动率具有"聚类"特性：大波动后往往跟着大波动，平静期后往往跟着平静期。但长期来看，极端的低波动（暴风雨前的宁静）和极端的高波动（恐慌释放）都会回归均值。
*   **策略**：
    *   **做多波动率 (Long Vol)**：在历史波动率极低（如 BTC 30天年化波动率 < 30%）时，买入**跨式期权 (Straddle)** 或 **宽跨式 (Strangle)**。虽然每天损耗 Theta，但一旦大行情爆发（无论涨跌），Gamma 收益将覆盖成本。
        *   *高级玩法 (Delta Hedging)*：即使持有到期未获利，也可以通过动态对冲 Delta 来赚取 Gamma Scalping 收益。
    *   **做空波动率 (Short Vol)**：在市场恐慌、隐含波动率 (IV) 极高时，卖出期权或做空波动率代币。这类似于"卖保险"，长期胜率高，但需防范黑天鹅。
    *   **方差互换 (Variance Swap)**：机构常用的纯波动率交易工具，直接对赌未来的实现波动率。

---

## 3. Hybrid Strategy: CTA + Factors
> "Trend tells you direction; Carry tells you conviction."

纯 CTA 在震荡市会亏损，纯 Carry 策略在单边大行情中可能跑输（或爆仓）。混合策略旨在结合两者的优点。

### 3.1 融合思路：信号过滤 (Signal Filtering)

我们不单纯依赖均线，而是加入 Carry 因子作为“红绿灯”。

#### 什么是 Crypto 的 Carry (携带收益)？

在传统外汇中，Carry 是高息货币的利息。在 Crypto 中，Carry 主要由以下构成：
1.  **永续合约资金费率 (Funding Rate)**：最直接的 Carry。多头付给空头（正费率）或空头付给多头（负费率）。年化收益率 = `资金费率 * 3 * 365`。
2.  **交割期货基差 (Futures Basis)**：远期价格 - 现货价格。通常是正的（Contango），随着到期日临近，基差收敛归零，空头获得无风险收益。
3.  **借贷利率 (Lending Rate)**：在 DeFi (Aave/Compound) 或 CEX 借出 USDT/BTC 的利息。

**策略逻辑示例：**

1.  **CTA 信号 (主方向)**：BTC 站上 60日均线 $\rightarrow$ 看多。
    *   *为什么是 60日？* 在加密市场，**季度 (Quarterly)** 是一个重要的周期。60日（约2个月，接近一个季度的大部分时间）通常能过滤掉月度级别的噪音，确认中期趋势。这也是机构资金（如 Grayscale, ETF）调仓和宏观政策传导的典型周期。
2.  **Carry 信号 (验证)**：
    *   如果资金费率正常或为负 $\rightarrow$ **全仓做多**（趋势向上 + 持仓成本低/有收益）。
    *   如果资金费率极高（年化 > 50%）$\rightarrow$ **轻仓做多或观望**（趋势虽然向上，但拥挤度过高，回调风险大，且持仓成本极高）。

### 3.2 融合思路：多策略组合 (Portfolio approach)

将资金分配给两个不相关的子策略：

$$ Portfolio = w_1 \cdot \text{Strategy}_{CTA} + w_2 \cdot \text{Strategy}_{Carry} $$

*   **CTA 子策略**：负责抓单边暴涨暴跌（Gamma Long）。
*   **Carry 子策略**：负责在横盘震荡时赚取资金费（Delta Neutral）。
    *   *如何运作？* 这是一个**期现套利 (Cash and Carry)** 策略。买入 1 BTC 现货，同时做空 1 BTC 永续合约。无论 BTC 涨跌，你的总资产价值（以 USD 计）不变。
    *   *收益来源*：你每天每 8 小时收取多头支付给你的资金费。
    *   *为什么互补？* 当市场横盘时，CTA 会因为假突破磨损本金，而 Carry 策略在持续“收租”。当市场暴涨时，CTA 赚大钱，Carry 策略收益平稳（甚至因为费率飙升赚更多）。
*   **效果**：平滑资金曲线，提高夏普比率（Sharpe Ratio）。

---

## 4. Implementation Guide (Python Logic)

以下是一个简化的混合策略代码逻辑，结合了均线趋势和资金费率过滤。

```python
import pandas as pd
import numpy as np

class HybridCTA:
    def __init__(self, df):
        # df columns: ['close', 'funding_rate']
        self.df = df
        
    def generate_signals(self):
        # 1. CTA Signal (Trend)
        self.df['sma_20'] = self.df['close'].rolling(window=20).mean()
        self.df['sma_60'] = self.df['close'].rolling(window=60).mean()
        
        # Basic Trend: 1 = Uptrend, -1 = Downtrend
        self.df['trend_signal'] = np.where(self.df['sma_20'] > self.df['sma_60'], 1, -1)
        
        # 2. Factor Signal (Carry / Funding Rate)
        # Calculate rolling average funding rate (e.g., 3 days)
        self.df['avg_funding'] = self.df['funding_rate'].rolling(window=8*3).mean()
        
        # Crowded Trade Filter: 
        # If funding is extremely positive (>0.05% per 8h), Longs are crowded.
        # If funding is extremely negative (<-0.05% per 8h), Shorts are crowded.
        crowded_threshold = 0.0005
        
        self.df['carry_filter'] = 1.0 # Default multiplier
        
        # 如果做多信号出现，但费率过高，减少仓位
        mask_long_crowded = (self.df['trend_signal'] == 1) & (self.df['avg_funding'] > crowded_threshold)
        self.df.loc[mask_long_crowded, 'carry_filter'] = 0.5  # Cut position in half
        
        # 如果做多信号出现，且费率为负（空头支付多头），这是极好的信号
        mask_long_value = (self.df['trend_signal'] == 1) & (self.df['avg_funding'] < 0)
        self.df.loc[mask_long_value, 'carry_filter'] = 1.2  # Increase position
        
        # 3. Final Position
        # Volatility Targeting logic can be added here
        self.df['position'] = self.df['trend_signal'] * self.df['carry_filter']
        
        return self.df

# 模拟数据分析
# result = HybridCTA(market_data).generate_signals()
```

### 关键要点 (Key Takeaways)

1.  **不要只看价格**：Crypto 市场有丰富的衍生品数据（Funding, OI, Liquidations），这些是传统市场没有的 Alpha 源。
2.  **趋势是王道，但成本很重要**：CTA 给你方向，Carry 给你持仓的底气。
3.  **波动率管理**：BTC 的波动率会在 30% 到 150% 之间切换，必须动态调整仓位，否则一次回撤就会致命。

#### 经典仓位管理方法 (Position Sizing)

*   **波动率倒数加权 (Inverse Volatility Weighting)**：
    *   $$ Position = \frac{Target Risk (e.g. 2\%)}{Daily Volatility (\%)} $$
    *   如果 BTC 日波动 5%，仓位 = 40%；如果日波动 10%，仓位自动降至 20%。这保证了你的账户每日风险始终在 2% 左右。
*   **凯利公式 (Kelly Criterion)**：
    *   $$ f^* = \frac{bp - q}{b} $$
    *   激进的数学最优解，通常在 Crypto 中使用 **Half-Kelly**（0.5倍凯利）来避免破产风险，因为 Crypto 的分布是厚尾的（Fat-tailed）。
*   **ATR 止损倒推法**：
    *   先确定止损距离（如 2倍 ATR）。
    *   计算单笔亏损额（如总资金的 1%）。
    *   $$ Size = \frac{Capital \times 1\%}{2 \times ATR} $$
    *   这是最适合趋势跟踪者的实战方法。

## 总结

对于 BTC 期货交易：
*   **初阶**：使用 15m/1h/4h 多周期共振做纯趋势跟踪。
*   **进阶**：引入 Carry 因子（资金费率），在拥挤交易时减仓，在负费率时加仓。
*   **高阶**：构建多策略组合，让 Trend 和 Carry 互相由于相关性低而互补，实现穿越牛熊的收益曲线。
