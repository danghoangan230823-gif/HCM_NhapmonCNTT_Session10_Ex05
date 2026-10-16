# 🚀 Smart-Farm: Hệ thống tưới tự động cho trang trại Lan

> Tài liệu hướng dẫn sử dụng và bàn giao cho người nông dân tại **trang trại Lan**.

## 📖 1. Mô tả

**Smart-Farm** là hệ thống IoT giúp trang trại đo nhiệt độ và độ ẩm đất, sau đó hỗ trợ quyết định tưới cây.

- **SoC ESP32:** tích hợp bộ xử lý và kết nối không dây; bộ nhớ cụ thể tùy phiên bản chip/bo mạch.
- **Mô hình IPO:** Input (đầu vào) → Process (xử lý) → Output (đầu ra).
- **Wi-Fi:** có thể gửi dữ liệu đến điện thoại khi firmware và ứng dụng đã được triển khai.

> **Phạm vi bản nộp:** dự án hiện có mô phỏng Python chạy trên MacBook. Nó chưa kết nối cảm biến, ESP32, rơ-le hoặc máy bơm thật.

## 🧩 2. Sơ đồ phối hợp phần cứng

```text
[DHT22: nhiệt độ] ------\
                        \
[Cảm biến độ ẩm đất] -----> [ESP32: CPU + bộ nhớ + Wi-Fi]
                        /                 |
[Nút bấm / ứng dụng] ---/                  | quyết định tưới
                                            v
                                         [Rơ-le]
                                            |
                                            v
                                        [Máy bơm]
                                            |
                                            v
                                         [Cây trồng]

[ESP32] ---------------- Wi-Fi ----------> [Điện thoại]
                                         trạng thái / số liệu
```

*Hình 1. Sơ đồ phối hợp phần cứng của hệ thống.*

## ⚙️ 3. Hướng dẫn cài đặt

### Chạy bản mô phỏng trên MacBook

1. Mở thư mục dự án trong VS Code.
2. Chọn **Terminal → New Terminal**.
3. Kiểm tra Python: `python3 --version`
4. Chạy chương trình:

```bash
python3 src/main.py
```

### Nếu triển khai trên ESP32 thật

Cần firmware riêng cho ESP32, thiết bị phần cứng phù hợp và cấu hình chân GPIO đúng. Ví dụ cấu hình:

```cpp
// config.h - ví dụ minh họa, chưa phải firmware hoàn chỉnh
#define WIFI_SSID       "TenWifiTrangTrai"
#define WIFI_PASSWORD   "ThayBangMatKhauCuaBan"
#define NGUONG_DO_AM    60
#define NGUONG_NHIET    32
#define LOG_MAX_DONG    100
```

Nếu dự án firmware đã được cấu hình bằng PlatformIO, lệnh nạp thường dùng là:

```bash
pio run --target upload
```

Lệnh này dùng cho dự án firmware ESP32, không nạp file Python vào ESP32. Không đưa mật khẩu Wi-Fi thật lên kho mã nguồn công khai.

## ✅ 4. Tính năng

- Đo nhiệt độ không khí bằng DHT22 khi có phần cứng.
- Đọc cảm biến độ ẩm đất.
- So sánh số đo với ngưỡng để quyết định tưới.
- Mô phỏng bật/tắt bơm.
- Mô phỏng trạng thái gửi dữ liệu qua Wi-Fi.
- Giới hạn bộ đệm log tối đa 100 dòng.
- Hẹn giờ tưới và cảnh báo lỗi cảm biến: **chưa triển khai trong bản mẫu**.

## 📊 5. Bảng thông số

| Thành phần | Thiết bị | Thông số chính | Vai trò (IPO) |
|---|---|---|---|
| Bộ xử lý | SoC ESP32 | Bộ xử lý và kết nối không dây; bộ nhớ tùy biến thể | Process |
| Cảm biến nhiệt độ | DHT22 | Dải nhiệt độ thường được công bố khoảng −40 đến 80 °C | Input |
| Cảm biến độ ẩm đất | Soil Moisture | Cần hiệu chuẩn theo loại cảm biến và đất thực tế | Input |
| Công tắc điện | Mạch rơ-le phù hợp | Đóng/ngắt mạch tải theo thiết kế | Output |
| Máy bơm | Bơm mini 12 V (ví dụ) | Dùng nguồn đúng định mức | Output |
| Kết nối | Wi-Fi 2.4 GHz trên phiên bản ESP32 phù hợp | Gửi dữ liệu khi firmware được triển khai | Output |

