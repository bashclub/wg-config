# wg-config

create a wireguard config for 2 endpoints with one command

#ä Installation
~~~
apt install wireguard qrencode
wget -O /usr/local/bin/wg-config https://git.bashclub.org/bashclub/wg-config/raw/branch/main/wg-config
chmod +x /usr/local/bin/wg-config
wg-config -h
~~~

## Usage
~~~
usage: wg_creator.sh [-h] 
  creates a wireguard configuration for both endpoints

    -e ENDPOINT_ADDR     Endpoint address on side A
    -t TUNNEL_ADDR       Tunnel address on side A
    -n NETWORKS          CIDR formatted networks accessible on side A (comma separated)
    -d DNS               DNS servers and suffixes accessible on side A (comma separated)
    -f FILENAME          Save side A to specified file (default: ./side-a.conf)
    -q                   Print side A as QR code
    -o                   Print side A as OPNsense template

    -E ENDPOINT          Endpoint address on side B
    -T TUNNEL_ADDR       Tunnel address on side B
    -N NETWORKS          CIDR formatted networks accessible on side B (comma separated)
    -D DNS               DNS servers and suffixes accessible on side B (comma separated)
    -F FILENAME          Save side B to specified file (default: ./side-b.conf)
    -Q                   Print side B as QR code
    -O                   Print side B as OPNsense template

    -p PORT              UDP Port (used on both sides, default: 51820)
    -k KEEPALIVE         Override PersistentKeepalive (default: 25)

    -h                   displays this help text
  ---------------------------------------------------------------------------
    (C) 2023             wg-creator by bashclub (https://github.com/bashclub)
                 Author: Thorsten Spille <thorsten@spille-edv.de>
  ---------------------------------------------------------------------------
  ~~~