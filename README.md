## 📌 Summary

| Setting         | Value                           |
| --------------- | ------------------------------- |
| Gateway         | `192.168.49.1:8383`             |
| Client Auth     | Certificate + Password          |
| Password (VPN)  | `0000`                          |
---
### 📥 Step 1: Download the Certificate

```bash
wget http://192.168.49.1:8181/netshare.p12
```

---

### 🔓 Step 2: Extract and Separate the CA Cert

Split the `.p12` file into its components:

```bash
openssl pkcs12 -in netshare.p12 -clcerts -nokeys -out netshare-cert.pem -password pass:123456
openssl pkcs12 -in netshare.p12 -cacerts -nokeys -out netshare-ca.pem -password pass:123456
openssl pkcs12 -in netshare.p12 -nocerts -nodes -out netshare-key.pem -password pass:123456
```

* `netshare-cert.pem` → Client certificate
* `netshare-ca.pem` → CA certificate (this is what we trust)
* `netshare-key.pem` → Private key

---

### 🛡️ Step 3: Install CA Certificate to System Trust

Move the CA certificate into your system's trusted CA store:

```bash
sudo cp netshare-ca.pem /usr/local/share/ca-certificates/netshare-ca.crt
sudo update-ca-trust extract  # On RHEL/Fedora/CentOS
# or
sudo update-ca-certificates   # On Debian/Ubuntu

# On Arch/Manjaro:
sudo trust anchor netshare-ca.pem
```

> **Verify it:**

```bash
trust list | grep "netshare"
```

---

### 🔧 Step 4: Update NetShare VPN Config

```bash
sudo nano /etc/NetworkManager/system-connections/NetShare.nmconnection
```

Modify the `[vpn]` section to **remove** `ignore-cert-warn=yes` and **ensure proper cert paths** are provided:

```ini
[vpn]
gateway=192.168.49.1:8383
user=def
password-flags=0
require-mppe=no
ca=/etc/ssl/certs/netshare-ca.pem
cert=/path/to/netshare-cert.pem
key=/path/to/netshare-key.pem
service-type=org.freedesktop.NetworkManager.sstp
```

> Replace `/path/to/` with the actual paths. You may prefer to place certs in `/etc/ssl/private/` or `/etc/NetworkManager/certs/`.

Ensure permissions:

```bash
sudo chmod 600 /path/to/netshare-key.pem
```

---

### 🔑 Step 5: Store VPN Password Securely

```bash
sudo nmcli connection modify NetShare vpn.secrets password=0000
```

---

### 🔄 Step 6: Restart NetworkManager and Connect

```bash
sudo systemctl restart NetworkManager
nmcli connection up NetShare
```
