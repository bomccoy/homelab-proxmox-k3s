# 🚀 Install Pi-hole + Unbound on Ubuntu Server
## 1. Update system
```
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y curl wget git vim htop
```

## 2. Install Unbound (recursive DNS resolver)
### 2.1 Install package
```
sudo apt install -y unbound
```

### 2.2 Fetch current root hints
```
sudo mkdir -p /var/lib/unbound
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
```

### 2.3 Create config for Pi-hole
```
sudo tee /etc/unbound/unbound.conf.d/pi-hole.conf >/dev/null <<'EOF'
server:
  interface: 127.0.0.1
  port: 5335
  do-ip4: yes
  do-udp: yes
  do-tcp: yes

  # Security
  auto-trust-anchor-file: "/var/lib/unbound/root.key"
  qname-minimisation: yes
  harden-glue: yes
  harden-dnssec-stripped: yes
  harden-referral-path: yes

  # Performance
  prefetch: yes
  serve-expired: yes
  cache-min-ttl: 60
  cache-max-ttl: 86400

  # Root hints
  root-hints: "/var/lib/unbound/root.hints"

  # Access
  access-control: 127.0.0.0/8 allow

  # Privacy
  hide-identity: yes
  hide-version: yes
EOF
```

### 2.4 Enable and test
```
sudo unbound-checkconf
sudo systemctl enable --now unbound

# Test: should return an IP
dig @127.0.0.1 -p 5335 example.com +short
```

## 3. Install Pi-hole (DNS sinkhole + dashboard)
### 3.1 Run installer
```
curl -sSL https://install.pi-hole.net | bash
```
During setup:
* Select your VM’s interface (e.g., ens18).
* Confirm static IP (e.g., 192.168.169.10).
* For upstream DNS, pick anything for now (we’ll switch to Unbound).
* Enable the Web Admin.
* Save the generated admin password (or reset later with pihole -a -p).

## 4. Configure Pi-hole to use Unbound
### 4.1 In Pi-hole admin (http://<VM-IP>/admin):
* Go to Settings → DNS.
* Disable all upstream DNS providers.
* Under Custom 1 (IPv4), enter:
```
127.0.0.1#5335
```
* Disable DNSSEC here (Unbound already validates).
* Save.

### 4.2 Test
```
dig @192.168.169.10 google.com +short
pihole -t
```
* You should see the query appear in Pi-hole logs.
* Unbound is now doing the real resolution, Pi-hole is filtering/logging.

## 5. (Optional) Conditional Forwarding for local hostnames
* In Pi-hole admin → Settings → DNS → Conditional Forwarding
* Example:
  * Local network: 192.168.169.0/24
  * IP of DHCP server/router: 192.168.169.1
  * Local domain: homelab.lan

## 6. Point your network at Pi-hole
* Update your router’s DHCP settings to advertise Pi-hole’s IP as DNS (e.g., 192.168.169.10).
* Add a secondary (like 1.1.1.1 or a second Pi-hole if you build one).
* Renew a client’s lease and check:
```
nslookup github.com
```
It should show Pi-hole as the DNS server.

✅ At this point:
* Pi-hole is your ad-blocking filter + dashboard.
* Unbound is your private recursive resolver.
* Together, you’re no longer dependent on Google/Cloudflare.
