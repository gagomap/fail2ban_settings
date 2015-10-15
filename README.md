This is fail2ban settings for fail2ban 0.8.13 on Ubuntu 14.04 LTS

## Install fail2ban

```bash
wget -O- http://neuro.debian.net/lists/trusty.us-ca.libre | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list

sudo apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu:80 0xA5D32F012649A5A9

sudo apt-get update

sudo apt-get install fail2ban
```

Link: 
```bash
http://neuro.debian.net/install_pkg.html?p=fail2ban
```

## Install settings

```bash
git clone git://github.com/gagomap/fail2ban_settings.git
```
Then copy the folder name "fail2ban" inside fail2ban_settings to /etc/fail2ban  (use "cp -rf" to overwrite)

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
```

## Permanently Ban Repeat Offenders With fail2ban
Link:

```bash
http://stuffphilwrites.com/2013/03/permanently-ban-repeat-offenders-fail2ban/
```
## Stop user enumeration plugin (must install)
Link:
```bash
https://wordpress.org/plugins/stop-user-enumeration/
```

## WP fail2ban plugin
Link:
```bash
https://wordpress.org/plugins/wp-fail2ban/
```

## Another way to stop brute force ssh  port (your ssh port)by using iptables

```bash
## we allow at max 5 new connections per minute

$IPTABLES -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
$IPTABLES -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP

## Limit the amount of connections on port 22 per remote-ip

$IPTABLES -A INPUT -p tcp --syn --dport 22 -m connlimit --connlimit-above 5 -j REJECT
```
Link:
```bash
https://github.com/bmaeser/iptables-boilerplate/issues/1#issuecomment-8935056
```
