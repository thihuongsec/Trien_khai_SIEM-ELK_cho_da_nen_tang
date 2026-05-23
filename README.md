# Triển khai hệ thống SIEM-ELK trên Windows và Linux
_Elastic Stack–ELK là nhóm các công cụ mã nguồn mở thuộc SIEM, chữ cái ELK là viết tắt của 3 sản phẩm đại diện cho nhóm Elastic Stack gồm: elasticsearch, kibana, logstash. ELK có thể thu log từ tất các máy trong mạng là máy chủ, các hệ điều hành (windows, ubuntu), hệ thống phòng thủ (crowdsec, iptables), từ các ứng dụng (mail, social app, media,…) miễn là được cấu hình để kết nối với ELK_

## Làm quen với các agent của ELK
* Filebeat: thu thập và điều phối các tệp tin logs.
*	Winlogbeat: dành riêng cho hệ điều hành windows, chức năng thu thập các event logs như Filebeat.
*	Metricbeat: thu thập các chỉ số về phần cứng và dịch vụ (CPU, RAM, Disk, Load) nhằm giám sát hiệu năng.
*	Packetbeat: phân tích dữ liệu mạng trực tiếp dùng cho giám sát mạng, có thể phát hiện được các hành vi quét cổng bằng cách phân tích các giao thức mạng http, htttps, dns, sql,…
*	Auditbeat: giám sát hoạt động của người dùng về việc tuân thủ các chính sách và kiểm tra tính toàn vẹn của tệp tin (ví dụ: xem ai đã thay đổi tệp tin).
*	Heartbeat: giám sát thời gian hoạt động, xem các dịch vụ có đang hoạt động thông qua việc kiểm tra URL, IP.
*	Functionbeat: dành cho kiến trúc không máy chủ (serverless) chính là một mô hình điện toán đám mây, chức năng thu thập dữ liệu từ môi trường cloud. 

## Mô hình triển khai
<img width="931" height="529" alt="image" src="https://github.com/user-attachments/assets/a8e62012-c26f-463f-95c6-50d949996294" />

Thực hiện xây dựng trên nền tảng ảo hóa VMware
1. Yêu cầu phần cứng
* RAM tối thiểu 4GB, dung lượng 40gb.
2. Yêu cầu phần mềm:
*	1 máy server ubuntu: cài ELK Stack, filebeat, kibana, filezilla.
*	1 máy client ubuntu: cài filebeat, auditbeat.
*	1 máy client windows: cài winlogbeat, auditbeat.
*	1 máy kali linux: có các công cụ cơ bản cho tấn công.

## Mô hình hoạt động
<img width="947" height="534" alt="image" src="https://github.com/user-attachments/assets/983a41e7-150e-473a-b97e-ff4729ff7ac8" />

## Chuẩn bị ban đầu
### 1. Cấu hình mạng NAT, IP cùng dải
### 2. Cài đặt, cấu hình ELK
* wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
* `sudo apt-get install apt-transport-https`
* `echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee`
* `sudo apt-get install elasticsearch`
* `sudo systemctl start elasticsearch.service    `
<img width="975" height="85" alt="image" src="https://github.com/user-attachments/assets/34dd14a0-f0fe-4886-8468-aff3d7ce0601" />

Khi cài đặt thành công, một chuỗi mã là key sẽ xuất hiện, cần lưu lại key này để dùng cho bước tới. Truy cập vào path sau để cấu hình ELK
* `sudo nano /etc/elasticsearch/elasticsearch.yml`
<img width="394" height="94" alt="image" src="https://github.com/user-attachments/assets/f4f86e60-3f62-4c8e-a5d9-62a99986879d" />

<img width="778" height="115" alt="image" src="https://github.com/user-attachments/assets/5827a590-0203-4424-b9ab-d0af1bc7a2f2" />

<img width="926" height="122" alt="image" src="https://github.com/user-attachments/assets/49fcf08e-8c73-43ea-bb1e-89dc0082ac6b" />

restart lại dịch vụ và kiểm tra trạng thái
<img width="975" height="239" alt="image" src="https://github.com/user-attachments/assets/04dea1b3-e5bb-42ff-8159-be7731cfb5d2" />

