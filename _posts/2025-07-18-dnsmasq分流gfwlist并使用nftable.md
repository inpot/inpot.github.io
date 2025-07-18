# dnsmasq分流gfwlist并使用nftable

#### 透明代理配置：dnsmasq (with nftset) + gfwlist + nftables + tproxy

本指南将详细说明如何在 Linux 系统上配置一个高效的透明代理。通过结合 `dnsmasq` 进行 DNS 解析和域名过滤（直接操作 `nftables` 集合），`nftables` 进行高级流量控制和重定向，以及 `tproxy` 实现流量透明代理，您可以将所有 GFW 列表中的域名流量透明地转发到代理服务器，而其他流量则直连。

------

### 前提条件

- 一台 Linux 系统（例如 Debian、Ubuntu、CentOS、Fedora 或 OpenWrt），拥有 **root 权限**。
- **`nftables` 已安装**。
- **`dnsmasq` 已安装且版本支持 `nftset` 功能**。您可以通过运行 `dnsmasq --version` 并查找输出中是否包含 `HAVE_NFTSET` 来确认。如果不支持，则需要回退到上一版指南中 `dnsmasq` 与 `ipset` 兼容层的方法。
- 一个正常工作的代理服务器（例如 Shadowsocks、V2Ray、Xray），监听在特定端口。本指南假定您的代理服务器监听在 **`127.0.0.1:12345`**，并且已配置为支持 **`tproxy` 透明代理模式**。
- 对 Linux 命令行和网络概念有基本了解。

------

### 步骤 1：配置 dnsmasq (集成 nftset)

我们将配置 `dnsmasq` 来解析域名。对于 GFW 列表中的域名，`dnsmasq` 将直接把解析到的 IP 地址添加到 `nftables` 中预定义的 `nftset` 集合中。

1. 准备 GFWList 转换为 dnsmasq nftset 格式：

   您需要一个脚本来定期下载最新的 GFWList 并将其转换为 dnsmasq 可用的 nftset 指令格式。这通常会生成类似以下内容的行（例如，保存为 /etc/dnsmasq.d/gfwlist.conf）：

   ```
   # 假设您的 nftables 表名为 'filter' (或 OpenWrt 中可能是 'fw4')，集合名为 'gfw_ips'
   nftset=/facebook.com/ip#filter#gfw_ips
   nftset=/google.com/ip#filter#gfw_ips
   nftset=/twitter.com/ip#filter#gfw_ips
   # ... 更多 nftset 行 ...
   
   # 如果需要处理 IPv6 流量，可以添加对应的 ip6 规则
   # nftset=/google.com/ip6#filter#gfw_ips
   ```

   **重要提示：** `ip#filter#gfw_ips` 中的 `filter` 是 `nftables` 的表名，`gfw_ips` 是您将在 `nftables` 中创建的集合名。请根据您的实际 `nftables` 配置调整。在 OpenWrt 中，默认的表名可能是 `fw4`。

2. 配置 dnsmasq：

   编辑或创建 /etc/dnsmasq.d/gfwlist.conf 文件，并包含上述生成的 nftset 行。

   确保您的主 `dnsmasq.conf` (例如 `/etc/dnsmasq.conf`) 包含您的 `gfwlist.conf` 所在的目录：

   ```
   conf-dir=/etc/dnsmasq.d
   ```

   同时，配置 `dnsmasq` 监听本地地址，并将其他查询转发到常规 DNS 服务器：

   ```
   # 监听本地地址，以便系统可以将 DNS 请求发送到这里
   listen-address=127.0.0.1
   
   # 上游 DNS 服务器，用于非 GFW 列表的域名解析
   server=8.8.8.8  # 示例：Google DNS
   server=1.1.1.1  # 示例：Cloudflare DNS
   
   # 可选：启用日志以便调试
   # log-queries
   # log-facility=/var/log/dnsmasq.log
   ```

3. 设置系统 DNS 指向 dnsmasq (localhost)：

   编辑 /etc/resolv.conf（或通过您的网络管理器配置）以指向 127.0.0.1：

   ```
   nameserver 127.0.0.1
   ```

   **注意：** 如果您的系统使用 `systemd-resolved` 或 NetworkManager 管理 DNS，您可能需要相应地配置它们（例如，编辑 `/etc/systemd/resolved.conf` 并设置 `DNS=127.0.0.1`）。

4. **重启 `dnsmasq`：**

   Bash

   ```
   sudo systemctl restart dnsmasq
   sudo systemctl enable dnsmasq
   ```

------

### 2：配置 nftables (包含 nftset 和 tproxy)

我们将使用 `nftables` 创建一个动态 IP 集合，并设置规则以透明代理所有匹配集合中 IP 的流量。

