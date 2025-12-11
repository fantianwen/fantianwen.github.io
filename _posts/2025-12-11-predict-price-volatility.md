---
layout:     post
title:      "Predicting Price Movements from Technical Indicators: A ML Approach"
subtitle:   "Using Multi-Timeframe KDJ/MACD Crossovers to Forecast BTC Price Changes"
date:       2025-12-11
author:     fantianwen
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - machine-learning
    - technical-analysis
    - btc
    - prediction
---

## 引言

传统技术分析告诉我们：KDJ 金叉/死叉、MACD 金叉/死叉是买入/卖出信号。但**信号出现后，价格会涨/跌多少？** 这个问题很少有量化答案。

本文将探讨：**使用机器学习预测技术指标交叉后的价格变动幅度是否可行？** 以及如何实现。

---

## 1. 这个想法有价值吗？(Is This Valuable?)

### 1.1 理论上的价值

**✅ 有价值的原因：**

1. **位置管理优化**：如果你知道交叉后平均涨 2%，你可以设置 1.5% 的止盈，而不是盲目等待。
2. **信号过滤**：如果模型预测涨幅 < 0.5%，这个信号可能不值得交易（手续费+滑点就吃掉了）。
3. **多时间周期共振**：5m/15m/1h/4h/1d 同时出现信号，预测的涨幅可能更大（动量叠加）。

**⚠️ 现实中的挑战：**

1. **市场效率**：如果这个模式真的有效，很快会被套利掉。
2. **过拟合风险**：技术指标交叉是事后定义的，容易在历史数据上过拟合。
3. **非线性关系**：价格变动受太多因素影响（流动性、新闻、大户操作），单纯看指标可能不够。

### 1.2 我的建议

**这个项目有价值，但需要调整目标：**

- ❌ **不要预测精确价格**（如"会涨 2.34%"）
- ✅ **预测价格变动区间**（如"有 70% 概率涨 1-3%"）
- ✅ **预测方向 + 置信度**（如"看涨，置信度 65%"）
- ✅ **作为辅助工具**，而不是唯一决策依据

---

## 2. 实现方法 (Implementation Methods)

### 2.1 问题定义

**输入 (Features):**
- 多时间周期 (5m, 15m, 1h, 4h, 1d) 的 KDJ 和 MACD 指标值
- 交叉信号（金叉/死叉）的强度
- 市场状态（波动率、成交量、资金费率等）

**输出 (Target):**
- **回归任务**：未来 N 根 K 线后的价格变动百分比
- **分类任务**：价格变动区间（如：<0.5%, 0.5-2%, >2%）

### 2.2 特征工程 (Feature Engineering)

```python
import pandas as pd
import numpy as np
import talib

class FeatureExtractor:
    def __init__(self):
        self.timeframes = ['5m', '15m', '1h', '4h', '1d']
    
    def extract_features(self, df_dict):
        """
        df_dict: {'5m': df_5m, '15m': df_15m, ...}
        每个 df 包含: open, high, low, close, volume
        """
        features = []
        
        for tf, df in df_dict.items():
            # 1. KDJ 指标
            k, d = talib.STOCH(df['high'], df['low'], df['close'])
            j = 3 * k - 2 * d
            
            # KDJ 交叉信号
            kdj_golden = (k > d) & (k.shift(1) <= d.shift(1))  # 金叉
            kdj_death = (k < d) & (k.shift(1) >= d.shift(1))  # 死叉
            
            # 2. MACD 指标
            macd, signal, hist = talib.MACD(df['close'])
            macd_golden = (macd > signal) & (macd.shift(1) <= signal.shift(1))
            macd_death = (macd < signal) & (macd.shift(1) >= signal.shift(1))
            
            # 3. 特征向量
            tf_features = {
                f'{tf}_kdj_k': k.iloc[-1],
                f'{tf}_kdj_d': d.iloc[-1],
                f'{tf}_kdj_j': j.iloc[-1],
                f'{tf}_kdj_golden': int(kdj_golden.iloc[-1]),
                f'{tf}_kdj_death': int(kdj_death.iloc[-1]),
                f'{tf}_macd': macd.iloc[-1],
                f'{tf}_macd_signal': signal.iloc[-1],
                f'{tf}_macd_hist': hist.iloc[-1],
                f'{tf}_macd_golden': int(macd_golden.iloc[-1]),
                f'{tf}_macd_death': int(macd_death.iloc[-1]),
                f'{tf}_volatility': df['close'].pct_change().std() * np.sqrt(288),  # 年化波动率
                f'{tf}_volume_ratio': df['volume'].iloc[-1] / df['volume'].rolling(20).mean().iloc[-1],
            }
            features.append(tf_features)
        
        # 4. 多周期共振特征
        # 统计有多少个周期同时出现金叉/死叉
        golden_count = sum([f[f'{tf}_kdj_golden'] for tf, f in zip(self.timeframes, features)])
        death_count = sum([f[f'{tf}_kdj_death'] for tf, f in zip(self.timeframes, features)])
        
        combined_features = {
            'multi_tf_golden_count': golden_count,
            'multi_tf_death_count': death_count,
            'signal_strength': golden_count - death_count,  # 正数=看涨，负数=看跌
        }
        
        # 合并所有特征
        all_features = {}
        for f in features:
            all_features.update(f)
        all_features.update(combined_features)
        
        return pd.Series(all_features)
```

