---
title: "VPS搭建V2Ray"
date: 2017-10-25T21:52:13+08:00
lastmod: 2017-10-25T21:52:13+08:00
draft: false
tags: ["VPS", "V2Ray"]
categories: ["技术"]

weight: 10

# you can close something for this content if you open it in config.toml.
comment: false
# toc: false
# reward: false
# mathjax: false
comments: false
---
<!-- toc -->
[V2Ray](https://www.v2ray.com) 原生支持 Socks、HTTP、VMess 等协议。（推荐使用VMess）

- 在一个进程中可以配置不同的端口使用不同的协议进行通讯。
- 通过不同的传入和传出协议组合，灵活转换通讯格式。

上述协议均可使用 TLS、TCP、WebSocket、mKCP 传输方式进行传输。

客户端支持Windows(测试部分通过)、MacOS(测试通过)、iOS(测试通过)、Android。

<!-- more -->
## V2Ray 服务端安装
我是在[DigitalOcean VPS](https://m.do.co/c/00a39cb0d92f)上使用Ubuntu16.04 x64系统搭建V2Ray服务端，在VPS中输入一句命令即可完成V2Ray服务端的安装：

`bash <(curl -L -s https://install.direct/go.sh)`
此脚本会自动安装以下文件：

- `/usr/bin/v2ray/v2ray`：V2Ray 程序；
- `/etc/v2ray/config.json`：配置文件；

此脚本会配置自动运行脚本。自动运行脚本会在系统重启之后，自动运行 V2Ray。目前自动运行脚本只支持带有 Systemd 的系统，以及 Debian / Ubuntu 全系列。

运行脚本位于系统的以下位置：

- `/lib/systemd/system/v2ray.service`: Systemd
- `/etc/init.d/v2ray`: SysV

脚本运行完成后，你需要：

1. 编辑 `/etc/v2ray/config.json` 文件来配置你需要的代理方式；
2. 运行 `service v2ray start` 来启动 V2Ray 进程；
3. 之后可以使用 service v2ray start|stop|status|reload|restart|force-reload 控制 V2Ray 的运行。


## V2Ray服务端普通配置

V2Ray的普通配置服务端只需要修改`/etc/v2ray/config.json`文件中的inbound部分，其他请别动。

### TCP

服务端配置（使用VMess协议）

```json
{
  "inbound": {
    "port": 31666, //服务端接收客户端数据的端口，客户端与服务端必须一致
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "892fae30-1616-4e76-a53e-43e951f2d107", //改成你的UUID，重要
          "level": 1,  //客户端与服务端必须一致
          "alterId": 64 //客户端与服务端必须一致
        }
      ]
    },
    "streamSettings":{
      "network":"tcp" //传输方式，客户端与服务端必须一致
    }
  },
  ... //未列出其他配置
}
```

注意：UUID事关安全，请不要乱用。你可以到[https://www.uuidgenerator.net](https://www.uuidgenerator.net)在线生成

修改完`/etc/v2ray/config.json`后，使用命令`service v2ray restart`重启V2Ray使配置生效。

### WebSocket

服务端配置（使用VMess协议）只需要把上面TCP配置的`"network":"tcp"`改为`"network":"ws"`，注意客户端也需要保持一致。

## V2Ray第三方客户端安装

v2ray官方提供了客户端程序但都不支持GUI操作，第三方客户端比较少，只能说可以凑合用。集合下载地址：[https://github.com/fun90/KXSW](https://github.com/fun90/KXSW)，改进过的Mac客户端[V2RayX](https://github.com/fun90/V2RayX/releases)（GUI更好）、[SwiftV2ray](https://github.com/fun90/SwiftV2ray)（配置是编辑文本，更灵活，提供了升级V2Ray内核功能）

### V2RayW (Windows)

以V2RayW为例，注意：测试时发现windows10下的Inbound protocol不能使用socket，只能选择http。

![](http://7xnocv.com1.z0.glb.clouddn.com/20171026162256_H5kxrQ_Screenshot.jpeg)

填写好服务器信息，右键选择V2Ray Rules，至此全部安装配置完成。

如果使用V2RayN可以参考上面的截图进行配置，不过还需要额外一步的Windows设置。(虽然已停止维护，V2RayN的界面操作比V2RayW更加完善)

![](http://7xnocv.com1.z0.glb.clouddn.com/20171026155220_PWVkfI_2017-10-26 15.43.04.gif)

### V2RayX (Mac)

界面与V2RayW几乎一致

### Shadowrocket (iOS)
配置非常简单
![](http://7xnocv.com1.z0.glb.clouddn.com/20171026165517_ET4b6R_IMG_3630.jpeg)

其他客户端可以查看[V2Ray官网介绍](https://www.v2ray.com/chapter_01/3rd_party.html)

## V2Ray高级配置

### TLS

可以查看[V2Ray配置指南中的TLS一节](https://toutyrater.github.io/advanced/tls.html)，非常详细。

/etc/v2ray/config.json

```json
{
  "inbound": {
    "port": 443, //推荐使用443，别的也可以
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "892fae30-1616-4e76-a53e-43e951f2d107",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings":{
      "network": "tcp",
      "security": "tls",
      "tlsSettings": {
        "certificates": [
          {
            "certificateFile": "/etc/v2ray/v2ray.crt",
            "keyFile": "/etc/v2ray/v2ray.key"
          }
        ]
      }
    }
  },
  ... //未列出其他配置
}
```

**注意事项**

服务端配置了TLS后，在Windows下使用V2RayW未能成功，可能是不支持。V2RayN是可以的。

![](http://7xnocv.com1.z0.glb.clouddn.com/20171026171206_rxsO6A_Screenshot.jpeg)

### 多用户

多用户只需要在服务端的config.json中修改inboundDetour，tcp配置例子如下：

```json
{
  "inbound": {
    ...
  },
  "inboundDetour":[
    {
      "port": 你的端口(不同于inbound), 
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "你的UUID(不同于inbound)",
            "level": 1,
            "alterId": 64
          }
        ]
      },
      "streamSettings":{
        "network": "tcp"
      }
    }
  ],
  "outbound": {
    ...
  },
  "outboundDetour": [
    ...
  ],
  "routing": {
    ...
  }
}
```



## 参考

[V2Ray官网](https://www.v2ray.com/)

[V2Ray配置指南](https://toutyrater.github.io)