1. 创建 nftables 配置文件：

   创建一个文件，例如 /etc/nftables.conf。

   注意： 替换 127.0.0.1:12345 为您的代理服务器地址和端口。

   代码段

   ```
   #!/usr/sbin/nft -f
   
   # 刷新现有规则，避免冲突
   flush ruleset
   
   # 定义一个 IPv4 和 IPv6 兼容的表 (或者 table ip filter，如果只处理 IPv4)
   table inet filter {
       # 定义一个名为 'gfw_ips' 的集合，dnsmasq 将填充此集合
       set gfw_ips {
           type ipv4_addr
           flags dynamic, timeout
           timeout 300m # IP 将在 30 分钟后过期，如果 dnsmasq 未刷新
       }
   
       # PREROUTING 链：处理所有进入本机的流量（无论源自何处），包括标记
       chain prerouting {
           type filter hook prerouting ingress priority -100; policy accept;
   
           # 1. 忽略局域网、本地以及代理服务器本身的流量，避免循环或不必要的代理
           # 注意：这里不需要排除代理服务器自身发出的流量，因为 prerouting 只处理入站流量
           ip daddr { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.1/8 } return
           # 2. 忽略发往本机 dnsmasq 的 DNS 查询，避免代理自身 DNS
           ip daddr 127.0.0.1 udp dport 53 return
   
           # 3. 对 GFW 列表的流量打上标记 (0x1)
           # 只有目的地是 GFW IP 的 TCP/UDP 流量会被标记
           ip daddr @gfw_ips tcp meta mark set 0x1
           ip daddr @gfw_ips udp meta mark set 0x1
   
           # 4. 根据标记进行 TPROXY 重定向
           # 只有带有 0x1 标记的流量才会被代理
           meta mark 0x1 tcp tproxy to 127.0.0.1:12345 accept
           meta mark 0x1 udp tproxy to 127.0.0.1:12345 accept
       }
   
       # OUTPUT 链：处理本机自身发出的流量
       chain output {
           type filter hook output priority -100; policy accept;
   
           # 1. 忽略局域网、本地以及代理服务器本身的流量
           # 这些流量通常也不需要通过代理，也不会被前面的 mark 规则影响
           ip daddr { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.1/8 } return
           # 2. 忽略发往本机 dnsmasq 的 DNS 查询
           ip daddr 127.0.0.1 udp dport 53 return
   
           # 3. **核心修改：** 仅代理带有 0x1 标记的本机发出流量
           # 只有在 prerouting 阶段被标记为 GFW 流量的数据包才会有 0x1 标记。
           # 本地代理进程发出的流量（例如，代理连接远程服务器的流量）不会在 prerouting 链中被打上 0x1 标记，
           # 因此，这些流量将不会匹配到这里的规则，从而避免被重复代理。
           meta mark 0x1 tcp tproxy to 127.0.0.1:12345 accept
           meta mark 0x1 udp tproxy to 127.0.0.1:12345 accept
       }
   }
   ```

   **`nftables` 规则解释：**

   - `table inet filter`：定义一个名为 `filter` 的表，可以同时处理 IPv4 和 IPv6 流量（`inet` 族）。如果只处理 IPv4，可以使用 `table ip filter`。
   - `set gfw_ips`：创建一个动态集合 `gfw_ips`，用于存储 IPv4 地址。这个集合将由 `dnsmasq` 填充。
   - `chain prerouting`：在路由决策之前处理所有传入数据包。
   - `chain output`：处理由本机进程生成的所有传出数据包。
   - `ip daddr { ... } return`：豁免本地、内网和代理自身的流量，防止循环和不必要的代理。
   - `ip daddr @gfw_ips tcp tproxy to 127.0.0.1:12345 accept`：**这是核心规则。** 如果数据包的目标 IP 地址在 `gfw_ips` 集合中，无论目标端口是什么，TCP 流量都将被透明代理到 `127.0.0.1:12345`。UDP 规则同理。

2. **加载 `nftables` 配置：**

   Bash

   ```
   sudo nft -f /etc/nftables.conf
   ```

3. **启用 `nftables` 随启动加载：**

   Bash

   ```
   sudo systemctl enable nftables
   sudo systemctl start nftables
   ```

------



### 步骤 3：启用 IP 转发和 TPROXY 路由



`tproxy` 的正常工作需要内核的 IP 转发功能以及特定的路由表配置。

1. 启用 IP 转发：

   编辑 /etc/sysctl.conf 并添加或取消注释以下行：

   ```
   net.ipv4.ip_forward = 1
   # 如果您处理 IPv6 流量，也请启用 IPv6 转发
   # net.ipv6.conf.all.forwarding = 1
   ```

   应用更改：

   Bash

   ```
   sudo sysctl -p
   ```

