#### 环境：
云主机配置（购买的vultr）：
Ubuntu 16.10 (GNU/Linux 4.8.0-30-generic x86_64)

#### navicat for mysql连接配置:

![](http://www.hiqiuyi.cn/Imgs/solution/ssh.jpg)

![](http://www.hiqiuyi.cn/Imgs/solution/%E5%B8%B8%E8%A7%84.jpg)

#### 问题描述：
用navicat for mysql通过SSH连接云主机时，出现如下问题：
>问题1：SSH Tunnel:Server does not support diffie-hellman-group1-sha1

>问题2：SSH Tunel:The negotiation of encryption algorthm is failed

#### 解决方法：

打开文件sshd_config：

```bash
vi /etc/ssh/sshd_config
```

在文件最后添加：
```bash
KexAlgorithms diffie-hellman-group1-sha1,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1

Ciphers 3des-cbc,blowfish-cbc,aes128-cbc,aes128-ctr,aes256-ctr
```

效果：
![](http://www.hiqiuyi.cn/Imgs/solution/sshd_config.jpg)

生成密钥︰
```bash
ssh-keygen -A
```

重启ssh服务：
```bash
service ssh restart
```

#### 参考文档：
<u><a href="https://www.veeam.com/kb1890">Не удается создать ssh tunnel</a></u>

<u><a href="https://www.veeam.com/kb1890">The negotiation of encryption algorithm is failed</a></u>