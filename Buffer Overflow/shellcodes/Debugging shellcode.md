> Creating shellcode and evaluting it for functionality

## Typical Bad Chars

1. Linux
	- Nullbyte: `\x00`
	- Carriage Return: `\x0a`
	- Line Feed: `\x0d`

## Remove Bad chars from selfmade shellcode

> with `msfvenom`  it's possible to alter the payload to fullfill certain conditions like not to contain `\x00` nullbytes

1. Create your shellcode
2. Modify it with `msfvenom` to fullfill your conditions
```bash
cat binfile.bin | msfvenom -p - -a -x86 --platform win -e x86/shikata_ga_nai -f c -b '\x00'
```

## Functionality check

Shellcode can be evaluated on the local system through a simple c/c++ construct as shown below:

```c
char code[] = "\x31\xc0"
"\xb8\x88\x13\x00\x00"
"\x50"
"\xbb\x11\x35\x86\x7d"
"\xff\xd3";

int main(int argc, char **argv) {
	int (*func) ();
	func = (int (*)()) code;
	(int) (*func)();
}
```

1. Replace the `code` section with your payload / shellcode
2. Compile for the target platform (obviously the same for which the payload was created)
3. Run the `.exe` or `.elf` on your local system and verify its functionality