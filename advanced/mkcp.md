# mKCP

V2Ray 引入了 [KCP](https://github.com/skywind3000/kcp) 传输协议，并且做了一些不同的优化，称为 mKCP，相对于常规的 TCP 来说，mKCP 在某些网络环境下具有更大的优势，但是 mKCP 有一个很明显的缺点就是会比 TCP 耗费更多的流量，所以请酌情使用。要了解的一点是，mKCP 与 KCPTUN 同样是 KCP 协议，但两者并不兼容。

## 配置

mKCP 的配置比较简单，只需在服务器的 inbound 和 客户端的 outbound 添加一个 streamSettings 并设置成 mkcp 即可。

### 服务器配置

```javascript
{
  "inbound": {
    "port": 16823,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "mkcp", //此处的 mkcp 也可写成 kcp，两种写法是起同样的效果
      "kcpSettings": {
        "mtu": 1350,
        "tti": 20,
        "uplinkCapacity": 5,
        "downlinkCapacity": 100,
        "congestion": true,
        "readBufferSize": 1,
        "writeBufferSize": 1,
        "header": {
          "type": "none"
        }
      }
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  }
}
```

### 客户端配置

```javascript
{
  "inbound": {
    "port": 1080,
    "protocol": "socks",
    "domainOverride": ["tls","http"],
    "settings": {
      "auth": "noauth"
    }
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "serveraddr.com",
          "port": 16823,
          "users": [
            {
              "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
              "alterId": 64
            }
          ]
        }
      ]
    },
    "streamSettings": {
      "network": "mkcp",
      "kcpSettings": {
        "mtu": 1350,
        "tti": 20,
        "uplinkCapacity": 5,
        "downlinkCapacity": 100,
        "congestion": true,
        "readBufferSize": 1,
        "writeBufferSize": 1,
        "header": {
          "type": "none"
        }
      }
    }
  }
}
```

### 说明

在上面的配置当中，与之前相比主要的变化在于多了一个 streamSettings，包含有不少参数：
* `network`: 网络的选择，要像上面的配置写成 kcp 或 mkcp 才会启用 mKCP
* `kcpSettings`: 包含一些关于 mKCP 设置的参数，有
  * `uplinkCapacity`: 上行链路容量，将决定 V2Ray 向外发送数据包的速率。单位为 MB
  * `downlinkCapacity`：下行链路容量，将决定 V2Ray 接收数据包的速率。单位同样是 MB
  * `header`：对于数据包的伪装
    * `type`：要伪装成的数据包类型

客户端的上行对于服务器来说是下行，同样地客户端的下行是服务器的上行，mKCP 设置当中服务器和客户端都有 uplinkCapacity 和 downlinkCapacity，所以客户端的上传速率由服务器的 downlinkCapacity 和客户端的 uplinkCapacity 中的最小值决定，客户端的下载速率也是同样的道理。因此，建议将服务器和客户端的 downlinkCapacity 设成一个很大的值，然后分别修改两端的 uplinkCapacity 以调整上下行速率。

还有一个 header 参数可以对 mKCP 进行伪装，这是 mKCP 的一个优势。具体的伪装类型在 type 参数设置，type 可以设置成 utp、srtp、wechat-video、dtls、wireguard 或者 none，这几个分别将 mKCP 数据伪装成 BT 下载、视频通话、微信视频通话、dtls、wireguard(一种新型 VPN)以及不进行伪装。**这里的 type 参数，客户端与服务器要一致。还有要时刻记住伪装仅仅是伪装。**

至于上述配置里有但是我没有说明的参数，是 V2Ray 的默认值，我个人建议是保持默认。如果你需要了解或者修改，请参考手册。

## 更新历史

- 2018-03-17 Update
- 2018-08-30 Update

