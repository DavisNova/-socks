# -socks





### 关于proxy-groups
1. 关于relay
1.1 样例
proxy-groups:
  # relay chains the proxies. proxies shall not contain a relay. No UDP support.
  # Traffic: clash <-> http <-> vmess <-> ss1 <-> ss2 <-> Internet
  - name: "relay"
    type: relay
    proxies:
      - http
      - vmess
      - ss1
      - ss2
1.2 解析
name: "relay":

这是代理组的名称，命名为 "relay"。

type: relay:

代理组的类型是 relay，表示该组内的代理会按照链式传输方式依次传递流量。

proxies:

这是一个包含多个代理的列表，流量会按照顺序通过这些代理。列表中的代理是 http、vmess、ss1、ss2。
当Clash接收到流量时，会先将流量传递给 http 代理，然后依次传递给 vmess、ss1 和 ss2 代理，最后流量到达互联网。

这个配置不支持UDP流量，只适用于TCP流量。

1.3 同时需要在proxies中有如下配置
proxies:
  - name: "http"
    type: http
    server: example-http.com
    port: 8080
    username: user
    password: pass

  - name: "vmess"
    type: vmess
    server: example-vmess.com
    port: 443
    uuid: your-uuid
    alterId: 0
    cipher: auto
    tls: true

  - name: "ss1"
    type: ss
    server: example-ss1.com
    port: 8388
    cipher: aes-256-gcm
    password: your-password

  - name: "ss2"
    type: ss
    server: example-ss2.com
    port: 8388
    cipher: aes-256-gcm
    password: your-password

可以使用其他代理组来进行中转


