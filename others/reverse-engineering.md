# Reverse engineering

## Summary
* [Analyse binary files](#analyse-binary-files)
	* [Analyse an ELF file](#analyse-an-elf-file)
	* [Analyse ELF file with GDB](#analyse-elf-file-with-gdb-on-cli)

## Analyse binary files
### Analyse an ELF file
```
rabin2 -g [elf file]
use ghidra
use strings command
```
### Analyze ELF file with GDB on CLI
```
> gdb [elf file]
> info functions --> list different functions in the binary
> b *0x0000000040281 --> sets a breakpoint
> run test
> info registers --> view the current state of registers
> x/s 0x7fffffffe2d0 --> print the strings at this memory address
```
