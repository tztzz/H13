> Short overview / summary of metasploit commands and specifics, when to use what and so on

## Top level

> Meaning you just started `msfconsole` and haven't entered a session

Searching for Exploits, Scanners and all kind of modules

```bash
msf> search type:exploit java
msf> use 133  # short cmd if you know the index of the module
```

Grep in msfconsole needs to be stated upfront the command which output you actually want to grep:

```bash
msf> grep excellent search type:exploit java
```

## Inside a session

Through `run` scripts can be invoked.

```bash
meterpreter> run post/windows/manage/migrate
```

Portforwarding, `<remoteIP>` and the port to your local machine port (through the current session!) -> check the [[Pivoting]] section!

```bash
meterpreter> portfwd add -L 127.0.0.1 -l 3389 -r <remoteIP> -p 3389
```

run as (`runas` in the windows commandline) via meterpreter

```bash
msf> use post/windows/manage/run_as
msf> set CMD c:\\path\\to\\your\\placed\\file.exe
msf> set user admin
...
```


## Start a handler

> Used if `msfvenom` payloads are deployed

```bash
msf> use exploit/multi/handler
...  # set options accordingly
```

## The loot folder

> Executions of scripts can result in depositing of information in the loot folder

```bash
cd /home/kali/.msf4/loot
```

