```python
for x in range(1, 256):
	val = "\\x" + "{:02x}".format(x)
	if val in ["\\x00"]:
		pass
	else:
		print("\\x" + "{:02x}".format(x), end='')
print()
```
