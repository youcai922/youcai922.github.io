#### 更新容器脚本

```sh
#容器名称
image=eem-oil/cet-oil-field-service
echo 更新 $image 镜像服务
containerId=`docker ps | grep "$image" | awk '{print $1}'`
imageName=`docker ps | grep "$image" | awk '{print $2}'`
echo 容器id: $containerId
echo 镜像名称: $imageName
docker stop $containerId
echo 停止容器成功
docker rm $containerId
echo 删除容器成功
#此处-m 1表示获取第一行数据
imageId=`docker images | grep -m 1 "$image" | awk '{print $3}'`
echo 镜像id: $imageId
docker rmi $imageId
echo 删除镜像成功
docker-compose up -d
echo 更新$image镜像服务成功
```
