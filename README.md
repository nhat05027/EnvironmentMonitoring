# Thiết Kế Hệ Thống Giám Sát Môi Trường Công Trình Dựa Trên ESP32

## 1. Yêu Cầu Hệ Thống (System Requirements)

### 1.1. Yêu Cầu Chức Năng (Functional Requirements)
- **Giám sát các thông số môi trường:**
  - Nhiệt độ (°C)
  - Độ ẩm (%RH)
  - Nồng độ bụi (PM2.5, PM10, sử dụng đơn vị μg/m³)
  - Tốc độ gió (m/s) và hướng gió
  - Nồng độ khí gas (CO, CO2, hoặc các khí độc hại khác, ppm)
  - Báo cháy (phát hiện khói và/hoặc nhiệt độ cao)
  - Hiện diện người (sử dụng cảm biến mmWave)
  - Phát hiện mưa (có/không hoặc cường độ mưa)
  - Áp suất khí quyển (hPa) để dự đoán thời tiết
- **Đồng bộ thời gian:**
  - Sử dụng RTC để đảm bảo dữ liệu có dấu thời gian chính xác, ngay cả khi mất kết nối mạng.
- **Kết nối cảm biến công nghiệp:**
  - Hệ thống phải có các header giao tiếp (I2C, SPI, UART, Analog, Digital) để kết nối với các cảm biến công nghiệp tiêu chuẩn.
- **Giao tiếp với datalogger:**
  - Mỗi node có cổng UART bổ sung dành riêng để giao tiếp với datalogger (bên cạnh Wi-Fi/LoRa).
- **Cấu trúc mạng lưới node:**
  - Các node cảm biến hoạt động độc lập và liên kết với nhau theo mô hình mạng lưới (mesh network) để truyền dữ liệu.
  - Dữ liệu từ các node được gửi về một datalogger trung tâm qua Wi-Fi, LoRa, hoặc UART.
- **Nguồn cấp:**
  - Hỗ trợ cấp nguồn qua pin sạc (Li-ion hoặc LiPo), năng lượng mặt trời, hoặc nguồn điện trực tiếp (5V/12V).
- **Giao tiếp dữ liệu:**
  - Truyền dữ liệu không dây (Wi-Fi hoặc LoRa) và có dây (UART) về datalogger.
  - Lưu trữ cục bộ (trên thẻ SD hoặc bộ nhớ flash) trong trường hợp mất kết nối.
- **Khả năng mở rộng:**
  - Hệ thống có thể thêm hoặc bớt node dễ dàng mà không cần cấu hình lại toàn bộ hệ thống.
- **Giao diện người dùng:**
  - Datalogger cung cấp giao diện web để theo dõi dữ liệu thời gian thực và lịch sử.
  - Ứng dụng di động (Android/iOS) để theo dõi dữ liệu và nhận cảnh báo.
- **Cảnh báo thời gian thực:**
  - Tích hợp còi hoặc đèn LED trên node để báo động tại chỗ khi phát hiện khói, hiện diện người bất thường, mưa lớn, hoặc các thông số vượt ngưỡng.
- **Bảo mật:**
  - Mã hóa dữ liệu truyền giữa node và datalogger (bao gồm qua UART) bằng AES-256.
- **Hiệu chỉnh cảm biến:**
  - Tích hợp cơ chế hiệu chỉnh định kỳ cho các cảm biến (đặc biệt là cảm biến khói và mmWave) để đảm bảo độ chính xác lâu dài.
- **Kết nối đám mây:**
  - Kết nối datalogger với dịch vụ đám mây (AWS IoT hoặc Google Cloud) để lưu trữ và phân tích dữ liệu từ xa.

### 1.2. Ràng Buộc Hệ Thống (System Constraints)
- **Môi trường hoạt động:**
  - Hoạt động trong điều kiện công trình khắc nghiệt: nhiệt độ từ -10°C đến 50°C, độ ẩm 10-95% RH, chống bụi và nước (ít nhất IP65).
