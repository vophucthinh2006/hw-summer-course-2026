# HW01: HỆ THỐNG GIÁM SÁT LÒ NHIỆT CÔNG NGHIỆP

## 1. MIÊU TẢ BÀI TOÁN

Trong các dây chuyền sản xuất, việc giám sát nhiệt độ của các lò thiêu hoặc lò nhiệt công nghiệp đòi hỏi hệ thống nhúng phải hoạt động cực kỳ chính xác và đồng thời. Hệ thống của bạn cần thực hiện chu trình khép kín bao gồm:

- Nhận gói tin cấu hình từ máy tính trung tâm để cập nhật ngưỡng nhiệt độ an toàn.
- Đọc liên tục giá trị từ cảm biến nhiệt độ.
- Điều khiển các thiết bị ngoại vi gồm Quạt làm mát và Đèn báo động dựa trên kết quả nhiệt độ.
- Toàn bộ chu trình vận hành trên được quản lý và thiết kế thông qua một **Sơ đồ trạng thái (Finite State Machine - FSM)** để đảm bảo tính an toàn hệ thống.

---

# 2. YÊU CẦU

Học viên phải hoàn thành **4 nhiệm vụ** (gồm viết 3 hàm logic trong file code và vẽ 1 sơ đồ FSM trong file báo cáo).

## Task 1 (Pointers & Memory): Bóc tách gói tin cấu hình

Hoàn thiện hàm:

```c
void parse_config(const uint8_t *config_packet, int16_t *high_threshold);
```

### Yêu cầu

- Nếu con trỏ đầu vào `NULL`, trả về `0`.
- Đọc mảng dữ liệu thô `config_packet` (gồm 2 byte).
- Học viên tiến hành gộp 2 byte này thành một số nguyên 16-bit có dấu (`int16_t`) theo định dạng **Little Endian**:
  - Byte Index 0 là byte thấp (**LSB**).
  - Byte Index 1 là byte cao (**MSB**).
- Giá trị sau khi gộp phải được ghi trực tiếp vào vùng nhớ mà con trỏ `high_threshold` đang trỏ tới.

---

## Task 2 (Compiler & Volatile): Đọc thanh ghi cảm biến phần cứng

Hoàn thiện hàm:

```c
int16_t read_temperature_reg(void *hw_sensor_reg);
```

### Yêu cầu

- Nếu con trỏ đầu vào `NULL`, trả về `0`.
- Hàm trả về giá trị nhiệt độ hiện tại bằng cách đọc trực tiếp từ ô nhớ/thanh ghi phần cứng thông qua con trỏ `hw_sensor_reg`.
- Cần:
  - Chọn kiểu phù hợp cho con trỏ.
  - Đảm bảo Compiler không tối ưu hóa lệnh này khi đặt trong vòng lặp vô hạn.

---

## Task 3 (Data Types & Bitwise): Thao tác bit điều khiển thiết bị xuất

Hoàn thiện hàm:

```c
void control_output(uint8_t *control_reg,
                    uint8_t fan_enable,
                    uint8_t alarm_enable);
```

### Yêu cầu

- Nếu con trỏ đầu vào `NULL`, trả về `0`.
- Tác động trực tiếp lên vùng nhớ của thanh ghi điều khiển hệ thống 8-bit `control_reg` bằng các toán tử bitwise theo quy tắc:

- Nếu `fan_enable = 1`:
  - BẬT Bit 0 (**Cooling Fan**) lên `1`.
- Nếu `fan_enable = 0`:
  - XÓA Bit 0 về `0`.

- Nếu `alarm_enable = 1`:
  - BẬT Bit 1 (**Alarm LED**) lên `1`.
- Nếu `alarm_enable = 0`:
  - XÓA Bit 1 về `0`.

- **Yêu cầu bắt buộc:** Các bit còn lại (từ Bit 2 đến Bit 7) phải được giữ nguyên trạng thái cũ, không được làm thay đổi dữ liệu của chúng.

---

## Task 4 (FSM): Thiết kế sơ đồ chuyển trạng thái FSM

Học viên **không viết code** cho phần này.

Hãy vẽ một sơ đồ khối trạng thái FSM (chụp ảnh vẽ tay hoặc dùng các công cụ như `draw.io`) dựa trên logic vận hành sau:

Hệ thống gồm 3 trạng thái chính:

- `STATE_SAFE`
- `STATE_WARNING`
- `STATE_EMERGENCY`

### Quy tắc chuyển trạng thái

- Trạng thái mặc định ban đầu là:

```text
STATE_SAFE
(Quạt TẮT, Đèn TẮT)
```

- Khi:

```text
Nhiệt độ hiện tại >= Ngưỡng high_threshold
```

Hệ thống lập tức chuyển sang:

```text
STATE_WARNING
(Quạt BẬT, Đèn TẮT)
```

- Nếu hệ thống ở `STATE_WARNING` liên tục quá **5 giây** mà nhiệt độ vẫn không hạ xuống dưới ngưỡng:

```text
STATE_EMERGENCY
```

- Nếu đang ở `STATE_WARNING` mà nhiệt độ thành công giảm xuống dưới ngưỡng:

```text
STATE_SAFE
```

---

# 3. RÀNG BUỘC

## Ràng buộc cú pháp

Bắt buộc sử dụng:

- Toán tử giải con trỏ `*`
- Các toán tử bitwise:

```c
&
|
~
<<
```

## Cấm gian lận logic

Cấm tính toán thủ công ra một hằng số tĩnh rồi gán trực tiếp.

Ví dụ:

```c
*control_reg = 0x01;
```

Code phải mang tính tổng quát.

## Cấm sử dụng

- Biến toàn cục (**Global variable**) để truyền hay chia sẻ dữ liệu giữa các hàm.

## File nộp bài

Bài nộp của học viên phải bao gồm:

1. File mã nguồn `.c` hoàn thiện 3 hàm.
2. File báo cáo chứa ảnh vẽ sơ đồ FSM ở Nhiệm vụ 4.

---

# 4. TESTCASE

Học viên có thể sử dụng bộ dữ liệu kiểm thử sau để đánh giá logic.

## Testcase 1 (Kiểm tra Pointer & Lập trình an toàn)

### Input 1

```text
config_packet = NULL
```

Kết quả:

```text
Hàm phải thoát an toàn, không làm crash chương trình.
```

### Input 2

```text
config_packet = {0x64, 0x00}
```

Kết quả:

```text
high_threshold nhận giá trị 100.
```

---

## Testcase 2 (Kiểm tra ép kiểu Volatile)

### Input

Một ô nhớ vùng ngoại vi có giá trị `105` được truyền qua con trỏ `void *`.

### Output mong đợi

```text
Hàm ép kiểu đúng, đọc và trả về chính xác số nguyên 105.
```

---

## Testcase 3 (Kiểm tra Quy trình Clear-then-Set)

### Input

```text
control_reg ban đầu = 0xFC (1111 1100)
fan_enable = 1
alarm_enable = 0
```

### Output mong đợi

```text
Giá trị mới của thanh ghi = 0xFD (1111 1101)
```

(Xóa 2 bit cũ rồi gán chuẩn xác bit mới.)

---

# Chúc các bạn làm bài vui vẻ!