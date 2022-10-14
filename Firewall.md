## Firewall 

### 端口开放
```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
```

### 证书的生成

```bash
mkdir /usr/local/nginx/conf/ssl
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /usr/local/nginx/conf/ssl/api.vanbal.test.key -out /usr/local/nginx/conf/ssl/api.vanbal.test.crt
```
根据提示操作 common name 为域名

### nginx 配置
```bash
server {

    listen 80;
    listen 443 ssl;
    server_name ~^(?<domain>[^.]+)\.vanbal.test$;
    root /usr/share/nginx/html/test/$domain;

    ssl_certificate /etc/nginx/ssl/test.crt;
    ssl_certificate_key /etc/nginx/ssl/test.key;

    index index.html;

    charset utf-8;

        location / {
                index index.html;
        }
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log /var/log/nginx/$domain-test-access.log combined;
    error_log  /var/log/nginx/$domain-test-error.log error;

    sendfile off;

}
```


### DNS 正向解释处理
```bash
# vim  /etc/named.rfc1912.zones

zone "vanbal.dev" IN {
        type master;
        file "vanbal.dev.zone";
        allow-update { none; };
};

# vim /var/named/vanbal.dev.zone

$TTL 1D
@       IN SOA @        admin.vanbal.dev. (
                                        20220826        ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.vanbal.dev.    # 必需
        MX  5   mail.vanbal.dev.  # 必需
        A       10.10.10.201

ns     A   10.10.10.201   # 必需
mail   A   10.10.10.201  # 必需

db      A   10.10.10.151
redis   A   10.10.10.151

*.api A  10.10.10.101
*   A   10.10.10.201

```
#### 开启防火墙
firewall-cmd --add-service=dns --permanent
firewall-cmd --zone=public --add-port=53/tcp --permanent
firewall-cmd --reload