- **Nguồn năng lượng:**
  - Pin phải đảm bảo hoạt động liên tục ít nhất 24 giờ khi không có nguồn sạc.
  - Hệ thống năng lượng mặt trời phải hoạt động hiệu quả trong điều kiện ánh sáng yếu (mây che phủ).
  - Tối ưu năng lượng cho cảm biến mmWave để giảm tiêu thụ pin.
- **Khoảng cách truyền dữ liệu:**
  - Khoảng cách truyền dữ liệu giữa các node tối thiểu 100m trong môi trường công trình (có vật cản) khi dùng Wi-Fi/LoRa.
  - UART chỉ dùng cho kết nối trực tiếp (khoảng cách < 10m).
- **Chi phí:**
  - Tổng chi phí phần cứng cho mỗi node nên dưới $70 (tương đương ~1.7 triệu VND, điều chỉnh do thêm cảm biến và linh kiện bổ sung) để dễ triển khai số lượng lớn.
- **Tần suất lấy mẫu:**
  - Cảm biến lấy mẫu ít nhất 1 lần/phút, có thể cấu hình tần suất qua phần mềm.
- **Độ chính xác:**
  - Nhiệt độ: ±0.5°C
  - Độ ẩm: ±3% RH
  - Bụi: ±10 μg/m³
  - Khí gas: ±5 ppm
  - Tốc độ gió: ±0.3 m/s
  - Báo cháy (khói): Phát hiện trong vòng 10 giây khi nồng độ khói vượt ngưỡng.
  - Hiện diện người: Phát hiện trong phạm vi 5-10m với độ nhạy cao.
  - Mưa: Phát hiện có/không mưa hoặc cường độ mưa (mm/h) với độ chính xác ±10%.
  - Áp suất khí quyển: ±1 hPa
  - RTC: Sai số thời gian < 1 giây/ngày.

## 2. Mô Tả Chi Tiết Hệ Thống (Detailed System Design)

### 2.1. Tổng Quan Hệ Thống
Hệ thống giám sát môi trường công trình bao gồm:
- **Node cảm biến:** Mỗi node sử dụng vi điều khiển ESP32, tích hợp các cảm biến (nhiệt độ, độ ẩm, bụi, gió, khí gas, báo cháy, hiện diện người, mưa, áp suất khí quyển), mô-đun RTC, còi/đèn LED, nguồn cấp (pin hoặc năng lượng mặt trời), và mô-đun truyền thông không dây (Wi-Fi/LoRa) cùng cổng UART bổ sung để giao tiếp với datalogger.
- **Datalogger trung tâm:** Thu thập dữ liệu từ các node qua Wi-Fi, LoRa, hoặc UART, lưu trữ, kết nối với đám mây, và cung cấp giao diện người dùng (web và di động).
- **Mạng lưới truyền thông:** Sử dụng giao thức Wi-Fi (cho khoảng cách ngắn), LoRa (cho khoảng cách xa, môi trường nhiều vật cản), hoặc UART (kết nối trực tiếp).
- **Nguồn năng lượng:** Kết hợp pin sạc, tấm pin mặt trời, và bộ điều khiển sạc để đảm bảo hoạt động liên tục.

### 2.2. Kiến Trúc Phần Cứng
#### 2.2.1. Node Cảm Biến
- **Vi điều khiển:** ESP32-WROOM-32
  - Tích hợp Wi-Fi và Bluetooth, hỗ trợ giao tiếp I2C, SPI, UART, ADC, và GPIO (Digital I/O).
  - Tiêu thụ năng lượng thấp với chế độ Deep Sleep.
