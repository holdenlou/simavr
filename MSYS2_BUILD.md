# Building simavr on Windows (MSYS2 / mingw64)

Tested on MSYS2 mingw64 (GCC 16.x). The POSIX-only pieces (`uart_pty`) and the
GLUT display widgets are auto-skipped; the core library, the `simavr` CLI and
`libsimavrparts` (including `uart_udp`) all build.

## Prerequisites

    pacman -S make        # make is NOT preinstalled in MSYS2

There is no libelf on mingw, so:

- `.elf` firmware loading is NOT supported (`ELF format is not supported by
  this build`).
- `.hex` firmware loading WORKS (built-in IHEX parser, no libelf):

      simavr -m atmega328p -f 16000000 D:/path/fw.hex

Pass Windows `D:/...` paths to the `simavr` binary (mingw programs cannot
resolve MSYS `/d/...` paths — they fail with "No such file or directory").

## Build + install

    make                      # builds libsimavr.a + run_avr (simavr CLI)
    make install              # -> /usr/bin/simavr, /usr/lib/libsimavr.a, /usr/include/simavr/
    make -C examples/parts    # builds libsimavrparts.a (incl. uart_udp; skips PTY + GLUT)

## UART over UDP

`uart_udp` is the UART bridge usable on Windows (PTY is not supported). Hosts
that use it must:

- call `WSAStartup` before any socket use (see davm-imp's `uart_udp_host.c`),
- link `-lws2_32 -lpthread`.

## What was patched vs upstream buserror/simavr

- `Makefile.common`: on Msys, `LIBDIR` is built with `cygpath -m` so mingw's
  cc can resolve the library path (otherwise the `run_avr.elf` link fails with
  `cannot find -lsimavr`).
- `examples/parts/Makefile`: on Msys, also filter out `*_glut.o` (no GLUT).
