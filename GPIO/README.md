# 📘 TÀI LIỆU HƯỚNG DẪN GIẢNG DẠY: CẤU HÌNH VÀ GIAO TIẾP NGOẠI VI GPIO TRÊN STM32F1

---

## 🕒 KHUNG PHÂN BỔ TIẾN TRÌNH SƯ PHẠM

| Thời lượng | Phân đoạn | Mục tiêu trọng tâm |
| :--- | :--- | :--- |
| **20 phút** | **Phần 1:** Đánh giá nhận thức nền tảng | Kích thích tư duy hệ thống thông qua câu hỏi định hướng thực tế. |
| **30 phút** | **Phần 2:** Kiến trúc phần cứng & GPIO Core | Nắm vững cơ chế Clock Gating, hệ thống Bus APB2 và cấu trúc các Mode GPIO. |
| **30 phút** | **Phần 3:** Thực hành cấu hình Digital Output | Sử dụng STM32CubeMX, hiểu và làm chủ các API HAL điều khiển LED. |
| **10 phút** | *Giải lao kỹ thuật* | Xử lý các xung đột phần cứng hoặc Driver trên máy học viên. |
| **30 phút** | **Phần 4:** Nguyên lý Digital Input & Ma trận phím | Phân tích hiện tượng dội phím, giải thuật toán quét tuần tự (Time-multiplexed). |
| **50 phút** | **Phần 5:** Thực hành quét Ma trận phím 2x2 | Tích hợp hệ thống, lập trình giải mã ma trận phím điều khiển LED. |
| **10 phút** | **Phần 6:** Đúc kết quy trình & Giao đề tài mở rộng | Hệ thống hóa quy trình thiết kế và định hướng tối ưu thuật toán. |

---

## 🎯 PHẦN 1: ĐÁNH GIÁ NHẬN THỨC NỀN TẢNG (Interactive Q&A)

### 1. Câu hỏi gợi ý điều phối lớp học
* **Câu hỏi 1:** *"Khi các bạn nhấn nút nguồn trên điện thoại hoặc tương tác với bàn phím máy tính, về mặt bản chất điện tử, làm thế nào để bộ vi xử lý nhận biết được hành động đó?"*
* **Câu hỏi 2:** *"Làm thế nào một chip vi điều khiển được đóng gói trong vỏ nhựa có thể truyền năng lượng hoặc ra lệnh để một bóng đèn LED bên ngoài phát sáng?"*

### 2. Định hướng tư duy cho học viên
Mọi vi xử lý/vi điều khiển về bản chất là một thực thể tính toán logic cô lập. Để tương tác với thế giới vật lý bên ngoài (nhận tín hiệu từ cảm biến, nút bấm hoặc xuất dòng điện điều khiển cơ cấu chấp hành), nó bắt buộc phải thông qua các cổng giao tiếp vật lý. Khối ngoại vi cơ bản và quan trọng nhất đảm nhiệm vai trò này chính là **GPIO (General-Purpose Input/Output - Cổng vào/ra chung)**.

---

## ⚙️ PHẦN 2: KIẾN TRÚC PHẦN CỨNG VÀ NGUYÊN LÝ KHỐI GPIO

### 1. Hệ thống Bus và Cơ chế Phân phối xung nhịp (Clock Gating)
Vi điều khiển STM32F103C8T6 dựa trên lõi **ARM Cortex-M3 (32-bit RISC)**. Điểm khác biệt lớn nhất của kiến trúc này so với các dòng 8-bit phổ thông là sự phân tầng năng lượng cực kỳ nghiêm ngặt để tối ưu hóa công suất tiêu thụ.

