> The fundamentals...

- Display help for a `cmdlet`:
```powershell
Get-Help Get-Process -Examples  # Get-Process beeing the cmdlet
```

- Location: `C:\Windows\System32\WindowsPowerShell\v1.0\`
- Versioncheck: `$PSVersionTable`
- ProcessCheck: `[Environment]::Is64BitProcess`
- Get 64-Bit version
	- `C:\windows\system32\WindowsPowerShell`
	- 32-Bit: `C:\windows\SysWOW64\WindowsPowerShell`

### Interesting cli switches for invoking powershell

- `-ExecutionPolicy bypass`: avoid restriction to not be able to run a script
	- shortened: `-ep Bypass` or `-ex by`
- `-WindowStyle hidden`: hide the window
	- shortened: `-W h`
- `-Command Get-Process` / `-Command "& { Get-EventLog -LogName security }"`
- `-EncodedCommand $encodedCommand`
	- shortened: `-ec`
- `-NoProfile`
- `-Version 2`: force usage of version 2

### Search for cmdlets

```powershell
Get-Command -Name *Firewall*  # search for cmdlet with firewall in name
```