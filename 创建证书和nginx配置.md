## 证书

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





## nginx 
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



























