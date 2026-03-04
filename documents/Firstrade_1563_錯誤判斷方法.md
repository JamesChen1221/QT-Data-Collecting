# Firstrade 錯誤代碼 1563 判斷方法

## 錯誤訊息
```
無法接受非流動性或低價股的開倉交易
參考代碼 1563
```

## 錯誤原因

Firstrade 對於以下類型的股票限制開倉交易：
1. **低價股（Penny Stock）**：股價過低的股票
2. **非流動性股票**：成交量過低、流動性不足的股票

---

## 判斷標準

### 1. 股價標準
- **高風險**：股價 < $1
- **中風險**：股價 < $5
- **低風險**：股價 ≥ $5

### 2. 流動性標準

#### 日均成交量
- **高風險**：< 50,000 股/天
- **中風險**：50,000 - 500,000 股/天
- **低風險**：> 500,000 股/天

#### 市值
- **高風險**：< $50M（5000萬美元）
- **中風險**：$50M - $300M
- **低風險**：> $300M

#### 買賣價差
- **高風險**：價差 > 5%
- **中風險**：價差 2% - 5%
- **低風險**：價差 < 2%

### 3. 交易所
- **OTC市場**（場外交易）：高風險
- **Pink Sheets**：高風險
- **主要交易所**（NYSE, NASDAQ）：相對安全

---

## Python 實作方法

### 方法 1: 基本檢查

```python
import yfinance as yf

def check_firstrade_1563_risk(ticker):
    """
    檢查股票是否可能觸發 Firstrade 1563 錯誤
    
    參數:
        ticker: 股票代碼
    
    返回:
        dict: {
            'is_high_risk': bool,      # 是否高風險
            'risk_level': str,         # 風險等級: 'low', 'medium', 'high'
            'price': float,            # 當前價格
            'avg_volume': int,         # 日均成交量
            'market_cap': float,       # 市值
            'reasons': list            # 風險原因列表
        }
    """
    try:
        stock = yf.Ticker(ticker)
        info = stock.info
        
        # 獲取關鍵指標
        current_price = info.get('currentPrice') or info.get('regularMarketPrice') or 0
        avg_volume = info.get('averageVolume') or info.get('averageVolume10days') or 0
        market_cap = info.get('marketCap', 0)
        exchange = info.get('exchange', '')
        bid = info.get('bid', 0)
        ask = info.get('ask', 0)
        
        reasons = []
        risk_score = 0
        
        # 1. 檢查股價
        if current_price < 1:
            reasons.append(f'極低價股 (${current_price:.2f} < $1)')
            risk_score += 3
        elif current_price < 5:
            reasons.append(f'低價股 (${current_price:.2f} < $5)')
            risk_score += 2
        
        # 2. 檢查成交量
        if avg_volume < 50000:
            reasons.append(f'極低成交量 ({avg_volume:,} < 50,000)')
            risk_score += 3
        elif avg_volume < 500000:
            reasons.append(f'低成交量 ({avg_volume:,} < 500,000)')
            risk_score += 1
        
        # 3. 檢查市值
        if market_cap > 0:
            market_cap_m = market_cap / 1_000_000
            if market_cap < 50_000_000:
                reasons.append(f'小市值 (${market_cap_m:.1f}M < $50M)')
                risk_score += 2
            elif market_cap < 300_000_000:
                reasons.append(f'中小市值 (${market_cap_m:.1f}M < $300M)')
                risk_score += 1
        
        # 4. 檢查交易所
        if 'OTC' in exchange.upper() or 'PINK' in exchange.upper():
            reasons.append(f'OTC市場 ({exchange})')
            risk_score += 3
        
        # 5. 檢查買賣價差
        if bid > 0 and ask > 0:
            spread_pct = (ask - bid) / bid * 100
            if spread_pct > 5:
                reasons.append(f'高買賣價差 ({spread_pct:.1f}%)')
                risk_score += 2
            elif spread_pct > 2:
                reasons.append(f'中等買賣價差 ({spread_pct:.1f}%)')
                risk_score += 1
        
        # 判斷風險等級
        if risk_score >= 5:
            risk_level = 'high'
            is_high_risk = True
        elif risk_score >= 2:
            risk_level = 'medium'
            is_high_risk = False
        else:
            risk_level = 'low'
            is_high_risk = False
        
        return {
            'ticker': ticker,
            'is_high_risk': is_high_risk,
            'risk_level': risk_level,
            'risk_score': risk_score,
            'price': current_price,
            'avg_volume': avg_volume,
            'market_cap': market_cap,
            'exchange': exchange,
            'reasons': reasons
        }
    
    except Exception as e:
        return {
            'ticker': ticker,
            'is_high_risk': None,
            'risk_level': 'unknown',
            'error': str(e)
        }


# 使用範例
result = check_firstrade_1563_risk('AAPL')
print(f"股票: {result['ticker']}")
print(f"風險等級: {result['risk_level']}")
print(f"高風險: {result['is_high_risk']}")
print(f"價格: ${result['price']:.2f}")
print(f"日均量: {result['avg_volume']:,}")
print(f"風險原因: {', '.join(result['reasons']) if result['reasons'] else '無'}")
```

