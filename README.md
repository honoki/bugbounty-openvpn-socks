## Description

This project uses Docker to run multiple VPN tunnels simultaneously from the same machine, and exposes a SOCKS proxy for each separate VPN connection.

## Setup

1. `git clone https://github.com/honoki/bugbounty-openvpn-socks`
2. Edit the `.env` file with the required credentials
3. Download the `ovpn` files of each platform to the corresponding `/vpn/<platform>` folder:
   * Hackerone: https://hackerone.com/settings/vpn
   * Intigriti: ?
   * YesWeHack: https://yeswehack.com/user/vpn
3. `docker-compose up`
4. Test the VPN connections with:
  ```bash
  curl ifconfig.io -x socks5://localhost:1080; #hackerone
  curl ifconfig.io -x socks5://localhost:1081; #intigriti
  curl ifconfig.io -x socks5://localhost:1082; #yeswehack
  ```
## Burp Suite

You can use the SOCKS proxy to tunnel Burp traffic through the VPN via "Project options":

![](image.png)

## SOCKS5

You can proxy traffic through the following ports:
* `localhost:1080` for the Hackerone VPN;
* `localhost:1081` for the Intigriti VPN;
* `localhost:1082` for the YesWeHack VPN;
