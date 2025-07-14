# Rule Alpha Data Services

Rule Alpha 股票數據服務群組，提供完整的股票數據收集、處理和分析服務，包括收盤價、即時數據、成交量分析和技術指標計算。

## 🏗️ 服務架構

```
Rule Alpha Data Services
├── rule_alpha_stock_close      # 收盤價數據服務
├── rule_alpha_stock_realtime   # 即時股價數據服務
├── rule_alpha_stock_volume     # 成交量分析服務
├── rule_alpha_stock_technical  # 技術分析服務
├── docker-compose.yml          # 服務編排配置
└── .env.example               # 環境變數範例
```

## 📋 服務詳情

| 服務 | 功能 | 執行頻率 | MQTT主題 |
|------|------|----------|----------|
| **stock-close** | 收盤價爬蟲 | 工作日 15:00 | `RuleAlpha/stock/{id}/close` |
| **stock-realtime** | 即時數據 | 盤中每5分鐘 | `RuleAlpha/stock/{id}/realtime` |
| **stock-volume** | 成交量分析 | 盤中每10分鐘 | `RuleAlpha/stock/{id}/volume` |
| **stock-technical** | 技術指標 | 工作日 16:00 | `RuleAlpha/stock/{id}/technical` |

## 🚀 快速部署

### 前置需求
- Docker & Docker Compose
- 運行中的Rule Alpha Manager (MariaDB, MQTT)
- 已配置cocaen網路

### 部署步驟

1. **克隆專案**
```bash
git clone https://github.com/yourusername/rule-alpha-data-services.git
cd rule-alpha-data-services
```

2. **配置環境變數**
```bash
cp .env.example .env
# 編輯 .env 檔案設定數據庫和MQTT連接資訊
```

3. **生產環境部署**
```bash
# 部署所有數據服務
docker-compose --profile production up -d

# 或僅部署數據服務群組
docker-compose --profile data-services up -d
```

4. **開發環境部署**
```bash
docker-compose --profile development up -d
```

## 📊 服務功能說明

### rule_alpha_stock_close (收盤價服務)
- **功能**: 收集每日股票收盤價數據
- **執行時間**: 工作日 15:00
- **數據來源**: 台灣證券交易所
- **輸出**: 收盤價、成交量、漲跌幅

### rule_alpha_stock_realtime (即時數據服務)
- **功能**: 收集盤中即時股價數據
- **執行頻率**: 盤中每5分鐘
- **數據來源**: 即時報價API
- **輸出**: 當前價格、買賣價、成交量

### rule_alpha_stock_volume (成交量分析服務)
- **功能**: 分析成交量異常和大單交易
- **執行頻率**: 盤中每10分鐘
- **分析指標**: 
  - 大單交易偵測 (預設 > 1000張)
  - 成交量激增 (預設 > 3倍平均值)
  - 成交量趨勢分析

### rule_alpha_stock_technical (技術分析服務)
- **功能**: 計算技術指標和交易信號
- **執行時間**: 工作日 16:00
- **技術指標**:
  - 移動平均線 (MA5, MA10, MA20, MA60)
  - 相對強弱指標 (RSI)
  - MACD指標
  - 布林帶 (Bollinger Bands)
  - KD指標

## 🔧 配置說明

### 環境變數

#### 數據庫配置
```bash
RULE_ALPHA_DB_NAME=rule_alpha          # 數據庫名稱
RULE_ALPHA_DB_USER=rule_alpha_user     # 數據庫用戶
RULE_ALPHA_DB_PASSWORD=your_password   # 數據庫密碼
```

#### MQTT配置
```bash
MQTT_USERNAME=cocaen                   # MQTT用戶名
MQTT_PASSWORD=your_mqtt_password       # MQTT密碼
```

#### 執行時間配置
```bash
CLOSE_SCHEDULE_TIME=15:00              # 收盤價執行時間
REALTIME_INTERVAL=5                    # 即時數據間隔(分鐘)
VOLUME_ANALYSIS_INTERVAL=10            # 成交量分析間隔(分鐘)
TECHNICAL_SCHEDULE_TIME=16:00          # 技術分析執行時間
```

#### 市場時間配置
```bash
MARKET_START=09:00                     # 開市時間
MARKET_END=13:30                       # 收市時間
```

