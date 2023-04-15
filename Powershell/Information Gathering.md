
- one-liner pingsweep

```powershell
1..254 | % {"192.168.1.$($_): $(Test-Connection -count 1 -comp 192.168.1.$($_) -quiet)"}
```

- portscan manually or with [powersploit](https://github.com/PowerShellMafia/PowerSploit)
```powershell
# manually:
$ports=(80,8000,8080,443,22);$ip=“10.0.10.5"; foreach ($port in $ports) {try{$socket=New-Object System.Net.Sockets.TcpClient($ip,$port);} catch{}; if ($socket -eq $null) {echo $ip":"$port" - Closed";}else{echo $ip":"$port" - Open"; $socket = $null;}}
```