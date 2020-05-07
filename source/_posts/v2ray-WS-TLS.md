---
title: v2ray+WS+TLS
date: 2020-05-06 16:41:48
tags:
- software
---
## 安装
### 1.1 服务端脚本安装
```bash
wget https://install.direct/go.sh && bash go.sh
```
### 1.2 安装Nginx
```bash
apt install nginx
```
## 注册域名
[https://sg.godaddy.com/](https://sg.godaddy.com/)

## CDN
[https://www.cloudflare.com/](https://www.cloudflare.com/)

### 3.1 cloudflare托管域名

### 3.2 在cloudflare下载TLS证书

## WebSocket+TLS+Web

### 2.1 WebSocket+TLS+Web

服务端配置
```bash
{
  "log": {
    "loglevel": "warning",
    "access": "/dev/null",
    "error": "/dev/null"
  },
  "inbounds": [{
    "listen":"127.0.0.1",
    "port": 10000,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "XXXX",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings":{
      "network":"ws",
      "wsSettings":{
        "path": "/bannedbook"
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {},
    "tag": "allowed"
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```
客户端配置
```bash
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "mydomain.com",
            "port": 443,
            "users": [
              {
                "id": "XXXX",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/bannedbook"
        }
      }
    }
  ]
}

```

Nginx配置
```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile  655350;
events {
	use epoll;
	worker_connections 65536;
}

http {
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	include /etc/nginx/mime.types;
	default_type application/octet-stream;
	access_log /var/log/nginx-access.log;
	error_log /var/log/nginx-error.log;

	gzip on;
  #----------------WS+SSL----------------------#
	server {
		listen  443 ssl;
		ssl on;
		ssl_certificate       /etc/v2ray/v2ray.crt;
		ssl_certificate_key   /etc/v2ray/v2ray.key;
		ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
		ssl_ciphers           HIGH:!aNULL:!MD5;
		server_name           mydomain.com;
        	location /bannedbook {
        		proxy_redirect off;
        		proxy_pass http://127.0.0.1:10000;
        		proxy_http_version 1.1;
        		proxy_set_header Upgrade $http_upgrade;
        		proxy_set_header Connection "upgrade";
        		proxy_set_header Host $http_host;

        		# Show realip in v2ray access.log
        		proxy_set_header X-Real-IP $remote_addr;
        		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        	}
	}
  #----------------WS--------------------------#
	server {
		listen 80 default_server;
		listen [::]:80 default_server;
		root /var/www/html;
		index index.html index.htm index.nginx-debian.html;
		server_name _;

		location / {
			try_files $uri $uri/ =404;
		}
		
    		location /bannedbook { 
	    		proxy_redirect off;
	    		proxy_pass http://127.0.0.1:10000; 
	    		proxy_http_version 1.1;
	    		proxy_set_header Upgrade $http_upgrade;
	    		proxy_set_header Connection "upgrade";
	    		proxy_set_header Host $http_host;
	
	    		proxy_set_header X-Real-IP $remote_addr;
	    		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;	
    		}
	}
}

```
