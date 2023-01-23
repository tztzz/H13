> Gathering more detailed information about a service provided by a machine or for the machine itself

| Service | Ports | Use |
| - | - | - |
| `NetBIOS` | 137(udp), 138(udp), 139(tcp) | Basic communication inside a local network. Can also be used for sharing files over the network. |
| `SMB` |  139, 445 | Server Message Block, basic file operations (change, share). Also messaging in combination with `IPC` |
| `SNMP` |  161(udp), 162(udp) | 161 primary port, 162 used for the messaging part. Primarily used for NW device communication and management | 

First ensure the services are available via a prior portscan as mentioned in [[Scanning]]

## NetBIOS

### 1. Enumerate services provided by remote host

**Windows**
```bash
nbtstat -A <targetIP>
> ...
> SRV77 <00> UNIQUE Registered # computername
> SRV77 <20> UNIQUE Registered # File & Printershare enabled!!
> FOO <00> GROUP Registered # Workgroup or domain
> ...
> MAC Address = 11-22-33-44-55-66
```

For detailed information on what the different suffix bytes stand for, check [MS-Docs](https://msdn.microsoft.com/en-us/library/cc224454.aspx)

**Linux**
```bash
nbtscan -v <targetIPorNetwork>  # same output as for windows
```

### 2. Display shares provided by remote host

**Windows**
```bash
net view <targetIP> # output self explanatory
```

**Linux**
```bash
smbclient -L <targetIP> # output also lists hidden shares
```

`smbclient` also lists hidden shares (usually shares with a trailing `$` after their name - `C$` aka admin share)

### 3. Mount enumerated shares

**Windows**
```bash
net use k: \\<targetIP>\<sharename> # use k or any other free letter (like C:\)
```

**Linux**
```bash
sudo mount.cifs //<targetIP>/<sharename> /media/K_share/ user=,pass= # anonymous login
```


### NULL sessions

> Allows displaying of information as `anonymous` user (user/pass empty), therefore `NULL` session

**Windows**
```bash
net use \\<target>\IPC$ "" /u:"" # u: user, other is empty password
> The command completed successfully.
```

Information dump with Windows can be done with additional tools - preferably use linux...

**Linux**
```bash
enum4linux <targetIP>  # default enumerates about 6 options
enum4linux -a <targetIP> -u <user> -p <pass>  # with credentials
```

As an alternative, you may use the interactive `rpcclient`, can also be used with credentials

```bash
rpcclient -N -U "" <targetIP> # N: nopwd, U: username
> help  # display all cmds - has autocomplete!
> enumalsgroups
> srvinfo
> enumprivs
```


## SNMP

> Simple Network Management Protocol, communication of network devices and their monitoring is mainly handled with this protocol - but can also expose data from servers

Versions
- `SNMPv1` and `SNMPv2`: Both use cleartext and are therefore prone to leak information
- `SNMPv3`: Uses encryption, but is still *vulnerable* to brute force attemtps

| Default community strings |
| - |
| public  /  private |

### 1. Sniff the network 

- Sniff the network for traps and community strings
- Read permission will suffice
- Scan for hosts with open `SNMP` ports and an exposed service

### 2. Try default community strings / brute force 

```bash
sudo nmap -sU -p 161 --script snmp-brute <targetIP> --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

> 161/udp open snmp
> snmp-brute:
>  public - Valid credentials  # string 'public' is a valid community string
>  private - Valid credentials # 'private' seems to be rw permission com. str.
```

To find other scripts from `nmap` concerning `snmp` search in the script folder:

```bash
ls -l /usr/share/nmap/scripts | grep -i snmp
```

Another tool is `onesixtyone`:

```bash
onesixtyone -c /usr/share/doc/onesixtyone/dict.txt <targetIP>
```

### 3. Extract information via SNMP

```bash
snmp-check <targetIP> -c <validCommunityString>
```

May reveal usernames, shares, OS, HWspecs, IPs, NICS etc.

```bash
snmpenum <targetIP> <communityString> <configfile>
# configfile under /usr/share/snmpenum/windows.txt
```

`snmpenum` is an alternative tool (has to be installed) and outputs the `OID`s in a more readable format (also comment out line 4 in `/etc/snmp/snmp.conf` to have it work)

### 4. Query specific attributes / query all values / write values

```bash
# query specific value
snmpwalk -v 2c -c public 192.168.22.33 system.sysContact.0
snmpwalk -v <snmpVersion> -c <communityString> <targetIP> <param>

# write specific value
snmpset -v 2c -c public 192.168.22.33 system.sysContact.0 s newvalue
snmpset -v <version> -c <communityString> <targetIP> <OID> <valueType> <newValue>
```

Interesting params, the `0` should select the value - may try with or without...
- `system.sysContact.0`
- `system.hrSWInstalledName`

To query all values from a remote system use

```bash
snmpwalk -v <snmpVersion> -c <communityString> <targetIP>
```

The walk may take some time - usually the enumeration in step 3 yields the same usable information.