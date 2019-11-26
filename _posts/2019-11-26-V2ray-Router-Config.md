V2ray Router Config

v2ray配置有两种方案，

1. 所有的到流量都转到v2ray, v2ray中配置路由策略。此配置简单，安装v2ray后，配置防火墙策略，若网络中有大流量迅雷或其他应用时，此方案cpu耗费高，网速可能会下降。
2. dnsmasq 根据gfwlist作域名分流，仅将gfwlist域名相关的流量交由v2ray处理，v2ray不作路由，仅处理gfwlist流量，此方案在低性能cpu上表现好。


参考 https://github.com/felix-fly/v2ray-dnsmasq-dnscrypt ,  https://gitee.com/felix-fly/v2ray-openwrt 

>
>1. dns-forwarder listen 1053端口，将dns请求转为tcp请求到8.8.8.8或其他可信的dns server
>2. dnsmasq负责所有的dns接收，添加gfwlist 的ipset文件到dnsmasq中。gfwlist中的域名转发到dns-forwarder的1053端口
>3. dnsmasq将gfwlist中的域名对应的ip添加到ipset(gfwlist)
>4. 将ipset(gfwlist)转到v2ray端口上
>5. v2ray不再提供路由。



# 1. 安装v2ray

   #### 上传文件

   https://gitee.com/felix-fly/v2ray-openwrt/releases
   选择相应的处理架构mt7620和mt7621均为常用的路由器cpu使用mipsle对应的包。
   解压,并写好config.json

   ```

   #先在路由器建目录
   mkdir /usr/bin/v2ray/
   mkdir/etc/v2ray/
   #copy v2ray相应文件至路由器
   scp v2ray user@192.168.x.x:/usr/bin/v2ray/
   scp config.json user@192.168.x.x:/etc/v2ray/v2ray.json
   #可以先测试安装是否正确
   v2ray -config /etc/v2ray/v2ray.json #查看是否能正常运行，不能正常运行就是cpu架构选择有误
   
   ```
   #### 添加自启动服务

  ```bash
   vi /etc/config/v2ray/v2ray.service
  ```
贴入以下内容保存退出

```
#!/bin/sh /etc/rc.common
# "new(er)" style init script
# Look at /lib/functions/service.sh on a running system for explanations of what other SERVICE_
# options you can use, and when you might want them.

START=80
ROOT=/etc/config/v2ray
SERVICE_WRITE_PID=1
SERVICE_DAEMONIZE=1

start() {
# limit vsz to 64mb (you can change it according to your device)
  ulimit -v 65536
  service_start $ROOT/v2ray
#  Only use v2ray via pb config without v2ctl on low flash machine
#  service_start $ROOT/v2ray -config=$ROOT/config.pb -format=pb
}

stop() {
  service_stop $ROOT/v2ray
}
```

服务自启动

```
chmod +x /etc/config/v2ray/v2ray.service
ln -s /etc/config/v2ray/v2ray.service /etc/init.d/v2ray
/etc/init.d/v2ray enable
```

# 2. dnsmasq配置
下载gfw.conf
https://github.com/felix-fly/v2ray-dnsmasq-dnscrypt/blob/master/gw.hosts
到/etc/config/proxy/
```bash
vi /etc/dnsmasq.conf
```
添加/etc/proxy到dnsmasq
```bash
conf-dir=/etc/config/proxy
```

# 3. dns转发设置
安装dns-forwarder或dns-over-https类似的dns转发软件，将1053端口进入的dns请求转发至可信的dns server上，完成解析。

# 4. 防火墙配置(dnsmasq分流方案)
```bash
ipset -N gfwlist iphash
#以下三个为dns server
ipset -A 8.8.8.8 gfwlist
ipset -A 8.8.4.4 gfwlist
ipset -A 1.1.1.1 gfwlist
iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 12345
```

# 5. 防火墙配置(v2ray处理所有流量方案)
```
iptables -t nat -N V2RAY
iptables -t nat -A V2RAY -d 0.0.0.0 -j RETURN
iptables -t nat -A V2RAY -d 127.0.0.1 -j RETURN
iptables -t nat -A V2RAY -d 192.168.1.0/24 -j RETURN
iptables -t nat -A V2RAY -d YOUR_SERVER_IP -j RETURN
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345
iptables -t nat -A PREROUTING -j V2RAY
```

# 6. v2ray 其他平台客户端
macos:v2rayU
ios: i2ray,shadowrocket
windows:v2rayN
Android:BifrostV,V2RayNG 
也可以跨平台Qv2ray。若遇版本更新不及时，可以找到安装文件夹下的v2ray手动替换