* **Nguyên lý Clock Gating:** Mặc định khi khởi động hệ thống, để tiết kiệm năng lượng, tất cả các khối ngoại vi đều bị cô lập và ngắt nguồn cấp xung nhịp (Clock). Khi một khối không có xung nhịp, các thanh ghi (Registers) của nó không thể dịch chuyển trạng thái, đồng nghĩa với việc CPU không thể đọc/ghi dữ liệu lên khối đó.
* **Hệ thống Bus APB2:** Khối ngoại vi GPIO (bao gồm các Port A, B, C...) được treo trực tiếp trên đường bus ngoại vi tốc độ cao **APB2**. Do đó, quy trình bắt buộc đầu tiên trong mọi mã nguồn cấu hình phần cứng là phải **kích hoạt xung nhịp cho bus APB2 tại Port tương ứng**. Nếu quên bước này, mọi thao tác điều khiển chân sau đó đều không có tác dụng.

### 2. Các chế độ cấu hình cấu trúc chân GPIO (GPIO Topologies)

#### A. Trạng thái Xuất tín hiệu (Digital Output)
Học viên cần phân biệt rõ hai cấu trúc liên kết nội tại:
* **Push-Pull (Đẩy - Kéo):** Chân chip được điều khiển bởi hai Transistor PMOS (phía trên) và NMOS (phía dưới) hoạt động đối nghịch. 
    * Khi xuất mức logic 1 (3.3V), PMOS dẫn, NMOS ngắt $\rightarrow$ dòng điện được "đẩy" từ nguồn cấp ra chân chip.
    * Khi xuất mức logic 0 (0V), PMOS ngắt, NMOS dẫn $\rightarrow$ dòng điện được "kéo" từ chân chip dập xuống Đất (GND). 
    * *Ứng dụng:* Chế độ tiêu chuẩn để điều khiển LED, linh kiện logic hoặc truyền tín hiệu tốc độ cao.
* **Open-Drain (Cực máng hở):** Transistor PMOS phía trên bị loại bỏ hoàn toàn khỏi cấu trúc điều khiển. 
    * Khi xuất mức logic 0, NMOS dẫn $\rightarrow$ chân chip nối xuống GND (0V).
    * Khi xuất mức logic 1, NMOS ngắt $\rightarrow$ chân chip rơi vào trạng thái trở kháng cao (**Hi-Z**), lơ lửng về điện áp. Muốn chân lên được mức 1, bắt buộc phải kết nối thêm một điện trở kéo lên bên ngoài (External Pull-up).
    * *Ứng dụng:* Giao tiếp bus nhiều thiết bị (I2C) hoặc chuyển đổi mức điện áp.

#### B. Trạng thái Nhận tín hiệu (Digital Input)
* **Hiện tượng trôi nổi (Input Floating):** Nếu cấu hình chân là Input không có trở kéo cố định, chân chip đóng vai trò như một chiếc ăng-ten bắt các nhiễu điện từ trường môi trường. Giá trị đọc về tại thanh ghi dữ liệu vào (IDR) sẽ nhảy hỗn loạn giữa 0 và 1.
* **Internal Pull-up (Điện trở kéo lên nội tại):** Để cố định trạng thái, một điện trở tích hợp sẵn trong chip (thường có giá trị từ $30\text{k}\Omega$ đến $50\text{k}\Omega$) sẽ được kích hoạt để kết nối chân chip với nguồn 3.3V. Trạng thái mặc định khi chưa có tác động phần cứng bên ngoài luôn ổn định ở mức logic 1.

---

## 💻 PHẦN 3: THỰC HÀNH CẤU HÌNH DIGITAL OUTPUT (Bật/Tắt LED)

### 1. Hướng dẫn thiết lập tầng cấu hình cấu trúc (STM32CubeMX)
Học viên sử dụng công cụ đồ họa để trực quan hóa việc thiết lập phần cứng. Trên bo mạch STM32F103C8T6 tiêu chuẩn, LED được đấu sẵn vào chân **PC13** (Port C, Pin 13).
1. Khởi động STM32CubeMX, chọn vi điều khiển `STM32F103C8T6`.
2. Tại giao diện sơ đồ chân (Pinout view), tìm chân **PC13**, nhấn chuột trái và cấu hình thành `GPIO_Output`.
3. Vào tab **System Core** $\rightarrow$ **GPIO**, chọn chân PC13 và thiết lập các thông số học thuật:
    * *GPIO Output Level:* High (hoặc Low tùy theo sơ đồ nguyên lý kích LED).
    * *GPIO Mode:* Output Push-Pull.
    * *GPIO Pull-up/Pull-down:* No pull.
    * *Maximum Output Speed:* Low hoặc Medium (để giảm nhiễu nhiễu bức xạ tần số cao khi chuyển mạch).