2. 配置 TPROXY 路由规则：

   tproxy 要求数据包在被代理后能正确路由回本地的代理服务。创建新的路由表并设置规则：

   Bash

   ```
   # 为标记为 fwmark 1 的数据包添加路由规则
   sudo ip rule add fwmark 1 lookup 100
   # 在表 100 中，所有目的地址都通过本地回环接口 (lo) 路由
   sudo ip route add local 0.0.0.0/0 dev lo table 100
   # 如果处理 IPv6
   # sudo ip -6 rule add fwmark 1 lookup 100
   # sudo ip -6 route add local ::/0 dev lo table 100
   ```

   为了使这些规则在系统重启后依然有效，您需要将其添加到开机启动脚本中。最常见的方法是创建一个 Systemd 服务或将其放入网络配置脚本中。

------



### 步骤 4：配置您的代理服务器



确保您的代理服务器（例如 V2Ray/Xray）已配置为监听 `127.0.0.1:12345`，并且**已启用 `tproxy` 或透明代理模式**。代理服务器必须能够接收并处理来自 `tproxy` 转发的流量。

**V2Ray/Xray 示例 (`config.json` 中的 `inbounds` 部分)：**

JSON

```
{
  "inbounds": [
    {
      "port": 12345,
      "listen": "127.0.0.1",
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp", // 必须包含 tcp 和 udp
        "tproxy": "tproxy",   // 启用 tproxy 模式
        "followRedirect": true // 遵循系统重定向
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"] // 启用流量嗅探以获取真实目标信息
      }
    }
  ],
  // ... 其他 outbounds, routing 规则等。
}
```

**重启您的代理服务器**以应用更改。

------



### 步骤 5：验证和测试



1. 检查 dnsmasq 日志：

   sudo journalctl -u dnsmasq 或查看 /var/log/dnsmasq.log (如果配置了日志)，确认域名解析正常，并且 nftset 指令没有报错。

2. 检查 nftables 集合 (nftset)：

   在 dnsmasq 运行一段时间并解析了一些 GFW 域名后，检查您的 nftset 内容：

   Bash

   ```
   sudo nft list set inet filter gfw_ips
   # 或根据您的表名：
   # sudo nft list set ip filter gfw_ips
   # sudo nft list set inet fw4 gfw_ips # 如果是 OpenWrt
   ```

   您应该能看到 GFW 域名对应的 IP 地址出现在此集合中，并且带有超时时间。

3. **测试连接：**

   - **访问 GFW 屏蔽网站：** (例如 `http://googleusercontent.com/youtube.com/3`、`facebook.com`)。这些流量应该通过您的代理。
   - **访问普通网站：** (例如 `baidu.com`)。这些流量应该直连，不经过代理。

   您可以使用 `curl` 或 `wget` 进行命令行测试，或者通过浏览器访问。检查代理服务器的日志，确认来自 GFW 站点的流量是否被正确接收。

4. 网络监控 (可选)：

   使用 tcpdump 或 wireshark 监控流量，确认流量是否按预期重定向：

   Bash

   ```
   # 监控您回环接口（代理监听处）的流量
   sudo tcpdump -i lo port 12345
   
   # 监控您主网络接口的流量 (例如 eth0)
   sudo tcpdump -i eth0 host <某个GFW网站的IP>
   ```

------



### 故障排除



- **`dnsmasq` `nftset` 不工作：** 再次确认您的 `dnsmasq` 版本是否支持 `HAVE_NFTSET`。如果不支持，需要使用旧的 `dnsmasq ipset` + `nftables` 兼容 `ipset` 的方法。
- **`nftables` 规则错误：** 运行 `sudo nft -f /etc/nftables.conf` 时检查是否有任何语法错误。
- **代理服务器问题：** 确保您的代理服务器已正确配置 `tproxy` 或透明代理模式，并且监听在 `127.0.0.1:12345`。检查代理服务器自身的日志。
- **IP 转发未开启：** 确保 `net.ipv4.ip_forward = 1` 已生效 (`sudo sysctl -p`)。
- **路由规则：** 确认 `ip rule` 和 `ip route` 的 `fwmark 1` 规则已正确设置并持久化。
- **防火墙冲突：** 确保没有其他防火墙规则（例如 `iptables`、`firewalld`）与 `nftables` 规则冲突。最好只使用 `nftables`。
- **`dnsmasq` 缓存：** 更改 `dnsmasq` 配置后，可能需要清除 DNS 缓存或重启 `dnsmasq`。
- **GFWList 更新脚本：** 确保您的 GFWList 转换脚本能够正确生成 `nftset` 格式的配置，并且定期更新。

------

希望这份详细的指南能帮助您成功设置透明代理！在配置过程中遇到任何问题，请随时提出。