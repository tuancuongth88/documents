
# Hướng dẫn Tạo Alarm cho CloudWatch

## 1. Cấu hình Role cho EC2

- Gán các quyền sau cho EC2: `AmazonSSMFullAccess`, `CloudWatchAgentAdminPolicy`, `AmazonSSMManagedInstanceCore`.
- Nếu EC2 của bạn đã có Role, hãy gán Role vào nhóm. Hiện tại, các dự án của **atmtm** đang sử dụng Role là **atmtc**.

## 2. Cài đặt Amazon CloudWatch Agent

- **Tải xuống CloudWatch Agent**:
  ```bash
  wget https://s3.ap-northeast-1.amazonaws.com/amazoncloudwatch-agent-ap-northeast-1/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm

- **Kiểm tra trạng thái Amazon SSM Agent**:
	 ```bash
	sudo systemctl status amazon-ssm-agent
- **Kích hoạt Amazon SSM Agent**:
	```bash
	sudo systemctl enable amazon-ssm-agent
## 3. Cấu hình CloudWatch Agent
- **Cài đặt gói Amazon CloudWatch Agent**:
	```bash
	sudo yum localinstall amazon-cloudwatch-agent.rpm
- **Chạy trình hướng dẫn cấu hình**:
	```bash
	sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
- **Khởi động CloudWatch Agent**:
	```bash
	sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start
- **Kiểm tra trạng thái CloudWatch Agent**:
	```bash
	sudo systemctl status amazon-cloudwatch-agent
## 4. Cài đặt và Cấu hình Collectd

- **Cài đặt Collectd**:
	```bash
	sudo yum install collectd
- **Kiểm tra trạng thái Collectd**:
	```bash
	sudo systemctl status collectd
- **Khởi động Collectd**:
	```bash
	sudo systemctl start collectd
- **Khởi động CloudWatch Agent với cấu hình từ SSM**:
	```bash
	sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c ssm:AmazonCloudWatch-yamapan
- **Kiểm tra trạng thái CloudWatch Agent**:
	```bash
	systemctl status amazon-cloudwatch-agent
## 5. Cấu hình các Alarm trong CloudWatch theo các tiêu chí sau:
-   **CPUCreditBalance**: < 288 cho 1 điểm dữ liệu trong vòng 5 phút.
-   **mem_used_percent**: > 80% cho 5 điểm dữ liệu trong vòng 5 phút.
-   **disk_used_percent**: > 80% cho 5 điểm dữ liệu trong vòng 5 phút.
-   **CPUUtilization**: > 80% cho 5 điểm dữ liệu trong vòng 5 phút.
