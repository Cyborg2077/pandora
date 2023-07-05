# 写在最前
- 思来想去还是搭建一个本地版罢（不然总是要科学上网），刚好可以放我虚拟机里跑，本教程基于[Pengzhile/pandora](https://github.com/pengzhile/pandora) 项目
- 前置要求：一台搭载了Contos7的虚拟机
 

# 安装Docker
- 关于Docker的安装和使用，详情可以参考我这篇文章 https://cyborg2077.github.io/2022/12/21/Docker/

## 卸载(可选)
 
- 如果之前安装过旧版本的Docker，可以使用下面命令卸载
``` BASH
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

## 安装Docker

- 首先先安装yum工具
``` BASH
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```

- 然后更新本地镜像源
``` BASH
# 设置Docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```

- 然后安装社区版Docker
``` BASH
yum install -y docker-ce
```

## 启动Docker

- Docker应用需要用到各种端口，挨个修改防火墙设置很麻烦，所以这里建议直接关闭防火墙
``` BASH
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```

- 通过命令启动Docker
``` BASH
# 启动docker服务（只执行此操作即可）
systemctl start docker 

# 停止docker服务（无需执行）
systemctl stop docker

# 重启docker服务（无需执行）
systemctl restart docker
```

- 然后输入命令，查看docker版本，如果可以看到版本号，则安装成功
``` BASH
docker -v
``` 

- 设置开机自启动
``` BSAH
systemctl enable docker
```


# 拉取镜像并启动容器
1. 拉取镜像
``` BASH
docker pull pengzhile/pandora
```
2. 启动容器，这里的9527可以换为你喜欢的端口（确保它没有被占用），由于docker已经设置为开机自启了，这里再设置 `--restart=always`可以保证该容器随docker启动而启动，从而实现你只需要将虚拟机开机，即可访问chatGPT了
``` BASH
docker run -e PANDORA_CLOUD=cloud -e PANDORA_SERVER=0.0.0.0:9527 -p 9527:9527 -d --restart=always pengzhile/pandora
```
3. 查看虚拟机ip，在输出结果中，找到带有 inet 地址的行，后面的一串数字就是本机的 IP 地址，例如：`192.168.101.128`
``` BASH
ifconfig
```
4. 在浏览器中输入`虚拟机ip:9527`，即可看到登录界面，点击最下方`Continue with Access Token`输入token即可登录使用，无需科学上网，定期刷新token即可
![](https://s1.ax1x.com/2023/06/14/pCnzCWV.png)
5. 关于token的获取，如果你有正常的openAI账号，访问https://chat.openai.com/api/auth/session 可以拿到token
    - 另外一种token的获取方式：https://ai.fakeopen.com/auth

# 其他设备访问
- 懒是第一生产力，我现在想在手机或者任何我的电子设备上访问ChatGPT，该怎么办呢？开放主机的一个端口供其他设备就好了（当然你的设备需要和主机在同一局域网内）

## Windows防火墙设置
- 控制面板->系统与安全->Windows防火墙->高级设置->入站规则->新建规则
![](https://s1.ax1x.com/2023/06/20/pC8B6Ve.png)
- `协议和端口`，选择`TCP`，`特定本地端口`，填写一个你喜欢的端口即可，我这里还是9527
![](https://s1.ax1x.com/2023/06/20/pC8BRPA.png)
- 然后一路Next就好了，都用默认的配置，最后的名称你可以起一个你喜欢的名称，我这里是ChatGPT

## 虚拟机端口转发
- 由于虚拟机采用的是NAT联网方式，我们点击菜单->编辑->虚拟网络编辑器->更改设置，选择VMnet8网络
![](https://s1.ax1x.com/2023/06/20/pC8DPIJ.png)
- 点击NAT设置添加端口转发规则，主机端口选择我们上一步开放的9527端口，虚拟机地址填写我们Docker容器中ChatGPT的启动端口
![](https://s1.ax1x.com/2023/06/20/pC8DmqO.png)
- 最后CMD查看本机ip，其他设备只要跟电脑在同一个局域网内，访问`ip:端口`即可正常使用
![](https://s1.ax1x.com/2023/06/20/pC8D0ij.jpg)