4. Đặt tên project, chọn Toolchain/IDE là **MDK-ARM (Keil IDE)** và nhấn **Generate Code**.

### 2. Phân tích mã nguồn và Ứng dụng API lớp HAL (Hardware Abstraction Layer)
Khi mã nguồn mở ra trên Keil IDE, hướng dẫn học viên tìm đến file `main.c` $\rightarrow$ hàm `main()` $\rightarrow$ vòng lặp thời gian thực `while(1)`. Phân tích bản chất các hàm API được sử dụng:

* **Hàm xuất trạng thái logic:**
```c
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
```
* **Hàm đảo trạng thái (Toggle):**
```c
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
```
Mỗi lần thực thi, hàm này tự động đảo ngược giá trị hiện tại của chân trong thanh ghi dữ liệu ra (ODR), tối ưu hóa tốc độ xử lý khi không cần quản lý logic điều kiện.

* **Hàm tạo độ trễ (Delay):**
```c
HAL_Delay(1000);
```
Sử dụng xung nhịp từ bộ đếm hệ thống Systick để tạm dừng việc thực thi lệnh của CPU trong một khoảng thời gian xác định (tính bằng mili-giây).

### 3. Quy trình thực hiện viết mã trực tiếp và nạp bằng STM32CubeProgrammer
Trong trường hợp không sử dụng công cụ sinh code tự động STM32CubeMX, học viên cần làm quen với việc cấu hình cấu trúc dữ liệu thủ công qua thư viện HAL trên Keil IDE, sau đó ứng dụng công cụ chuyên dụng STM32CubeProgrammer để hoàn thiện chu trình nạp chip.
### A. Khởi tạo ngoại vi bằng mã nguồn C
Học viên định nghĩa hàm cấu hình chân PC13 trực tiếp trong file main.c:
```c
void GPIO_PC13_Init(void)
{
    // 1. Kích hoạt xung nhịp cho Port C trên Bus APB2 (Clock Gating)
    __HAL_RCC_GPIOC_CLK_ENABLE();

    GPIO_InitTypeDef GPIO_InitStruct = {0};

    // 2. Cấu hình các thông số cấu trúc cho chân PC13
    GPIO_InitStruct.Pin = GPIO_PIN_13;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;   // Output Push-Pull
    GPIO_InitStruct.Pull = GPIO_NOPULL;           // Không dùng trở kéo
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;  // Tốc độ thấp để giảm nhiễu nhiễu bức xạ
    
    // 3. Khởi tạo thanh ghi cấu hình vật lý thông qua API HAL
    HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
}
```
### B. Quy trình biên dịch và nạp Flash với STM32CubeProgrammer

Để triển khai mã nguồn xuống phần cứng mà không sử dụng các trình nạp tích hợp của IDE, quy trình thực hiện được chuẩn hóa qua các bước sau:

* **Bước 1: Thiết lập xuất file thực thi (Trên Keil IDE)**
    * Mở giao diện dự án, truy cập vào mục **Options for Target** (Biểu tượng chiếc đũa thần trên thanh công cụ).
    * Di chuyển đến Tab **Output**, tích chọn vào hộp kiểm **Create HEX File**.
    * Nhấn **OK** để lưu cấu hình. Tiến hành nhấn tổ hợp phím **Build (F7)** để biên dịch toàn bộ dự án và sinh file `.hex` trong thư mục đầu ra.