- **Cảm biến và phương thức giao tiếp:**
  - **Nhiệt độ, độ ẩm, áp suất khí quyển:** BME280 (I2C, đo nhiệt độ ±0.5°C, độ ẩm ±3% RH, áp suất ±1 hPa).
  - **Bụi:** SDS011 (UART, đo PM2.5 và PM10).
  - **Tốc độ và hướng gió:** Cảm biến gió công nghiệp (analog hoặc UART, ví dụ: Davis Anemometer).
  - **Khí gas:** MQ-135 (analog, đo CO, CO2, NH3) hoặc cảm biến công nghiệp như SGX Sensortech (I2C/UART).
  - **Báo cháy:** Cảm biến khói quang học (ví dụ: MQ-2 hoặc cảm biến công nghiệp như Kidde Smoke Sensor, analog hoặc digital output) để phát hiện khói và/hoặc nhiệt độ cao.
  - **Hiện diện người:** Cảm biến mmWave (ví dụ: LD2410, UART hoặc digital output) để phát hiện chuyển động và sự hiện diện người trong phạm vi 5-10m.
  - **Mưa:** Cảm biến mưa (ví dụ: YL-83 hoặc cảm biến công nghiệp, digital output) để phát hiện có/không mưa hoặc đo cường độ mưa.
  - **RTC:** DS3231 (I2C, sai số < 1 giây/ngày) để cung cấp dấu thời gian chính xác.
- **Cảnh báo thời gian thực:**
  - Còi (buzzer) hoặc đèn LED tích hợp trên node, kích hoạt khi phát hiện khói, hiện diện người bất thường, mưa lớn, hoặc các thông số vượt ngưỡng.
- **Header giao tiếp:**
  - 2x I2C header (cho BME280, DS3231, và cảm biến công nghiệp I2C).
  - 1x SPI header (dự phòng cho cảm biến tốc độ cao).
  - 4x UART header (1 cho SDS011, 1 cho cảm biến gió, 1 cho mmWave LD2410, 1 dành riêng cho giao tiếp với datalogger).
  - 4x Analog input (cho MQ-135, cảm biến khói, hoặc cảm biến công nghiệp).
  - 4x Digital I/O (cho cảm biến mưa, cảm biến khói digital, mmWave, hoặc còi/đèn LED).
- **Nguồn cấp:**
  - Pin LiPo 3.7V 2000mAh.
  - Tấm pin mặt trời 6V 2W với bộ điều khiển sạc TP4056.
  - Đầu vào nguồn DC 5V/12V cho ứng dụng cố định.
- **Lưu trữ cục bộ:** Khe cắm thẻ microSD để lưu dữ liệu khi mất kết nối.
- **Vỏ bảo vệ:** Hộp nhựa hoặc kim loại đạt chuẩn IP65, chống bụi và nước.

#### 2.2.2. Datalogger Trung Tâm
- **Phần cứng:** Raspberry Pi 4 hoặc ESP32 với bộ nhớ flash lớn hơn.
- **Lưu trữ:** SSD hoặc thẻ SD 32GB để lưu dữ liệu dài hạn.
- **Giao tiếp:** Wi-Fi/LoRa để nhận-union nhận dữ liệu từ node, UART để kết nối trực tiếp với node gần, Ethernet/4G cho kết nối đám mây.
- **Màn hình:** Tùy chọn màn hình LCD 7 inch để hiển thị dữ liệu tại chỗ.
- **Kết nối đám mây:** Hỗ trợ AWS IoT hoặc Google Cloud để lưu trữ và phân tích dữ liệu từ xa.

### 2.3. Kiến Trúc Phần Mềm
- **Firmware Node (ESP32):**
  - Sử dụng Arduino framework hoặc ESP-IDF.
  - Lấy mẫu cảm biến theo tần suất cấu hình (mặc định 1 phút/lần).
  - Đọc dấu thời gian từ DS3231 để gắn vào dữ liệu cảm biến.
  - Hiệu chỉnh định kỳ các cảm biến (khói, mmWave) bằng thuật toán tự động so sánh với giá trị chuẩn.
  - Tối ưu năng lượng cho mmWave: Chỉ kích hoạt mmWave khi phát hiện chuyển động ban đầu qua cảm biến khác (ví dụ: cảm biến gió hoặc mưa).
  - Lưu trữ dữ liệu cục bộ trên thẻ SD nếu mất kết nối.
  - Gửi dữ liệu qua Wi-Fi (MQTT protocol), LoRa (giao thức tùy chỉnh), hoặc UART (JSON qua serial) đến datalogger, mã hóa bằng AES-256.
  - Chế độ Deep Sleep để tiết kiệm năng lượng.
