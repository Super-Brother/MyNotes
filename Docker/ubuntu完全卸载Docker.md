ubuntu 完全干净的卸载docker

1. 删除某软件,及其安装时自动安装的所有包

```
sudo apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc
```

2. 查看删除docker其他有没有没有卸载干净的包

```
dpkg -l | grep docker
```

3. 卸载相应的包

```
sudo apt-get autoremove docker-ce-*
```

4. 删除docker的相关配置&目录

```
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /var/lib/docker
```

5. 确定docker卸载完毕

```
docker --version
```

