# TPBank API - Tự động đồng bộ giao dịch và gửi webhook

![TPBank API](https://socialify.git.ci/phamquangvinhfpt/tpbank-api-unofficial-2025/image?description=1&descriptionEditable=API%20t%E1%BB%B1%20%C4%91%E1%BB%99ng%20%C4%91%E1%BB%93ng%20b%E1%BB%99%20giao%20d%E1%BB%8Bch%20TPBank%20v%C3%A0%20g%E1%BB%ADi%20webhook&font=Inter&forks=1&issues=1&language=1&name=1&owner=1&pattern=Plus&pulls=1&stargazers=1&theme=Auto)

## 📋 Giới thiệu

API này cung cấp dịch vụ **tự động đồng bộ giao dịch từ TPBank** và gửi thông báo đến webhook endpoint của bạn. Hoàn hảo cho các ứng dụng cần theo dõi giao dịch ngân hàng theo thời gian thực như:

- 💳 Cổng thanh toán tự động
- 🏪 Hệ thống quản lý cửa hàng  
- 📊 Theo dõi doanh thu realtime
- 🔔 Thông báo giao dịch cho nhân viên

## ✨ Tính năng

- ✅ **Tự động đồng bộ** giao dịch theo lịch (cron job)
- ✅ **Gửi webhook** với filter tùy chọn (nhận tiền, chuyển đi, hoặc cả 2)
- ✅ **Transaction Filters** - Lọc giao dịch theo loại category
- ✅ **Redis Transaction Tracking** - Phát hiện giao dịch mới, tránh duplicate
- ✅ **Telegram Alert System** - Nhận thông báo giao dịch mới qua Telegram Bot
- ✅ **Revenue Statistics API** - Thống kê doanh thu theo tháng/năm với biểu đồ
- ✅ **Telegram Bot Commands** - Lệnh `/revenue` để xem thống kê trực tiếp
- ✅ **API REST** để truy vấn giao dịch thủ công
- ✅ **Swagger UI** cho documentation
- ✅ **Docker** ready - chạy với 1 lệnh (~15MB)
- ✅ **Retry logic** thông minh khi gặp lỗi
- ✅ **Logs** chi tiết để debug
- 🛡️ **Account Protection** - Tự động stop sau 2 lần login thất bại để tránh bị khóa tài khoản

## 🚀 Cài đặt nhanh với Docker

### Bước 1: Lấy Device ID từ TPBank

1. Mở trình duyệt **đã từng đăng nhập** và **xác minh khuôn mặt** TPBank
2. Truy cập https://ebank.tpb.vn/retail/vX/ và đăng nhập
3. Nhấn **F12** → Tab **Console**
4. Paste lệnh sau và Enter:

```javascript
localStorage.deviceId
```

5. Copy giá trị trả về (dạng: `7fl36P1VEz1s7qxzfr8QyKKVToNBZma3PW0mp2rRVkZSr`)

![Device ID](./imgs/deviceId.png)

### Bước 2: Cấu hình và chạy

1. **Download docker-compose file:**

```bash
wget https://raw.githubusercontent.com/phamquangvinhfpt/tpbank-api-unofficial-2025/main/docker-compose.example.yml
# Hoặc
curl -O https://raw.githubusercontent.com/phamquangvinhfpt/tpbank-api-unofficial-2025/main/docker-compose.example.yml
```

2. **Sửa file `docker-compose.example.yml`:**

```yaml
environment:
  # Thông tin đăng nhập TPBank
  TPBANK_USERNAME: "your_phone_number"       # Số điện thoại
  TPBANK_PASSWORD: "your_password"           # Mật khẩu  
  TPBANK_DEVICE_ID: "your_device_id"         # Device ID từ bước 1
  TPBANK_ACCOUNT_NUMBER: "your_account_number" # Số tài khoản
  
  # Webhook nhận thông báo giao dịch
  WEBHOOK_URL: "https://your-webhook-url.com/transactions"
  WEBHOOK_FILTER_TYPE: "all"  # all, money_in, transfer_out, both, custom
  WEBHOOK_HEADER_X_API_KEY: "your-secret-key"
  
  # Redis (Optional - Transaction Tracking)
  REDIS_ENABLED: "true"
  REDIS_URL: "redis:6379"
  REDIS_PASSWORD: "your_redis_password"
  
  # Telegram (Optional - Alerts & Bot Commands)
  TELEGRAM_ENABLED: "true"
  TELEGRAM_BOT_TOKEN: "your_bot_token"
  TELEGRAM_CHAT_ID: "your_chat_id"
```

3. **Khởi chạy:**

```bash
docker-compose -f docker-compose.example.yml up -d
```

🎉 **Xong!** API đang chạy tại `http://localhost:8089`

## 📡 API Endpoints

### Health Check
```bash
GET /api/v1/health
```

### Swagger Documentation
```bash
GET /swagger/index.html
```

### Transaction Endpoints

#### 1. Lấy giao dịch có phân trang (Khuyến nghị) ⭐

**API mới - Hiệu suất tốt nhất, có thể filter theo categories**

```bash
POST /api/v1/transactions/paginated
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109",
  "page": 1,
  "page_size": 50,
  "categories": ["transaction_CategoryMoneyIn", "transaction_CategoryTransfer"]  // optional
}
```

**Response (không filter):**

```json
{
  "status": "success",
  "data": {
    "transactions": [...],
    "pagination": {
      "page": 1,
      "page_size": 50,
      "count": 50,
      "has_more": true
    }
  }
}
```

**Response (có filter categories):**

```json
{
  "status": "success",
  "data": {
    "transactions": [...],
    "pagination": {
      "page": 1,
      "page_size": 50,
      "count": 30,
      "has_more": true
    },
    "total_before_filter": 50,
    "filter_type": "custom"
  }
}
```

**Lưu ý:**
- `page`: Trang hiện tại (bắt đầu từ 1)
- `page_size`: Số giao dịch mỗi trang (1-400)
- `has_more`: `true` nếu còn trang tiếp theo
- `categories`: Tùy chọn - filter theo danh sách categories
- `total_before_filter`: Số giao dịch trước khi filter (chỉ có khi filter)
- `filter_type`: Loại filter đang dùng (chỉ có khi filter)

#### 2. Lấy tất cả giao dịch (Không phân trang)

⚠️ **Cảnh báo:** API này lấy tất cả giao dịch, có thể chậm nếu có nhiều giao dịch. Khuyến nghị dùng API phân trang.

```bash
POST /api/v1/transactions
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109"
}
```

#### 3. Lấy giao dịch N ngày gần nhất
```bash
POST /api/v1/transactions/last-days
Content-Type: application/json

{
  "days": 7
}
```

#### 4. Lấy giao dịch nhận tiền (Money In)

**Filter chỉ lấy giao dịch nhận tiền vào tài khoản**

```bash
POST /api/v1/transactions/money-in
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "count": 3,
    "total_before_filter": 10,
    "filter_type": "money_in",
    "transactions": [...]
  }
}
```

#### 5. Lấy giao dịch chuyển tiền đi (Transfer Out)

**Filter chỉ lấy giao dịch chuyển tiền đi (bao gồm cả rút ATM/QR)**

```bash
POST /api/v1/transactions/transfer-out
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "count": 2,
    "total_before_filter": 10,
    "filter_type": "transfer_out",
    "transactions": [...]
  }
}
```

#### 6. Lấy giao dịch theo category tùy chọn

**Filter theo danh sách categories cụ thể**

```bash
POST /api/v1/transactions/by-category
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109",
  "categories": [
    "transaction_CategoryMoneyIn",
    "transaction_CategoryTransfer"
  ]
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "count": 5,
    "total_before_filter": 10,
    "filter_type": "custom",
    "transactions": [...]
  }
}
```

**Các categories có sẵn:**
- `transaction_CategoryMoneyIn`: Nhận tiền
- `transaction_CategoryTransfer`: Chuyển khoản
- `transaction_CategoryCashOut`: Rút tiền ATM
- `transaction_CategoryPayBill`: Thanh toán hóa đơn
- `transaction_CategoryTopUp`: Nạp tiền điện thoại
- `transaction_CategoryWithdrawal`: Rút tiền (QR/POS)

### Cronjob Endpoints

#### Kích hoạt đồng bộ thủ công
```bash
POST /api/v1/cronjob/trigger
```

#### Lấy trạng thái cronjob
```bash
GET /api/v1/cronjob/status
```

### Revenue Statistics (Thống kê doanh thu) 📊

#### API thống kê doanh thu

**Thống kê theo tháng/năm với biểu đồ:**

```bash
POST /api/v1/statistics/revenue
Content-Type: application/json

{
  "from_date": "20240101",
  "to_date": "20241231",
  "type": "monthly",
  "include_chart": true
}
```

**Parameters:**
- `from_date`: Ngày bắt đầu (YYYYMMDD)
- `to_date`: Ngày kết thúc (YYYYMMDD)
- `type`: `"monthly"` (theo tháng) hoặc `"yearly"` (theo năm)
- `include_chart`: `true` để tạo biểu đồ PNG (base64)

**Response:**

```json
{
  "status": "success",
  "data": {
    "type": "monthly",
    "total_income": 600000000,
    "total_expense": 240000000,
    "net_revenue": 360000000,
    "total_count": 540,
    "data": [
      {
        "period": "2024-01",
        "total_income": 50000000,
        "total_expense": 20000000,
        "net_revenue": 30000000,
        "count": 45
      }
    ],
    "chart_url": "data:image/png;base64,..."
  }
}
```

#### Telegram Bot Command

**Xem thống kê doanh thu trực tiếp trong Telegram:**

```
/revenue          # Thống kê theo tháng (6 tháng gần nhất)
/revenue month    # Thống kê theo tháng
/revenue year     # Thống kê theo năm (12 tháng gần nhất)
```

**Định dạng message:**
- 📊 **Biểu đồ PNG** hiển thị trên (line chart 3 đường với labels rõ ràng)
- 📝 **Text caption** hiển thị dưới (tổng quan, chi tiết 5 kỳ gần nhất)

**Setup Telegram Bot:**

1. Tạo bot với [@BotFather](https://t.me/botfather)
2. Lấy Bot Token
3. Lấy Chat ID (gửi message cho bot rồi gọi `getUpdates`)
4. Set webhook:

```bash
curl -X POST "https://api.telegram.org/bot<BOT_TOKEN>/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://your-domain.com/api/v1/telegram/webhook"}'
```

**Tối ưu hóa:**
- ✅ **Token reuse**: Không login lại nếu token còn hợp lệ
- ✅ **Date format**: Hỗ trợ cả YYYY-MM-DD và DD/MM/YYYY
- ✅ **Chart labels**: Format ngắn gọn (T5/25 cho tháng 5/2025)
- ✅ **Message layout**: Chart với caption (dễ xem ngay ảnh)
- ✅ **Cloudflare bypass**: Whitelist Telegram IPs để tránh 403 Forbidden

**Troubleshooting Webhook:**

Nếu bot không phản hồi, kiểm tra:

```bash
# Kiểm tra webhook status
curl "https://api.telegram.org/bot<BOT_TOKEN>/getWebhookInfo"

# Nếu thấy lỗi 403 Forbidden từ Cloudflare:
# 1. Vào Cloudflare Dashboard → Security → WAF → Tools
# 2. Tạo IP Access Rule:
#    - IP: 149.154.160.0/20 → Action: Allow
#    - IP: 91.108.4.0/22 → Action: Allow
# 3. Hoặc tắt Bot Fight Mode cho path /api/v1/telegram/webhook

# Xóa webhook và set lại để clear pending updates
curl "https://api.telegram.org/bot<BOT_TOKEN>/deleteWebhook?drop_pending_updates=true"
curl -X POST "https://api.telegram.org/bot<BOT_TOKEN>/setWebhook" \
  -d '{"url": "https://your-domain.com/api/v1/telegram/webhook"}'
```

## 📊 Response mẫu

```json
{
  "status": "success",
  "data": {
    "count": 3,
    "transactions": [
      {
        "id": "13712911687",
        "amount": "2000000",
        "description": "NGUYEN VAN A chuyen tien",
        "transactionDate": "20250109",
        "transactionType": "IN",
        "balance": "21000000",
        "referenceNumber": "FT24068710833711",
        "channel": "TRANSFER",
        "accountNo": "your_account_number",
        "toAccount": "",
        "runningBalance": "21000000",
        "transactionStatus": "SUCCESS"
      }
    ]
  }
}
```

## 🔔 Webhook Format

Khi có giao dịch mới, API sẽ POST đến webhook URL của bạn:

```json
{
  "timestamp": "2025-01-09T14:30:00+07:00",
  "account_no": "your_account_number",
  "transactions": [
    {
      "id": "21947404159",
      "category": "transaction_CategoryMoneyIn",
      "amount": "35000586",
      "description": "XXXX XXXX XXXX CHUYEN KHOAN",
      "creditDebitIndicator": "CRDT",
      "transactionDate": "2025-09-03",
      "currency": "VND",
      "runningBalance": "35000586"
    }
  ],
  "summary": {
    "total_count": 10,
    "total_debit": 2000000,
    "total_credit": 5000000,
    "fetched_from": "20250901",
    "fetched_to": "20250910"
  }
}
```

### Webhook Filter Types

Bạn có thể cấu hình webhook để chỉ gửi các loại giao dịch nhất định:

**1. Tất cả giao dịch (Mặc định)**
```yaml
WEBHOOK_FILTER_TYPE: "all"
```

**2. Chỉ giao dịch nhận tiền**
```yaml
WEBHOOK_FILTER_TYPE: "money_in"
```
Ví dụ: Nhận chuyển khoản, nhận lương, hoàn tiền

**3. Chỉ giao dịch chuyển tiền đi**
```yaml
WEBHOOK_FILTER_TYPE: "transfer_out"
```
Ví dụ: Chuyển khoản, thanh toán hóa đơn

**4. Cả nhận và chuyển**
```yaml
WEBHOOK_FILTER_TYPE: "both"
```
Loại bỏ: Rút ATM, phí dịch vụ

**5. Tùy chỉnh categories**
```yaml
WEBHOOK_FILTER_TYPE: "custom"
WEBHOOK_FILTER_CATEGORY: "transaction_CategoryMoneyIn,transaction_CategoryTransfer,transaction_CategoryCashMoney"
```

**Danh sách Categories:**
- `transaction_CategoryMoneyIn` - Nhận tiền
- `transaction_CategoryTransfer` - Chuyển tiền
- `transaction_CategoryCashMoney` - Rút ATM/QR
- `transaction_CategoryOther` - Phí dịch vụ

## ⚙️ Cấu hình nâng cao

### Thay đổi lịch chạy cronjob

Mặc định: chạy **mỗi 5 phút**. Để thay đổi, sửa:

```yaml
CRONJOB_SCHEDULE: "0 */5 * * * *"  # Format: giây phút giờ ngày tháng thứ
```

Ví dụ:
- `0 * * * * *` - Mỗi phút
- `0 0 * * * *` - Mỗi giờ
- `0 0 0 * * *` - Mỗi ngày lúc 00:00
- `*/30 * * * * *` - Mỗi 30 giây

### Số ngày lấy giao dịch

```yaml
CRONJOB_FETCH_DAYS: "7"  # Lấy 7 ngày gần nhất
```

### Bật debug logs

```yaml
LOGGER_LEVEL: "debug"  # Thay vì "info"
```

## 🐛 Troubleshooting

### Lỗi "Account Number Invalid"
- Kiểm tra `TPBANK_ACCOUNT_NUMBER` phải là **số tài khoản**, không phải số điện thoại

### Lỗi "Token expired"
- Device ID đã hết hạn, lấy lại Device ID mới từ trình duyệt

### Lỗi "Login failed" / 401 Unauthorized
- Kiểm tra username/password có đúng không
- Đảm bảo đã xác minh khuôn mặt trên trình duyệt trước đó
- ⚠️ **CẢNH BÁO QUAN TRỌNG**: Ứng dụng sẽ **TỰ ĐỘNG DỪNG** sau **2 lần login thất bại** để bảo vệ tài khoản của bạn khỏi bị khóa (TPBank khóa tài khoản sau 5 lần đăng nhập sai)

### Webhook không nhận được data
- Kiểm tra `WEBHOOK_URL` có đúng không
- Kiểm tra webhook server có chạy không
- Xem logs: `docker-compose logs -f tpbank-api`

## 📂 Logs

Logs được lưu trong folder `./logs`:

```bash
# Xem logs realtime
docker-compose logs -f

# Xem logs trong container
docker exec -it tpbank-api-production cat /logs/app.log
```

## 🛑 Dừng service

```bash
docker-compose -f docker-compose.example.yml down
```

## 🔄 Update version mới

```bash
docker-compose -f docker-compose.example.yml pull
docker-compose -f docker-compose.example.yml up -d
```

## 📦 Docker Images

- **Docker Hub:** `phamquangvinh/tpbank-api:latest`
- **Size:** ~25MB (scratch-based)
- **Architecture:** amd64

## 🛠️ Tech Stack

- **Go** 1.25
- **Gin** - Web framework
- **Cron** - Job scheduler
- **Zap** - Structured logging
- **Viper** - Configuration management
- **Swagger** - API documentation
- **Docker** - Containerization

## 📄 Bản quyền

- ✅ **Cho phép** sử dụng **thương mại**: tạo cổng thanh toán, thông báo giao dịch,...
- ❌ **Không cho phép**: mở dịch vụ tương tự Casso.vn, VietQR.io để kinh doanh

## ⚠️ Miễn trừ trách nhiệm

**⚠️ QUAN TRỌNG - VUI LÒNG ĐỌC KỸ:**

### Sử Dụng API Không Chính Thức

Dịch vụ này sử dụng **API không chính thức** của TPBank mà **không có sự đồng ý** từ ngân hàng. Do đó:

- ❌ **Không chịu trách nhiệm** cho bất kỳ vấn đề pháp lý nào phát sinh
- ❌ **Không đảm bảo** tính chính xác, độ tin cậy của dữ liệu
- ❌ **Không đảm bảo** dịch vụ hoạt động liên tục (API có thể thay đổi bất cứ lúc nào)
- ⚠️ **Rủi ro bảo mật**: Device ID có thể bị lộ nếu không bảo mật tốt

### Khuyến cáo

- 🔒 **BẢO MẬT** thông tin đăng nhập và Device ID
- 💼 **Tham khảo chuyên gia pháp lý** trước khi sử dụng cho mục đích kinh doanh
- 🚫 **Không sử dụng** cho các giao dịch quan trọng, số tiền lớn
- 📞 **Liên hệ TPBank** để sử dụng API chính thức nếu có nhu cầu thương mại
- 🛡️ **Account Protection**: Ứng dụng có cơ chế bảo vệ tài khoản tự động stop sau 2 lần login thất bại

### Vi phạm tiềm ẩn

Việc sử dụng API không chính thức có thể:
- Vi phạm **Điều khoản sử dụng** của TPBank
- Vi phạm **Quy định pháp luật** về bảo mật ngân hàng
- Dẫn đến **khóa tài khoản** nếu ngân hàng phát hiện hoặc đăng nhập sai quá 5 lần

### Cơ chế bảo vệ tài khoản

Để tránh tình trạng bị khóa tài khoản do đăng nhập sai nhiều lần, ứng dụng đã tích hợp:

- 🛡️ **Tự động theo dõi** số lần login thất bại
- ⚠️ **Cảnh báo** khi đạt ngưỡng nguy hiểm
- 🛑 **Dừng ứng dụng** tự động sau **2 lần thất bại liên tiếp**
- 📊 **Logs chi tiết** để debug và kiểm tra

**Lưu ý:** TPBank sẽ khóa tài khoản sau **5 lần đăng nhập sai**. Ứng dụng stop ở lần thứ 2 để đảm bảo an toàn.

**➡️ SỬ DỤNG DỊCH VỤ NÀY ĐỒNG NGHĨA BẠN CHẤP NHẬN MỌI RỦI RO**

## 🤝 Đóng góp

Mọi đóng góp đều được hoan nghênh! Vui lòng:

1. Fork repo
2. Tạo branch mới (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Mở Pull Request

## 📧 Liên hệ

- **GitHub Issues:** [Report bugs](https://github.com/phamquangvinhfpt/tpbank-api-unofficial-2025/issues)
- **Docker Hub:** [phamquangvinh/tpbank-api](https://hub.docker.com/r/phamquangvinh/tpbank-api)

## 📝 License

MIT License - Xem [LICENSE](LICENSE) để biết thêm chi tiết.

---

**⭐ Nếu project hữu ích, hãy cho 1 star nhé! ⭐**

**Made with ❤️ in Vietnam 🇻🇳**
