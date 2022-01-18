# Ubuntu 20.04 server

---

Yêu cầu: Đã cài đặt Ubuntu 20.04 server

## Bước 1: Cài Apache + Cập nhật Firewall

Cập nhật các gói tin hệ thống

```console
sudo apt update
```

Với lệnh sudo nếu yêu cầu nhật mật khẩu thì cần nhập mật khẩu để xác nhận.

Sau đó cài đặt Apache

```console
sudo apt install apache2
```

Nếu có yêu cầu xác nhận thì chọn `Y`

Bây giờ cần cấu hình tường lửa để cho phép lưu lượng HTTP. UFW có thể làm điều này, chạy lệnh để xem các ứng dụng có sẵn UFW:

```
sudo afw app list
```

Terminal sẽ giống như sau:

Chúng ta chỉ cần cho luồng truy cập port 80 nên sẽ chỉ cần Apache:

```
sudo ufw allow in "Apache"
```

Xem ufw đã hoạt động chưa thì dùng:

```
sudo ufw status
```

Nếu status không phải là `active` thì chạy lệnh:

```
sudo ufw enable
```

Lúc này đã có thể kiểm tra Apache đã hoạt động chưa bằng cách truy cập vào IP của Ubuntu server.
Để kiểm tra IP của Ubuntu server thì có thể curl tới 1 trang web bất kì:

```
curl
```

## Bước 2: Cài MySQL

Để trang web có thể lưu trữ dữ liệu thì chúng ta cần có database. MySQL là database phổ biến nhất cho PHP.

Chúng ta sẽ dùng apt để cài MySQL:

```console
sudo apt install mysql-server
```

Nếu cần phải xác thực thì chọn `Y`.

Khi mà cài đặt MySQL xong, thì cần nâng cao bảo mật cho cơ sở sữ liệu. Mình sẽ cài một script bảo mật cho MySQL. Script này sẽ xóa một vài cài đặt mặc định và khóa quyền truy cập tới cơ sở dữ liệu:

```console
sudo mysql_secure_installation
```

Lúc này bạn sẽ nhận được yêu cầu cấu hình `VALIDATE PASSWORD PLUGIN`. Nếu chọn `Y` thì bạn sẽ cần đặt chọn 1 trong 3 mức độ mật khẩu được định sẵn. Dù có chọn `Y` hay không thì vẫn cần phải cài đặt mật khẩu cho `root user` của database. Nếu cấu hình `VALIDATE PASSWORD PLUGIN` thì khi nhập mật khẩu mình sẽ biết được độ mạnh của mật khẩu. Sau đó chọn `Y` nếu muốn kết thúc và bấm `enter`.

Có thể đăng nhập vào mysql bằng lệnh:

```console
sudo mysql
```

Để đăng xuất khỏi mysql có thể dùng lệnh

```console
exit
```

## Bước 3: Cài đặt PHP

Nếu như Apache để phân phối nội dung, MySQL để lưu trữ, quản lý dữ liệu thì PHP sẽ giúp xử lý câu lệnh để hiển thị nội dung với người dùng. Ngoài việc cài `php` thì cũng cần cài `php-sql` để giao tiếp với cơ sở dữ liệu và `libapache2-mod-php` để kích hoạt Apache xử lý các tệp PHP. Để cài đặt cả 3 gói trên cần chạy lệnh:

```console
sudo apt install php libapache2-mod-php php-mysql
```

Mình kiểm tra phiên bản PHP bằng câu lệnh:

```console
php -v
```

## Bước 4: Tạo máy chủ ảo (Virtual Host) cho trang web

Khi dùng Apache, mình có thể tạo ra các máy chủ ảo để lưu trữ nhiều tên miền trên 1 server. Mình sẽ ví dụ tạo tên miền có tên là `web1`. Tên miền này có thể thay bằng bất kì tên miền nào.

