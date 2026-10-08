# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## 1. Mục tiêu

Thiết lập và quản trị tường lửa UFW trên Ubuntu Server để bảo vệ máy chủ chạy nhiều dịch vụ đồng thời.

Các dịch vụ dự kiến:

* SSH: Port `22/tcp`
* Nginx Web: Port `80/tcp`
* Spring Boot: Port `8082/tcp`
* MySQL: Port `3306/tcp`

Yêu cầu bảo mật:

* Mặc định chặn tất cả kết nối đi vào.
* Cho phép SSH để quản trị máy chủ.
* Cho phép HTTP để phục vụ website.
* Cho phép ứng dụng Spring Boot truy cập từ xa.
* Không cho phép truy cập MySQL từ Internet.

---

## 2. Môi trường thực hiện

* Hệ điều hành: Ubuntu Server 24.04 LTS
* Firewall: UFW
* Web Server: Nginx
* Public IP: `221.121.4.73`

---

## 3. Cài đặt UFW

Cập nhật danh sách package:

```bash
sudo apt update
```

Cài đặt UFW:

```bash
sudo apt install ufw -y
```

Kiểm tra trạng thái ban đầu:

```bash
sudo ufw status
```

---

## 4. Cấu hình chính sách mặc định

Chặn toàn bộ kết nối đi vào:

```bash
sudo ufw default deny incoming
```

Cho phép các kết nối đi ra:

```bash
sudo ufw default allow outgoing
```

Kết quả mong muốn:

```text
Default incoming policy changed to 'deny'
Default outgoing policy changed to 'allow'
```

---

## 5. Cho phép các dịch vụ cần thiết

### 5.1. Cho phép SSH - Port 22

```bash
sudo ufw allow 22/tcp
```

Mục đích: cho phép kết nối SSH để quản trị máy chủ từ xa.

### 5.2. Cho phép HTTP - Port 80

```bash
sudo ufw allow 80/tcp
```

Mục đích: cho phép người dùng truy cập website thông qua Nginx.

### 5.3. Cho phép Spring Boot - Port 8082

```bash
sudo ufw allow 8082/tcp
```

Mục đích: cho phép kiểm tra ứng dụng Spring Boot từ xa.

---

## 6. Không cho phép MySQL - Port 3306

Không tạo luật `ALLOW` cho port `3306`.

Nếu trước đó đã tạo nhầm rule:

```bash
sudo ufw allow 3306/tcp
```

thì xóa bằng:

```bash
sudo ufw delete allow 3306/tcp
```

Port `3306/tcp` sẽ bị chặn theo chính sách mặc định:

```text
deny incoming
```

Điều này giúp hạn chế việc truy cập trực tiếp MySQL từ Internet.

---

## 7. Kích hoạt UFW

Sau khi đã cho phép SSH, HTTP và Spring Boot, kích hoạt UFW:

```bash
sudo ufw enable
```

Nếu hệ thống hiển thị cảnh báo:

```text
Command may disrupt existing ssh connections.
Proceed with operation (y|n)?
```

chọn:

```text
y
```

---

## 8. Kiểm tra trạng thái UFW

Sử dụng lệnh:

```bash
sudo ufw status verbose
```

Kết quả thực tế:

```text
Status: active
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
8082/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp                     ALLOW       Anywhere (v6)
8082/tcp                   ALLOW       Anywhere (v6)
```

---

## 9. Phân tích kết quả

### Port 22/tcp

```text
22/tcp ALLOW IN Anywhere
```

Cho phép kết nối SSH từ bên ngoài để quản trị máy chủ.

### Port 80/tcp

```text
80/tcp ALLOW IN Anywhere
```

Cho phép truy cập website Nginx thông qua HTTP.

### Port 8082/tcp

```text
8082/tcp ALLOW IN Anywhere
```

Cho phép truy cập ứng dụng Spring Boot từ xa.

### Port 3306/tcp

Không xuất hiện trong danh sách `ALLOW`.

Do chính sách mặc định là:

```text
Default: deny (incoming)
```

nên kết nối từ Internet tới MySQL port `3306` bị chặn.

---

## 10. Kết luận

Đã hoàn thành cấu hình tường lửa UFW theo yêu cầu:

* [x] UFW được cài đặt và kích hoạt.
* [x] Mặc định từ chối kết nối vào (`deny incoming`).
* [x] Cho phép SSH port `22/tcp`.
* [x] Cho phép HTTP port `80/tcp`.
* [x] Cho phép Spring Boot port `8082/tcp`.
* [x] Không cho phép MySQL port `3306/tcp`.
* [x] Chính sách kết nối ra ngoài được cho phép (`allow outgoing`).

Hệ thống đã được cấu hình để chỉ mở các cổng dịch vụ cần thiết và hạn chế truy cập trực tiếp tới cơ sở dữ liệu MySQL từ Internet.