* **Bước 2: Kết nối vật lý hệ thống (Phần cứng)**
    * Sử dụng mạch nạp chuyên dụng **ST-Link v2** (hoặc tương đương) kết nối với máy tính thông qua cổng USB.
    * Đấu nối chính xác các đường tín hiệu từ mạch nạp sang các chân tương ứng trên bo mạch Blue Pill (`STM32F103C8T6`) bao gồm 4 đường dây bắt buộc:
        1. `SWDIO` (Serial Wire Data Input/Output)
        2. `SWCLK` (Serial Wire Clock)
        3. `GND` (Ground)
        4. `3.3V` (Power Supply)
* **Bước 3: Thao tác cấu hình nạp chip (Trên phần mềm STM32CubeProgrammer)**
    1. Khởi động ứng dụng **STM32CubeProgrammer**. Tại bảng cấu hình kết nối phía bên phải màn hình, chọn phương thức giao tiếp là **ST-LINK**, sau đó nhấn nút **Connect** để thiết lập liên kết thời gian thực với vi điều khiển.
    2. Sau khi kết nối thành công, di chuyển đến thanh điều hướng bên trái, chọn Tab **Erasing & Programming** (Biểu tượng mũi tên xanh hướng xuống).
    3. Tại trường dữ liệu **File Path**, nhấn nút **Browse** và trỏ đường dẫn trực tiếp đến file `.hex` vừa được biên dịch thành công từ Keil IDE.
    4. Tích chọn kích hoạt 2 chế độ kiểm soát:
        * **Verify programming:** Hệ thống sẽ tự động đối chiếu dữ liệu trên Flash sau khi nạp với file gốc để kiểm tra tính toàn vẹn, tránh lỗi dữ liệu.
        * **Run after programming:** Cho phép vi điều khiển tự động Reset và thực thi mã nguồn ngay lập tức sau khi chu trình nạp kết thúc mà không cần ngắt nguồn cấp.
    5. Nhấn nút **Start Programming** để bắt đầu quá trình xóa (Erase) và đổ dữ liệu (Write) vào bộ nhớ Flash của chip.
---

## 🔍 PHẦN 4: NGUYÊN LÝ DIGITAL INPUT & MA TRẬN PHÍM 2x2

### 1. Bài toán tối ưu chân bằng cấu trúc Ma trận phím 2x2
Khi thiết kế hệ thống có nhiều nút bấm, việc kết nối mỗi nút với 1 chân GPIO độc lập sẽ làm cạn kiệt tài nguyên chân của vi điều khiển. Với ma trận phím 2x2, ta xếp 4 nút bấm thành cấu trúc mạng lưới gồm 2 Hàng (Rows) và 2 Cột (Cols). Lúc này, số chân tiêu tốn chỉ còn $2 + 2 = 4$ chân thay vì 4 chân độc lập kèm các điện trở treo phức tạp ngoài mạch.

**Sơ đồ tư duy hình học:**

```text
            Cột 1 (PA2)    Cột 2 (PA3)
                 │              │
Hàng 1 (PA0) ────┼──[Nút 1]─────┼──[Nút 2]
                 │              │
Hàng 2 (PA1) ────┼──[Nút 3]─────┼──[Nút 4]
```
**Nguyên lý Quét tuần tự (Time-multiplexed Scanning):**
* **Cấu hình:** 2 chân Hàng ($R_1, R_2$) được cấu hình làm `GPIO_Output`. 2 chân Cột ($C_1, C_2$) được cấu hình làm `GPIO_Input Pull-up` (mặc định luôn đọc về mức 1 nhờ điện trở kéo lên nội tại).
* **Cơ chế quét:** CPU sẽ lần lượt kéo duy nhất một Hàng xuống mức logic 0, hàng còn lại giữ ở mức 1. Ngay tại thời điểm đó, CPU tiến hành đọc đồng thời trạng thái của 2 Cột. Nếu một nút bấm tại giao điểm của Hàng đang quét và Cột đang đọc được nhấn, mức logic 0 từ Hàng sẽ truyền qua tiếp điểm nút bấm, kéo điện áp của Cột đó sụt từ 1 xuống 0. Bằng cách đối chiếu tọa độ `[Hàng ép xuống 0][Cột đọc về 0]`, hệ thống định vị chính xác phím đang tương tác.

