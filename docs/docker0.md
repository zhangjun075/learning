# 修改docker0网桥

Docker默认使用的网桥 docker0 的网段是 172.17.0.1。如果想修改按照如下步骤：

**第一步 删除原有配置**
```
sudo service docker stop
sudo ip link set dev docker0 down
sudo brctl delbr docker0
sudo iptables -t nat -F POSTROUTING
```

**第二步 创建新的网桥**
```
sudo brctl addbr docker0
sudo ip addr add 192.168.200.1/24 dev docker0
sudo ip link set dev docker0 up
```

**第三步 配置Docker的文件**

注意： 这里是 增加下面的配置
```
vi /etc/docker/daemon.json
{
    ...
    "bip": "192.168.200.1/24",
    ...
}
```

docker的配置现在都是用daemon.json了，不需要去设置DOCKER_OPTS, 所以网络上很多的资料早已过时。

**第四步 重启主机**
```
sudo reboot
```