- **Datalogger:**
  - Chạy Node-RED hoặc Python script để thu thập và xử lý dữ liệu từ Wi-Fi, LoRa, hoặc UART.
  - Lưu trữ dữ liệu vào cơ sở dữ liệu SQLite hoặc InfluxDB với dấu thời gian từ RTC.
  - Cung cấp giao diện web (Flask hoặc Node-RED) để theo dõi dữ liệu, bao gồm trạng thái báo cháy, hiện diện người, mưa, và áp suất khí quyển.
  - Gửi cảnh báo qua email/SMS hoặc ứng dụng di động khi thông số vượt ngưỡng (khói, hiện diện bất thường, mưa lớn, v.v.).
  - Kết nối với AWS IoT hoặc Google Cloud để lưu trữ và phân tích dữ liệu từ xa.
- **Ứng dụng di động:**
  - Ứng dụng Android/iOS sử dụng Flutter hoặc React Native để hiển thị dữ liệu thời gian thực, lịch sử, và gửi thông báo đẩy.
- **Giao thức truyền thông:**
  - Wi-Fi: MQTT với broker Mosquitto, mã hóa AES-256.
  - LoRa: Giao thức tùy edital với mã hóa AES-256.
  - UART: Truyền dữ liệu JSON hoặc CSV qua serial (baud rate 115200), mã hóa AES-256.

### 2.4. Mạng Lưới (Mesh Network)
- **Cấu trúc:** Sử dụng ESP-Mesh (Wi-Fi) hoặc LoRaWAN (LoRa). UART dùng cho kết nối trực tiếp với datalogger.
- **Chức năng:**
  - Các node tự động kết nối với node gần nhất để chuyển tiếp dữ liệu về datalogger (qua Wi-Fi/LoRa).
  - Node gần datalogger sử dụng UART để truyền dữ liệu trực tiếp, giảm tải mạng không dây.
  - Tự động tái cấu hình khi một node bị mất kết nối.
- **Khoảng cách:** LoRa hỗ trợ 1km trong môi trường công trình, Wi-Fi 100m, UART < 10m.

### 2.5. Nguồn Năng Lượng
- **Pin:** Pin LiPo 2000mAh đảm bảo hoạt động 24-48 giờ (tùy tần suất lấy mẫu).
- **Năng lượng mặt trời:** Tấm pin 6V 2W với bộ điều khiển sạc TP4056, hỗ trợ sạc trong điều kiện ánh sáng yếu.
- **Quản lý năng lượng:** 
  - ESP32 chuyển sang chế độ Deep Sleep giữa các chu kỳ lấy mẫu.
  - mmWave chỉ kích hoạt khi cần để tiết kiệm pin (ví dụ: khi phát hiện chuyển động qua cảm biến khác).

### 2.6. Danh Sách Cảm Biến và Phương Thức Giao Tiếp
| **Cảm Biến**                | **Model Ví Dụ**         | **Phương Thức Giao Tiếp** | **Chức Năng**                              |
|-----------------------------|-------------------------|---------------------------|--------------------------------------------|
| Nhiệt độ, Độ ẩm, Áp suất    | BME280                  | I2C                       | Đo nhiệt độ (±0.5°C), độ ẩm (±3% RH), áp suất (±1 hPa) |
| Bụi (PM2.5, PM10)          | SDS011                  | UART                      | Đo nồng độ bụi (μg/m³)                     |
| Tốc độ và Hướng gió         | Davis Anemometer        | Analog/UART               | Đo tốc độ gió (m/s) và hướng gió           |
| Khí gas                     | MQ-135                  | Analog                    | Đo nồng độ CO, CO2, NH3 (ppm)             |
| Báo cháy (khói/nhiệt độ)    | MQ-2 hoặc Kidde Sensor  | Analog/Digital            | Phát hiện khói hoặc nhiệt độ cao           |
| Hiện diện người             | LD2410 (mmWave)         | UART/Digital              | Phát hiện chuyển động/người trong 5-10m    |
| Mưa                         | YL-83                   | Digital                   | Phát hiện có/không mưa hoặc cường độ mưa   |
| RTC (Thời gian thực)        | DS3231                  | I2C                       | Cung cấp dấu thời gian chính xác           |
| Giao tiếp với Datalogger    | -                       | UART                      | Truyền dữ liệu trực tiếp đến datalogger    |