### 2. Hiện tượng Dội phím (Button Bouncing) và Giải pháp Khử nhiễu phần mềm
* **Bản chất vật lý:** Nút bấm là một cấu trúc cơ khí đàn hồi. Khi có lực tác động, các tiếp điểm kim loại không đóng khít ngay lập tức mà bị nảy liên tục trong khoảng thời gian từ $5\text{ms} \dots 20\text{ms}$ trước khi đi vào trạng thái xác lập ổn định. Vì vi điều khiển thực thi lệnh ở tốc độ cực cao, chuỗi xung nhiễu quá độ này sẽ bị nhận nhầm thành hàng chục hành vi bấm nút liên tục.
* **Giải thuật Khử nhiễu bằng phần mềm (Software Debouncing):**
    * *Bước 1:* Phát hiện tín hiệu sụt áp xuống mức 0 tại chân Input.
    * *Bước 2:* Gọi hàm trễ `HAL_Delay(20);` để tạm dừng hệ thống, bỏ qua toàn bộ giai đoạn nhiễu quá độ cơ học.
    * *Bước 3:* Đọc lại trạng thái chân Input. Nếu chân vẫn duy trì ở mức 0 $\rightarrow$ Xác nhận hành vi nhấn nút là hợp lệ.

---

## 🛠️ PHẦN 5: THỰC HÀNH LẬP TRÌNH QUÉT MA TRẬN PHÍM 2x2

### 1. Bài tập thực hành mẫu tại lớp
**Đề bài:** Thiết kế hệ thống giao tiếp ma trận phím 2x2 sử dụng Port A. Cấu hình chân `PA0, PA1` làm Hàng ($R_1, R_2$) và `PA2, PA3` làm Cột ($C_1, C_2$). Thiết lập thêm 4 chân trên Port B (`PB0, PB1, PB2, PB3`) làm Output nối với 4 LED đơn bên ngoài.

Viết chương trình thực hiện kịch bản xuất điện áp ra LED:
* Nhấn **Nút 1** (Hàng 1, Cột 1) $\rightarrow$ LED 1 (PB0) sáng, các LED khác tắt.
* Nhấn **Nút 2** (Hàng 1, Cột 2) $\rightarrow$ LED 2 (PB1) sáng, các LED khác tắt.
* Nhấn **Nút 3** (Hàng 2, Cột 1) $\rightarrow$ LED 3 (PB2) sáng, các LED khác tắt.
* Nhấn **Nút 4** (Hàng 2, Cột 2) $\rightarrow$ LED 4 (PB3) sáng, các LED khác tắt.

