> Section for specific knowledge concerning operating systems and special cases

## Linux

tbd

## Windows

### Run programs in another user context within shell

Requirements
- User and password

Run an application in a shell without graphical interface if message `this program cannot be run in DOS mode` appears

```powershell
cmd> runas /user:<username> app.exe
```

