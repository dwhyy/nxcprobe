# nxcprobe
Multi-protocol credential validator built on NetExec — test recovered credentials across SMB, LDAP, WinRM, RDP, MSSQL, SSH, and FTP in a single command.


# nxcspray

A bash wrapper around [NetExec](https://github.com/Pennyw0rth/NetExec) that automates credential spraying across multiple protocols simultaneously. Built for internal penetration testing and Active Directory assessments.

## Features

- Spray credentials across **SMB, LDAP, WinRM, RDP, MSSQL, SSH, and FTP** in a single command
- Supports **usernames, passwords, and NTLM hashes** — individually or from files
- **Port check** before attempting each protocol — skips closed ports automatically
- **WinRM fallback** — checks port 5985, falls back to 5986 automatically
- **Domain and local authentication** — spray both in one pass with `--both`
- **Target flexibility** — single IP, CIDR range, IP range, or hosts file
- Skips protocols that don't support `--local-auth` (LDAP, SSH, FTP) automatically

## Installation

```bash
sudo cp nxcspray /usr/local/bin/nxcspray
sudo chmod +x /usr/local/bin/nxcspray
```

## Usage

```
nxcspray <protocols|all> <targets> -u <username|file> -p <password|file> [options]
nxcspray <protocols|all> <targets> -u <username|file> -H <hash|file> [options]
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
nxcspray all 10.10.10.10 -u bob -p Password123

# Subnet spray
nxcspray all 10.10.10.0/24 -u bob -p Password123

# IP range spray
nxcspray all 10.10.10.1-25 -u bob -p Password123

# Hosts file
nxcspray all hosts.txt -u bob -p Password123

# File spray — users and passwords from files
nxcspray all hosts.txt -u users.txt -p passwords.txt

# Specific protocols only
nxcspray smb,ldap,winrm hosts.txt -u bob -H aad3b435b51404eeaad3b435b51404ee

# Pass-the-Hash
nxcspray smb 10.10.10.10 -u administrator -H aad3b435b51404eeaad3b435b51404ee --local-auth

# Spray domain and local auth in one pass
nxcspray all hosts.txt -u users.txt -p passwords.txt --both --continue-on-success

# Hash file spray
nxcspray smb hosts.txt -u SAMusers -H SAMNThashes --local-auth --continue-on-success
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
