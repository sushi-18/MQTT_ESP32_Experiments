## Bố trí thí nghiệm 
- Tạo 1 broker miễn phí trên HiveMQ
    + Sau khi tao thành công 1 broker miễn phí trên HiveMQ, tiếp tục tạo 1 username và password rồi Connect 
  ![Hinh 1](https://github.com/sushi-18/MQTT_ESP32_Experiments/blob/main/MQTT_ESP32/image/anh1.png)
- Dùng thư viện PubSubClient trên ESP32 kết nối với một MQTT Broker HiveMQ.
- Sử dụng thư viện Ticker, một thư viện chuẩn trong Arduino để gọi hàm publish một cách đều đặn và bất đồng bộ, mỗi giây (1s) một lần:
    + Mã: `mqttPulishTicker.attach(1, mqttPublish)`
- Subscribe tới topic `esp32/echo_test` ngay sau khi MQTT connect thành công
- Gọi hàm `mqttClient.loop()` trong main loop để handle các thông điệp nhận được từ broker (bất đồng bộ, event driven) bất kỳ lúc nào. 
- Phát hiện mất kết nối MQTT `if (!mqttClient.connected())` trong main loop để kết nối lại `mqttReconnect()` ngay khi phát hiện mất kết nối.

## Kịch bản thí nghiệm

- Sau khi ESP32 khởi động, sẽ kết nối WiFi vào một điểm phát AP đã định (ssid, và pass trong secrets/wifi.h) --> thành công
- Sẽ thấy MQTT Client kết nối đến broker thành công và bắt đầu gửi (publish) và nhận (subscribe) số đếm tăng dần trong `echo_topic` đều đặn
- Khi đó sẽ tiến hành ngắt điểm phát WiFi, tiện nhất là phát wifi từ điện thoại để bật ngắt nó nhanh chóng trong tầm tay
- Quan sát phản ứng của MQTT Client trong mã khi mất kết nối WiFi giữa chừng, 
- Sau đó bật lại điểm phát WiFi và quan sát khả năng khôi phục kết nối, và quan sát việc mất gói tin trong quá trình kết nối.

## Mục đích 
- Xem việc ngắt kết nối từ bên dưới chồng Internet Protocol có ảnh hưởng tới lớp trên không. Ở đây là lớp WiFi (link layer) bị ngắt --> lớp TCP/IP bị ngắt --> có ảnh hưởng tới lớp ứng dụng MQTT trên cùng hay không? ESP core lib sẽ in ra thông điệp lỗi như nào (có báo lỗi từ lớp dưới lên lớp bên trên hay không?)
- Quan sát sự bỏ mặc việc mất thông điệp trong QoS = 0. 
- Hiểu rõ hơn về cơ chế hoạt động của MQTT client bên trên tầng TCP/IP, nhất là cơ chế phát hiện mất kết nối và khôi phục kết nối ở lớp vật lý, rất hay xảy ra trong thực tế.

## Kết quả
Quan sát thông điệp in ra theo thời gian ta thấy một vài điều thú vị ngoài dự kiến như sau:

![Hinh 2](https://github.com/sushi-18/MQTT_ESP32_Experiments/blob/main/MQTT_ESP32/image/anh2.png)
**Hình 2**

1. Thư viện PubSubClient có thể gọi hàm publish trước khi thiết lập kết nối thành công với broker
- **Hình 2** cho thấy: có hai thông điệp được "publish" là 0 và 1 trước cả khi MQTT kết nối thành công
- Sau khi kết nối WiFi thành công, thì `Attempting MQTT connection...` mất khoảng 2s (có thể là 3s) để thiết lập kết nối (mỗi lần publish là 1s).

2. Thông điệp đầu tiên mà ESP32 nhận được từ broker chính là cái `retained message`(This is retained message):
- Xem **Hình 2**

3. Cơ chế Echo hoạt động bình thường như dự kiến, kể cả với QoS 0:
- Trên **Hình 2**
- Điều này không có gì lạ, vì khi mội kết nối được thiết lập thì lớp TCP/IP truyền thông điệp rất tốt 
- Không quan sát thấy bị mất gói tin lần nào kể cả việc publish và subscribe với QoS = 0. 

![Hình 3](https://github.com/sushi-18/MQTT_ESP32_Experiments/blob/main/MQTT_ESP32/image/anh3.png)
**Hình 3**

4. Khi ngắt điểm phát WiFi (AP):
- **Hình 3** ngắt tín hiệu từ bộ phát WiFi - ví dụ: trên điện thoại, giữa chừng khi ESP32 đang publish thông điệp 18 (xem hình 3).

![Hình 4](https://github.com/sushi-18/MQTT_ESP32_Experiments/blob/main/MQTT_ESP32/image/anh4.png)
**Hình 4**

## Kết luận 

Việc "làm các thí nghiệm" trong công nghệ lập trình là vô cùng hữu ích ở nhiều phương diện:

- hiểu rõ hơn về tương tác của các thành phần trong mã
- hiểu rõ hơn về các trường hợp không được nói trong tài liệu nhưng có thể xảy ra trong thực tế (edge cases)
- giúp người lập trình hiểu rõ hơn về API của các thư viện mình sắp dùng 
- cũng là quá trình tiếp cận các thư viện và công nghệ mới hiệu quả vì nó cần phải động não mà cũng khá đơn giản.
