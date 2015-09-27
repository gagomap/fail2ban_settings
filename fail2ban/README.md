This is fail2ban settings for fai2ban 0.8.11 on ubuntu 14.04 LTS

## Install:

```bash
git clone git://github.com/gagomap/fail2ban_settings.git
```
Then copy the folder name "fail2ban" inside to /etc/fail2ban

## Note:

You can change user@yourdomain.com to your mail adress, and port 22 to your ssh port in jail.local.

## Nginx http limit requests module

In /etc/nginx/nginx.conf file under http{..} block, add following

```bash
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```
10m is size of zone. 1MB can hold 16000 states. I think this means 16000 unique IP addresses. In case you have way too many sites or very high traffic sites, you may want to increase it to 20MB or 100MB.

1r/s means 1 request per second is allowed. You cannot specify fractions. If you want to slowdown further, means less requests per second try 30r/m which means 30 requests per min, effectively 1 request per 2 second.


You can add something like below to server{..} block:

```bash
location = /wp-login.php {
    limit_req   zone=one  burst=1 nodelay;
    include fastcgi_params;
    fastcgi_pass 127.0.0.1:9000;
}
```

Link:
```bash
https://rtcamp.com/tutorials/nginx/fail2ban/
```bash