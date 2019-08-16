# Installation & Configuration Guide

> This guide is referenced from this amazing author from this [link](https://www.tipsforchina.com/how-to-setup-a-fast-shadowsocks-server-on-vultr-vps-the-easy-way.html). The author's guide is by far the best guide for setting up your own shadowsocks server. In case this site ever gone down, you refer to this guide here.

Note: I highly recommend that you use **ShadowsocksR (SSR)**, instead of **Shadowsocks (SS)**. Noticed the extra '**R**'. ShadowsocksR (SSR) is a newer version that supports obfuscation, which can make your shadowsocks traffic look more like regular https web traffic. This can prevent your speed from getting throttled by your network or ISP. The server that is made using this guide will be compatible with both SS and SSR clients (if you chose the recommended parameters in this guide).

## Pre-requisite

1. You must have a linux server. This guide is made and is tested for Ubuntu LTS 18.04.
2. You must be able to SSH into the linux server and login as `root`. If you are not the `root` user, then ensure that you have sudo rights.

## Install ShadowsocksR

First thing to do is to update the server.

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

> If you encountered `A new version of configuration file /etc/default/grub is available, but the version installed currently has been locally modified. What do you want to do about modified configuration file grub?"`, just keep everything as **default**.

Once you have kept everything updated, enter the following command to initiate the Shadowsocks installation:

```bash
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh && chmod +x shadowsocksR.sh && ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

In the installation process, you will be ask to configure some parameters for your Shadowsocks server. I highly recommend the following:

```
Password: YOURPASSWORD  # this password will be used in your ShadowsocksR client later
Port: 443
cipher: chacha20
protocol: auth_sha1_v4_compatible
obfs: http_simple_compatible
```

Hit 'Enter' after you have selected the recommended parameters above and it will start the installation process. Once the installation process is done, your server has successfully installed Shadowsocks and you may began using it in your Shadowsocks client.

If you wish to make any changes later, simply enter the command below:

```bash
nano /etc/shadowsocks.json
```

Once you are done with the configuration, hit 'Ctrl + O', 'Enter' and then 'Ctrl + X' to save and exit the editor. 

Every time you make changes to this file, you need to restart shadowsocks so the changes will take effect. Restart shadowsocks using the command below (if you have changed the config file).

```bash
/etc/init.d/shadowsocks restart
```

## Optimising Server

This section is optional, however, it is highly recommended. You will get 2-10x speed once you have done these optimisation.

### Install BBR

Google BBR is a TCP congestion control algorithm that can give a huge speed boost on networks with high packet loss (basically all of the networks in/out of China).

Check if BBR is already installed in the server by entering the command below:

```bash
lsmod | grep bbr
```

If you see a text output from this command with the words "tcp_bbr" and a number beside it, then you already have BBR. **You can skip the next command.** If you do not see this word "tcp_bbr", then enter the command below:

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

> If you have an incompatible kernel, you will be asked to reboot your server after the kernel is changed.


### Optimize Kernel Settings

Enter the following command to modify the kernel settings:

```bash
nano /etc/sysctl.conf
```

Add the following text at the bottom of the file after the `net.ipv4.tcp_congestion_control = bbr` line.

```
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
```

Hit 'Ctrl + O', 'Enter' and then 'Ctrl + X' to save and exit the editor.

Apply the new setting using the command below:

```bash
sysctl -p
```

### Optimize System Settings

Enter the following command to configure the `limit.conf`:

```bash
nano /etc/security/limits.conf
```

Add these lines to the bottom of the file:

```
* soft nofile 51200
* hard nofile 51200
```

Hit 'Ctrl + O', 'Enter' and then 'Ctrl + X' to save and exit the editor.

Enter the following command to configure the `pam.d`:

```bash
nano /etc/pam.d/common-session
```

Add these lines to the bottom of the file:

```
session required pam_limits.so
```

Hit 'Ctrl + O', 'Enter' and then 'Ctrl + X' to save and exit the editor.

Enter the following command to configure the `profile`:

```bash
nano /etc/profile
```

Add these lines to the bottom of the file:

```
ulimit -n 51200
```

Hit 'Ctrl + O', 'Enter' and then 'Ctrl + X' to save and exit the editor.

Finally, enter the following command:

```bash
ulimit -n 51200
```

Restart shadowsocks to apply these optimisation:

```bash
/etc/init.d/shadowsocks restart
```

## Shadowsocks Client

The standard Shadowsocks (SS) client is no longer stable in China. I recommend using the ShadowsocksR (SSR) client if you are in China which can make your shadowsocks traffic look more like regular https web traffic. This can prevent your speed from getting throttled by your network or ISP.

### SSR Clients (recommended for China)

[ShadowsocksR for Windows](https://github.com/shadowsocksrr/shadowsocksr-csharp/releases)
[ShadowsocksR for Android](https://github.com/shadowsocksrr/shadowsocksr-android/releases)
[ShadowsocksR for Mac](https://github.com/qinyuhang/ShadowsocksX-NG-R/releases) or the recommended way to install is via brew, `brew cask install shadowsocksx-ng-r`.
[iOS Shadowrocket]($2.99)

Not recommended (anymore):

iOS Wingy (whatsapp no longer working and has limited support for SSR)
iOS Potato Lite (whatsapp no longer working)

> Note: Apple has removed all VPN and Shadowsocks apps from the China version of the app store. If your iTunes account is registered with a Chinese address, you need to create a new iTunes account with a foreign address to download these apps. If you are using a USA iTunes account but don't have a US credit card to buy apps, you can always buy a $5 USA iTunes gift card on Taobao.

### Configuring Shadowsocks Client

All shadowsocks client has similiar fields – it should not be too hard.

> Assuming you have set up your shadowsocks server configuration as per the guide's recommendation.

```
Server - The Public IP address of your server
Port - 443
Password - (whatever password you chose)
Encryption - chacha20
Protocol - origin or auth_sha1_v4 (if you choose auth_sha1_v4_compatible for your server, this option is only available for SSR clients)
Obfs - http_simple for obfuscation or plain for no obfuscation (this option is only available in SSR clients)
```

If you encounter any additional settings, please leave it as default or empty.

#### Global vs PAC Mode

Global will route all domains through the proxy, while PAC will only use the proxy for a specific list of blocked websites such as Google, Facebook, etc and use your ISP connection for everything else. Not every blocked website is part of this PAC list. And even foreign websites that are not blocked are very slow if not using a proxy or VPN.

For this reason, I recommend using the Global mode. It's easy enough to enable/disable that you can conveniently switch it off if you need to access some Chinese websites.

Other programs, e.g. SSH clients, may need to set manually to use the system proxy or use a `SOCKS5 proxy` on server `127.0.0.1 port 1080` (Windows) or `127.0.0.1 port 1086` (Mac). The proxy settings can usually be found in the advanced settings for most applications.

#### Set up proxy for your terminal

First, install privoxy via brew.

```bash
brew install privoxy
```

Open the configuration file at `/usr/local/etc/privoxy/config`

```bash
nano /usr/local/etc/privoxy/config
```

Add the following line at the end of the config file:

```
listen-address 0.0.0.0:8118
forward-socks5 / localhost:1080 .
```

**or you can append directly into the config file using this command:

```bash
echo 'listen-address 0.0.0.0:8118\nforward-socks5 / localhost:1080 .' >> /usr/local/etc/privoxy/config
```

Start up privoxy by using this command:

```bash
sudo /usr/local/sbin/privoxy /usr/local/etc/privoxy/config
```

Check if the startup is successful:

```bash
netstat -na | grep 8118
```

If you get the following message, it means that the startup is successful:

```
tcp4	0	0  *.8118		*.*		LISTEN
```

##### Use privoxy in terminal

Enter the following command to make terminal go into proxy:

```bash
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
```

To cancel the proxy, enter the following command:

```bash
unset http_proxy
unset https_proxy
```

If you close the terminal, the proxy will be disabled. If you need to make it permanent, add the command in your shell profile, e.g. `.bash_profile`, `.zshrc`, etc.

```bash
vim ~/.zshrc
-----------------------------------------------------
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
-----------------------------------------------------
```

Reload your profile to take effect immediately:

```bash
source ~/.zshrc
```

If you want to add a switch toggle in your shell profile, add this following function:

```
function proxy_off(){
    unset http_proxy
    unset https_proxy
    echo -e "Proxy is off!"
}

function proxy_on() {
    export no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com"
    export http_proxy="http://127.0.0.1:8118"
    export https_proxy=$http_proxy
    echo -e "Proxy is on!"
}
```

Test if your proxy is working:

```bash
curl ip.gs # current IP: 8.8.8.8 From: Los Angeles, California, USA choopa.com 
```