## 🧠 6. Cách CPU xử lý lệnh tưới

CPU thực hiện chu kỳ **Fetch – Decode – Execute**:

1. **Fetch (Nạp lệnh):** CPU lấy lệnh máy tiếp theo từ bộ nhớ chương trình.
2. **Decode (Giải mã):** CPU giải mã lệnh để xác định thao tác cần thực hiện.
3. **Execute (Thực thi):** CPU thực hiện thao tác, chẳng hạn đọc cảm biến hoặc cập nhật tín hiệu điều khiển rơ-le theo logic chương trình.

`TUOI_VUON_LAN` chỉ là tên lệnh ở mức ứng dụng trong ví dụ, không phải lệnh máy nguyên thủy của CPU. Trước khi bật bơm, firmware cần kiểm tra độ ẩm và các điều kiện an toàn.

## 🔺 7. Từ dữ liệu đến hành động – mô hình DIKW

Ví dụ cảm biến đọc được **35 °C**:

```text
                  /\
                 / W  \        WISDOM (Hành động)
                /------\       Lập lịch tưới phù hợp, giới hạn thời gian
               /   K    \      KNOWLEDGE (Tri thức)
              /----------\     Nóng và đất khô có thể làm cây cần thêm nước
             /     I      \    INFORMATION (Thông tin)
            /--------------\   35 °C cao hơn ngưỡng tham khảo 32 °C
           /       D        \  DATA (Dữ liệu)
          /------------------\ Cảm biến ghi nhận: 35
```

| Tầng | Tên | Ví dụ tại trang trại Lan |
|---|---|---|
| **D** | Data | Giá trị thô `35` |
| **I** | Information | `35 °C` cao hơn ngưỡng `32 °C` |
| **K** | Knowledge | Nóng và đất khô có thể làm cây tăng nhu cầu nước |
| **W** | Wisdom | Kiểm tra độ ẩm đất, tưới có giới hạn và tránh tưới thừa |

**Lưu ý:** nhiệt độ cao không nên là điều kiện duy nhất để bật bơm. Cần xem độ ẩm đất và các điều kiện an toàn.

## ⚠️ 8. Lưu ý an toàn

- Không chạm vào dây điện hoặc đầu nối khi hệ thống đang cấp nguồn.
- Giữ ESP32 và mạch điện tránh nước, mưa trực tiếp.
- Dùng nguồn và rơ-le đúng định mức; không nối máy bơm trực tiếp vào chân GPIO.
- Nên có nút dừng thủ công, giới hạn thời gian bơm và trạng thái an toàn khi lỗi.
- Kiểm tra cảm biến theo lịch bảo trì và hướng dẫn nhà sản xuất.

### Khi bộ nhớ dành cho nhật ký bị sử dụng quá nhiều

1. Nếu bơm hoạt động bất thường, dùng công tắc/nút dừng an toàn để tắt bơm.
2. Không thao tác dây điện khi mạch đang cấp nguồn; khởi động lại theo quy trình thiết bị.
3. Giới hạn log trong bộ nhớ, ví dụ chỉ giữ **100 dòng mới nhất**.
4. Nếu cần lưu lâu dài, ghi log ra thẻ nhớ hoặc máy chủ thay vì giữ toàn bộ trong RAM.
5. Sau khi khôi phục, kiểm tra cảm biến, rơ-le và bơm trước khi chạy tự động.

Ví dụ cấu hình minh họa:

```cpp
#define LOG_MAX_DONG 100
#define LOG_GHI_THE_NHO true
```

Các macro trên chỉ có tác dụng khi firmware được viết để sử dụng chúng. Giới hạn số dòng log không tự đảm bảo toàn bộ RAM không bao giờ đầy.

> **Nguyên tắc:** ưu tiên điều khiển an toàn hơn ghi log. Khi thiếu bộ nhớ, bỏ log không thiết yếu nhưng không bỏ qua các bước kiểm tra an toàn.

## 🧪 9. Kiểm thử

Chạy `python3 src/main.py`. Chương trình sẽ:
- In dữ liệu Input mẫu.
- So sánh độ ẩm đất với ngưỡng.
- Hiển thị trạng thái bơm mô phỏng.
- Tạo 150 dòng log và xác nhận chỉ giữ tối đa 100 dòng.

Kết quả mong đợi: `ĐẠT` nếu số log còn giữ không vượt quá 100.

## 👤 10. Thông tin dự án

- **Tác giả:** *(điền tên của bạn)*
- **Lớp / Môn học:** *(điền tên lớp và môn học)*
- **Bài tập:** Session08 – Ex05
