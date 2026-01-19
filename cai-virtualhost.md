# HƯỚNG DẪN CÀI ĐẶT VIRTUAL HOST + SSL (CERTBOT) TRÊN AMAZON LINUX 2023 (AL2023)

Tài liệu này hướng dẫn cấu hình VirtualHost cho Apache (`httpd`) trên Amazon Linux 2023, kết hợp với Certbot để cài đặt SSL Let’s Encrypt.

---

## Bước 1: Dừng dịch vụ Apache

```bash
sudo systemctl stop httpd
```

## Bước 2: Tạo file VirtualHost
Tạo file cấu hình VirtualHost, ví dụ ```example.conf```:

```bash
sudo vi /etc/httpd/conf.d/example.conf
```

Nội dung cấu hình ban đầu như sau
(Lưu ý: thay đổi `ServerName`, `DocumentRoot`, đường dẫn SSL cho phù hợp với ứng dụng của bạn):

```bash
<VirtualHost *:80>
    ServerName atmtc-daisei-log-dev-api.vw-dev.com
    ServerAlias atmtc-daisei-log-dev-api.vw-dev.com
    DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public

    <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public/>
        AllowOverride All
    </Directory>

    Alias /.well-known /var/www/html/.well-known

    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{HTTPS} off
        RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
    </IfModule>
</VirtualHost>

<VirtualHost *:443>
    ServerName atmtc-daisei-log-dev-api.vw-dev.com
    DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public

    Alias /.well-known /var/www/html/.well-known

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/cert.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/chain.pem

    <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public>
        AllowOverride All
    </Directory>
</VirtualHost>

```

## Bước 3: Comment cấu hình SSL và redirect để chạy Certbot
Trước khi chạy Certbot, chỉnh sửa lại file example.conf và comment toàn bộ phần SSL + redirect HTTPS như sau:
```bash
<VirtualHost *:80>
    ServerName atmtc-daisei-log-dev-api.vw-dev.com
    ServerAlias atmtc-daisei-log-dev-api.vw-dev.com
    DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public

    <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public/>
        AllowOverride All
    </Directory>

    # Alias /.well-known /var/www/html/.well-known
    # <IfModule mod_rewrite.c>
    #     RewriteEngine On
    #     RewriteCond %{HTTPS} off
    #     RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
    # </IfModule>
</VirtualHost>

# <VirtualHost *:443>
#     ServerName atmtc-daisei-log-dev-api.vw-dev.com
#     DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public
#     Alias /.well-known /var/www/html/.well-known
#     SSLEngine on
#     SSLCertificateFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/cert.pem
#     SSLCertificateKeyFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/privkey.pem
#     SSLCertificateChainFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/chain.pem
#     <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public>
#         AllowOverride All
#     </Directory>
# </VirtualHost>

```
## Bước 4: Lưu file và khởi động Apache
```bash
sudo systemctl start httpd
```
check lại xem đã vào được trang web khi không có ssl hay chưa. nếu vào dc rồi thì quay lại vào ssh chạy lệnh sau
```bash
sudo systemctl stop httpd
```


## Bước 5: Chạy Certbot để xin SSL

```bash
sudo certbot certonly
```

Chọn các tùy chọn:
- Authentication method: 1 (Apache)
- Domain: chọn đúng domain, ví dụ:
`4: atmtc-daisei-log-dev-api.vw-dev.com`

## Bước 6: Bỏ comment để bật HTTPS + redirect
Sau khi Certbot chạy xong, mở lại file:
```bash
sudo vi /etc/httpd/conf.d/example.conf
```
Khôi phục toàn bộ cấu hình như sau:

```bash
<VirtualHost *:80>
    ServerName atmtc-daisei-log-dev-api.vw-dev.com
    ServerAlias atmtc-daisei-log-dev-api.vw-dev.com
    DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public

    <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public/>
        AllowOverride All
    </Directory>

    Alias /.well-known /var/www/html/.well-known

    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{HTTPS} off
        RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
    </IfModule>
</VirtualHost>

<VirtualHost *:443>
    ServerName atmtc-daisei-log-dev-api.vw-dev.com
    DocumentRoot /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public

    Alias /.well-known /var/www/html/.well-known

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/cert.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/atmtc-daisei-log-dev-api.vw-dev.com/chain.pem

    <Directory /var/www/atmtc-daisei-log/ATMTC-DAISEI-LOG-BE/public>
        AllowOverride All
    </Directory>
</VirtualHost>
```
## Bước 7: Lưu file và restart Apache
```bash
sudo systemctl restart httpd
```

Hoàn tất
- Truy cập ứng dụng qua HTTPS
- HTTP tự động redirect sang HTTPS
- Chứng chỉ SSL được quản lý bởi Certbot

