---
title: "WSL: Real Linux Inside Your Windows"
date: 2026-07-12
draft: false
tags: ["wsl", "linux", "windows"]
categories: ["infrastructure"]
description: "What WSL is, how to install WSL2, manage distributions, understand the two file systems, run cross-OS commands, and limit resource usage with .wslconfig."
---

> 🎥 This article has a companion video on my [YouTube channel](https://www.youtube.com/@ropizzolato) (in Portuguese). The video link will be added here once it's published.
<!-- When the video is published, replace the line above with: {{</* youtube VIDEO_ID */>}} -->

**WSL (Windows Subsystem for Linux)** is a Windows 10/11 feature that lets you use real Linux resources directly from Windows, officially supported by Microsoft. It's not new, but it remains one of the most practical ways to study and work with Linux — no dual-boot, no hypervisor setup. If you're an IT student getting started, this is probably the simplest entry point.

## What is WSL?

WSL is a compatibility layer that runs **real Linux** inside Windows. It's not traditional emulation or a classic virtual machine — although under the hood WSL2 uses lightweight virtualization via Hyper-V.

WSL is currently at **version 2**. Version 1, simpler and less compatible, is nearly obsolete — avoid it. WSL2 advantages:

- Real Linux kernel
- Near-total compatibility
- Better performance in many scenarios
- Lightweight virtualization (via Hyper-V)

## What can you do with WSL?

A lot: run `bash`, `ssh`, `grep`, and other tools; use package managers like `apt` and `yum`; run Python, Node, Go, Docker, and Git; compile code; automate tasks; access Linux servers. For many professionals, WSL has completely replaced dual-boot.

A huge differentiator is the **Windows integration**: you can reach your `C:\` drive through the `/mnt/c` mount point, run Linux commands from PowerShell (and Windows commands from inside Linux), and integrate with VS Code.

### Limitations and caveats

- WSL is **not a replacement for a production Linux server** — I/O performance can vary a lot.
- Watch out for security: handle sensitive files with care.
- Not every corporate environment allows WSL.

Where I recommend it: study environments, development, task automation, and hybrid setups — when you natively live on Windows but still need Linux.

## Installing WSL2

The official documentation is at [learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install). Prerequisites: Windows 10 version 2004+ (build 19041+) or Windows 11 — the most common case today, since Windows 10 support ended in October 2025.

Open **PowerShell as administrator** and run:

```powershell
wsl --install
```

Wait for it to finish, then verify the installed version:

```powershell
wsl --version
```

## Managing distributions

List what's installed and what's available online:

```powershell
wsl --list --verbose
wsl --list --online
```

I'll install Ubuntu 24.04 (an LTS release, with extended support) and also Fedora 43:

```powershell
wsl --install -d Ubuntu-24.04
wsl --install -d FedoraLinux-43
```

After each install, set a username and password. To confirm you're on real Linux:

```bash
cat /etc/os-release
```

To leave WSL and return to PowerShell, type `exit`.

Other useful commands:

```powershell
wsl --list                       # list installed distros
wsl --unregister <DistroName>    # remove a distro
wsl --setdefault Ubuntu-24.04    # set the default distro (or -s)
wsl -u root                      # log in as root (handy to reset a password with passwd)
wsl --help                       # all options
```

Once you're inside your distro, the first thing I suggest is updating the system:

```bash
sudo apt update && sudo apt upgrade
```

The `&&` chains commands: the second one only runs if the first succeeds (return code 0). You can check the last command's return code with `echo $?` — try `ls` on a non-existent path and watch it change to 2. `man ls` documents the *Exit Status* values.

## The two file systems

This is the most important concept for using WSL well:

| Path | File system | What it is |
| --- | --- | --- |
| `/mnt/c/...` | NTFS (Windows) | Your `C:\` drive, mounted inside Linux |
| `/home/<user>/...` | ext4 (Linux) | Virtual disk managed by WSL |
| `\\wsl$\<Distro>\...` | ext4 (Linux) | A bridge for Windows to reach the Linux filesystem |

The path `C:\Users\<User>\Project` is equivalent to `/mnt/c/Users/<User>/Project` — same files. Meanwhile `/home/<user>` lives on an ext4 virtual disk that isn't directly on NTFS.

Try it yourself: inside WSL, run `cd /mnt/c/`, create a folder with `mkdir` and a file with `touch`, then check Windows File Explorer — they're there.

**Important tip:** for development environments, avoid working under `/mnt/c` — keep your projects in your user's `/home`, where performance is much better.

## Windows and VS Code integration

From inside your `/home`, you can call Windows programs:

```bash
explorer.exe .   # opens File Explorer in the current folder
code .           # opens VS Code in the current folder (via the WSL extension)
```

The trailing dot means the current directory (use `pwd` to see where you are). In VS Code, install the official Microsoft **WSL** extension if it doesn't open in remote mode automatically.

Let's test something that only works on Linux. Create a `wsl` folder, and inside it a `hello.sh` file:

```bash
#!/bin/bash
echo "Hello from Linux via WSL"
date
uname -a
```

Make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh
```

`uname -a` prints system information — including the real Linux kernel running inside Windows.

## Cross-OS commands

You can mix both worlds. In **PowerShell**, running a Linux command:

```powershell
ipconfig | wsl grep -i ipv4
```

`ipconfig` is native to Windows; `grep` filters the output on Linux (`-i` makes it case-insensitive). A plain `wsl ls` instead of `dir` works too.

The reverse also works — Windows commands **inside WSL** (don't forget the `.exe`):

```bash
notepad.exe .wslconfig
ls -la | findstr.exe "search_term"
```

Remember: to run a Linux tool from PowerShell, it must be installed in the distro. On Ubuntu/Debian, check with `dpkg -l | grep <package>` and install with `apt install`. On Fedora/Red Hat, use `rpm -qa | grep <package>` and `yum install`. A classic example: `ifconfig` (legacy, replaced by the `ip` command) requires the `net-tools` package:

```bash
sudo apt install net-tools -y    # Ubuntu/Debian
sudo yum install net-tools -y    # Fedora/Red Hat
```

The `-y` flag answers "yes" automatically to the package manager's prompt. Worth noting: `apt` and `yum` resolve dependencies automatically — unlike raw `dpkg` and `rpm`.

## Limiting resources with .wslconfig

WSL loves to consume resources — it's like Chrome devouring RAM. Without limits, it keeps eating memory until your system starts caching to disk and slows down. The fix is the `.wslconfig` file at `%UserProfile%\.wslconfig` (that is, `C:\Users\<your_user>\.wslconfig`):

```ini
[wsl2]
memory=4GB
processors=2
swap=2GB
swapFile=C:\\wsl\\swap.vhdx
localhostForwarding=true
```

- `memory`, `processors`, and `swap` cap WSL2's global usage.
- `swapFile` sets where the swap file lives (the backslash is doubled because it's an escape character). Without it, the default is `%UserProfile%\AppData\Local\Temp\swap.vhdx`.
- `localhostForwarding=true` is great for exposing applications — for example, `python3 -m http.server 8000` inside WSL, then open `localhost:8000` in a Windows browser.

The [full list of settings is in the official docs](https://learn.microsoft.com/en-us/windows/wsl/wsl-config). You can also set limits **per distro**, but that's a topic for another article.

## Shutting down WSL

A good practice is shutting WSL down when you're done, freeing memory and CPU:

```powershell
wsl --shutdown
```

In rare cases the process insists on staying alive. If that happens, force-kill it:

```powershell
taskkill /f /im wslservice.exe
```

`/f` forces termination and `/im` targets the process by name (you can use `/pid` as well).

## Conclusion

WSL has many more features — this article is the foundation for upcoming content on WSL, Linux, and systems administration. If you have questions or topic suggestions, find me on [YouTube](https://www.youtube.com/@ropizzolato), [GitHub](https://github.com/rpizzolato), or [LinkedIn](https://linkedin.com/in/rpizzolato).
