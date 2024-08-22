---
title: "Docker总结"
aliases: 
tags: [Docker]
date: 2024-08-22
time: 15:00
---

## Docker宿主机磁盘满了处理方法

### 清理Docker资源

1. 删除未使用的容器
```sh
  docker container prune
  ```
2. 删除未使用的镜像
```sh
  docker image prune
  ```
 3. 删除未使用的卷
 ```sh
  docker volume prune
  ```
 4. 删除未使用的网络
 ```sh
  docker network prune
  ``` 
5. 全量清理
  - 清理所有未使用的资源
```sh
  docker system prune
  ```
  - 清理未使用的镜像和卷
```sh
  docker system prune -a --volumes
  ```
### 增加Docker主机的存储空间

  - 增加磁盘空间
  - 重新分区，后迁移docker存储目录

### 迁移Docker储存目录

  Docker存储目录一般存放在  `/var/lib/docker` 目录

  1. 停止Docker服务
```sh
  sudo systemctl stop docker
  ```
  2. 移动存储目录
```sh
  sudo rsync -aP /var/lib/docker /new/docker/dir
  ```
  3. 更新Docker配置
    编辑Docker配置文件 `/etc/docker/daemon.json`， 添加或更新 `data-root` 选项。
```json
  {
    "data-root": "/new/docker/dir"
  }
  ```
  4. 重启Docker服务
```sh
  sudo systemctl start docker
  ```