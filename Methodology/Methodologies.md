> An approach of what to do when a new target (client, network, whatever) is ecnountered

## Network

1. Scan for alive targets
	1. [[Scanning#Approach on scanning]]
	2. On an exploited client
		1. Linux: [[Linux Exploitation/Information Gathering#Local enumeration]] and following
		2. Windows: [[Powershell/Information Gathering]]
		3. Both: [[Network Security/Post Exploitation#Mapping the internal network]]
2. Passively listen on the line for possible broadcast messages
	1. Attacker: Wireshark
	2. On Victim: `tcpdump`, `pktmon`
3. Scan for open ports
	1. TCP
	2. UDP
	3. ALL PORTS! (Exam probably till port 10000)

## Clients

1. Check the available services (open ports)
	1. Vulnerabilities (check version)
	2. Misconfiguration (check behaviour)
2. Exploit
3. Local enumeration
	1. Files (`txt`, `xml`, `config`, `account`, `password`, `swap-file`)
	2. Local Services (`docker`, `chronjobs` unkown apps)
	3. Persistance (get passwords, place ssh key, create new user)
4. Privesc
5. Lateral Movement
	1. Pivoting (yay)