### 2.3 标签生成 (Label Generation)

```python
def generate_labels(df, lookforward_periods=20):
    """
    生成未来 N 根 K 线的价格变动标签
    
    df: 包含 close 价格的 DataFrame
    lookforward_periods: 预测未来多少根 K 线
    """
    # 计算未来价格变动
    future_price = df['close'].shift(-lookforward_periods)
    current_price = df['close']
    
    # 价格变动百分比
    price_change_pct = (future_price - current_price) / current_price * 100
    
    # 分类标签（可选）
    def classify_change(pct):
        if pct < -2:
            return 0  # 大跌
        elif pct < -0.5:
            return 1  # 小跌
        elif pct < 0.5:
            return 2  # 横盘
        elif pct < 2:
            return 3  # 小涨
        else:
            return 4  # 大涨
    
    df['target_regression'] = price_change_pct  # 回归目标
    df['target_classification'] = price_change_pct.apply(classify_change)  # 分类目标
    
    return df
```

### 2.4 模型选择 (Model Selection)

#### 方案 A: 回归模型（预测具体涨幅）

```python
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error, r2_score

class PriceMovementPredictor:
    def __init__(self):
        # 尝试多种模型
        self.models = {
            'rf': RandomForestRegressor(n_estimators=100, max_depth=10),
            'gbm': GradientBoostingRegressor(n_estimators=100, learning_rate=0.1),
        }
    
    def train(self, X, y):
        X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
        
        results = {}
        for name, model in self.models.items():
            model.fit(X_train, y_train)
            y_pred = model.predict(X_test)
            
            mae = mean_absolute_error(y_test, y_pred)
            r2 = r2_score(y_test, y_pred)
            
            results[name] = {
                'model': model,
                'mae': mae,
                'r2': r2,
            }
            
            print(f"{name} - MAE: {mae:.4f}, R2: {r2:.4f}")
        
        # 选择最佳模型
        best_model_name = min(results, key=lambda x: results[x]['mae'])
        self.best_model = results[best_model_name]['model']
        
        return results
    
    def predict(self, X):
        return self.best_model.predict(X)
```

#### 方案 B: 分类模型（预测涨跌区间）

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix

class PriceMovementClassifier:
    def __init__(self):
        self.model = RandomForestClassifier(n_estimators=100, max_depth=10, class_weight='balanced')
    
    def train(self, X, y_class):
        X_train, X_test, y_train, y_test = train_test_split(X, y_class, test_size=0.2, random_state=42)
        
        self.model.fit(X_train, y_train)
        y_pred = self.model.predict(X_test)
        
        print(classification_report(y_test, y_pred))
        print("\nConfusion Matrix:")
        print(confusion_matrix(y_test, y_pred))
        
        return self.model
    
    def predict_proba(self, X):
        """返回每个类别的概率"""
        return self.model.predict_proba(X)
```

#### 方案 C: 时间序列模型（LSTM/Transformer）

```python
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense, Dropout

class LSTMPredictor:
    def __init__(self, sequence_length=60):
        self.sequence_length = sequence_length
    
    def prepare_sequences(self, data, target):
        """将数据转换为时间序列格式"""
        X, y = [], []
        for i in range(len(data) - self.sequence_length):
            X.append(data[i:i+self.sequence_length])
            y.append(target[i+self.sequence_length])
        return np.array(X), np.array(y)
    
    def build_model(self, input_shape):
        model = Sequential([
            LSTM(50, return_sequences=True, input_shape=input_shape),
            Dropout(0.2),
            LSTM(50, return_sequences=False),
            Dropout(0.2),
            Dense(25),
            Dense(1)  # 回归输出
        ])
        model.compile(optimizer='adam', loss='mse', metrics=['mae'])
        return model