### 方法 2: 批次檢查

```python
def batch_check_1563_risk(tickers):
    """
    批次檢查多支股票的 1563 風險
    
    參數:
        tickers: 股票代碼列表
    
    返回:
        DataFrame: 包含所有股票的風險評估
    """
    import pandas as pd
    
    results = []
    for ticker in tickers:
        print(f"檢查 {ticker}...")
        result = check_firstrade_1563_risk(ticker)
        results.append(result)
    
    df = pd.DataFrame(results)
    return df


# 使用範例
tickers = ['AAPL', 'NVDA', 'TSLA', 'Q']
risk_df = batch_check_1563_risk(tickers)

# 篩選高風險股票
high_risk_stocks = risk_df[risk_df['is_high_risk'] == True]
print("\n高風險股票:")
print(high_risk_stocks[['ticker', 'risk_level', 'price', 'avg_volume', 'reasons']])
```

### 方法 3: 整合到現有程式

```python
# 在 calculate_indicators.py 中加入檢查
def calculate_with_risk_check(ticker, date):
    """
    計算指標並檢查 1563 風險
    """
    # 先檢查風險
    risk_result = check_firstrade_1563_risk(ticker)
    
    if risk_result['is_high_risk']:
        print(f"  ⚠️ 警告: {ticker} 可能觸發 Firstrade 1563 錯誤")
        print(f"     風險原因: {', '.join(risk_result['reasons'])}")
    
    # 繼續計算其他指標
    result = calculate_rsi_adx_sequences(ticker, date)
    
    # 將風險資訊加入結果
    if result:
        result['1563_risk'] = risk_result
    
    return result
```

---

## 實際應用建議

### 1. 交易前檢查
在下單前先檢查股票是否可能觸發 1563 錯誤：

```python
# 檢查單支股票
ticker = 'EXAMPLE'
risk = check_firstrade_1563_risk(ticker)

if risk['is_high_risk']:
    print(f"❌ {ticker} 可能無法在 Firstrade 開倉")
    print(f"原因: {', '.join(risk['reasons'])}")
else:
    print(f"✅ {ticker} 可以交易")
```

### 2. 建立監控清單
定期檢查持倉或觀察清單中的股票：

```python
# 從 Excel 讀取股票清單
import pandas as pd

df = pd.read_excel('量化交易.xlsx', sheet_name='資料庫')
tickers = df['公司代碼'].unique()

# 批次檢查
risk_df = batch_check_1563_risk(tickers)

# 匯出高風險清單
high_risk = risk_df[risk_df['is_high_risk'] == True]
high_risk.to_excel('高風險股票清單.xlsx', index=False)
```

### 3. 在 Excel 中標記
在「量化交易.xlsx」中新增「1563風險」欄位，自動標記高風險股票。

---

## 注意事項

### 1. 限制
- ⚠️ **無法 100% 準確**：Firstrade 的實際限制清單可能每天變動
- ⚠️ **只能推測**：透過公開指標推測，無法直接查詢 Firstrade 限制清單
- ⚠️ **需要更新**：股票的價格和流動性會變化，需定期重新檢查

### 2. 建議
- ✅ **建立黑名單**：記錄已知觸發 1563 的股票
- ✅ **保守判斷**：寧可誤判為高風險，也不要漏掉真正的高風險股票
- ✅ **人工確認**：對於重要交易，建議人工確認是否能在 Firstrade 交易

### 3. 其他券商
不同券商對低價股和非流動性股票的限制可能不同：
- **Interactive Brokers**：限制較寬鬆
- **TD Ameritrade**：類似 Firstrade
- **Robinhood**：限制某些 OTC 股票

---

## 常見問題

### Q1: 為什麼我的股票突然無法交易？
A: 可能原因：
- 股價跌破 $5 或 $1
- 成交量大幅下降
- 被移至 OTC 市場
- Firstrade 更新限制清單

### Q2: 如何避免 1563 錯誤？
A: 建議：
- 避免交易股價 < $5 的股票
- 選擇日均量 > 500,000 的股票
- 優先選擇大型股（市值 > $1B）
- 避免 OTC 市場股票

### Q3: 已持有的股票觸發 1563 怎麼辦？
A: 
- 可以平倉（賣出）
- 無法加倉（買入）
- 考慮轉移到其他券商

---

## 更新記錄

- 2026-03-05: 初版建立
- 包含基本判斷標準和 Python 實作方法
