---
layout:     post                    # 使用的布局（不需要改）
title:      Week Summary               # 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2025-11-30            # 时间
author:     fantianwen                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - tag1
    - tag2
---

## 段落1

1. 单币种：

1.1 find the abivous trend and enter, less trades but high profit. control the stoploss.

2. multi-coins trategy:

2.1 find coins without relationship and use long-short strategies to profit.

## 段落2

1. from the system/all markets aspects, the more stable the whole system is, the high of the real risk is higer

---

## 分析与思考

### 段落1 - 交易策略分析

**1.1 单币种策略 - "找到明显趋势再入场，减少交易次数但提高利润，控制止损"**

这是典型的**趋势跟踪**哲学，非常明智。大多数散户交易者过度交易，最终被手续费和滑点拖垮。

**BTC/USDC 示例：**
- 当 BTC 在 2023 年 10 月突破 3 万美元阻力位时，这就是一个"明显趋势"信号
- 与其频繁做 50 次短线交易，不如找准一次机会，用合理的仓位入场
- 设置止损在 2.8 万美元（关键支撑位），目标 4 万美元以上
- 风险回报比 = 1:5，即使只有 40% 的胜率也能保持盈利

**2.1 多币种长空策略 - 寻找无关联性的币种进行对冲**

这本质上是**配对交易**或**市场中性策略**，对于降低方向性风险非常有效。

**示例：**
- 做多 BTC（价值存储叙事）+ 做空某个 meme 币（市场不确定时）
- 如果市场暴跌：BTC 跌 10%，meme 跌 30% → 净盈利
- 如果市场暴涨：BTC 涨 15%，meme 涨 10% → 净盈利
- 关键：找到在波动条件下相关性会断裂的币对

#### 如何找到相关性断裂的币对？

**哲学原理：**

核心思想是：在正常市场中，很多币走势相似（高相关性）。但在极端波动时，不同币的"本质属性"会暴露出来，相关性会断裂。

**类比：** 风平浪静时，所有船都一起摇晃。但当风暴来临，结实的大船稳住了，脆弱的小船会翻。

**实操方法：**

```python
import pandas as pd
import numpy as np

# 1. 计算滚动相关系数
def find_divergent_pairs(prices_df, window=30):
    """
    prices_df: DataFrame with coin prices (columns = coins)
    """
    # 计算收益率
    returns = prices_df.pct_change()
    
    # 计算滚动相关系数
    rolling_corr = returns['BTC'].rolling(window).corr(returns['MEME_COIN'])
    
    # 找到相关性显著下降的时刻
    corr_drop = rolling_corr.diff()
    
    return rolling_corr, corr_drop

# 2. 筛选标准
# - 正常时期相关性 > 0.7
# - 波动时期相关性 < 0.3
# - 这种币对适合做配对交易
```

**寻找币对的思路：**

| 类别 | 高波动时表现 | 适合配对 |
|------|-------------|---------|
| **BTC** | 相对稳定，资金避险流入 | 做多 |
| **ETH** | 中等波动 | 中性 |
| **山寨币** | 大幅下跌，流动性枯竭 | 做空 |
| **Meme币** | 极端波动，可能暴跌 | 做空 |
| **稳定币** | 可能脱锚（风险！） | 谨慎 |

**推荐币对：**
- BTC vs SHIB/DOGE（BTC稳，meme崩）
- ETH vs 小市值DeFi币
- BNB vs 新上线的币

### 段落2 - 系统性风险视角

> "系统越稳定，真实风险越高"

这是一个深刻的观察，呼应了**明斯基的金融不稳定假说**：

> *"稳定孕育着不稳定"*

**加密货币中的例子：**
- 2021 年：一切看起来都很稳定，杠杆到处累积
- Terra/LUNA、三箭资本、FTX 都显得"稳定"，提供一致的收益率
- 当它破裂时，连锁反应是灾难性的

**对你的 BTC/USDC 套利：**
- 当价差持续收窄且可预测时 → 所有人都会涌入
- 流动性提供者变得自满
- 一次黑天鹅事件（交易所问题、脱锚事件）可能抹掉数月的收益

### 对套利策略的建议

既然你目前在做 BTC/USDC 套利：

1. **监控 USDC 锚定稳定性**（记住 2023 年 3 月的 SVB 事件）

#### 如何系统性监控 USDC 锚定稳定性？

