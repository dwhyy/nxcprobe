# nxcprobe
Multi-protocol credential validator built on NetExec — test recovered credentials across SMB, LDAP, WinRM, RDP, MSSQL, SSH, and FTP in a single command.

A bash wrapper around [NetExec](https://github.com/Pennyw0rth/NetExec) that automates credential spraying across multiple protocols simultaneously. Built for internal penetration testing and Active Directory assessments.

## Features

- Spray credentials across **SMB, LDAP, WinRM, RDP, MSSQL, SSH, and FTP** in a single command
- Supports **usernames, passwords, and NTLM hashes** — individually or from files
- **Port check** before attempting each protocol — skips closed ports automatically
- **WinRM fallback** — checks port 5985, falls back to 5986 automatically
- **Domain and local authentication** — spray both in one pass with `--both`
- **Target flexibility** — single IP, CIDR range, IP range, or hosts file
- Skips protocols that don't support `--local-auth` (LDAP, SSH, FTP) automatically

## Use Cases

**Penetration Testing & Red Team**
After recovering credentials through any means — SAM/SYSTEM dumps, LSA secrets, Kerberoasting, plaintext config files — quickly validate where those credentials work across the entire environment without manually running netexec per protocol.

**Multi-Protocol Coverage**
In real-world environments and certification labs like OSCP/CPTS, relying on a single protocol risks missing valid authentication paths. SMB may be disabled or firewalled on a host while WinRM or RDP remains open — a local credential that appears dead on SMB could grant full access over a different protocol. nxcprobe covers all supported protocols simultaneously, ensuring no valid path is missed.

**Active Directory Assessments**
Spray recovered domain and local credentials across all in-scope hosts in one pass. The `--both` flag handles domain and local authentication simultaneously, and protocols that don't support local auth (LDAP, SSH, FTP) are skipped automatically to prevent errors and wasted time.

## Installation

```bash
git clone https://github.com/dwhyy/nxcprobe.git
cd nxcprobe
sudo cp nxcprobe /usr/local/bin/nxcprobe
sudo chmod +x /usr/local/bin/nxcprobe
```

## Usage

```
nxcprobe <protocols|all> <targets> -u <username|file> -p <password|file> [options]
nxcprobe <protocols|all> <targets> -u <username|file> -H <hash|file> [options]
```

Protocols can be specified as a comma-separated list with no spaces (e.g. `smb,winrm,rdp`) or use `all` to spray every supported protocol.

### Options

| Flag | Description |
|---|---|
| `-u` | Username or path to username file |
| `-p` | Password or path to password file |
| `-H` | NTLM hash or path to hash file |
| `--local-auth` | Use local authentication only |
| `--both` | Spray domain auth then local auth |
| `--no-bruteforce` | Pair each username with its corresponding password (no cross-product) |
| `--continue-on-success` | Continue spraying after a hit |
| `--no-port-check` | Skip port check and attempt all protocols regardless |

## Examples

```bash
# Single target, all protocols
nxcprobe all 10.10.10.10 -u bob -p Password123

# Subnet spray
nxcprobe all 10.10.10.0/24 -u bob -p Password123

# IP range spray
nxcprobe all 10.10.10.1-25 -u bob -p Password123

# Hosts file
nxcprobe all hosts.txt -u bob -p Password123

# File spray — users and passwords from files
nxcprobe all hosts.txt -u users.txt -p passwords.txt

# Specific protocols only
nxcprobe smb,ldap,winrm hosts.txt -u bob -H aad3b435b51404eeaad3b435b51404ee

# Pass-the-Hash
nxcprobe smb 10.10.10.10 -u administrator -H aad3b435b51404eeaad3b435b51404ee --local-auth

# Spray domain and local auth in one pass
nxcprobe all hosts.txt -u users.txt -p passwords.txt --both --continue-on-success

# Hash file spray
nxcprobe smb hosts.txt -u SAMusers -H SAMNThashes --local-auth --continue-on-success
```

## Protocol Defaults

| Protocol | Port | Local Auth |
|---|---|---|
| SMB | 445 | ✓ |
| LDAP | 389 | ✗ |
| WinRM | 5985 / 5986 | ✓ |
| RDP | 3389 | ✓ |
| MSSQL | 1433 | ✓ |
| SSH | 22 | ✗ |
| FTP | 21 | ✗ |

WinRM checks port 5985 first and falls back to 5986 automatically.

## Target Format

| Format | Example |
|---|---|
| Single IP | `10.10.10.10` |
| CIDR | `10.10.10.0/24` |
| Range | `10.10.10.1-25` |
| Hosts file | `hosts.txt` |

Hosts files support `#` comments and blank lines — both are skipped automatically.

## Requirements

- [NetExec](https://github.com/Pennyw0rth/NetExec) (`nxc`) installed and in PATH
- Python 3 (for CIDR expansion)
- Bash 4+

## Notes

- Passwords and hashes with special characters are automatically quoted when passed as strings
- Files are passed unquoted so NetExec reads them directly
- Port check has a 3 second timeout per host per protocol
- `--local-auth` is silently skipped for protocols that don't support it (LDAP, SSH, FTP)
