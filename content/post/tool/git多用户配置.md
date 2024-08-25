---
title: "Git多用户配置"
aliases: 
tags: [Tool]
date: 2024-08-24
time: 10:57
---

## 用户分类
  - git 用户分类  global (全局) local（仓库）级别
  - 优先级 local > global
## 全局配置

- 查看配置
```bash
git config --list
``` 
- 清空全局 user.name email

```bash
git config --global --unset user.name
git config --global --unset user.email
```
- 设置全局 user.name email

```bash
git config --global user.name "hobocat"
git config --global user.email "hobocat@126.com"
```

## 本地配置

  配置单个仓库的用户信息

  - 清空 user.name email

    ```bash
    git config --unset user.name
    git config --unset user.email
    ```
  - 设置 user.name email

    ```bash
    git config user.name "hobocat"
    git config user.email "hobocat@126.com"
    ```

## SSH 多用户配置

#### 生成秘钥

- 生成命令
```bash
ssh-keygen -t {算法名} -f {文件路劲及文件名} -C "解释信息，一般是邮箱"
```

- 生成gitee仓库的SSH
```bash
  ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitee -C "hobocat@126.com"
```
- 生成github仓库的SSH

```bash
  ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "hobocat@126.com"
```
- 生成公司仓库的SSH

```bash
  ssh-keygen -t rsa -f ~/.ssh/id_rsa.company -C "hobocat@126.com"
```
### 将 ssh-key 分别添加到 ssh-agent 信任列
```bash
  ssh-agent bash
  ssh-add ~/.ssh/id_rsa.gitee
  ssh-add ~/.ssh/id_rsa.github
  ssh-add ~/.ssh/id_rsa.company
```
### 添加公钥到自己的 git 账户中
| 使用命令，copy公钥，到 git 账户中粘贴即可。或者打开文件复制，带 pub 的文件
```bash
  pbcopy < ~/.ssh/id_rsa.gitee.pub
```
#### git平台添加SSH

- [gitee方式](https://help.gitee.com/base/account/SSH%E5%85%AC%E9%92%A5%E8%AE%BE%E7%BD%AE)


#### 配置多平台
| 在生成密钥的.ssh 目录下，新建一个config文件，然后配置不同的仓库，
```yml
#Default gitHub user Self
Host github.com
HostName github.com
User HObOCat #默认就是git，可以不写
IdentityFile ~/.ssh/id_rsa.github
	
# gitee的配置
host gitee.com  # 别名,最好别改
Hostname gitee.com #要连接的服务器
User HObOCat #用户名
#密钥文件的地址，注意是私钥
IdentityFile ~/.ssh/id_rsa_gitee

#Add gitLab user 
Host xxxx
HostName xxxx
User HObOCat
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa.company

```

