# nginx_auto_ssl

## 1. 获取证书

```sh
# 拉取镜像
sudo docker pull neilpang/acme.sh

# 运行
# https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker

# 创建目录
mkdir ./acme
mkdir ./acme/domains
mkdir ./acme/domains/staneee.com
mkdir ./acme/domains/cyanstream.com

# 启动容器
sudo docker run --network=host \
  --restart=always -d  \
  -v "$(pwd)/acme":/acme.sh  \
  --name acme neilpang/acme.sh daemon
  
  
# 注册  zerossl
sudo docker exec acme --register-account  -m 邮箱地址 --server zerossl


# 注意，申请域名证书需要配置相关的参数，请参照 https://github.com/acmesh-official/acme.sh/wiki 指定key和val,此处我使用dnspod

# 申请泛域名证书 staneee.com
sudo docker exec \
-e DP_Id= \
-e DP_Key= \
acme --issue --dns dns_dp \
-d *.staneee.com \
--cert-file	       /acme.sh/domains/staneee.com/cert.cert \
--key-file         /acme.sh/domains/staneee.com/key.key \
--ca-file          /acme.sh/domains/staneee.com/ca.cer \
--fullchain-file   /acme.sh/domains/staneee.com/fullchain.pem

# 申请泛域名证书 cyanstream.com
sudo docker exec \
-e DP_Id= \
-e DP_Key= \
acme --issue --dns dns_dp \
-d *.cyanstream.com \
--cert-file	       /acme.sh/domains/cyanstream.com/cert.cert \
--key-file         /acme.sh/domains/cyanstream.com/key.key \
--ca-file          /acme.sh/domains/cyanstream.com/ca.cer \
--fullchain-file   /acme.sh/domains/cyanstream.com/fullchain.pem

```


## 2. 编辑nginx配置
请自行编辑 `conf.d` 目录中nginx配置文件

## 3. 启动nginx容器
```
# 拉取nginx镜像
sudo docker pull nginx:1.17.10

# 启动容器
sudo docker run --network=host \
  --restart=always -d \
  -v "$(pwd)/acme/domains":/domains \
  -v "$(pwd)/www":/var/www \
  -v "$(pwd)/conf.d":/etc/nginx/conf.d \
  --name nginx nginx:1.17.10
```
