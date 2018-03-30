title: Xshell添加隧道，浏览器设置代理
categories:
- 工具使用
---
## 一、问题描述
> 由于考虑安全因素和避免繁琐操作的原因，阿里云服务器上的端口没有对外开放。
> 但很多时候有在本地访问的需要。

## 二、解决方法
> Xshell添加隧道，浏览器设置代理

## 三、设置步骤

### Xshell添加隧道

> 1.先正确配置成能够登录到阿里云主机。  
> 2.点击隧道标签。
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-29/tunnel.png)</div>

> 3.点击添加按钮。  
> 4.配置规则。`类型（方向）`选择`Dynamic(SOCKET4/5)`;`监听端口`设置成端口号（比如6666），这个端口号必须和后面代理的端口号一致。
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-29/tunnel_rule.png)</div>

> 5.点击确定。这样隧道添加完成。
    
### 浏览器设置代理

> 1.下载代理插件，方便快捷切换代理。 这里借助Chrome浏览器的`Proxy SwitchySharp`插件。（360浏览器也有这个插件）
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-29/proxy_switchy.png)</div>

> 2.配置代理。
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-29/proxy_result.png)</div>

> 3.切换代理。这样你就可以访问阿里云主机的相关网页了。

<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-29/switch.png)</div>

ps：访问过程中xshell不能断开，否则也无法正常访问。





