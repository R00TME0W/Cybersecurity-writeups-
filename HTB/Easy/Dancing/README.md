# Dancing — HackTheBox Write-up

**Difficulty:** Easy
**OS:** Windows
**Category:** SMB Enumeration

---

## Overview

Dancing is an Easy-rated Windows box on HackTheBox that focuses entirely on SMB enumeration. There's no exploit chain here — the whole path to the flag comes down to correctly enumerating exposed SMB shares and finding one that doesn't require valid credentials. It's a great box for getting comfortable with `nmap` service detection and `smbclient`, two tools that show up constantly in real-world SMB assessments.

## Recon

### Nmap

```
nmap -sC -sV <IP>
```

The scan revealed several open ports typical of a Windows host running SMB services:

| Port | Service |
|---|---|
| 135/tcp | MSRPC |
| 139/tcp | NetBIOS-SSN |
| 445/tcp | SMB / Microsoft-DS |
| 548/tcp | Possibly related to Windows/NetBIOS enumeration |

Port **445/tcp** immediately stood out as the port of interest — this is the standard port for SMB running directly over TCP (as opposed to port 139, which carries SMB over NetBIOS).

### Enumeration

#### SMB — Port 445

With SMB confirmed open, the next step was to enumerate the available shares using `smbclient`:

```
smbclient -L //<IP>
```

This listed four shares:

- `ADMIN$`
- `C$`
- `IPC$`
- `WorkShares`

`ADMIN$`, `C$`, and `IPC$` are default administrative shares present on most Windows hosts. `WorkShares`, on the other hand, is a custom share — and notably, it was accessible without a valid password, making it the obvious next target.

## Foothold

Since `WorkShares` didn't require real credentials, connecting to it was straightforward:

```
smbclient //<IP>/WorkShares
```

Once inside the SMB shell, listing the contents showed two directories, each apparently tied to a different user:

```
smb: \> ls
Amy.J
James.P
```

**Checking Amy.J's directory:**

```
smb: \> cd Amy.J
smb: \Amy.J\> ls
worknotes.txt
```

The file was pulled to the local machine with:

```
smb: \Amy.J\> get worknotes.txt
```

After exiting the SMB shell and reading it locally with `cat worknotes.txt`, it turned out not to contain the flag — just some notes, but useful confirmation that the enumeration approach was working.

**Checking James.P's directory:**

Reconnecting to the share and navigating to the second user's folder turned up the actual target:

```
smb: \> cd James.P
smb: \James.P\> ls
flag.txt
```

## Root / System

The flag was downloaded the same way as the previous file:

```
smb: \James.P\> get flag.txt
smb: \James.P\> exit
```

And read locally:

```
cat flag.txt
```

This returned the flag, completing the box. No privilege escalation was needed — the misconfigured share permissions gave direct access to the flag.

## Vulnerabilities Summary

| Category | Vulnerability | Why It Mattered |
|---|---|---|
| Access Control | `WorkShares` SMB share allowed anonymous/guest access | Let an unauthenticated user browse and download files without any credentials |
| Information Disclosure | Sensitive files (`worknotes.txt`, `flag.txt`) stored in user-accessible share directories | Exposed data that should have required proper authentication to reach |
| Configuration | No restriction on share enumeration | Allowed a full share listing via `smbclient -L`, revealing internal share names and structure |

## Lessons Learned / Takeaways

- **SMB enumeration is often the whole battle.** Before jumping to exploits, checking what shares exist and whether they allow anonymous access can hand you the objective directly — no CVE required.
- **`smbclient -L` vs. connecting directly** — listing shares first gives a map of what's available before committing to a connection, which is a good habit for larger, more complex boxes.
- **Distinguishing local vs. remote command context matters.** Commands like `ls`, `cd`, and `get` run inside the `smbclient` shell, while `nmap` and `cat` run on the local machine — a small distinction, but one worth being deliberate about when documenting a walkthrough.
- This box reinforced the basics of SMB as a protocol (ports 139 vs. 445, its role in file/printer sharing on Windows networks) — fundamentals that come up again and again in more complex Active Directory boxes.

## References

- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [smbclient man page](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
