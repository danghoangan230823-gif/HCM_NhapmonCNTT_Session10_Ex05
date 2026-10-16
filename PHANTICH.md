# Smart-Farm – Phân tích và kiểm thử

## 1. Mục tiêu
Mô phỏng hệ thống tưới cây tự động theo mô hình Input – Process – Output (IPO) và kiểm thử khi log tăng quá nhiều.

> Đây là mô phỏng Python, chưa kết nối ESP32, DHT22, cảm biến độ ẩm đất, rơ-le hoặc máy bơm thật.

## 2. Input / Process / Output

| Loại | Thành phần | Dữ liệu / hành động |
|---|---|---|
| Input | DHT22 | Nhiệt độ không khí (°C) |
| Input | Cảm biến độ ẩm đất | Độ ẩm đất (%) |
| Input | Nút bấm / ứng dụng | Lệnh tưới tay |
| Process | ESP32 (mô phỏng) | Đọc số đo, so sánh với ngưỡng |
| Output | Rơ-le (mô phỏng) | Đóng/ngắt tín hiệu điều khiển |
| Output | Máy bơm (mô phỏng) | Bật/tắt tưới |
| Output | Wi-Fi/điện thoại (mô phỏng) | Gửi trạng thái và số đo |

## 3. Sơ đồ phối hợp phần cứng

```text
[DHT22: nhiệt độ] ----\
                      \
[Cảm biến độ ẩm đất] ---> [ESP32: CPU + RAM + Flash + Wi-Fi]
                      /               |
[Nút bấm / ứng dụng] -/                | quyết định tưới
                                       v
                                  [Rơ-le]
                                       |
                                       v
                                  [Máy bơm]
                                       |
                                       v
                                    [Cây trồng]

[ESP32] ---- Wi-Fi ----> [Điện thoại: trạng thái / biểu đồ]
```

## 4. Vì sao ESP32 là SoC?
SoC (System on Chip) là hệ thống tích hợp nhiều khối chức năng trên một chip. ESP32 tích hợp bộ xử lý và Wi-Fi/Bluetooth (tùy phiên bản); RAM và bộ nhớ flash có thể được tích hợp hoặc bố trí theo biến thể bo mạch. Vì vậy, nên tránh nói rằng mọi phiên bản ESP32 đều chứa CPU, RAM, Flash và Wi-Fi hoàn toàn trên cùng một khuôn chip.

## 5. Kiểm thử: log tăng quá nhiều
### Rủi ro
- Bộ nhớ dùng cho log tăng cao có thể làm chương trình thiếu RAM, chậm hoặc treo.
- Lệnh tưới có thể bị trễ nếu chương trình xử lý không tốt.
- Nếu điều khiển bơm không có giới hạn an toàn, bơm có thể chạy quá lâu.
- Rút điện đột ngột không phải là cách xử lý lỗi đầu tiên nên chọn.

### Cách xử lý an toàn
1. Nếu bơm đang chạy bất thường, dùng nút dừng/biện pháp ngắt bơm an toàn.
2. Thực hiện khởi động lại theo quy trình của thiết bị; không tùy tiện rút điện khi đang ghi dữ liệu.
3. Giới hạn log trong bộ nhớ: `LOG_MAX_DONG = 100`.
4. Lưu log lâu dài vào thẻ nhớ hoặc máy chủ nếu cần; không giữ tất cả log trong RAM.
5. Kiểm tra lại cảm biến, rơ-le và bơm sau khi khôi phục.

### Phòng ngừa
- Giới hạn số log trong RAM.
- Ưu tiên tác vụ điều khiển an toàn hơn việc ghi log.
- Có giới hạn thời gian chạy bơm và cơ chế tắt bơm khi lỗi.
- Không coi việc xóa log là giải pháp duy nhất cho mọi lỗi RAM.

## 6. Cách chạy trên MacBook bằng VS Code
1. Mở thư mục này trong VS Code.
2. Mở Terminal: **Terminal → New Terminal**.
3. Kiểm tra Python: `python3 --version`
4. Chạy: `python3 main.py`

## 7. Kết quả mong đợi
- Hiển thị dữ liệu Input.
- So sánh độ ẩm đất với ngưỡng 35%.
- Hiển thị trạng thái bơm mô phỏng.
- Tạo 150 log nhưng chỉ giữ tối đa 100 log.
- Báo `ĐẠT` nếu số log không vượt quá giới hạn.
