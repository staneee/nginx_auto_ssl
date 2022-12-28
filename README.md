# nginx_auto_ssl

## 1. 获取证书

```sh
# 拉取镜像
sudo docker pull neilpang/acme.sh:3.0.5

# 运行
# https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker

# 创建目录
mkdir ./acme
mkdir ./acme/domains
mkdir ./acme/domains/staneee.com

# 启动容器
sudo docker run --network=host \
  --restart=always -d  \
  -v "$(pwd)/acme":/acme.sh  \
  --name acme neilpang/acme.sh:3.0.5 daemon
  
  
# 注册  zerossl
sudo docker exec acme --register-account  -m 邮箱地址 --server zerossl


# 注意，申请域名证书需要配置相关的参数，请参照 https://github.com/acmesh-official/acme.sh/wiki 指定key和val,此处我使用dnspod

# 申请泛域名证书 staneee.com
## 第一次申请
sudo docker exec \
-e DP_Id= \
-e DP_Key= \
acme --issue \
--dns dns_dp \
-d *.staneee.com \
--cert-file	       /acme.sh/domains/staneee.com/cert.cert \
--key-file         /acme.sh/domains/staneee.com/key.key \
--ca-file          /acme.sh/domains/staneee.com/ca.cer \
--fullchain-file   /acme.sh/domains/staneee.com/fullchain.pem

## 第二次申请，手动更新
sudo docker exec \
acme --issue \
--dns dns_dp \
-d *.staneee.com \
--cert-file	       /acme.sh/domains/staneee.com/cert.cert \
--key-file         /acme.sh/domains/staneee.com/key.key \
--ca-file          /acme.sh/domains/staneee.com/ca.cer \
--fullchain-file   /acme.sh/domains/staneee.com/fullchain.pem \
--force

# 申请泛域名证书 prod.staneee.com
## 第一次申请
sudo docker exec \
-e DP_Id= \
-e DP_Key= \
acme --issue  \
--dns dns_dp \
-d *.prod.staneee.com \
--cert-file	       /acme.sh/domains/prod.staneee.com/cert.cert \
--key-file         /acme.sh/domains/prod.staneee.com/key.key \
--ca-file          /acme.sh/domains/prod.staneee.com/ca.cer \
--fullchain-file   /acme.sh/domains/prod.staneee.com/fullchain.pem

## 第二次申请，手动更新
sudo docker exec \
acme --issue  \
--dns dns_dp \
-d *.prod.staneee.com \
--cert-file	       /acme.sh/domains/prod.staneee.com/cert.cert \
--key-file         /acme.sh/domains/prod.staneee.com/key.key \
--ca-file          /acme.sh/domains/prod.staneee.com/ca.cer \
--fullchain-file   /acme.sh/domains/prod.staneee.com/fullchain.pem \
--force

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