#### 交易監控閾值
```bash
BIG_TRADE_THRESHOLD=1000               # 大單門檻(張)
VOLUME_SPIKE_THRESHOLD=3.0             # 成交量異常倍數
STRONG_SIGNAL_THRESHOLD=80.0           # 強信號門檻
```

## 📈 數據流程

```
1. 數據收集
   ├── stock-close: 收盤價 → 數據庫
   ├── stock-realtime: 即時價格 → 數據庫
   ├── stock-volume: 成交量 → 數據庫
   └── stock-technical: 技術指標 → 數據庫

2. 數據處理
   ├── 數據清洗和驗證
   ├── 異常值檢測
   └── 數據標準化

3. 消息發布
   ├── MQTT消息發布
   ├── 實時通知
   └── 警報觸發

4. 數據存儲
   ├── 歷史數據歸檔
   ├── 索引優化
   └── 備份機制
```

## 🛠️ 管理操作

### 服務管理
```bash
# 查看服務狀態
docker-compose ps

# 查看特定服務日誌
docker-compose logs -f rule_alpha_stock_close

# 重啟服務
docker-compose restart rule_alpha_stock_realtime

# 停止所有服務
docker-compose down

# 更新服務
docker-compose pull && docker-compose up -d
```

### 單獨執行服務
```bash
# 執行收盤價數據收集
docker-compose run --rm rule_alpha_stock_close python main.py --once

# 執行技術分析
docker-compose run --rm rule_alpha_stock_technical python main.py --analyze

# 測試即時數據連接
docker-compose run --rm rule_alpha_stock_realtime python main.py --test
```

### 數據維護
```bash
# 清理過期日誌
find ./*/logs -name "*.log" -mtime +30 -delete

# 備份數據庫
docker-compose exec -T mariadb mysqldump rule_alpha > backup_$(date +%Y%m%d).sql

# 檢查數據完整性
docker-compose run --rm rule_alpha_stock_close python main.py --verify
```

## 📊 監控和警報

### 性能監控
- 數據收集成功率
- API響應時間
- 數據庫連接狀態
- MQTT消息發布狀態

### 警報配置
```bash
# 數據收集失敗警報
ALERT_ON_DATA_FAILURE=true

# API超時警報 (秒)
API_TIMEOUT_ALERT=30

# 數據延遲警報 (分鐘)
DATA_DELAY_ALERT=10
```

### 日誌查看
```bash
# 查看今日收盤價日誌
docker-compose logs rule_alpha_stock_close | grep $(date +%Y-%m-%d)

# 查看錯誤日誌
docker-compose logs | grep ERROR

# 查看成交量異常警報
docker-compose logs rule_alpha_stock_volume | grep "VOLUME_SPIKE"
```

## 🔐 安全配置

### API安全
- 限制API請求頻率
- 使用HTTPS連接
- API密鑰管理

### 數據安全
- 數據庫連接加密
- 敏感資訊加密存儲
- 定期安全掃描

### 網路安全
- Docker網路隔離
- 防火牆配置
- 存取日誌記錄

## 🐛 故障排除

### 常見問題

1. **數據收集失敗**
   ```bash
   # 檢查API連接
   docker-compose run --rm rule_alpha_stock_close python main.py --test-api
   
   # 檢查網路連接
   docker-compose run --rm rule_alpha_stock_close ping tw.stock.yahoo.com
   ```

2. **MQTT連接問題**
   ```bash
   # 檢查MQTT連接
   docker-compose run --rm rule_alpha_stock_close python main.py --test-mqtt
   
   # 查看MQTT日誌
   docker-compose logs mosquitto
   ```

3. **數據庫連接失敗**
   ```bash
   # 檢查數據庫連接
   docker-compose run --rm rule_alpha_stock_close python main.py --test-db
   
   # 查看數據庫日誌
   docker-compose logs mariadb
   ```

### 調試模式
```bash
# 啟用詳細日誌
export LOG_LEVEL=DEBUG

# 單獨運行服務進行調試
docker-compose run --rm -e LOG_LEVEL=DEBUG rule_alpha_stock_close python main.py
```

## 📞 技術支持

- **維護者**: arch.twn1@gmail.com
- **問題報告**: GitHub Issues
- **文檔**: [Rule Alpha Data Services Docs](https://docs.rule-alpha.com/data-services)

## 📄 授權協議

MIT License