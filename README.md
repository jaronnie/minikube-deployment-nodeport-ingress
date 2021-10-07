# minikube-deployment-nodeport-ingress

## 启动 minikube 集群

### 安装 minikube

> 建议安装 v1.20.0 verison

下载地址：https://github.com/kubernetes/minikube/releases/tag/v1.20.0

### 安装 virtualbox

> 建议安装 v6.1.26 version

下载地址：https://www.virtualbox.org/wiki/Downloads

### 启动 minikube 单节点

> 需要稍等几分钟
>
> 不建议指定阿里云的源，一堆坑，建议找一个好一点的 vpn 进行下面的命令

```shell
minikube start -driver=virtualbox
```

### 开启 dashboard

> 验证完毕后自动跳转至 dashboard 页面

```shell
minikube dashboard
```

## 开启 ingress

> 这一步会有很多问题

```shell
minikube addons enable ingress
```

一直卡在这个地方是因为有一个镜像一直拉不下来

先看看哪个镜像xx拉不下来

按步骤执行下面的命令

```shell
docker pull xx
docker tag oldTag newTag
minikube image load newTag
minikube delete
minikube start --driver=virtualbox --cache-images
minikube addons enable ingress
```

如果还有问题的话，进入 dashboard 中的 ingress-nginx 命令空间，删除对应的 deployment 中 image 后的 sha 值即可！

## 运行服务

```shell
docker build -t "gocloudcoder.com:kube-go-app:v1" .
kubectl apply -f deployment.yaml
kubectl apply -f service-nodeport.yaml
kubectl apply -f service-ingress1.yaml
kubectl apply -f service-ingress2.yaml
kubectl apply -f service-ingress3.yaml
```

## 配置 nfsmasq 泛型解析域名

> 即 *.kube.go-app.com 定向到 192.168.99.158(minikube ip)

```shell
brew install nfsmasq
sudo brew services start dnsmasq
sudo echo "address=/kube.go-app.com/192.168.99.158" >> /usr/local/etc/dnsmasq.conf
sudo mkdir /etc/reslover
sudo echo "nameserver 127.0.0.1" >> /etc/reslover/kube.go-app.com
sudo echo "nameserver 192.168.99.158" >> /etc/reslover/kube.go-app.com
sudo brew services restart dnsmasq
```

```shell
$ curl service1.kube.go-app.com/health
<h1>Hello World</h1>%

$ curl service2.kube.go-app.com/health
<h1>Hello World</h1>%

$ curl service3.kube.go-app.com/health
<h1>Hello World</h1>%

$ curl service4.kube.go-app.com/health
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
