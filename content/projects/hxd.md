---
title: hxd
date: 2026-11-25
description: A hexdump tool in Go, with offsets and length limits.
tags: [go, tools]
---

[hxd](https://github.com/KalebHawkins/hxd) prints a file as a hex dump. Each line shows 16 bytes: the offset, the bytes in hex, and the printable ASCII characters.

```
$ hxd -f game.exe -s 0x40 -n 48
0x00000040: 0e 1f ba 0e | 00 b4 09 cd | 21 b8 01 4c | cd 21 54 68 | ........!..L.!Th
0x00000050: 69 73 20 70 | 72 6f 67 72 | 61 6d 20 63 | 61 6e 6e 6f | is program canno
0x00000060: 74 20 62 65 | 20 72 75 6e | 20 69 6e 20 | 44 4f 53 20 | t be run in DOS
```

## Flags

| Flag | Meaning |
|---|---|
| `-f`, `--file` | File to dump |
| `-s`, `--offset` | Start offset, decimal or prefixed (`0x40`, `0o100`) |
| `-n`, `--length` | Maximum bytes to dump |

## What I learned building it

The read loop is the core of the tool. `Read` can return fewer bytes than the buffer holds, so every step uses `buf[:n]`, never the whole buffer. Getting that wrong produced a size check that counted 224 bytes in a 203-byte file.

`-n` wraps the file in `io.LimitReader` instead of adding counting logic to the loop. `-s` uses `Seek`, and the offset column starts from the position `Seek` returns, so the labels always match the file.

The first version wrote each byte straight to stdout. Dumping a 4 MB file took 26.5 seconds. Building each line in a `strings.Builder` and writing once per line brought it to 1.7 seconds.

The dump function takes an `io.Reader` and an `io.Writer`, so it does not care whether the bytes come from a file, stdin, or another process's memory. That last one is where this is heading.
