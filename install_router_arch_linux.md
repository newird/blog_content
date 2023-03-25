---
title: 'Installing a Router on Arch Linux'
date: '2023-03-23'
tags: ['arch Linux', 'Router','clash']
draft: false
summary: ' Installing a Router on Arch Linux .'
---



# Installing a Router on Arch Linux

For various reasons, I decided to install a software router on my Arch Linux system. However, I wasn't fond of using EXI or any other pre-built solution. As a dedicated Arch Linux user, I wanted to install the router on my system and was grateful to find many users sharing their experiences on how to do it.

## Changing the Network Card Names (Optional)

This step is optional but recommended for better organization. You can retrieve your MAC address by running the ip a command, which is short for ip address show. I had four network interfaces, so I added the following lines to the /etc/udev/rules.d/10-network.rules file:

```
# /etc/udev/rules.d/10-network.rules
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="MAC_ADDRESS", NAME="extern0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="MAC_ADDRESS", NAME="intern0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="MAC_ADDRESS", NAME="intern1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="MAC_ADDRESS", NAME="intern2"
```

### Reloading the Config
To apply the changes, run the following commands:

```shell
udevadm control --reload
udevadm trigger
```

## Setting the WAN
To set the WAN (Wide Area Network), create the /etc/systemd/network/20-wired-external-dhcp.network file with the following content:

```code
# /etc/systemd/network/20-wired-external-dhcp.network
[Match]
Name=extern0 #if you skip the rename step ,you can find your interface by `ip a` ,normally ,it is called enp* 

[Network]
DHCP=yes
IPv6AcceptRA=yes
IPv6PrivacyExtensions=yes
```

Then, start the system resolver by running the following commands:

```
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
systemctl enable --now systemd-networkd systemd-resolved
```

## Setting the LAN

To set up the LAN (Local Area Network), first create the bridge by adding the following content to the /etc/systemd/network/br_lan.netdev file:

```code
# /etc/systemd/network/br_lan.netdev
[NetDev]
Name=br_lan
Kind=bridge
```
Then, add the following content to the /etc/systemd/network/10-bind-br_lan.network file to bind the intern* interface to the bridge:
```code
# /etc/systemd/network/10-bind-br_lan.network
[Match]
Name=intern*  # Replace `intern*` with the name of your interface. You can find it by running `ip a`.

[Network]
Bridge=br_lan
```
Finally, allocate an IP address for the internal network by creating the /etc/systemd/network/21-wired-internal.network file with the following content:

```code
# /etc/systemd/network/21-wired-internal.network
[Match]
Name=br_lan

[Link]
Multicast=yes

[Network]
Address=10.0.0.1/24
MulticastDNS=yes
IPMasquerade=both
#IPv6SendRA=yes
#DHCPPrefixDelegation=yes
#IPv6AcceptRA=no

#[DHCPPrefixDelegation]
#UplinkInterface=extern0
#SubnetId=1
#Announce=yes
```

## Using  systemd-resolved on Arch Linux

In this guide, we'll be using clash as our rule-based proxy, and systemd-resolved as our DNS resolver temporarily. After we create ourselves a route, we'll be disabling systemd-resolved.

### Checking if our network works

Before we begin, let's check if our network works with systemd-resolved. To do this, create a configuration file for the DNS resolver:


```code
# /etc/systemd/resolved.conf.d/listen-on-internal.conf
[Resolve]
DNSStubListenerExtra=10.0.0.1
```

Then restart the systemd-resolved service:

```code
systemctl restart systemd-resolved
```

Next, start the dhcp service. If you don't have it installed, you can run `pacman -S dhcp` to install it:

```shell
# /etc/dhcpd.conf
option domain-name-servers 10.0.0.1, 8.8.8.8;
option subnet-mask 255.255.255.0;
option routers 10.0.0.1;
subnet 10.0.0.0 netmask 255.255.255.0 {
    range 10.0.0.100 10.0.0.250;
}
```

Since I have multiple network interfaces, I need to use this command to start the dhcp service on the interface we choose:

```shell
cp /usr/lib/systemd/system/dhcpd4.service /etc/systemd/system/dhcpd4@.service
```

then edit `/etc/systemd/system/dhcpd4@.service	`

```code
#vim /etc/systemd/system/dhcpd4@.service
- ExecStart=/usr/bin/dhcpd -4 -q -cf /etc/dhcpd.conf -pf /run/dhcpd4/dhcpd.pid 
+ ExecStart=/usr/bin/dhcpd -4 -q -cf /etc/dhcpd.conf -pf /run/dhcpd4/dhcpd.pid %I
```

