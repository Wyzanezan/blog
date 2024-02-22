---
title: 通过cloudflare的tunnel技术实现内网穿透
date: 2024-02-02 14:25:31
tags: 网络
categories: 网络
---

cloudflare是一家提供一系列面向网站、应用程序和API的互联网安全性和性能增强服务的公司，它提供的服务主要包括内容分发网络（CDN）、分布式域名系统（DNS）服务、防御分布式拒绝服务（DDoS）攻击的解决方案和安全防护措施等。

<!--more-->

我们在cloudflare上购买域名后，可以使用它的 tunnel 技术实现内网穿透，将我们的域名绑定到一个本地服务上。

下面分享一下操作步骤。

操作步骤（以windows为例）：

1. 安装 cloudflared（在官网或者github下载并安装，https://github.com/cloudflare/cloudflared/releases）

2. 执行以下命令，登录到cloudflare进行授权

   cloudflared tunnel login
   执行后会弹出一个页面，进行授权即可。授权完成后，会在 .cloudflare 目录下生成一个 cert.pem 文件

3. 创建隧道 tunnel

   cloudflared tunnel create <隧道名字>
   创建完成后，会在 .cloudflare 目录中生成一个 uuid.json 的文件

4. 把域名指向对应的隧道

   cloudflared tunnel route dns <隧道名字> <域名>
   完成后，cloudflare 会自动添加一条 CNAME 记录到对应的域名

5. 配置 cloudflared

   编辑文件 .\cloudflared\config.yml，添加如下内容：

   ```yml
   tunnel: local
   credentials-file: C:\Users\10579\.cloudflared\e9e54272-7145-485d-af9f-d620558d5c7a.json
   
   ingress:
     - hostname: blog.wangzan.net
       service: http://localhost:8080
     - service: http_status:404
   ```

   配置完成后，可以测试一下文件是否正确：

   cloudflared tunnel --config .\cloudflared\config.yml ingress validate

   测试规则是否命中：

   cloufdlared tunnel --config .\cloudflared\config.yml ingress rule http://wangzan.net

6. 测试运行

   .\cloudflared.exe --loglevel debug --transport-loglevel warn --config C:\Users\wyzane\.cloudflared\config.yml tunnel run <隧道UUID>



以上就是通过cloudflare配置内网穿透的步骤。



参考：

https://blog.cloudflare.com/zh-cn/elevate-load-balancing-with-private-ips-and-cloudflare-tunnels-a-secure-path-to-efficient-traffic-distribution-zh-cn/

https://bra.live/setup-home-server-with-cloudflare-tunnel/

https://github.com/anderspitman/awesome-tunneling



