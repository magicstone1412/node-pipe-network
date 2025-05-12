# Running a PoP Node on Pipe Network Testnet

This guide outlines the steps to set up a Point of Presence (PoP) node on the Pipe Network testnet (PipeQuest) using Ubuntu 24.04. By successfully running a node, you can contribute to the decentralized CDN built on Solana and potentially earn rewards during PipeQuest seasons (e.g., Season 1, May 5–12, 2025). If you encounter issues, check the [Pipe Network Discord](https://discord.gg/pipe-network) or [official documentation](https://docs.pipe.network/) for updates.

> 💡 If you're using Ubuntu 22.04, follow [this guide](https://www.cyberciti.biz/faq/how-to-upgrade-from-ubuntu-22-04-lts-to-ubuntu-24-04-lts/) to upgrade to 24.04.

![image](https://github.com/user-attachments/assets/8e2882fc-b11b-4611-89a6-8c07499188c7)

---

## Prerequisites

### Hardware Requirements

* **Recommended:** 4-core CPU, 16 GB RAM, 100 GB+ SSD

### Software Requirements

* Ubuntu 24.04 (LTS)
* `wget`, `libssl-dev`, and `ca-certificates` (installed during setup)
* Invite code from Pipe Network (via email after registration)
* Solana wallet address for rewards

### Network Requirements

* Stable internet connection (1 Gbps recommended)
* Public IP or proper port forwarding (Ports **80 TCP**, **443 TCP**)
* Firewall configured to allow ports 80 and 443

### Additional Requirements

* Discord or Telegram account
* Email confirmation from Pipe Network team

---

## Step-by-Step Guide

### 1. Set Up Your Environment

#### Install Dependencies

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install libssl-dev ca-certificates wget -y
```

#### Optimize Network Settings

```bash
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'
```

```bash
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

#### Increase File Descriptor Limits

```bash
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
exit
```

> Reconnect your VPS via SSH:


#### Enable Firewall & Open Ports

```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

---

### 2. Configure Your Node

#### Find Your IP informaton to put in the config file

```bash
 curl ipinfo.io
```

#### Set Up Directories

```bash
sudo mkdir -p /opt/popcache/logs
cd /opt/popcache
```

#### Download and Install the Node Binary

```bash
wget https://download.pipe.network/static/pop-v0.3.0-linux-x64.tar.gz
sudo tar -xzf pop-v0.3.0-linux-x64.tar.gz
chmod 755 /opt/popcache/pop
```

#### Create Configuration File

```bash
nano /opt/popcache/config.json
```

Paste the following and replace placeholders:

```json
{
  "pop_name": "your-pop-name",
  "pop_location": "city, country",
  "invite_code": "your-invite-code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "your-name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your-discord-username",
    "telegram": "your-telegram-handle",
    "solana_pubkey": "your-solana-wallet-address"
  }
}
```

The `cache_config` may change based on the official guide at [https://docs.pipe.network/nodes/testnet](https://docs.pipe.network/nodes/testnet).


##### Fields to Edit:

* **`pop_name`**: Unique name for your node (e.g., `MyPipeNode`).
* **`pop_location`**: City and country of your VPS (e.g., `Ho Chi Minh, VN`).
* **`invite_code`**: Your invite code from the Pipe Network email.

**Inside `identity_config`:**

* **`node_name`**: Name for your node (e.g., `magicstone1412`).
* **`name`**: Your personal or organization name.
* **`email`**: Your email address.
* **`website`**: Your website (optional; use a valid URL or leave as is).
* **`discord`**: Your Discord username (optional).
* **`telegram`**: Your Telegram handle (optional).
* **`solana_pubkey`**: Your Solana wallet address for rewards.

Save and exit the editor: `Ctrl+O`, `Enter`, then `Ctrl+X`.


#### Create a Systemd Service File

```bash
sudo nano /etc/systemd/system/popcache.service
```

Paste the following:

```ini
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
```

---

### 3. Run the PoP Node

#### Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable popcache
sudo systemctl start popcache
```

#### Verify the Service

```bash
sudo systemctl status popcache
sudo journalctl -u popcache -f
```

---

### 4. Verify Node Operation

#### Check Health Endpoint

```bash
curl http://localhost/health
```

#### Check Node State

Open in browser:

```
https://<your-vps-ip>/state
```

---

### 5. Participate in PipeQuest

* **Monitor Rewards:** Rewards depend on data cached (1x to 3x multiplier).
* **Dashboard:** Use community channels or Pipe dashboard (if available).
* **Correct Wallet:** Ensure `solana_pubkey` is accurate in your config.

> Stay active to earn **consistency bonuses** across seasons.

---

### 6. Monitor and Maintain Your Node

#### Monitor Logs

```bash
tail -f /opt/popcache/logs/stdout.log
tail -f /opt/popcache/logs/stderr.log
```

#### Basic Commands

```bash
sudo systemctl stop popcache
sudo systemctl restart popcache
sudo journalctl -u popcache -n 100
```

#### Update Node

```bash
LATER
```

---

## Troubleshooting

* **Service fails to start:**

  ```bash
  journalctl -u popcache -f
  cat /opt/popcache/logs/stderr.log
  sudo netstat -tulnp | grep -E '80|443'
  ```

* **Health endpoint fails:**

  * Check `ufw status`, ensure service is running.

---

## Additional Notes

* **Costs:** VPS \~\$10/month (e.g., Hetzner, Contabo)
* **Rewards:** Based on performance and audited post-season
* **Testnet:** v0.3.0 (as of May 11, 2025)
* **Mainnet:** Expected Summer 2025

---

## Sources

* [Official Docs](https://docs.pipe.network/)
* [PipeQuest Blog](https://pipe.network/blog/announcing-pipequest)
* [Original Guide](https://maouam.nodelab.my.id/pipe-network/testnet/)