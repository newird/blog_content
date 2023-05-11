---
title: 'connect to the school net without browser'
date: '2023-05-11'
tags: ['school net', 'server' ,'NJU']
draft: false
summary: 'connect to the school net without browser'
---

# Connecting to School Network Without a Browser

## Motivation

I have a Linux server installed without a GUI, hence, there's no browser available. However, without logging in, I can't access the Internet. So, I found a solution to connect to the Internet using just the terminal. There are many pioneers who have written scripts to log in, but these scripts are outdated. Therefore, I found a new, simpler way to log in.

## Tools

The only tool you'll need is a proxy tool, such as `squid`, which I used in this case. Fortunately, we have an Arch Linux mirror in NJU, so I can install it by running `sudo pacman -S squid` as I can't use the public Internet.

Then, you need to modify the configuration file. This step can be dangerous, so remember to stop squid after you've logged in.

```code
#sudo nvim /etc/squid/squid.conf
# this is dangerout but effiective
- http_access deny all
+ http_access allow all
```

After that, you can start the squid service by running:

```shell
sudo systemctl start squid
```

## Connection Process

Having started our proxy server, we can then use our PC or cellphone. In my case, I used my cellphone. Locate the WALN-NJU Wi-Fi and add a manual proxy with your server IP address and port 3128 (you can find this in squid.conf). After that, you can access the login page on your phone.

## conclusion

I found a new way to log in to the school network for servers without a GUI, only via a proxy tool named squid. This method is simpler than using outdated login scripts, and it avoids dealing with the CAPTCHA on the new login page.

