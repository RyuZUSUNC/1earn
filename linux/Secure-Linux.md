# Secure-Linux👨🏻‍💻
`linux安全加固+应急响应的tips`
[TOC]

## shell
### 会话
```bash
who  #查看当前登录用户
w    #查看登录用户行为
last #查看登陆用户历史
pkill -u linfengfeiye #直接剔除用户

ps -ef| grep pts/0  #得到用户登录相应的进程号pid后执行
kill -9 pid #安全剔除用户
```

## 系统管理
### 进程
```bash
ulimit -u 20 #临时性允许用户最多创建 20 个进程,预防类似fork炸弹
vim /etc/security/limits.conf
    user1 - nproc 20       #退出后重新登录，就会发现最大进程数已经更改为 20 了
```

---

## net
### 端口🕵🏻‍
```bash
lsof -i:22  #查22端口

ss -tnlp
ss -tnlp | grep ssh
ss -tnlp | grep ":22"

etstat -tnlp
netstat -tnlp | grep ssh

nmap -sV -p 22 localhost
```

### Firewall👮🏻‍
```bash
firewall-cmd --permanent --zone=public --remove-service=ssh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=internal --add-source=1.1.1.1
firewall-cmd --reload
firewall-cmd --list-services  #查看防火墙设置
```

在上面的配置中，如果有人尝试从 1.1.1.1 去 ssh，这个请求将会成功，因为这个源区域（internal）被首先应用，并且它允许 ssh 访问。

如果有人尝试从其它的地址，如 2.2.2.2，去访问 ssh，它不是这个源区域的，因为和这个源区域不匹配。因此，这个请求被直接转到接口区域（public），它没有显式处理 ssh，因为，public 的目标是 default，这个请求被传递到默认动作，它将被拒绝。

如果 1.1.1.1 尝试进行 http 访问会怎样？源区域（internal）不允许它，但是，目标是 default，因此，请求将传递到接口区域（public），它被允许访问。

现在，让我们假设有人从 3.3.3.3 拖你的网站。要限制从那个 IP 的访问，简单地增加它到预定义的 drop 区域，正如其名，它将丢弃所有的连接：
```bash
firewall-cmd --permanent --zone=drop --add-source=3.3.3.3
firewall-cmd --reload
```
下一次 3.3.3.3 尝试去访问你的网站，firewalld 将转发请求到源区域（drop）。因为目标是 DROP，请求将被拒绝，并且它不会被转发到接口区域（public）。


