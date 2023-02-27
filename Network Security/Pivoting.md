> This section is a collection of pivoting techniques. Mainly commands with the metasploit framework

Through pivoting it is possible to scan/enumerate/exploit targets in a new subnet or behind a firewall/DMZ from a bastion host (host acts as some kind of proxy).

## Enable pivoting through an established meterpreter session

### 0. Prerequisites

- established meterpreter session on a host
- Victim host must have connection to other NW or machine

### 1. Create a route into another network

```bash
msf> use post/multi/manage/autoroute
msf> set SESSION 1  # session on bastion host
msf> set SUBNET 10.10.10.0  # target network
msf> set NETMASK /16  # if different from /24
msf> run
```

Routes can be display with the `route` command in the console.

Also they can be added 'manually':

```bash
msf> route add <networkIP> <subnetMask> <sessionNumber>
msf> route add <networkIP>/24 <sessionsNumber>
```

### 2. Using the Pivot within metasploit frameowrk

This should be working already. If traffic is to go to the network covered by the defined route, metasploit will handly it automatically.

### 3. Start the `SOCKS` proxy for outside tools to pivot

```bash
msf> use auxiliary/server/socks_proxy
msf> set VERSION 4a  # optionally version 5 works also
msf> run
```

Socks proxy is started as a job in metasploit and runs in the background (check via `jobs` command).

- Also check if local config for proxychains matches the `PORT` and `VERSION` parameter stated within metasploit.

```bash
sudo nano /etc/proxychains4.conf  # or similar config name
> version 4a  # if using version 4
> port 1080
```

With this config and metasploit running the socks server, we can use `proxychains` in order to use other tools over this tunnel.

### 4. Use `proxychains` with `nmap`

> Remember: `SOCKS` is a service on the upper layers of the OSI model - `ICMP` and other low-level functions will not work with proxychains!

```bash
# scan ftp port with nmap
sudo proxychains nmap -Pn -sTV -n -p21 <targetIP>

# scan top 50 (most common) ports with nmap
sudo proxychains nmap -sT -Pn -n <targetIP> --top-ports 50
```

## Portforwarding

> Using metaslpoit

```bash
meterpreter> portfwd add -l <localPort> -r <targetIP> -p <targetPort>
meterpreter> portfwd add -l 3333 -r 10.0.0.15 -p 3389
```

Example above: `RDP` connect to local port 3333 will actually goto 10.0.0.15:3389

![[m6_portforward.png]]

Listener on attacker machine forwarding traffic to the target over the compromised host. Requires an already exploited victim to use as intermediary.