**监控指标：**

```python
# USDC 稳定性监控脚本
import requests

def monitor_usdc_peg():
    """监控 USDC 锚定状态"""
    
    alerts = []
    
    # 1. 价格偏离度 (从 CoinGecko/Binance API)
    usdc_price = get_usdc_price()  # 理想值 = 1.0
    deviation = abs(usdc_price - 1.0)
    if deviation > 0.005:  # 偏离超过 0.5%
        alerts.append(f"⚠️ USDC 价格偏离: {usdc_price}")
    
    # 2. Curve 3pool 比例 (USDC/USDT/DAI)
    # 正常: 各占 ~33%
    # 危险: USDC 占比 > 40% (大家在抛售 USDC)
    curve_balance = get_curve_3pool_balance()
    if curve_balance['USDC'] > 0.40:
        alerts.append(f"⚠️ Curve池USDC占比过高: {curve_balance['USDC']:.1%}")
    
    # 3. Circle 储备金透明度
    # 监控 Circle 官方公告和储备报告
    
    # 4. 链上大额转账
    # 监控 USDC 大额转入交易所 (可能是机构抛售)
    
    return alerts

# 数据源
APIS = {
    'coingecko': 'https://api.coingecko.com/api/v3/simple/price?ids=usd-coin&vs_currencies=usd',
    'curve': 'https://api.curve.fi/api/getPools/ethereum/3pool',
    'etherscan': 'USDC大额转账监控'
}
```

**关键监控点：**

1. **USDC/USD 价格** — 偏离 > 0.5% 就要警惕
2. **Curve 3pool 比例** — USDC 占比异常升高 = 危险信号
3. **Circle 官方公告** — 关注储备金相关新闻
4. **Twitter/社交媒体情绪** — 恐慌往往先从社交媒体传播
2. **跟踪各交易所的资金费率和基差**

#### 如何跟踪资金费率和基差？

**什么是资金费率和基差？**

- **资金费率 (Funding Rate):** 永续合约多空双方定期支付的费用
  - 正费率 = 多头付给空头（市场偏多）
  - 负费率 = 空头付给多头（市场偏空）
  
- **基差 (Basis):** 期货价格 - 现货价格
  - 正基差 = 期货溢价（看涨情绪）
  - 负基差 = 期货折价（看跌情绪）

**监控脚本：**

```python
import ccxt
import pandas as pd

class FundingRateMonitor:
    def __init__(self):
        self.exchanges = {
            'binance': ccxt.binance({'options': {'defaultType': 'future'}}),
            'okx': ccxt.okx({'options': {'defaultType': 'swap'}}),
            'bybit': ccxt.bybit({'options': {'defaultType': 'linear'}}),
        }
    
    def get_funding_rates(self, symbol='BTC/USDT:USDT'):
        """获取各交易所资金费率"""
        rates = {}
        for name, exchange in self.exchanges.items():
            try:
                funding = exchange.fetch_funding_rate(symbol)
                rates[name] = {
                    'funding_rate': funding['fundingRate'],
                    'next_funding_time': funding['fundingTimestamp'],
                }
            except Exception as e:
                rates[name] = {'error': str(e)}
        return rates
    
    def get_basis(self, symbol='BTC/USDT'):
        """计算期货-现货基差"""
        # 获取现货价格
        spot = self.exchanges['binance'].fetch_ticker(symbol)
        spot_price = spot['last']
        
        # 获取永续合约价格
        perp = self.exchanges['binance'].fetch_ticker(f"{symbol}:USDT")
        perp_price = perp['last']
        
        basis = (perp_price - spot_price) / spot_price * 100
        return {
            'spot_price': spot_price,
            'perp_price': perp_price,
            'basis_percent': basis
        }
    
    def alert_check(self):
        """检查异常情况"""
        alerts = []
        
        rates = self.get_funding_rates()
        basis = self.get_basis()
        
        # 资金费率异常高 (> 0.1% 每8小时 = 年化 > 100%)
        for ex, data in rates.items():
            if 'funding_rate' in data and abs(data['funding_rate']) > 0.001:
                alerts.append(f"⚠️ {ex} 资金费率异常: {data['funding_rate']:.4%}")
        
        # 基差异常 (> 1%)
        if abs(basis['basis_percent']) > 1:
            alerts.append(f"⚠️ 基差异常: {basis['basis_percent']:.2f}%")
        
        return alerts

# 使用示例
monitor = FundingRateMonitor()
print(monitor.get_funding_rates())
print(monitor.get_basis())
print(monitor.alert_check())
```

