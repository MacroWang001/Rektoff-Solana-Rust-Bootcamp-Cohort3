# Commands

**Note:** This document is a reference to Daniel’s GitHub repository.[^1]

Some commands to inspect `.data`, `.bss`, and `.rodata` sections in a compiled Rust (or C) program using standard Linux command-line tools. You can use `grep` on the output files to search for your uniquely named string literals or variable names, such as `FIRST_CONST` or `SECOND`, to confirm where they are stored in memory. To learn more about the commands you can use run `man readelf`, or `man objdump` to read the manuals - or you could use google.

First, compile the Rust program to a binary:
```bash
rustc -C opt-level=0 -C debuginfo=0 src/main.rs -o target_binary
```
- `-C opt-level=0` disables optimizations to preserve symbols.
- `-C debuginfo=0` omits debug info to keep the binary minimal.

To list all sections in the binary:
```bash
readelf -S ./target_binary > out-readelf-sections
```
- `-S` prints section headers from the ELF file.
- Look/search for the `.data`, `.bss`, `.rodata` sections.

To view the contents of the read-only data segment:
```bash
objdump -s -j .rodata ./target_binary > out-objdump-rodata
```
- `-s` dumps section contents.
- `-j .rodata` specifies the `.rodata` segment.

To view the contents of the initialized writable data segment:
```bash
objdump -s -j .data ./target_binary > out-objdump-data
```
- Same as above, but for `.data`.
- Not easy to interpret as is pointer and integer not characters

To view zero-initialized statics (metadata only):
```bash
objdump -s -j .bss ./target_binary > out-objdump-bss
```
- `.bss` will be empty / zeroed out as it is data that is zero-initialised.

To list global symbols and their segment classification:
```bash
nm -C ./target_binary > out-nm
```
- `-C` demangles Rust symbol names for readability.

To confirm if `.bss` is present and see its size:
```bash
readelf -S ./target_binary | grep .bss > out-bss-section
```

Summary of segment purposes:

| Segment   | Purpose                                | Writable | In File        |
|-----------|----------------------------------------|----------|----------------|
| `.rodata` | Constants, string literals              | No       | Yes            |
| `.data`   | Initialized static/global variables     | Yes      | Yes            |

| `.bss`    | Zero-initialized static/global vars     | Yes      | No (size only) |


[^1]: https://github.com/dkcumming/statics.

