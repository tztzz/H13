- C2 Framework [EMPIRE](https://github.com/BC-SECURITY/Empire) (Install manually - kali is fked up, installation takes 10-15 minutes)
	- Defaultcreds for *StarKiller*: `empireadmin:password123`

## Pivoting with empire
> The builtin socks proxy only forwards onto on target port...

1. Run the builtin webproxy within empire
2. Invoke the `powershell_management_invoke_socksproxy` and use attacker IP as `remote` system
3. execute
4. use `proxychains` for usage of the victim host