**如何解读这些信息？**

| 指标 | 数值 | 含义 | 套利机会 |
|------|------|------|---------|
| 资金费率 | > +0.05% | 多头过热 | 做空永续 + 做多现货 |
| 资金费率 | < -0.05% | 空头过热 | 做多永续 + 做空现货 |
| 基差 | > +1% | 期货溢价高 | 卖期货 + 买现货 |
| 基差 | < -1% | 期货折价 | 买期货 + 卖现货 |

**推荐数据源：**

- **Coinglass:** https://www.coinglass.com/FundingRate
- **Laevitas:** https://app.laevitas.ch/
- **各交易所 API:** Binance, OKX, Bybit
3. **"明显趋势"的洞察也适用这里**：当套利价差显著扩大时，那就是你的"趋势"信号——但也可能意味着系统压力

#### 当套利价差显著扩大时，如何设计策略？

**策略设计框架：**

当价差扩大时，有两种可能：
1. **正常波动** — 价差会回归，套利机会
2. **系统风险** — 价差继续扩大，可能爆仓

**分层策略设计：**

```python
class SpreadStrategy:
    def __init__(self):
        # 价差阈值设置
        self.normal_spread = 0.1  # 0.1% 正常价差
        self.entry_spread = 0.3   # 0.3% 开始入场
        self.danger_spread = 1.0  # 1.0% 危险信号
        self.max_position = 100   # 最大仓位百分比
    
    def calculate_position_size(self, current_spread):
        """
        价差越大，仓位越小（反直觉但更安全）
        """
        if current_spread < self.entry_spread:
            return 0  # 价差太小，不值得做
        
        if current_spread > self.danger_spread:
            return self.max_position * 0.2  # 危险区域，只用20%仓位
        
        # 正常区域：价差越大，仓位越大，但有上限
        position = min(
            (current_spread / self.entry_spread) * 30,  # 基础仓位
            self.max_position * 0.6  # 上限60%
        )
        return position
    
    def entry_decision(self, spread, market_context):
        """
        入场决策
        """
        decision = {
            'action': 'WAIT',
            'position_size': 0,
            'stop_loss': None,
            'take_profit': None,
        }
        
        # 检查市场背景
        if market_context['usdc_deviation'] > 0.01:  # USDC脱锚风险
            decision['action'] = 'CLOSE_ALL'
            decision['reason'] = 'USDC风险'
            return decision
        
        if market_context['funding_rate_extreme']:  # 资金费率极端
            decision['action'] = 'REDUCE'
            decision['reason'] = '资金费率异常'
            return decision
        
        # 正常套利逻辑
        if spread >= self.entry_spread:
            decision['action'] = 'ENTER'
            decision['position_size'] = self.calculate_position_size(spread)
            decision['stop_loss'] = spread * 2  # 价差翻倍就止损
            decision['take_profit'] = self.normal_spread  # 回归正常就止盈
        
        return decision
```

**关键策略原则：**

1. **分批入场**
   - 价差 0.3% → 入场 20% 仓位
   - 价差 0.5% → 再入场 20%
   - 价差 0.8% → 再入场 20%（但要更谨慎）

2. **设置硬止损**
   - 如果价差扩大到 2%，说明有系统性问题，立刻平仓
   - 不要死扛！

3. **监控退出信号**
   - USDC 开始脱锚 → 立刻退出
   - 交易所出现提币问题 → 立刻退出
   - 资金费率极端异常 → 减仓

4. **保持流动性**
   - 永远保留 30% 资金不动
   - 确保能在任何时候快速退出

**决策流程图：**

```
价差扩大
    │
    ├─→ 检查 USDC 稳定性 ─→ 异常 ─→ 退出/不入场
    │
    ├─→ 检查资金费率 ─→ 极端 ─→ 谨慎/减仓
    │
    └─→ 一切正常 ─→ 分批入场套利
                        │
                        ├─→ 价差回归 ─→ 止盈
                        │
                        └─→ 价差继续扩大 ─→ 触发止损
```

### 总结

你的总结显示你同时在**战术层面**（交易执行）和**战略层面**（系统风险）思考问题。这很罕见且很有价值。继续保持这种思维方式！

