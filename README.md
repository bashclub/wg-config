# wg-config

> Development takes place at https://code.bashclub.org/bashclub/wg-config

create a wireguard config for 2 endpoints with one command

## Installation
~~~
apt install wireguard qrencode
wget -O /usr/local/bin/wg-config https://code.bashclub.org/bashclub/wg-config/raw/branch/main/wg-config
chmod +x /usr/local/bin/wg-config
wg-config -h
~~~

`qrencode` is only needed for `-q`/`-Q`.

## Usage
~~~
usage: wg-config [-h]
  creates a wireguard configuration for both endpoints

    -a NAME              Name of side A (OPNsense instance/peer name, default: side-a)
    -e ENDPOINT_ADDR     Endpoint address on side A (omit for clients without fixed address)
    -t TUNNEL_ADDR       Tunnel address on side A
    -n NETWORKS          CIDR formatted networks accessible on side A (comma separated)
    -d DNS               DNS servers and suffixes accessible on side A (comma separated)
    -r KEY_OR_CONF       Reuse the private key of side A (key file or existing wg config)
    -f FILENAME          Save side A to specified file (default: ./NAME.conf)
    -q                   Print side A as QR code
    -o                   Print side A as OPNsense values

    -A NAME              Name of side B (OPNsense instance/peer name, default: side-b)
    -E ENDPOINT          Endpoint address on side B (omit for clients without fixed address)
    -T TUNNEL_ADDR       Tunnel address on side B
    -N NETWORKS          CIDR formatted networks accessible on side B (comma separated)
    -D DNS               DNS servers and suffixes accessible on side B (comma separated)
    -R KEY_OR_CONF       Reuse the private key of side B (key file or existing wg config)
    -F FILENAME          Save side B to specified file (default: ./NAME.conf)
    -Q                   Print side B as QR code
    -O                   Print side B as OPNsense values

    -p PORT              UDP port of side A (and of side B unless -P is set, default: 51820)
    -P PORT              UDP port of side B (e.g. the port a FRITZ!Box chose itself)
    -k KEEPALIVE         Override PersistentKeepalive (default: 25)

    -h                   displays this help text
~~~

Both tunnel addresses (`-t`, `-T`) are required. Generated files are created with mode 0600.
`Endpoint` and `PersistentKeepalive` are only written if the other side has an endpoint address.

## Which option ends up where

Rule of thumb: **`AllowedIPs` always describes the other side.** In the config of side B it lists
what side B reaches through the tunnel; in the config of side A it lists what side B may send.
`wg-quick` adds a route into the tunnel for every entry in `AllowedIPs`.

| Option      | written to config of | as           | meaning                                        |
|-------------|----------------------|--------------|------------------------------------------------|
| `-t`        | side A               | `Address`    | tunnel address of side A                       |
| `-T`        | side B               | `Address`    | tunnel address of side B                       |
| `-t` + `-n` | side B               | `AllowedIPs` | what side B reaches through the tunnel         |
| `-T` + `-N` | side A               | `AllowedIPs` | what side A accepts from side B                |
| `-e`        | side B               | `Endpoint`   | where side B connects to                       |
| `-E`        | side A               | `Endpoint`   | omitted: side A gets no `Endpoint` line        |

The own tunnel address is always added to the `AllowedIPs` of the other side, so a client
without networks behind it (laptop, smartphone) needs no `-N`.

## Examples

### Server and admin laptop / smartphone
The laptop has no fixed address, so `-E` is omitted and the server config gets no `Endpoint`.
Paste side B into the WireGuard app (laptop) or add `-Q` and scan the QR code (smartphone/tablet).
~~~
wg-config -a server -e vpn.example.com -t 10.10.10.1/24 -n 10.10.20.0/24 \
          -A thorsten -T 10.10.10.10/32 \
          -f /etc/wireguard/wg0.conf
~~~

### Another client for the same server
Reuse the server key, otherwise the existing clients stop working. Copy the `[Peer]` block
of the new server side into `/etc/wireguard/wg0.conf`.
~~~
wg-config -a server -e vpn.example.com -t 10.10.10.1/24 -n 10.10.20.0/24 -r /etc/wireguard/wg0.conf \
          -A tablet -T 10.10.10.11/32 -f ./server-tablet.conf -Q
~~~

### Site to site with OPNsense
Side B is printed as values for VPN > WireGuard (Instances and Peers), in the order of the OPNsense dialogs.
~~~
wg-config -a hq -e vpn.example.com -t 10.10.30.1/30 -n 10.10.20.0/24 \
          -A branch -E branch.example.com -T 10.10.30.2/30 -N 192.168.10.0/24 \
          -p 51821 -O
~~~

### FRITZ!Box
A FRITZ!Box imports the plain config of side B. It uses no transfer network: take the LAN
of the FRITZ!Box as tunnel address of side B (e.g. `-T 192.168.178.1/24 -N 192.168.178.0/24`).
Some firmware versions did not keep `ListenPort` on import and chose a random port. Check the
port in the FRITZ!Box after the import; if it differs, change the port in the `Endpoint` line
of side A. If the port of the FRITZ!Box is known beforehand, set it with `-P PORT`.

  ---------------------------------------------------------------------------
    (C) 2023             wg-creator by bashclub (https://github.com/bashclub)
                 Author: Thorsten Spille <thorsten@spille-edv.de>
  ---------------------------------------------------------------------------