Apache trên Ubuntu 20.04 sẽ mặc định lấy tài nguyên từ tư mục /`var/www/html`. Nó chỉ hoạt động tốt cho 1 website đơn lẻ. Nếu có nhiều website được tại thì nên thay `/var/www/html` bằng cấu trúc `/var/www/web1` . `/var/www/html` vẫn sẽ là thư mục mặc định để cung cấp nếu yêu cầu từ client không khớp với bất kì website nào.

Tạo thư mục cho `web1`

```console
sudo mkdir /var/www/web1
```

Cấp quyền sở hữ thư mục bằng biến `$USER`, biến này sẽ tham chiếu tới người dùng hiện tại trong hệ thống.

```console
sudo chown -R $USER:$USER /var/www/web1
```

Sau đó tạo 1 file cấu hình trong `sites-available` cho `web1` bằng lệnh `nano`:

```
sudo nano /etc/apache2/sites-available/web1.conf
```

Chỉnh sửa thông tin như sau:

```
<VirtualHost *:80>
    ServerAdmin admin@localhost
    ServerName web1
    ServerAlias www.web1
    DocumentRoot /var/www/web1
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Lưu và đóng file bằng cách chọn `Ctrl+X` sau đó chọn `Y` rồi `Enter`. Mình đã cài để cho `/var/www/web1` trở thành thư mục gốc.

Bây giờ mình sẽ dùng `a2ensite` để mở kích hoạt máy chủ ảo mới:

```
sudo a2ensite web1
```

Nếu không sử dụng tên miền tùy chỉnh thì cần phải vô hiệu hóa trang web mặc định của Apache:

```
sudo a2dissite 000-default
```

Để chắc rẳng tệp cấu hình không có lỗi có thể chạy lệnh:

```
sudo apache2ctl configtest
```

Và cuối cùng cần reload lại những thay đổi của Apache để chúng có hiệu lực:

```
sudo systemctl restart apache2
```

Bây giờ trang web đã hoạt động nhưng nội dung của trang web vẫn trống. Mình sẽ tạo một dung hiển thị cho trang web bằng cách tạo file `index.html` trong thư mục `/var/www/web1`:

```
sudo nano /var/www/web1/index.html
```

Và tạo nội dung sau:

```
<html>
    <head>
        <title>Welcome to web1!</title>
    </head>
    <body>
        <h1>Success!  The web1 server block is working!</h1>
    </body>
</html>
```

Bây giờ mình có thể truy cập vào website thông qua địa chỉ IP của server bằng trình duyệt.

Mặc định server sẽ ưu tiên `index.html` là web mặc định, mình có thể thay đổi:

```
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Bây giờ có thể tùy chỉnh thứ tự bằng cách thay đổi thứ tự phần nội dung sau `DirectoryIndex`:

```
<IfModule mod_dir.c>
        DirectoryIndex index.html index.php index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Nếu thay đổi thì cần lưu, đóng tệp và khởi động lại Apache:

```
sudo systemctl restart apache2
```

Để có thể vào web qua domain thì cần chỉnh lại nội dung của file `/etc/hosts` của client, mình sẽ thêm vào file nội dung sau:

```
Server_IP web1.com
```

Vậy là có thể truy cập thông qua domain `web1.com`.

## Bước 5: Kiểm tra PHP trên máy chủ web.

Để kiểm tra PHP trên máy chủ web mình sẽ tạo 1 file `info.php` trong thư mục `web1`:

```
sudo nano /var/www/web1/info.php
```

Thêm nội dung sau:

```
<?php
phpinfo();
```

Lưu và đóng tệp. Sau đó có thể kiểm tra bằng cách truy cập vào

```
http://web1_IP/info.php
```

# Cài đặt Wordpress

## Bước 1: Tạo 1 cơ sở dữ liệu và tài khoản Wordpress

Đăng nhập vào cơ sở dữ liệu thông qua tài khoản `root`:

```
mysql -u root -p
```

Nếu ở phần trước cài đặt mật khẩu thì mình cần nhập đúng mật khẩu đã cài đặt.

---

**Lưu ý**

Nếu không sua cậ được ngay bằng cách trên thì cần đăng nhập như người dùng hệ thống và tạo một tài khoản `root`:

```
sudo mysql -u root
```

Sau đó tạo tài khoản mật khẩu của người dùng `root`:

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '12345678';
```

