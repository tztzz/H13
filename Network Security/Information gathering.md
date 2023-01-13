> The scanning for targets in form of `hostnames`, `domains`, `subdomains`, `IP addresses` and `IP ranges` can be devided in passive (OSINT) and active.

| What to scan for |
| - |
| DNS, Network blocks, IP addresses, Ports, Services, OS, alive hosts, systems |

This is a cyclic process and can be repeated inside an internal network to reveal new information concerning the above mentioned items.

## Full scale infrastructure info gathering
> Can be done against public reachable instances or internal DNS server inside a company network!
>This is a boiled down howto!

1. Gather general information from outside the network

```bash
dig +nocmd targetdomain.com [+short|PTR|MX|NS|AXFR] +noall +answer @8.8.8.8
# use one of there params in sqaure brackets AXFR is zonetransfer
```

2. Gather information from within the target network

```bash
fierce --domain targetdomain.com --dns-server 192.168.0.1 # ip of internal DNS
```

3. Resolve the hostnames to IP addresses

```bash
nslookup hostname.targetdomain.com 192.168.0.1 # optional: provide IP for dns to send query to
```

4. Resolve IPs to hostname (reverse lookup)

```bash
nslookup 193.99.144.80 8.8.8.8 # resolve IP by dns server 8.8.8.8
```

5. Determine IP-blocks 
6. Check what hosts and systems are online in the different discovered IP ranges

```bash
# fping
fping -q -a -g 192.168.0.0/24 -r 0 -e # q:quiet, a:alive hosts, g:ip range, r:retries, e:elapsed time
```

```bash
# nmap do sudo!
sudo nmap -sn 192.168.0.0/24 --disable-arp-ping # sends multiple packets to check if a host is alive (80,443,icmp ...)
sudo nmap -sn -P_ 192.168.0.0/24 --disable-arp-ping # replace '_' with E/M/P/U [defaultport 80]
sudo nmap -sn -PS22,53 192.168.0.1 # sent TCP SYN to port 22 and 53
sudo nmap -sS -sU -p53 -n 192.168.0.0/24 # scan for DNS servers in the network
```

- nmap Probe options `A/S/U/Y`: SYN/ACK, UDP or SCTP (alternative to TCP/UDP)

```bash
# hping3
sudo hping3 -1 192.168.0.x --rand-dest -I eth0 # -1 icmp, -2 udp (port 0 default)
sudo hping3 -F 192.168.0.1 -c 3 # -F: FIN, -U: URGENT, -P: PUSH, -X: XMAS (flags)
```

**IMPORTANT:** Simple `ICMP` scans may not reveal all hosts as perimeter- or client firewall may filter these specifically (`nmap` uses multiple protocols by default other tools need more hands on)

7. Find new `DNS` in the target network an enumerate again for new IPs and alive hosts