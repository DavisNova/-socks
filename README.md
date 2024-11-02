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




2. fallback
2.1 样例
  - name: "fallback-auto"
    type: fallback
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
2.2 解析
name: "fallback-auto":

这是代理组的名称，命名为 fallback-auto。

type: fallback:

代理组的类型是 fallback，表示这个组会自动检测和切换代理，以确保连接的稳定性和可用性。

proxies:

包含的代理列表：ss1、ss2 和 vmess1。这意味着流量会通过这些代理中的一个，如果某个代理不可用，会自动切换到下一个。

url:

这是用于检测代理可用性的测试URL。在这个例子中，使用了 http://www.gstatic.com/generate_204，这是一个常用的测试URL，返回一个204状态码，用于快速检测连接是否正常。

interval:

检测间隔时间，单位为秒。在这个例子中，设置为300秒（即5分钟），表示每隔5分钟检测一次代理的可用性。

默认选择的代理服务器： 是的，默认选择的代理服务器是列表中的第一个代理服务器，即 ss1。

如果ss1故障，是否按顺序切换到ss2： 是的，如果 ss1 故障，Clash 会按顺序切换到 ss2。

如果已经切换到了ss2，此时ss1恢复正常，是否会切回ss1： 是的，Clash 会在检测到 ss1 恢复正常后，重新切回 ss1，因为它是列表中的第一个代理服务器。

如果切换到ss2，此时ss2故障，是按顺序切换到vmess1还是切换到ss1： 如果切换到 ss2 后， ss2 也故障，Clash 会按顺序切换到列表中的下一个代理服务器，即 vmess1。如果 vmess1 也故障，Clash 会按顺序检查代理列表，重新尝试连接到 ss1。

总结一下，Clash的fallback代理组会按照代理列表的顺序进行切换，并且在检测到故障代理恢复正常后，会重新切换回该代理，以确保连接的稳定性和连续性。

默认情况下，Clash会选择 ss1 作为初始代理。

如果 ss1 故障，Clash会按顺序切换到 ss2，然后是 vmess1。

如果检测到 ss1 恢复正常，Clash会重新切回到 ss1，因为它是列表中的第一个代理。

这种行为设计的目的是为了确保使用代理列表中的优先级顺序来提供最稳定的连接。






