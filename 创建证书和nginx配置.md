# docker + acme.sh + nginx 实现自动且免费的泛域名ssl证书

---

## 配置acme.sh 

#### 拉取镜像

> 参考 https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker

```shell
sudo docker pull neilpang/acme.sh
```



#### 创建目录

> 在当前目录创建一个acme目录，acme目录中domains根据域名创建对应的目录

```shell
mkdir ./acme
mkdir ./acme/domains
mkdir ./acme/domains/staneee.com
mkdir ./acme/domains/cyanstream.com
```

#### 启动容器

> 运行acme，并映射对应的目录

```shell
sudo docker run --network=host \
  --restart=always -d  \
  -v "$(pwd)/acme":/acme.sh  \
  --name acme neilpang/acme.sh daemon
```

#### 申请证书

> 申请域名证书到对应的目录

```shell
# 申请泛域名证书 staneee.com
sudo docker exec \
 -e DP_Id= \
 -e DP_Key= \
 acme --issue --dns dns_dp \
 -d *.staneee.com \
 --cert-file	    /acme.sh/domains/staneee.com/cert.cert \
 --key-file         /acme.sh/domains/staneee.com/key.key \
 --ca-file          /acme.sh/domains/staneee.com/ca.cer \
 --fullchain-file   /acme.sh/domains/staneee.com/fullchain.pem

# 申请泛域名证书 cyanstream.com
sudo docker exec \
 -e DP_Id= \
 -e DP_Key= \
 acme --issue --dns dns_dp \
 -d *.cyanstream.com \
 --cert-file	    /acme.sh/domains/cyanstream.com/cert.cert \
 --key-file         /acme.sh/domains/cyanstream.com/key.key \
 --ca-file          /acme.sh/domains/cyanstream.com/ca.cer \
 --fullchain-file   /acme.sh/domains/cyanstream.com/fullchain.pem
```

---

## 配置nginx 

#### 拉取镜像

```shell
sudo docker pull nginx:1.17.10
```

#### 创建目录

> 创建nginx的配置目录和静态资源目录
>
> config.d 为配置目录，将nginx的相关配置都放到这个目录中
>
> www 为静态资源目录

```shell
mkdir ./config.d
mkdir ./www
```

#### 运行nginx

> 运行nginx，将对应的路径映射到容器

```shell
sudo docker run --network=host \
  --restart=always -d \
  -v "$(pwd)/acme/domains":/domains \
  -v "$(pwd)/www":/var/www \
  -v "$(pwd)/conf.d":/etc/nginx/conf.d \
  --name nginx nginx:1.17.10
```





---

## nginx配置实例

```nginx
# ------------------------- frp -------------------------
# *.dev.staneee.com
server {
    listen 443 ssl;
    server_name *.dev.staneee.com;
    ssl_certificate /domains/staneee.com/fullchain.pem;
    ssl_certificate_key /domains/staneee.com/key.key;

    access_log off;
    location / {
        proxy_pass http://127.0.0.1:7778/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header Connection keep-alive;
        proxy_set_header real-ip $remote_addr;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 7200;
        port_in_redirect off;
    }
}

# *.riven.staneee.com
server {
    listen 80;
    server_name *.riven.staneee.com;

    location / {
        proxy_pass http://127.0.0.1:7778/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header Connection keep-alive;
        proxy_set_header real-ip $remote_addr;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 7200;
        port_in_redirect off;
    }
}

# ------------------------- main -------------------------

server {
    listen 80;
    server_name staneee.com;

    # redict to https www domain
    return 301 https://www.staneee.com$request_uri;
}

server {
    listen 80;
    server_name www.staneee.com;
    
    # redict to https www domain
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name www.staneee.com;
    ssl_certificate /domains/staneee.com/fullchain.pem;
    ssl_certificate_key /domains/staneee.com/key.key;

    access_log off;
    location / {
        root /var/www/staneeecom;
        index index.html;
        port_in_redirect off;
    }
}
```