```

### 2.5 完整流程示例

```python
# 1. 数据准备
feature_extractor = FeatureExtractor()
df_dict = {
    '5m': get_klines('BTCUSDC', '5m'),
    '15m': get_klines('BTCUSDC', '15m'),
    '1h': get_klines('BTCUSDC', '1h'),
    '4h': get_klines('BTCUSDC', '4h'),
    '1d': get_klines('BTCUSDC', '1d'),
}

# 2. 特征提取
features_list = []
for i in range(len(df_dict['5m']) - 100):  # 留出足够的历史数据
    current_data = {tf: df.iloc[max(0, i-100):i+1] for tf, df in df_dict.items()}
    features = feature_extractor.extract_features(current_data)
    features_list.append(features)

X = pd.DataFrame(features_list)

# 3. 标签生成（使用 1h 数据作为基准）
df_1h = df_dict['1h']
df_1h = generate_labels(df_1h, lookforward_periods=20)  # 预测未来 20 根 1h K 线
y = df_1h['target_regression'].iloc[100:].values  # 对齐特征

# 4. 训练模型
predictor = PriceMovementPredictor()
results = predictor.train(X, y)

# 5. 预测
current_features = feature_extractor.extract_features(df_dict)
predicted_change = predictor.predict([current_features])
print(f"预测未来价格变动: {predicted_change[0]:.2f}%")
```

---

## 3. 关键注意事项 (Critical Considerations)

### 3.1 数据泄露 (Data Leakage)

⚠️ **常见错误**：使用未来数据预测未来
- ❌ 在计算特征时使用了 `df.shift(-1)`（未来数据）
- ✅ 所有特征必须只使用历史数据

### 3.2 过拟合 (Overfitting)

**防范措施：**
1. **时间序列交叉验证**：不要随机分割，要按时间顺序分割
2. **特征选择**：使用特征重要性（Random Forest）筛选真正有用的特征
3. **正则化**：L1/L2 正则化，Dropout（神经网络）

### 3.3 模型评估

**不要只看 R² 或准确率！**

```python
def evaluate_model(y_true, y_pred):
    # 1. 回归指标
    mae = mean_absolute_error(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    
    # 2. 方向准确率（更重要！）
    direction_accuracy = np.mean(np.sign(y_true) == np.sign(y_pred))
    
    # 3. 盈亏模拟
    # 假设：预测涨就做多，预测跌就做空
    returns = np.where(y_pred > 0, y_true, -y_true)  # 如果预测涨，收益=y_true；预测跌，收益=-y_true
    total_return = np.sum(returns)
    sharpe = total_return / np.std(returns) / np.sqrt(252) if np.std(returns) > 0 else 0
    
    print(f"MAE: {mae:.4f}")
    print(f"方向准确率: {direction_accuracy:.2%}")
    print(f"模拟总收益: {total_return:.2f}%")
    print(f"模拟夏普比率: {sharpe:.2f}")
```

---

## 4. 实际应用建议 (Practical Recommendations)

### 4.1 不要完全依赖模型

**模型输出应该作为：**
- ✅ 信号强度的参考（"这个信号历史平均涨 2%，但模型预测只涨 0.3%，可能不值得做"）
- ✅ 止盈目标的参考（"模型预测涨 1.5%，我设置 1.2% 止盈"）
- ❌ 唯一的交易决策（"模型说涨就做多"）

### 4.2 持续更新模型

- **定期重训练**：市场在变化，模型需要适应
- **在线学习**：使用新数据增量更新模型
- **A/B 测试**：对比使用模型 vs 不使用模型的策略表现

### 4.3 结合其他信息

**模型特征应该包括：**
- 技术指标（KDJ, MACD）
- 市场微观结构（订单簿深度、大单）
- 链上数据（交易所流入流出）
- 情绪指标（恐惧贪婪指数）

---

## 总结

**预测技术指标交叉后的价格变动是有价值的，但需要：**

1. **调整目标**：预测区间而非精确值，预测方向+置信度
2. **多时间周期**：5m/15m/1h/4h/1d 共振信号更有价值
3. **特征工程**：不仅看指标值，还要看信号强度、市场状态
4. **模型选择**：从简单（Random Forest）开始，再尝试复杂（LSTM）
5. **严格评估**：方向准确率 > 精确预测，模拟交易验证

**记住：模型是工具，不是圣杯。** 最好的策略是：**技术分析 + 机器学习 + 风险管理** 的组合。