`12345678` là mật khẩu mình sử dụng, có thể thay thế bằng mật khẩu khác.
Sau đó thoát khỏi cơ sở dữ liệu bằng cách gõ `exit`. Vậy là sau đấy có thể đăng nhập cơ sở dữ liệu bằng câu lệnh dưới đây:

```
mysql -u root -p
```

---

Sau khi vào được `mysql`, mình sẽ tạo một cơ sở dữ liệu có tên `wordpress` để lưu trữ thông tin bằng lệnh:

```
mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

Sau đó mình sẽ tạo một tài khoản người dùng có tên `wordpressuser` với password là `trungdoxuan` để vận hành cơ sở dữ liệu `wordpress`:

```
mysql> CREATE USER 'wordpressuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'trungdoxuan';
```

Mình sẽ cấp tất cả các quyền cho user này:

```
mysql> GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
```

Cập nhật những thay đổi của MySQL:

```
mysql> FLUSH PRIVILEGES;
```

Và thoát khỏi MySQL:

```
mysql> exit;
```

## Bước 2: Update và cài bổ sung các gói của PHP

Cần update apt và cài bổ sung thêm một vài PHP extensions:

```
sudo apt update
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

Sau khi đã cài xong thì cần khởi động lại Apache2:

```
sudo systemctl restart apache2
```

## Bước 3: Điều chỉnh cấu hình Apache để .htaccess có thể ghi đè, tạo lại.

### Cho phép ghi đè .htaccess

Mở tệp cầu hình của wordpress:

```
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Bây giờ mình sẽ cài đặt với thư mục gốc của `web1`:

```
<Directory /var/www/web1/>
    AllowOverride All
</Directory>
```

Lưu nội dung chỉnh sửa và thoát.

### Cho phép Rewrite Module

Bật chế độ `mod_rewrite`:

```
sudo a2enmod rewrite
```

Kiểm tra lại các cú pháp trên:

```
sudo apache2ctl configtest
```

Sau khi thấy thông báo `Syntax OK` thì sẽ khởi động lại Apache để kích hoạt các thay đổi:

```
sudo systemctl restart apache2
```

## Bước 4: Tải xuống Wordpress

Mình sẽ mở thư mục `/tmp` và tải xuống bản nén:

```
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```

Giải nén tệp vừa tải:

```
tar xzvf latest.tar.gz
```

Có thể tạo 1 file `.htaccess` để sử dụng sau này:

```
touch /tmp/wordpress/.htaccess
```

Tiếp theo mình sẽ copy file cấu hình mẫu sang file cấu mình mà Wordpress chạy:

```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

Tạo một thư mục update để Wordpress sẽ không gặp vấn đề về quyền khi update trong tương lai:

```
mkdir /tmp/wordpress/wp-content/upgrade
```

Bây giờ mình sẽ copy toàn bộ nội dung tới thư mục của `web1`:

```
sudo cp -a /tmp/wordpress/. /var/www/web1
```

Cung cấp quyền sở hữu cho tất cả người dùng, nhóm người dùng `www-data` - đây là nhóm người dùng mà Apache web server sử dụng:

```
sudo chown -R www-data:www-data /var/www/web1
```

Mình sử dụng câu lệnh `find` để cấp quyền cho các file và thư mục trong `web1`:

```
sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
```

Cuối cùng mình sẽ cấu hình lại một vài thông tin để wordpress có thể kết nối đến MySQL:

```
sudo nano /var/www/wordpress/wp-config.php
```

Mình sẽ chỉnh nội dung các dòng sau:

```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'trungdoxuan' );
define( 'WP_DEBUG' ,true );
```

Lưu và đóng file.

## Bước 6: Hoàn thiện và cài đặt trên web.

Bây giờ mình sẽ vào được dẫn `web1.com/index.php` để tùy chình wordpress bằng giao diện.
