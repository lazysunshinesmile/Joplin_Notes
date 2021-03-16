Android shell命令查询ip，网关，DNS - 一样菜 - 博客园

`以下各值常见于所有的基本配置文件中：`

`* DEVICE=name,这里name是物理设备的名字（动态分配的PPP设备应当除外，`

`它的名字是“逻辑名”。`

`* IPADDR=addr, 这里addr是IP地址。`

`* NETMASK=mask, 这里mask是网络掩码。`

`* NETWORK=addr, 这里addr是网络地址。`

`* BROADCAST=addr, 这里addr是广播地址。`

`* GATEWAY=addr, 这里addr是网关地址。`

`* ONBOOT=answer, 这里answer取下列值之一：`

`o yes -- 该设备将在boot时被激活。`

`o no -- 该设备不在boot时激活。`

`* USERCTL=answer, 这里answer取下列值之一：`

`o yes --非root用户可以控制该设备。`

`o no -- 非root用户不允许控制该设备。`

`* BOOTPROTO=proto, 这里proto取下列值之一：`

`o none -- 不使用boot时协议。`

`o bootp -- 使用bootp协议。`

`o dhcp --使用dhcp协议。`

`终端:查询IP地址: ifconfig -a`

`修改局域网IP:`

`1.以 root 登录`

`2.修改配置文件`

`/etc/sysconfig/network-scripts/ifcfg-eth0`

`文件内容如下：`

`\DEVICE=eth0`

`HWADDR=00:0C:29:A2:8C:B2`

`ONBOOT=yes`

`TYPE=Ethernet`

`NETMASK=255.255.255.0`

`IPADDR=192.168.1.11 -> 修改为 192.168.1.12`

`GATEWAY=192.168.1.1`

`reboot`

`ifconfig eth0 新ip 然后编辑/etc/sysconfig/network-scripts/ifcfg-eth0，修改ip`

`ifconfig eth0 新IP`

`然后编辑/etc/sysconfig/network-scrIPts/ifcfg-eth0，修改IP`

`一、修改IP地址`

`[aeolus@db1 network-scrIPts]$ vi ifcfg-eth0`

`DEVICE=eth0`

`ONBOOT=yes`

`BOOTPROTO=``static`

`IPADDR=219.136.241.211`

`NETMASK=255.255.255.128`

`GATEWAY=219.136.241.254`

`二、修改网关`

`vi /etc/sysconfig/network`

`NETWORKING=yes`

`HOSTNAME=Aaron`

`GATEWAY=192.168.1.1`

`三、修改DNS`

`[aeolus@db1 etc]$ vi resolv.conf`

`nameserver 202.96.128.68`

`nameserver 219.136.241.206`

`四、重新启动网络配置`

`/etc/init.d/network restart`

`修改IP地址`

`即时生效:`

`# ifconfig eth0 192.168.0.20 netmask 255.255.255.0`

`启动生效:`

`修改/etc/sysconfig/network-scrIPts/ifcfg-eth0`

`修改``default` `gateway`

`即时生效:`

`# route add default gw 192.168.0.254`

`启动生效:`

`修改/etc/sysconfig/network-scrIPts/ifcfg-eth0`

`修改DNS`

`修改/etc/resolv.conf`

`修改后可即时生效，启动同样有效`

`修改host name`

`即时生效:`

`# hostname fc2`

`启动生效:`

`修改/etc/sysconfig/networknetcfg eth0 dhcp`