## Bố trí thí nghiệm 
- Cài đặt https://mosquitto.org/ borker và MQTTX trên máy
- Thử nghiệm với localhost: Mở Admintrator:Command Prompt
    + Dùng lệnh 'net start mosquitto' để bắt đầu khởi động mosquitto broker 
    + DÙng lệnh 'mosquitto -v' để theo dõi chi tiết về quá trình kết nối, các sự kiện xuất bản và đăng ký của các client
    + Cấu hình listener 1883, listener 8883, username, password cho localhost; với listener 8883 cần tạo self-signed certificate
- Sử dụng phần mềm MQTTX để kiểm tra kết nối với localhost port 1883 và 8883
(Ngoài ra, có thể sử dụng 1 MQTT broker công khai test.mosquitto.org do Eclipse Mosquitto cung cấp)

## Kịch bản thí nghiệm
- Kiểm tra các trạng thái kết nối thành công 
- Thiết lập một client để publish tin nhắn lên topic esp32/echo_test
- Dùng MQTTX kiểm tra tin nhắn được gửi và nhận trong thời gian thực.
- Thử nghiệm với các chủ đề khác nhau và xác nhận hoạt động ổn định của broker, bao gồm cả cổng 1883 và 8883

## Mục đích 
- Đánh giá tính ổn định và hiệu suất của Mosquitto broker trên các cổng.
- Xác nhận tính chính xác của dữ liệu truyền qua MQTT: Đảm bảo tin nhắn được gửi và nhận đúng nội dung, không bị mất mát.

## Kết quả
1. Dùng MQTTX kiểm tra kết nối thành công  
- Cấu hình hoàn thiện cho 2 cổng 1883 và 8883 trên MQTTX với Broker localhost (ví dụ cấu hình cho cổng 8883 ở **Hình 1**)
![Hình 1](./images/hinh3.png "Hình 1")
**Hình 1**
    + Nhập đầy đủ các thông tin Name, Host, Port, Client ID, Username, Password
    + Đối với cổng 8883, cần địa chỉ chứng chỉ CA đã tạo
- **Hình 2** và **Hình 3** cho thấy đã kết nối thành công giữa 2 cổng 1883 và 8883
![Hình 2](./images/hinh1.png "Hình 2")
**Hình 2**
![Hình 3](./images/hinh4.png "Hình 3")
**Hình 3**

2. Kiểm tra tin nhắn gửi, nhận
- Gửi một tin nhắn (publish) thử nghiệm tới MQTT broker Mosquitto đang chạy trên localhost qua cổng 1883 
- Dùng MQTTX kiểm tra tin nhắn được gửi và nhận trong thời gian thực qua cổng 8883 (**Hình 5**)
![Hình 5](./images/hinh5.png "Hình 5")
**Hình 5**


## Kết luận 
- Kết quả thu được từ việc sử dụng MQTTX để kiểm tra kết nối, gửi và nhận tin nhắn cho thấy Mosquitto broker đáp ứng tốt yêu cầu về độ tin cậy và hiệu suất, kể cả khi sử dụng các cấu hình bảo mật như TLS.
- Thông qua việc cấu hình và thử nghiệm với localhost (có thể thử nghiệm thêm broker công khai test.mosquitto.org), ta thấy tính bảo mật và chính xác khi dùng Mosquitto trong việc giao tiếp qua các cổng khác nhau. 
