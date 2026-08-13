# Wireshark FTP Credential Sniffing Lab

Kali Linux + Ubuntu VMs · packet capture and analysis · portfolio project

## Overview

This lab demonstrates why plaintext network protocols are a security risk. Using two virtual machines — Kali Linux as the attacker/analyst and Ubuntu Server as the target — I set up an FTP server, captured the network traffic between the two machines with Wireshark, and recovered a login username and password sent in cleartext.

**Skills demonstrated:** virtual machine networking, Wireshark packet capture, protocol analysis, display filtering, TCP stream reconstruction, and identifying an authentication vulnerability.

## Lab Environment

| Component | Detail |
|---|---|
| Attacker / analyst machine | Kali Linux, IP `192.168.74.128` |
| Target machine | Ubuntu Server, IP `192.168.74.131` |
| Hypervisor | VMware, both VMs on the same subnet |
| Capture tool | Wireshark 4.6.4, capturing on interface `eth0` |
| Service under test | `vsftpd` (FTP server) |

## Tools Used

- **Wireshark** — packet capture and protocol analysis
- **vsftpd** — FTP server installed on the Ubuntu target
- Standard Linux command-line tools (`ip`, `ping`, `ftp`, `systemctl`)

## Steps Performed

### 1. Confirmed network connectivity between the two VMs

```bash
ip a
ping -c 4 192.168.74.128
```

Result: 33/33 packets received, 0% packet loss.

### 2. Installed and started an FTP server on Ubuntu

```bash
sudo apt update
sudo apt install vsftpd -y
sudo systemctl start vsftpd
sudo systemctl status vsftpd
```

Result: `vsftpd.service` reported `active (running)`.

### 3. Created a test account to authenticate with

```bash
sudo adduser ftptest
```

### 4. Started a Wireshark capture on Kali before connecting

Selected the `eth0` interface and started the capture before any FTP traffic was generated, so the full authentication handshake would be recorded.

### 5. Connected to the FTP server and logged in

```bash
ftp 192.168.74.131
```

Logged in as `ftptest`, then closed the session with `bye`.

### 6. Filtered the capture

Applied the Wireshark display filter:
### 7. Reconstructed the full session

Right-clicked the `PASS` packet and selected **Follow → TCP Stream** to view the login exchange as plain text.

## Key Finding

FTP transmits usernames and passwords as **unencrypted plaintext**. Anyone with visibility into the network segment can passively capture login credentials with no specialized tools beyond Wireshark itself.

![FTP login filtered in Wireshark](ftp-capture-filtered.png)

![Plaintext credentials in Follow TCP Stream](ftp-plaintext-credentials.png)

## Risk

Plaintext credential transmission is a confidentiality failure. It exposes account credentials to any attacker with network access, with no exploitation required — only passive observation.

## Recommendation

Replace FTP with an encrypted alternative:

- **SFTP** (SSH File Transfer Protocol)
- **FTPS** (FTP over TLS/SSL)

More broadly: prefer HTTPS over HTTP, and SSH over Telnet, for the same reason.

## Lessons Learned

- Packet capture timing matters — starting Wireshark *after* a login already happened misses the credentials entirely.
- VMs on the same hypervisor network segment can see each other's traffic by default, which mirrors exactly how this kind of interception happens on a real network.
- Confirming connectivity with `ping` before troubleshooting Wireshark saves time.

## How to Reproduce

1. Create two VMs on the same virtual network (able to `ping` each other).
2. Install `vsftpd` on one VM; create a test user.
3. Start a Wireshark capture on the other VM before connecting.
4. Connect via `ftp <target-ip>` and log in.
5. Filter to `ftp` and use **Follow → TCP Stream** on the `PASS` packet.