### 2.7. Các tính năng nâng cao
- **Cảm biến áp suất khí quyển:** BME280 để dự đoán thời tiết.
- **Cảnh báo thời gian thực:** Còi/đèn LED trên node để báo động tại chỗ khi phát hiện khói, hiện diện người bất thường, mưa lớn, hoặc các thông số vượt ngưỡng.
- **Kết nối đám mây:** Datalogger kết nối với AWS IoT hoặc Google Cloud để lưu trữ, phân tích dữ liệu, và cung cấp truy cập từ xa.
- **Ứng dụng di động:** Ứng dụng Android/iOS để theo dõi dữ liệu và nhận thông báo đẩy.
- **Bảo mật:** Mã hóa dữ liệu truyền (Wi-Fi, LoRa, UART) bằng AES-256.
- **Hiệu chỉnh cảm biến:** Cơ chế tự động hiệu chỉnh định kỳ cho cảm biến khói và mmWave, sử dụng dữ liệu tham chiếu từ cảm biến khác hoặc đám mây.
- **Tối ưu năng lượng cho mmWave:** Kích hoạt mmWave chỉ khi phát hiện chuyển động ban đầu qua cảm biến gió, mưa, hoặc cảm biến khác.
- **UART ưu tiên:** Sử dụng UART cho các node gần datalogger để tăng độ tin cậy và giảm tải mạng không dây.

## 3. Sơ Đồ Khối Hệ Thống
```
[Node 1] ---- [Node 2] ---- [Node 3]
   |             |             |
   |----[Wi-Fi/LoRa/UART]----[Datalogger]----[Cloud/App]
[Cảm biến: Nhiệt độ, Độ ẩm, Áp suất, Bụi, Gió, Gas, Báo cháy, mmWave, Mưa, RTC]
[Còi/LED, Nguồn: Pin, Năng lượng mặt trời]
```

## 4. Kế Hoạch Triển Khai
- **Giai đoạn 1:** Chọn và thử nghiệm cảm biến (bao gồm BME280, báo cháy, mmWave, mưa, RTC), phát triển firmware cho ESP32 với hỗ trợ UART, còi/LED, và hiệu chỉnh cảm biến.
- **Giai đoạn 2:** Thiết kế và thử nghiệm mạng lưới node với Wi-Fi/LoRa và UART.
- **Giai đoạn 3:** Xây dựng datalogger, giao diện web, ứng dụng di động, và kết nối đám mây (AWS IoT/Google Cloud).
- **Giai đoạn 4:** Triển khai thực tế tại công trình, hiệu chỉnh và tối ưu hóa.

## 5. Ngân Sách Ước Tính (cho mỗi node)
- ESP32-WROOM-32: $5
- BME280: $3
- SDS011: $20
- MQ-135: $2
- Cảm biến gió: $10
- Cảm biến báo cháy (MQ-2): $2
- Cảm biến mmWave (LD2410): $5
- Cảm biến mưa (YL-83): $2
- RTC (DS3231): $2
- Còi/đèn LED: $1
- Pin LiPo + TP4056: $5
- Tấm pin mặt trời: $3
- Vỏ IP65 + linh kiện khác (bao gồm mạch UART): $5
- **Tổng:** ~$65 (có thể tối ưu xuống dưới $60 khi sản xuất số lượng lớn).