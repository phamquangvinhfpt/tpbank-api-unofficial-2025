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
- ✅ **Gửi webhook** khi có giao dịch mới
- ✅ **API REST** để truy vấn giao dịch thủ công
- ✅ **Swagger UI** cho documentation
- ✅ **Docker** ready - chạy với 1 lệnh
- ✅ **Retry logic** thông minh khi gặp lỗi
- ✅ **Logs** chi tiết để debug

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
wget https://raw.githubusercontent.com/phamquangvinhfpt/tpbank-api-unofficial/main/docker-compose.example.yml
# Hoặc
curl -O https://raw.githubusercontent.com/phamquangvinhfpt/tpbank-api-unofficial/main/docker-compose.example.yml
```

2. **Sửa file `docker-compose.example.yml`:**

```yaml
environment:
  # Thông tin đăng nhập TPBank
  TPBANK_USERNAME: "your_phone_number_bank"              # Số điện thoại
  TPBANK_PASSWORD: "your_password_here"      # Mật khẩu  
  TPBANK_DEVICE_ID: "your_device_id_here"    # Device ID từ bước 1
  TPBANK_ACCOUNT_NUMBER: "your_account_number_bank_here"       # Số tài khoản
  
  # Webhook nhận thông báo giao dịch
  WEBHOOK_URL: "https://your-webhook-url.com/transactions"
  WEBHOOK_HEADER_X_API_KEY: "your-secret-key"
```

3. **Khởi chạy:**

```bash
docker-compose -f docker-compose.example.yml up -d
```

🎉 **Xong!** API đang chạy tại `http://localhost:8089`

## 📡 API Endpoints

### Health Check
```bash
GET /health
```

### Swagger Documentation
```bash
GET /swagger/index.html
```

### Lấy giao dịch theo khoảng thời gian
```bash
POST /api/v1/transactions
Content-Type: application/json

{
  "from_date": "20250101",
  "to_date": "20250109"
}
```

### Lấy giao dịch N ngày gần nhất
```bash
POST /api/v1/transactions/last-days
Content-Type: application/json

{
  "days": 7
}
```

### Kích hoạt đồng bộ thủ công
```bash
POST /api/v1/cronjob/trigger
```

### Lấy trạng thái cronjob
```bash
GET /api/v1/cronjob/status
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
        "accountNo": "10000453128",
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
  "timestamp": "2025-01-09T14:30:00Z",
  "summary": {
    "total_transactions": 5,
    "total_credit": 10000000,
    "total_debit": 2000000,
    "net_amount": 8000000
  },
  "transactions": [
    {
      "id": "13712911687",
      "amount": "2000000",
      "description": "NGUYEN VAN A chuyen tien",
      "transactionDate": "20250109",
      "transactionType": "IN"
    }
  ]
}
```

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

### Lỗi "Login failed"  
- Kiểm tra username/password
- Đảm bảo đã xác minh khuôn mặt trên trình duyệt trước đó

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

### Vi phạm tiềm ẩn

Việc sử dụng API không chính thức có thể:
- Vi phạm **Điều khoản sử dụng** của TPBank
- Vi phạm **Quy định pháp luật** về bảo mật ngân hàng
- Dẫn đến **khóa tài khoản** nếu ngân hàng phát hiện

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