### 2. Hướng dẫn phân tích cấu trúc mã nguồn tuyến tính
Để học viên hiểu sâu luồng đi của dòng điện và logic quét, chương trình cần được viết một cách tường minh, tách biệt rõ ràng quy trình xử lý cho từng Hàng trong vòng lặp `while(1)`:
```c
/* ================= CHU KỲ QUÉT HÀNG 1 ================= */
// Bước 1: Dập Hàng 1 xuống 0, giữ Hàng 2 bằng 1
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET); 
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);   
HAL_Delay(1); // Thời gian trễ ngắn cho phép điện áp trên đường truyền ổn định

// Kiểm tra Cột 1 (PA2) -> Phát hiện Nút 1
if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET) {
    HAL_Delay(20); // Khử nhiễu dội phím cơ học
    if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET) {
        // Tác vụ: Bật duy nhất LED 1, tắt các LED còn lại
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3, GPIO_PIN_RESET);
        
        while(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET); // Vòng lặp khóa chờ nhả nút
    }
}

// Kiểm tra Cột 2 (PA3) -> Phát hiện Nút 2
if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET) {
    HAL_Delay(20);
    if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET) {
        // Tác vụ: Bật duy nhất LED 2
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_2 | GPIO_PIN_3, GPIO_PIN_RESET);
        
        while(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET);
    }
}

/* ================= CHU KỲ QUÉT HÀNG 2 ================= */
// Bước 1: Trả Hàng 1 về mức 1, dập Hàng 2 xuống mức 0
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);   
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET); 
HAL_Delay(1);

// Kiểm tra Cột 1 (PA2) -> Phát hiện Nút 3
if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET) {
    HAL_Delay(20);
    if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET) {
        // Tác vụ: Bật duy nhất LED 3
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_2, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_3, GPIO_PIN_RESET);
        
        while(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET);
    }
}

// Kiểm tra Cột 2 (PA3) -> Phát hiện Nút 4
if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET) {
    HAL_Delay(20);
    if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET) {
        // Tác vụ: Bật duy nhất LED 4
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_3, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_RESET);
        
        while(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3) == GPIO_PIN_RESET);
    }
}
```
### 🔍 Danh sách các lỗi hệ thống thường gặp (TA cần tập trung kiểm tra)
* **Quên lệnh khôi phục trạng thái Hàng:** Học viên sau khi quét xong Hàng 1 quên kéo chân Hàng 1 lên lại mức 1 mà đã dập tiếp Hàng 2 xuống 0. Hậu quả là cả hai hàng đều bằng 0, dẫn đến việc bấm nút ở hàng này nhưng chip nhận diện nhầm sang hàng khác.
* **Thiếu thời gian trễ trích mẫu (Micro-delay):** STM32 thực thi lệnh ở thang đo nano-giây. Ngay sau câu lệnh ghi trạng thái Hàng xuống 0, nếu đọc chân Cột ngay lập tức, trạng thái điện áp trên đường dây vật lý chưa kịp xả hết (do hiệu ứng điện dung ký sinh trên bo testboard). Điều này khiến giá trị đọc về bị sai lệch. Thêm `HAL_Delay(1);` là giải pháp tình thế hiệu quả cho người mới.

---

## 📝 PHẦN 6: ĐÚC KẾT QUY TRÌNH & GIAO ĐỀ TÀI MỞ RỘNG

### 1. Hệ thống hóa quy trình cấu hình hệ thống nhúng
Bất kể ngoại vi phức tạp nào sau này (UART, SPI, I2C, Timers), cấu trúc thiết lập cấu hình luôn tuân theo mô hình 3 bước bất biến:
* **Clock Activation:** Kích hoạt đường cấp năng lượng dẫn tới ngoại vi (Hệ thống Bus APB/AHB).
* **Hardware Configuration:** Thiết lập chức năng vật lý cho các cổng kết nối (Định hình cấu trúc các Mode vật lý).
* **Application Logic:** Xây dựng thuật toán điều khiển hoặc lấy mẫu tuần hoàn trong vòng lặp thời gian thực `while(1)`.

### 2. Đề tài mở rộng về nhà (Assignment)
Đoạn code thực hành tại lớp là code tuyến tính (Hardcode) nhằm mục đích tường minh hóa nguyên lý. Nhược điểm lớn của phương pháp này là làm phình to mã nguồn và gây lãng phí tài nguyên CPU nghiêm trọng do việc sử dụng các hàm trễ khóa tuyến tính (`HAL_Delay`).

**Yêu cầu tối ưu:** Học viên ứng dụng kiến trúc dữ liệu mảng (`array`) để lưu trữ danh sách các chân Hàng/Cột, thiết lập vòng lặp `for` lồng nhau để viết lại toàn bộ chương trình quét ma trận phím thành một hàm chức năng duy nhất:

```c
uint8_t KEYPAD_2x2_Scan(void);
```
Hàm này có nhiệm vụ quét toàn bộ ma trận và trả về giá trị từ 1 đến 4 tương ứng với số thứ tự của phím được nhấn, đồng thời áp dụng giải thuật tối ưu để không gây đóng băng hệ thống khi người dùng nhấn giữ nút lâu. Từ đó, chuẩn bị nền tảng tư duy để mở rộng hệ thống lên ma trận lớn hơn như 4x4 hoặc ma trận phím máy tính 104 phím.
