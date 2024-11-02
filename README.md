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


3. 关于load-balance
3.1实例
  # load-balance: The request of the same eTLD+1 will be dial to the same proxy.
  - name: "load-balance"
    type: load-balance
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
    # strategy: consistent-hashing # or round-robin

3.2 解析
name: "load-balance":

这是代理组的名称，命名为 load-balance，表示这是一个负载均衡组。

type: load-balance:

代理组的类型是 load-balance，表示这个组会在多个代理之间分配流量。

proxies:

这是一个包含多个代理的列表，在这个例子中，包含 ss1、ss2 和 vmess1 这三个代理。这意味着流量会被分配到这三个代理中。

url:

这是用于检测代理可用性的测试URL。在这个例子中，使用了 http://www.gstatic.com/generate_204，这是一个常用的测试URL，返回一个204状态码，用于快速检测连接是否正常。

interval:

检测间隔时间，单位为秒。在这个例子中，设置为300秒（即5分钟），表示每隔5分钟检测一次代理的可用性。

示例情境：
假设你有以下三个代理服务器：

ss1：一个Shadowsocks代理服务器，位于 ss://example-ss1.com。

ss2：另一个Shadowsocks代理服务器，位于 ss://example-ss2.com。

vmess1：一个VMess代理服务器，位于 vmess://example-vmess.com。

在实际使用中，Clash会根据负载均衡策略将流量分配到这些代理服务器：

初始连接：

Clash会根据负载均衡策略选择一个代理服务器，例如 ss1。

检测和切换：

每隔5分钟，Clash会通过 http://www.gstatic.com/generate_204 检测这些代理的可用性。

如果检测到 ss1 有问题，Clash会自动切换到 ss2 或 vmess1，根据当前的负载和可用性选择最佳代理。

流量分配：

当所有代理都可用时，Clash会根据负载均衡算法（如轮询或随机）在 ss1、ss2 和 vmess1 之间分配流量。

这有助于优化连接性能，避免单一代理过载或故障。

1. 负载均衡算法是在内核中的吗？
是的，Clash 的负载均衡算法是在其内核中实现的。内核负责执行负载均衡策略，并在不同的代理之间分配流量。

2. 首选的默认负载均衡算法是哪一个？
Clash 内核默认采用的负载均衡算法是 Round Robin（轮询）。这种算法会依次将流量分配给列表中的每个代理，以达到均衡负载的目的。

3. 可以指定选择负载均衡算法吗？
在当前版本的 Clash 中，负载均衡策略是内置在内核中的，并且默认使用轮询算法。用户无法通过配置文件指定使用其他负载均衡算法（例如随机或最少连接数）。如果需要特定的负载均衡策略，可能需要自定义开发或修改Clash的源码。

4. URL 是用来查看节点的可用性的吗？
是的，url 用于检测代理节点的可用性。Clash 会定期向该 URL 发送请求，并根据响应状态码判断代理节点是否正常工作。在你的配置中使用了 http://www.gstatic.com/generate_204 这个 URL，它返回 204 状态码，是一个常用的检测连接是否正常的测试 URL。

链接数量如何进行分配
Clash 的 load-balance 类型代理组在处理高并发任务时，会依据其负载均衡算法（如轮询）在不同代理之间分配流量。

2. 如果 ss1 已连接，会切换到 ss2 吗？
在 load-balance 组中，流量分配并不是简单地“连接满一个代理后再切换到下一个”，而是依据负载均衡算法进行分配。

轮询（Round Robin）：例如，如果当前有多个并发连接，Clash 会依次将这些连接分配给 ss1、ss2 和 vmess1，确保每个代理得到均匀的流量。这意味着即使 ss1 已连接，新的连接也会分配给 ss2 和 vmess1，而不是全部集中在 ss1 上。

随机（Random）：如果是随机负载均衡算法，每个新连接会随机分配给 ss1、ss2 或 vmess1。

最少连接数（Least Connections）：如果使用最少连接数算法，Clash 会将新连接分配给当前连接数最少的代理，以均衡各代理的负载。

3. 高并发任务中的流量分配
在高并发任务中，Clash 会持续监控每个代理的可用性并执行健康检查（如通过 url: 'http://www.gstatic.com/generate_204'）。根据检测结果和负载均衡算法，流量会动态分配：

健康检查：每隔一定时间（如300秒），Clash会测试所有代理的可用性。

