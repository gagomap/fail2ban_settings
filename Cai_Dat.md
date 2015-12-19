### Đây là hướng dẫn cài đặt cho Debian7|8 hoặc Ubuntu 12.04|14.04LTS. Nếu bạn muốn cài cho Centos thì cần phải đổi đường dẫn các file log trong file jail.local cho phù hợp. Thiết lập mặc định dùng cho nginx, tương thích hoàn toàn với EasyEngine.

## Trước tiên, chúng ta cài đặt fail2ban bằng lệnh

```bash
wget -O- http://neuro.debian.net/lists/trusty.us-ca.libre | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list

sudo apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu:80 0xA5D32F012649A5A9

sudo apt-get update

sudo apt-get install fail2ban
```
Link nguồn: 
```bash
http://neuro.debian.net/install_pkg.html?p=fail2ban
```

## Cập nhật settings bằng lệnh sau

```bash
\curl -sSL https://raw.githubusercontent.com/gagomap/install_fail2ban/master/install_fail2ban.sh > /usr/local/bin/install_fail2ban.sh

chmod +x /usr/local/bin/install_fail2ban.sh

sh /usr/local/bin/install_fail2ban.sh
```
Sau đó bạn có thể tùy chỉnh thiết lập rồi thực hiện lệnh:

```bash
service fail2ban restart
```
Khi nào bạn muốn cập nhật lại settings, thực hiện lại những lệnh trên.
Đường dẫn mặc định của file jail.local là ```/etc/fail2ban/jail.local```

## Note:
Mặc định mình để địa chỉ mail trong file jail.local là ...@dautu365.com (tìm dòng: destmail=..., dest=...).

Bạn phải đổi lại địa chỉ mail này thành địa chỉ mail của bạn, để fail2ban gửi mail báo khi ban ip nào đó.

Ngoài ra, bạn có thể đổi port ssh (mặc định là 22) trong file jail.local (sshd, sshd-dos, ssh-repeater) cho phù hợp.

Trong file nginx-badbots.conf, bạn có thể thay đổi bot bạn muốn chặn. Mặc định mình chặn hai bot MJ12bot và Ahref, nếu bạn cần dùng hai bot này, hãy xóa nó ở dùng badbots=...

#### Nếu bạn không cài suhosin, hãy disable [suhosin] trong file jail.local (chuyển true thành false).

## Hỗ trợ chống Brute forece Mysql, đặc biệt khi bị tấn công qua XSS, SQL injection,...

Bạn tạo file auth.conf tại thư mục /etc/mysql/conf.d với nội dung sau:

```bash
[mysqld]
log_error = /var/log/mysql/error.log
log-warning = 2
```

Sau đó khởi động lại mysql:

```bash
service mysql restart
```

## Cài đặt thêm cho Wordpress:
Bạn bắt buộc phải cài đặt thêm 2 plugin sau:

### Stop user enumeration plugin - hỗ trợ chống dò username

Nguồn:
```bash
https://wordpress.org/plugins/stop-user-enumeration/
```

### WP fail2ban plugin - hỗ trợ chống brute forces đăng nhập wordpress 

Nguồn:
```bash
https://wordpress.org/plugins/wp-fail2ban/
```
## Hỗ trợ thêm cho plugin chống spam Antispam-bee

Antispam-bee là plugin chống spam cực kì hiệu quả, nó hỗ trợ ghi log ra file riêng, để fail2ban có thể đọc và ban ip.
Bạn thêm dòng này vào file wp-config.php của site bạn, đặt trước dòng "/* That's all, stop editing! Happy blogging. */":

```bash
define('ANTISPAM_BEE_LOG_FILE', 'path/to/file/spam.log');
```

Trên hệ điều hành Debian hoặc Ubuntu, bạn thực hiện lệnh sau để khởi tạo file log cho Antispam-bee:

```bash
sudo touch /path/to/file/spam.log
sudo chown www-data:www-data /path/to/file/spam.log
```

Bạn phải đổi path/to/file/spam.log  thành đường dẫn tới file spam.log (chẳng hạn /var/www/site_cua_ban.com/spam.log).

### Chú ý: Nếu bạn không sử dụng antispam-bee, hãy vô hiệu hóa [antispam-bee] trong file jail.local (chuyển true thành false)

Nguồn:
```bash
https://github.com/pluginkollektiv/antispam-bee/wiki/Dokumentation#logdatei-f%C3%BCr-fail2ban
https://gist.github.com/sergejmueller/5622883
https://blog.shadypixel.com/spam-log-plugin/
```

## Module Nginx http limit requests (yêu cầu bắt buộc)

Đây là module rất hay của nginx để chống ddos
Trong file /etc/nginx/nginx.conf, ở trong block http{..}, thêm vào dòng sau (nếu chưa có):

```bash
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```

10m là kích cỡ của zone. 1MB có thể giữ 16000 states, hay 16000 địa chỉ IP. Trong trường hợp bạn có nhiều site trên 1 VPS hoặc site có truy cập cao, bạn có thể tăng kích cỡ của zone lên 20M hoặc 100M.

1r/s (1request/s) nghĩa là trung bình 1 lượt truy vấn/giây. Bạn có thể thay đổi tùy theo lượng truy cập site. Mặc định 1r/s tương đương với 60r/p. 
Tuy nhiên không nên đặt quá cao.
Bất cứ lượt truy vấn/giây vượt quá quy định, fail2ban sẽ tự động ban ngay lập tức ip đó. ([nginx-limit-req])

Bạn có thể thêm dòng sau vào block server[..] để chống ddos cho đường dẫn cụ thể:

```bash
location = /wp-login.php {
    limit_req   zone=one  burst=1 nodelay;
    include fastcgi_params;
    fastcgi_pass 127.0.0.1:9000;
}
```

## Ban vĩnh viễn ip nào đó lặp lại hành động sau khi đã bị ban.

Đọc thêm tại đây:
Nguồn:

```bash
http://stuffphilwrites.com/2013/03/permanently-ban-repeat-offenders-fail2ban/
```

## Hỗ trợ CSF và APF

CSF: Configserver Security and Firewall
APF: Advanced Policy Firewall

Để cài đặt chung fail2ban với CSF hoặc APF, bạn cần đổi "banaction = iptables-multiport" trong file jail.local, thành csf hoặc apf (tương ứng với csf.conf hoặc apf.conf trong thư mục action.d)
Để hỗ trợ ban vĩnh viễn ip như trên, bạn cần sửa lại file iptables-repeater.conf cho phù hợp. (Thay iptables bằng csf hoặc apf)