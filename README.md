# OSB Kernel

**OSB (Operating System Basic)** is a bootable, bare-metal operating system kernel written in C# and compiled with NativeAOT. It runs directly on x86-64 hardware (or QEMU) with no host OS, no BCL, and no runtime dependencies.

The kernel provides a DOS-like user experience: drive letters, backslash paths, internal commands (`DIR`, `CD`, `TYPE`, `COPY`, `DEL`, `GREP`, ...), external programs, preemptive multitasking, and a text console over VGA and serial.

## Features

- **Bootable** — Multiboot2 ELF, boots from ISO or raw disk image
- **Freestanding** — no .NET runtime, no BCL, no GC; pure NativeAOT
- **Memory management** — physical frame allocator, per-process address spaces
- **Preemptive scheduler** — round-robin over a task table, ring-3 user processes
- **ext4 filesystem** — ATA PIO disk driver with simplified ext4 read/write on `A:`
- **DOS-like shell** — `DIR`, `CD`, `TYPE`, `COPY`, `DEL`, `REN`, `MD`, `RD`, `GREP`, `TRASH`, `USER`, `SUDO`, pipes (`A | B`), and more
- **Per-user trash** — `DEL` moves files to `$HOME\TRASH`; `TRASH /U` undeletes
- **User system** — accounts, passwords, sudoers, ownership and permission bits
- **OSLANG interpreter** — embedded 0.72 interpreter for `.OSL` scripts
- **Pipes** — `A | B` captures A's console output as B's input
- **Per-command help** — every command accepts `/?` for a help page

## Building

Requires .NET SDK 8.0+ and `mke2fs` (e2fsprogs).

```bash
cd Osb.Kernel
./build.sh          # builds osb.iso
./build.sh run      # build + run in QEMU (serial stdio)
./build.sh run-gui  # build + run with VGA window
./build.sh check    # validate Multiboot2 ELF
```

## Running

Boot `osb.iso` in QEMU, VirtualBox, or on real hardware. The kernel starts a login prompt — log in as `root` (passwordless by default).

```
login: root
root@OSB A:\OSB\ROOT> DIR
root@OSB A:\OSB\ROOT> HELP
root@OSB A:\OSB\ROOT> OSLANG SCRIPT.OSL
```

## Repository Structure

```
Osb.Kernel/
├── Osb.Kernel.csproj        # Project file
├── Kernel.cs                # Boot orchestration
├── boot.S / sched.S / user.S # Assembly entry points
├── linker.ld                # Linker script
├── grub.cfg                 # GRUB configuration
├── build.sh                 # Build script
├── Cpu/                     # GDT, IDT, PIT, RTC, syscalls
├── Hardware/                # VGA, serial, keyboard, console
├── Memory/                  # Frame allocator, layout constants
├── Processes/               # Scheduler, task table, address spaces
├── Storage/                 # ATA PIO, ext4, CWD
├── Shell/                   # Shell, environment, users, commands
│   └── Commands/            # DIR, CD, TYPE, DEL, GREP, TRASH, ...
├── Oslang*.cs               # Embedded OSLANG 0.72 interpreter
├── MiniCoreLib.cs           # Minimal runtime stubs for NativeAOT
└── README.md                # This file
```

## Architecture

```
Boot (boot.S)
  → PIC/PIT/RTC init
  → GDT + IDT
  → Physical memory detection
  → Scheduler init
  → Environment + Shell
  → Login prompt
```

The kernel is fully freestanding: no exceptions in normal control flow, no managed heap allocation, fixed buffers and stackalloc throughout.

## License

[MIT](LICENSE)
