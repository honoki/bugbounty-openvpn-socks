## Description

This project uses Docker to run multiple VPN tunnels simultaneously from the same machine, and exposes a SOCKS proxy for each separate VPN connection.

![Diagram of OpenVPN/SOCKS ](docs/diagram.png)

## Setup

1. `git clone https://github.com/honoki/bugbounty-openvpn-socks`
2. Edit the `.env` file with your platform credentials
   * **Note -** If your password or mail address contains a dollar character, you need to escape it with 3 backslashes, e.g. `pas$w0rd` becomes `pas\\\$w0rd`.
   * If you want to allow unauthenticated access to your SOCKS proxy, remove the `PROXY_USERNAME` and `PROXY_PASSWORD`
3. Download the `ovpn` files of each platform to the corresponding `/vpn/<platform>` folder:
   * Hackerone: https://hackerone.com/settings/vpn
   * Intigriti: ?
   * YesWeHack: https://yeswehack.com/user/vpn
3. `docker-compose up`
4. Check that the VPN connections are working:
  ```bash
  curl ifconfig.io # direct
  curl ifconfig.io -k -x socks5://username:password@localhost:1080; #hackerone
  curl ifconfig.io -k -x socks5://username:password@localhost:1081; #intigriti
  curl ifconfig.io -k -x socks5://username:password@localhost:1082; #yeswehack
  ```
## SOCKS5

The SOCKS5 proxy ports will be available on:
* `localhost:1080` for the Hackerone VPN;
* `localhost:1081` for the Intigriti VPN;
* `localhost:1082` for the YesWeHack VPN;

Below is a selection of tools that have SOCKS proxy support out of the box:

```bash
nuclei -proxy-socks-url socks5://username:password@localhost:1080
ffuf -x socks5://username:password@localhost:1080
curl -x socks5://username:password@localhost:1080
```

If you are using tools without built-in proxy support, you can use [`proxychains`](https://github.com/haad/proxychains) to force everything through the proxy regardless.

## Burp Suite

You can use the SOCKS proxy to tunnel Burp traffic through the VPN via "Project options":

![](docs/burp.png)

## BBRF

If you're a [BBRF](https://github.com/honoki/bbrf-client) user, you will not be surprised to learn this integrates nicely. BBRF v1.2 contains some additional features to store and use proxy configurations.

First, I recommend deploying this project on a public VPS. (It runs nicely alongside [BBRF Server](https://github.com/honoki/bbrf-server) if you have it deployed!) Please make sure to set up authentication on your SOCKS proxies by editing the `PROXY_USER` and `PROXY_PASSWORD` variables in `.env`.

After that, configure your proxy settings in BBRF as follows:

```bash
bbrf proxy set hackerone socks5://user:pass@yourserver.com:1080
bbrf proxy set intigriti-1 socks5://user:pass@yourserver.com:1081
bbrf proxy set yeswehack socks5://user:pass@yourserver.com:1082
bbrf proxy set intigriti-2 socks5://user:pass@yourserver.com:1083
```

Now you can update or create a program's proxy settings with a custom tag `proxy` as follows:

```bash
bbrf program update my_hackerone_program -t proxy:hackerone
bbrf new secret_program -t proxy:intigriti-1
```

Now update your automation scripts to always send traffic through the right tunnel, e.g.:

```bash
# use the vpn config in whatever tool you're running
# note that the use of double quotes will allow this
# to work even if `bbrf proxy` returns an empty string
curl -x "$(bbrf proxy)" ifconfig.co
```