### 3. Cài đặt kibana
* `sudo apt-get install kibana`
* `sudo nano /etc/kibana/kibana.yml`
<img width="515" height="207" alt="image" src="https://github.com/user-attachments/assets/288f1d2f-e59b-47c9-a0d7-7471d551105e" />

Sau khi cấu hình xong, restart lại dịch vụ và kiểm tra trạng thái hoạt động.
<img width="975" height="225" alt="image" src="https://github.com/user-attachments/assets/d55b1656-7e8f-41db-a7e2-50607b17658a" />

### 4. Enroll kibana từ web đến máy chủ cài ELK
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/0fd7e6f6-29a7-456b-9985-b286547c016c" />

### 5. Cài agent (filebeat, audit beat) trên linux
* `sudo apt-get install filebeat`
* `sudo nano /etc/filebeat/filebeat.yml`
<img width="688" height="353" alt="image" src="https://github.com/user-attachments/assets/d29d155c-d2d9-4870-886f-166640c890fc" />

<img width="975" height="553" alt="image" src="https://github.com/user-attachments/assets/06f171b4-a157-45e3-ae48-276ed6f33d9a" />

Tìm kiếm modul system bằng lệnh `filebeat modules list` và khởi động
<img width="808" height="108" alt="image" src="https://github.com/user-attachments/assets/8bcbe40d-4529-4103-966f-aa3a10e07e76" />

Cấu hình để module này biết thu log gì và gửi về đâu `sudo nano /etc/filebeat/modules.d/system.yml`
<img width="629" height="644" alt="image" src="https://github.com/user-attachments/assets/55d2d694-c4fc-4cef-8792-3945ef118d5c" />

Lưu và thoát cấu hình, cài thêm các esset cần thiết với lệnh dưới
<img width="975" height="187" alt="image" src="https://github.com/user-attachments/assets/c8dbdd5f-a591-4d96-94b9-acdec331a801" />

Trạng thái hoạt động tốt
<img width="975" height="250" alt="image" src="https://github.com/user-attachments/assets/dbd3b93e-28b1-4c36-a16a-7d42ec385382" />

Tương tự như trên, cài audit beat
<img width="875" height="78" alt="image" src="https://github.com/user-attachments/assets/acb6c652-82ae-401b-9101-a785c7de8b37" />

<img width="975" height="151" alt="image" src="https://github.com/user-attachments/assets/72a29903-98cf-4985-962f-757146732e0a" />

<img width="975" height="531" alt="image" src="https://github.com/user-attachments/assets/5d358654-ea11-428c-81d6-597d5e39634f" />

### 6. Cài agent trên windows
Cài winlogbeat trên internet và install vào máy windows bằng lần lượt lệnh sau:
* `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force`
* `PowerShell.exe -ExecutionPolicy UnRestricted -File .\install-service-winlogbeat.ps1`
<img width="975" height="225" alt="image" src="https://github.com/user-attachments/assets/f60b0bc6-9b57-4c51-8e44-50a392d08a57" />

* `Start-Service winlogbeat` khởi chạy dịch vụ
<img width="928" height="445" alt="image" src="https://github.com/user-attachments/assets/769c6d68-6d4b-410d-8845-4c3eed90e62d" />

Cấu hình winlogbeat như dưới bằng notepad
<img width="926" height="573" alt="image" src="https://github.com/user-attachments/assets/902dfb70-31c4-4887-a4c1-064b50b2e19c" />

<img width="915" height="642" alt="image" src="https://github.com/user-attachments/assets/a72853ea-fd26-4292-a46f-e1e60d6b1d60" />

Cài thêm các esset để chạy được agent này `.\winlogbeat.exe setup -e`. Sau đó vào system audit policy để chọn danh mục log muốn hệ thống ghi lại. Cuối cùng update các policy vừa thay đổi `gpupdate /force`.

## Thực hành
### Lab 01: Tìm hiểu và sử dụng ELK
### Lab 02: Quét port, service từ máy attacker và tìm vuln của app bằng công cụ nmap
### Lab 03: Tấn công và SOAR

## Kết luận
Nội dung chi tiết về thiết lập môi trường, cấu hình trên các máy, hướng dẫn sử dụng SIEM-ELK, kết quả của một số thử nghiệm trên môi trường ảo và phân tích các kết quả ấy được trình bày chi tiết trong tài liệu .docx đã tải lên trên repo này.
















