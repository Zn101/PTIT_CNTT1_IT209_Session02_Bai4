# Exercise 4: Server Firewall Configuration (UFW & DigitalOcean Cloud Firewall)

## 1. Objectives

- Configure UFW (Uncomplicated Firewall) inside Ubuntu to filter incoming traffic.
- Combine it with a DigitalOcean Cloud Firewall at the infrastructure layer, so traffic is filtered at the network edge before it reaches the Droplet.
- Result: a two-layer firewall that only allows SSH (22/tcp) and HTTP (80/tcp).

## 2. Architecture

```
Internet
   |
   v
[Layer 2] DigitalOcean Cloud Firewall  -> Inbound: 22/tcp, 80/tcp only
   |
   v
[Layer 1] UFW on Ubuntu (Droplet)      -> default deny incoming, allow 22/tcp, 80/tcp
   |
   v
Nginx (80) / OpenSSH (22)
```

## 3. Layer 1: UFW (inside the OS)

### 3.1 Commands

```bash
# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH first (to avoid locking yourself out) and HTTP
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp

# Enable the firewall
sudo ufw enable
```

> Note: always allow port 22 **before** running `sudo ufw enable`, otherwise the current SSH session may be cut off.

### 3.2 Verification

```bash
sudo ufw status verbose
```

Output:

```text
<PASTE THE OUTPUT OF `sudo ufw status verbose` HERE>
```

Expected output (for reference):

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
```

## 4. Layer 2: DigitalOcean Cloud Firewall

### 4.1 Steps

1. Log in to the DigitalOcean Console.
2. Go to **Networking -> Firewalls -> Create Firewall**.
3. Name the firewall, e.g. `ptit-web-fw`.
4. **Inbound Rules**:

   | Type  | Protocol | Port | Sources                      |
   |-------|----------|------|------------------------------|
   | SSH   | TCP      | 22   | All IPv4, All IPv6 (or my IP) |
   | HTTP  | TCP      | 80   | All IPv4, All IPv6           |

5. **Outbound Rules**: keep the default (allow all).
6. In **Apply to Droplets**, select my Droplet and click **Create Firewall**.

### 4.2 Screenshots

**Inbound Rules:**

![Cloud Firewall inbound rules](./images/cloud-firewall-inbound.png)

**Firewall applied to the Droplet:**

![Cloud Firewall applied to Droplet](./images/cloud-firewall-droplet.png)

## 5. Testing

| # | Test (from my personal machine)            | Command                          | Expected result        | Actual result |
|---|--------------------------------------------|----------------------------------|------------------------|---------------|
| 1 | SSH connection                             | `ssh <user>@<DROPLET_IP>`        | Success                | <fill in>     |
| 2 | HTTP access on port 80                     | `curl -I http://<DROPLET_IP>`    | `200 OK`               | <fill in>     |
| 3 | Ping                                       | `ping <DROPLET_IP>`              | Timeout / blocked      | <fill in>     |
| 4 | Scan a closed port (e.g. 3306)             | `nc -zv -w 5 <DROPLET_IP> 3306`  | Timeout / blocked      | <fill in>     |
| 5 | Scan common ports                          | `nmap -Pn -p 22,80,443,3306 <DROPLET_IP>` | 22, 80 open; others filtered | <fill in> |

Test output:

```text
<PASTE THE TEST OUTPUT HERE>
```

## 6. Conclusion

- UFW is **active**, with default deny for incoming traffic and default allow for outgoing traffic.
- Only ports **22/tcp** and **80/tcp** are accepted by both layers.
- The Cloud Firewall drops unwanted traffic at the network edge, while UFW acts as a second line of defense inside the OS (defense in depth). If one layer is misconfigured, the other still protects the server.
- All other ports are blocked (connections time out).

## 7. Submitted files

- `README.md` (this report)
- `images/cloud-firewall-inbound.png`
- `images/cloud-firewall-droplet.png`
- Text output of `sudo ufw status` (included in section 3.2)
