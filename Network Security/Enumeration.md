> Gathering more detailed information about a service provided by a machine or for the machine itself

| Service | Ports | Use |
| - | - | - |
| `NetBIOS` | 137(udp), 138(udp), 139(tcp) | Basic communication inside a local network. Can also be used for sharing files over the network. |
| `SMB` |  139, 445 | Server Message Block, basic file operations (change, share). Also messaging in combination with `IPC` |


## Enumeration

First ensure the services are available via a prio portscan as mentioned in [[Scanning]]

### NetBIOS

#### 1. Enumerate services provided by remote host

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

#### 2. Display shares provided by remote host

**Windows**
```bash
net view <targetIP> # output self explanatory
```

**Linux**
```bash
smbclient -L <targetIP> # output also lists hidden shares
```

`smbclient` also lists hidden shares (usually shares with a trailing `$` after their name - `C$` aka admin share)

#### 3. Mount enumerated shares

**Windows**
```bash
net use k: \\<targetIP>\<sharename> # use k or any other free letter (like C:\)
```

**Linux**
```bash
sudo mount.cifs //<targetIP>/<sharename> /media/K_share/ user=,pass= # anonymous login
```


#### NULL sessions

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


