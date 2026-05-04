# 🛡️ WireGuard VPN Implementation Guide  
**Multi-Platform Setup (Linux, Windows, Windows Server)**

---

## 📌 Overview

This project provides a complete guide for deploying a WireGuard VPN infrastructure supporting:

- Linux Server (Recommended)
- Windows Server (Alternative)
- Windows Clients
- Linux Clients

WireGuard is a modern VPN solution known for its speed, simplicity, and strong encryption.

---

## 🧱 Network Architecture

Client Devices (Windows/Linux)
        │
        ▼
Encrypted Tunnel (UDP 51820)
        │
        ▼
VPN Server (Linux / Windows Server)
        │
   ┌────┴────┐
   ▼         ▼
Internal     Internet
Network

---

## 🌐 IP Addressing Scheme

| Device        | IP Address   |
|--------------|-------------|
| VPN Server   | 10.0.0.1/24 |
| Client 1     | 10.0.0.2/24 |
| Client 2     | 10.0.0.3/24 |

---

## ⚙️ Requirements

### Server
- Linux (Ubuntu/Debian) OR Windows Server 2019/2022
- Public IP or Port Forwarding
- UDP Port 51820 open

### Clients
- Windows 10/11 or Linux
- WireGuard installed

---

## 🐧 Linux Server Setup (Recommended)

### Install WireGuard
```
sudo apt update
sudo apt install wireguard -y
```

### Generate Keys
```
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

### Configure Server
```
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 51820

PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

### Enable IP Forwarding
```
sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
sudo sysctl -p
```
### Start WireGuard
```
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```
---

## 🖥️ Windows Server Setup

### Install WireGuard
```
https://www.wireguard.com/install/
```
### Configuration
```
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 51820
DNS = 8.8.8.8
```

### Enable Routing (PowerShell)
```
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" -Name IPEnableRouter -Value 1
```

### Configure NAT
```
New-NetNat -Name "WireGuardNAT" -InternalIPInterfaceAddressPrefix 10.0.0.0/24
```

### Allow Firewall
```
New-NetFirewallRule -DisplayName "WireGuard VPN" -Direction Inbound -Protocol UDP -LocalPort 51820 -Action Allow
```
---

## 💻 Windows Client Setup
```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_PUBLIC_IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
---

## 🐧 Linux Client Setup
```
sudo apt install wireguard -y

sudo nano /etc/wireguard/wg0.conf
```

```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_PUBLIC_IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

```
sudo wg-quick up wg0
```
---

## 🔗 Add Client to Server
```
[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```
---

## 🧪 Testing
```
ping 10.0.0.1

sudo wg
```
---

## 🔒 Security Best Practices

- Keep private keys secure  
- Assign unique IP per client  
- Restrict AllowedIPs  
- Remove inactive peers  
- Rotate keys periodically  

---

## 📊 Deployment Recommendation

| Scenario | Recommendation |
|----------|--------------|
| Production | Linux Server |
| Testing | Windows Server |
| Mixed Clients | Fully Supported |

---

## 📎 Default Settings

Port: 51820 (UDP)  
Network: 10.0.0.0/24  

---

## 👨‍💻 Author

IT Department Documentation  
WireGuard VPN Implementation Guide
