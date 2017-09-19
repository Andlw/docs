一、安装docker并启动
  # yum install docker -y
  # systemctl start docker

二、下载docker registry 镜像
  # docker search registry
  # docker pull registry

三、启动registry仓库
  # docker run --rm -d -p 5000:5000 --privileged=true -v \     /data/docker/data/registry:/var/lib/registry -v \ /data/openresty/nginx/conf/:/root/certs  -e  \ REGISTRY_HTTP_TLS_CERTIFICATE=/root/certs/htres_cn.pem -e \ REGISTRY_HTTP_TLS_KEY=/root/certs/htres_cn.key registry

四、测试能不能上传和下载镜像
  # docker pull busybox
  # docker tag busybox registry.htres.cn:5000/busybox
  # docker push registry.htres.cn:5000/busybox
  # docker pull registry.htres.cn:5000/busybox

五、增加nginx反向代理
  #、、、、、、、、、、、、、、、
