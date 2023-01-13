> Scannig the network for live hosts, enumerating their ports and determining OS and provided services via tools like `nmap` and `hping3` in conjunction with wireshark.

## Some Basics

### TCP loophole

> Or: when is a port open, closed or filtered

The `TCP` standard has a rather strick definition on how a packet should be handled:

| TCP loophole |
| - |
| TCP compliant system receives packet without `SYN`, `RST`, `ACK` returns: |
| `RST` for **closed** port |
| Response ( `SYNACK` if accepted) if port is **open** |
| No response if filtered (FW drops packet) |

> Problem nowadays: Firewall and IDS may interfere in the process and alter the behavior so `nmap` returns `open|filtered` as a response and further manual checking is required

### Informations to tools

### nmap

| option | description / recommendation |
| - | - |
| `-oG` | stores the output in grepable format |
| sudo | recommended, as some operations need root access for raw sockets |

| `nmap` script categories |
| - |
| Explanation, see [here](https://nmap.org/book/nse-usage.html) |
| auth, broadcast, brute, default, discovery, dos, exploit, external, fuzzer, intrusive, malware, safe, version, vuln |

```bash
sudo nmap -sC <targetIPorNetwork>
sudo nmap --script-updatedb # update script database
nmap --script-help "ftp*" and discovery # search for ftp related scripts in category discovery
sudo nmap --script smb-enum-shares <targetIP> -p <targetPort(445)>
sudo nmap --script auth <targetIP> # execute all scripts in a category
```


### wireshark

Can be used to check the work of `hping3` and `nmap` (or when packets are created manually via `scapy`)

Filters, easy and combinable
- Protocols: `dns`, `tcp` , `udp` and `icmp` (or other protocols) add to the filter
- Logical operatiors: filters can be combined with `and` / `or` also with brackets etc.


## Approach on scanning

From a logical point of view the order should be
1. Check for live hosts in the network
2. Check open ports per host (encounter FW in this step)
3. Fingerprint the operating system (OS)
4. Determine services provided behind the ports

## Initial scan for live hosts in a network

> Simple check which IPs have live hosts to set a baseline for further enumeration of provided ports, protocols and services

- Simple ping sweep with `nmap` 

```bash
sudo nmap -PE -sn -n <targetNetwork> -oA pingsweep
```

- Sweep with `fping`

```bash
fping -q -a -g <targetNetwork>
```

- Or with `hping3`

```bash
sudo hping3 -1 10.0.10.x --rand-dest -I eth0
```


## Port scan

> Given we have enumerated live hosts and therefore IP addresses in the previous step, we can gather more information per target IP in the network.
> Starting with ports open on the different devices

For this task different approaches can be taken through using the tools already mentioned (`nmap` / `hping3` and others)

**Important:** Always scan with `TCP` (usually default) **AND** `UDP` for open ports on a system

### TCP

- Port enumeration with `nmap` 

```bash
sudo nmap -sS <targetNetwork>  # TCP SYN scan
```

- Port enumeration with `hping3`
This is on a per-host basis - if multiple hosts shall be scanned make a script

```bash
sudo hping3 -1 <targetIP> --scan 1-1000  # TCP ICMP scan
```


### UDP

```bash
sudo nmap -sU <targetIpOrNetwork> # UDP scan -> takes longer
```

```bash
sudo hping3 -2 <targetIP> --scan 1-1000 # -2 for UDP
```

> `nmap` seems to be the better solution


### Scan for open ports from specific source ports

> This can be used to avoid the first filter in our scans, when only specific source ports are allowed to communicate with a opened port

```bash
sudo nmap -sS -p <targetport> <targetIPorNetwork> --source-port <sourceport>  # source port option!
```

```bash
sudo hping3 -S -s <sourceport> -k -p <targetport> <targetIP>  # -k option to keep the sourceport steady, otherwise hping will increase by 1 
```

So in case you want to check from what sourceport a targetport is reachable, omit the `-k` option from the `hping3` command and set the sourceport to `0`.

### idle scan

> Use a network internal client as a source of information when trying to determine if ports are open and your sourceIP is not allowed / blocked...

This procedure requires a zombie host in the target network (zombie: no network traffic generated) with an steadily, global increasing `IP ID` (see [here](https://blog.apnic.net/2018/06/18/a-closer-look-at-ip-headers/) ) to be able to detect incoming traffic.

1. Identify a zombie host

`IP` and `Port` is needed to identify a zombie and to later utilize it!

```bash
sudo nmap --script ipidseq <targetIP> -p <targetPort>
```

`nmap` will state `_ipidseq: Incremental!`, wether there is much traffic or not is note stated.

```bash
sudo hping3 -S -r -p <targetPort> <targetIP>
```

For `hping3` the counter should increase only by `id=+1` in order to be useable!

2. Scan the port through your identified zombie host

Packets are forged with source IP of the zombie and IP of the target

```bash
sudo nmap -sI <zombieIP:zombiePort> <targetIP> -p <targetPort(s)> -Pn --packet-trace  # last option to see what packets are sent
```

-> Should work, but `nmap` states zombiehost not useable...

```bash
# terminal 1
sudo hping3 -S -r -p <zombiePort> <zombieIP>  # Check for id+=1
# terminal 2
sudo hping3 -a <zombieIP> -S -p <targetPort> <targetIP> # forge actual probing packet
```

`hping3` option requires 2 windows:
- terminal 1: Monitor the value of `id+=`. If `+=1` no response from host (**port closed**) if `id+=2` (or more) every time you start your command in the 2nd terminal, the port is probably **open**.
- terminal 2: send forged packets and check output in terminal 1

## OS fingerprinting

> Determine what operating system is running on the different hosts in the network, achieved through probing open ports and the behaviour of responses (depending on IP-stack implementation of the OS).

```bash
sudo nmap -A -n -iL <fileWithHostIPs>
```

This enumeration will be extensive and also take some time, at the end it has a rather good guess on the OS and also additional information on the target.
For a less aggressive approach try:

```bash
sudo nmap -O -v -n <targetIPorNetwork> --osscan-guess
```


## Service Fingerprinting

> Try guessing the services running behind the open ports.

```bash
sudo nmap -sV -iL <fileWithHostIPs> -n
```

Optional the `-A` option can be added for a more aggressive approach and for enabling the scripts!


## Other additional enumerations

Check which protocols are supported:

```bash
nmap -sO <targetIP>  # Flips BITs in IP protocol field in the packet
```