动态分配：如果某个代理（如 ss1）不可用，Clash 会自动将流量分配到其他可用代理（如 ss2 或 vmess1）。

故障恢复：如果检测到 ss1 恢复正常，Clash 会将部分或全部流量重新分配给 ss1，取决于当前的负载和策略。

这种分配机制确保了在高并发和负载条件下，流量能够均匀分布在多个代理之间，提供更稳定和高效的连接。


4.    select 变种 interface-name: en1        routing-mark: 6667
4.1 样例
  - name: en1
    type: select
    interface-name: en1
    routing-mark: 6667
    proxies:
      - DIRECT
4.2解析
假设你的路由器有以下网络接口：

en1

en2

en3

WiFi

你希望通过WiFi接口传输某些特定的流量，并指定使用特定的代理。

在操作系统中配置路由规则，使标记为特定值（如 6668 ）的流量通过WiFi接口传输。例如，在Linux系统中，可以使用ip rule和ip route命令来设置路由规则：

bash
ip rule add fwmark 6668 table 100
ip route add default via 192.168.1.1 dev wifi table 100
Clash配置文件：

在Clash的配置文件中添加WiFi接口的配置：

yaml
proxy-groups:
  - name: wifi
    type: select
    interface-name: wifi
    routing-mark: 6668
    proxies:
      - DIRECT
      - Proxy-1
      - Proxy-2
添加代理服务器配置：

定义你的代理服务器配置，例如：

yaml
proxies:
  - name: "Proxy-1"
    type: ss
    server: example-ss1.com
    port: 8388
    cipher: aes-256-gcm
    password: your-password

  - name: "Proxy-2"
    type: vmess
    server: example-vmess.com
    port: 443
    uuid: your-uuid
    alterId: 0
    cipher: auto
    tls: true
流量规则：

在Clash的规则中，将特定的流量路由到WiFi接口的代理组：

yaml
rules:
  - DOMAIN-SUFFIX,special-domain.com,wifi
  - MATCH,load-balance
整个配置文件示例
yaml
mixed-port: 7890
allow-lan: true
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

proxies:
  - name: "Proxy-1"
    type: ss
    server: example-ss1.com
    port: 8388
    cipher: aes-256-gcm
    password: your-password

  - name: "Proxy-2"
    type: vmess
    server: example-vmess.com
    port: 443
    uuid: your-uuid
    alterId: 0
    cipher: auto
    tls: true

proxy-groups:
  - name: wifi
    type: select
    interface-name: wifi
    routing-mark: 6668
    proxies:
      - DIRECT
      - Proxy-1
      - Proxy-2

  - name: load-balance
    type: load-balance
    proxies:
      - Proxy-1
      - Proxy-2
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

rules:
  - DOMAIN-SUFFIX,special-domain.com,wifi
  - MATCH,load-balance
在这个配置中，访问 special-domain.com 的流量会通过WiFi接口传输，并且可以选择DIRECT或指定的代理 (Proxy-1 和 Proxy-2)。其他流量将通过负载均衡代理组进行分配



如果你要求所有通过WiFi的流量都走代理，那么确实不需要加上 - DOMAIN-SUFFIX,special-domain.com,wifi 这条规则。

你可以直接设置一条通配符规则，将所有流量通过 wifi 代理组处理。这么做将确保任何通过WiFi接口传输的流量都走指定的代理。

更新后的配置文件示例
yaml
mixed-port: 7890
allow-lan: true
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090

proxies:
  - name: "Proxy-1"
    type: ss
    server: example-ss1.com
    port: 8388
    cipher: aes-256-gcm
    password: your-password

  - name: "Proxy-2"
    type: vmess
    server: example-vmess.com
    port: 443
    uuid: your-uuid
    alterId: 0
    cipher: auto
    tls: true

proxy-groups:
  - name: wifi
    type: select
    interface-name: wifi
    routing-mark: 6668
    proxies:
      - Proxy-1
      - Proxy-2

  - name: load-balance
    type: load-balance
    proxies:
      - Proxy-1
      - Proxy-2
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

rules:
  - MATCH,wifi
在这个配置中：

所有通过WiFi接口的流量都将按照 wifi 代理组进行处理。

wifi 代理组包含 Proxy-1 和 Proxy-2，意味着所有WiFi流量会通过这两个代理之一。

这样设置后，所有通过WiFi的流量将会走指定的代理，无需再添加具体的域名规则。