Finally, restart the service:

```shell
systemctl daemon-reload
systemctl enable --now dhcpd4@br_lan.service
```

### Setting up internal forwarding

To start using clash, we need to set up internal forwarding. Create a new configuration file:

```
nvim /etc/sysctl.d/30-ipforward.conf
net.ipv4.ip_forward=1
net.ipv6.ip_forward=1
```

after the step you can connect the internet use your route ,but the really wonderful world is just starting

## Installing clash

Now that we have our network set up, we can install  [clash](https://pacman.ltd/). On Arch Linux, we can use pacman:

```shell
sudo pacman -S clash-premium-bin
```

### Configuring clash as a service

Next, we need to modify clash to run as a service. Create a new file at `/etc/systemd/system/clash.service`:

```shell
# vim /etc/systemd/system/clash.service
[Unit]
Description=A rule based proxy in Go for neko.
After=network.target

[Service]
Type=exec
Restart=on-abort
ExecStart=/usr/bin/clash -d /etc/clash #this is where you config file locate 

[Install]
WantedBy=multi-user.target
```

### Using clash as our DNS resolver

We can now use clash as our DNS resolver. Disable and remove systemd-resolved and create a new resolv.conf file:

```shell
sudo systemctl disable --now systemd-resolved
sudo rm /etc/resolv.conf
echo 'nameserver 127.0.0.1' | sudo tee /etc/resolv.conf
```

the head of clash config, the most will be given by your VPN provider ,if you build it by yourself ,apparently you are more experienced than me .

```code
port: 7890
socks-port: 7891
redir-port: 7892
allow-lan: true
mode: rule
log-level: silent
external-controller: '0.0.0.0:9090'
secret: ''
tun:
  enable: true
  stack: system
  dns-hijack:
   - tcp://8.8.8.8:53
   - udp://8.8.8.8:53
dns:
  enable: true
  ipv6: true
  listen: '0.0.0.0:53'
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16 # take care of this ,it will be use later ,
  nameserver:
    - 1.1.1.1
    - 8.8.8.8
    - 'tcp://223.5.5.5'
  fallback:
    - 'tls://223.5.5.5:853'
    - 'https://223.5.5.5/dns-query'
  fallback-filter:
    geoip: true
    ipcidr:
      - 240.0.0.0/4
```

## test clash

let change the route table ,and test the connection

```shell
sudo ip route add default via 198.18.0.1 dev utun table 233 # the ip is just on clash/config.yaml
sudo ip route add 198.18.0.0/16 dev utun table 233
sudo ip route add 10.0.0.0/16 dev intern0 table 233
sudo ip rule add from 10.0.0.0/24 table 233
```

after this you can visit the website that you can't visit before, okay, but if we do this every time, it is nasty.

Let's do it automatically.

## clash script

make a directory for clash

```shell
sudo mkdir /etc/clash/scripts
```

And we need to create the route table after Clash starts:

```
# vim /etc/clash/scripts/setup.sh
#!/bin/bash

#wait for TUN device
while ! ip address show utun > /dev/null; do
    sleep 0.2
done

ip route flush table 233
ip route add default via 198.19.0.1 dev utun table 233
ip route add 198.19.0.0/16 dev utun table 233
ip route add 10.0.0.0/16 dev enp3s0 table 233

ip rule add from 10.0.0.0/16 table 233
```

and delete the table after clash stop

```code
# vim /etc/clash/scripts/unsetup.sh
#!/bin/bash

ip rule delete from all table 233
ip route flush table 233
```

at last ,we need add this script to clash.service 

```code
  [Service]
  Type=exec
  Restart=on-abort
  ExecStart=/usr/bin/clash -d /etc/clash
+ ExecStartPost=+/etc/clash/scripts/setup.sh
+ ExecStopPost=+/etc/clash/scripts/unsetup.sh
```

## Conclusion

Now you should be able to connect to the internet with the help of Clash! :smile:

## reference

[最好的軟路由 Powered By Arch Linux（其一：基本網路設定） | YHNdnzj's Blog](https://yhndnzj.com/2021/08/13/arch-is-the-best-router/)

[Router - ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Router)

[用 Arch Linux 做软路由 — 凌莞咕噜咕噜～ (nyac.at)](https://nyac.at/posts/archlinux-router)
