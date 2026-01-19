# BIND DNS Server for Local Network

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Enabled-2496ED?logo=docker)](https://www.docker.com/)
![Maintained](https://img.shields.io/badge/Maintained%3F-yes-green.svg)

A containerized BIND9 DNS server solution for managing domain names and DNS resolution within your local network. Perfect for internal network DNS management, development environments, and local domain hosting.

[Quick Start](#-quick-start) • [Configuration](#-configuration) • [Features](#-features) • [Troubleshooting](#-troubleshooting)

</div>

---

## 📋 Overview

This project provides a fully containerized BIND9 DNS server setup using Docker and Docker Compose. It enables you to:

- **Host custom domain names** on your local network
- **Manage DNS records** for internal services
- **Control query resolution** with ACL-based access control
- **Forward DNS queries** to external resolvers as fallback
- **Deploy easily** with pre-configured Docker setup

> ⚠️ **Important Note:** This configuration requires **static IP addresses** for reliable DNS resolution. Ensure all devices and the DNS server have fixed IPs before deployment.

---

## ✨ Features

- 🐳 **Docker Containerized** - Consistent environment across platforms
- 🔒 **ACL-Based Security** - Control which networks can query the DNS
- 🔄 **DNS Forwarding** - Fallback to external resolvers (e.g., Cloudflare 1.1.1.1)
- ⚙️ **Easy Configuration** - Simple YAML-based setup
- 🔧 **Customizable Zones** - Add multiple DNS zones and records
- 📦 **Persistent Storage** - Data retained in named volumes

---

## 🚀 Quick Start

### Prerequisites

- Docker ([Install Docker](https://docs.docker.com/get-docker/))
- Docker Compose ([Install Docker Compose](https://docs.docker.com/compose/install/))
- Git
- Network with static IP configuration

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Illucious/BIND-DNS-server.git
   cd BIND-DNS-server
   ```

2. **Update network configuration** (see [Configuration](#-configuration) section)

3. **Start the DNS server:**
   ```bash
   docker-compose up -d
   ```

4. **Verify the service:**
   ```bash
   docker-compose logs bind9
   ```

---

## ⚙️ Configuration

### Step 1: Configure Network Access (named.conf)

Edit `config/named.conf` to specify which networks can query your DNS server:

```conf
acl internal {
    192.168.0.0/24;      # Your local network subnet
    172.21.30.0/24;      # DNS server subnet
    172.18.0.0/24;       # Additional authorized networks
    172.21.0.0/24;       # Add more as needed
};

options {
    forwarders {
        1.1.1.1;         # Cloudflare DNS (or use 8.8.8.8, 9.9.9.9, etc.)
    };
    allow-query { internal; };        # Only allow configured networks
    allow-query-cache { internal; };  # Cache for authorized networks
};
```

**Update the ACL with your actual network ranges and DNS server IP.**

### Step 2: Define DNS Zones

Configure your DNS zones in `config/named.conf`:

```conf
zone "example.home" IN {
    type master;
    file "/etc/bind/example-home.zone";
};
```

### Step 3: Add DNS Records

Edit `config/example-home.zone` to add your DNS records:

```dns
$TTL 10d
$ORIGIN example.home.

@           IN      SOA     ns.example.home.        info.example.home. (
                            2024052900      ; Serial
                            12h             ; Refresh
                            15m             ; Retry
                            3w              ; Expire
                            2h              ; Minimum TTL
                            )

            IN      NS      ns.example.home.
ns          IN      A       172.21.30.44    # Your DNS server IP

; DNS Records
mail        IN      A       192.168.0.10
web         IN      A       192.168.0.20
db          IN      A       192.168.0.30
```

---

## 📦 Docker Compose Setup

The `docker-compose.yml` configuration:

```yaml
version: '3'

services:
  bind9:
    container_name: bind9-dns
    image: ubuntu/bind9:latest
    environment:
      - TZ=Asia/Kolkata
      - BIND9_USER=root
    ports:
      - "53:53/tcp"      # DNS over TCP
      - "53:53/udp"      # DNS over UDP
    volumes:
      - ./config:/etc/bind
      - .cache:/var/cache/bind
      - .records:/var/lib/bind
    restart: unless-stopped
    networks:
      - bind-network

networks:
  bind-network:
    driver: bridge
```

**Port Requirements:**
- **Port 53/TCP** - DNS queries over TCP
- **Port 53/UDP** - DNS queries over UDP

---

## 🔧 Usage

### Start the DNS Server

```bash
# Start in detached mode (background)
docker-compose up -d

# Start with logs visible
docker-compose up

# Start specific service
docker-compose up bind9
```

### View Logs

```bash
# View current logs
docker-compose logs bind9

# Follow logs in real-time
docker-compose logs -f bind9

# View last 50 lines
docker-compose logs --tail=50 bind9
```

### Stop the DNS Server

```bash
# Stop container (data persists)
docker-compose stop

# Stop and remove containers
docker-compose down
```

### Test DNS Resolution

From any device on your network:

```bash
# macOS/Linux
nslookup mail.example.home 192.168.X.X
dig mail.example.home @192.168.X.X

# Windows
nslookup mail.example.home 192.168.X.X
```

Replace `192.168.X.X` with your DNS server's IP address.

---

## 🖥️ Client Configuration

### Option 1: Per-Device Configuration

**Windows:**
1. Open Network Settings → Change adapter options
2. Right-click network connection → Properties
3. Select "IPv4" → Properties
4. Set DNS server to your BIND DNS server IP

**macOS:**
1. System Preferences → Network → Advanced
2. DNS tab → Add your DNS server IP

**Linux:**
```bash
# Edit /etc/resolv.conf (Ubuntu/Debian)
echo "nameserver 192.168.X.X" | sudo tee /etc/resolv.conf
```

### Option 2: Network-Wide Configuration

Configure your router's DHCP settings to use the BIND DNS server as the primary DNS resolver for all devices.

---

## 📝 Directory Structure

```
bind-dns/
├── docker-compose.yml      # Docker Compose configuration
├── config/
│   ├── named.conf          # BIND configuration & ACLs
│   └── example-home.zone   # DNS zone file
├── .cache/                 # BIND cache (auto-created)
├── .records/               # BIND records (auto-created)
└── README.md              # This file
```

> **Note:** Folders `.cache` and `.records` are created automatically on first run. If not created, manually create them with:
> ```bash
> mkdir .cache .records
> ```

---

## 🐛 Troubleshooting

### Container won't start

**Check logs:**
```bash
docker-compose logs bind9
```

**Verify named.conf syntax:**
```bash
docker-compose exec bind9 named-checkconf /etc/bind/named.conf
```

### DNS queries not resolving

1. **Check ACL configuration** - Ensure your network is in the `acl internal` block
2. **Verify server IP** - Confirm clients are querying the correct DNS server IP
3. **Check firewall** - Ensure port 53 (TCP/UDP) is open on the server
4. **Restart service:**
   ```bash
   docker-compose restart bind9
   ```

### Zone file syntax errors

**Validate zone file:**
```bash
docker-compose exec bind9 named-checkzone example.home /etc/bind/example-home.zone
```

### Permissions issues

**Reset permissions:**
```bash
docker-compose exec bind9 chown -R bind:bind /var/cache/bind /var/lib/bind
```

---

## 📚 DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | `web IN A 192.168.0.10` |
| **AAAA** | IPv6 address | `web IN AAAA 2001:db8::1` |
| **CNAME** | Alias | `alias IN CNAME web.example.home.` |
| **MX** | Mail server | `example.home IN MX 10 mail.example.home.` |
| **NS** | Nameserver | `@ IN NS ns.example.home.` |
| **TXT** | Text records | `example.home IN TXT "v=spf1 -all"` |

---

## 📖 Additional Resources

- [BIND9 Official Documentation](https://www.isc.org/bind/)
- [DNS Record Types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
- [Docker Documentation](https://docs.docker.com/)
- [RFC 1035 - DNS Protocol](https://tools.ietf.org/html/rfc1035)

---

## ⚖️ License

This project is licensed under the **MIT License** - see the LICENSE file for details.

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📧 Support

For issues, questions, or suggestions:
- Open an [Issue](https://github.com/Illucious/BIND-DNS-server/issues)
- Submit a [Pull Request](https://github.com/Illucious/BIND-DNS-server/pulls)

---

<div align="center">

Made with ❤️ for the open-source community

⭐ If you found this helpful, please consider giving it a star!

</div>
