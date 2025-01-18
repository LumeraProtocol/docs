#  Lumera Protocol Validator Operations Manual

## Introduction

This manual provides detailed instructions for operating a validator node on the Lumera Protocol. It covers hardware requirements, software setup, security practices, common operations, and troubleshooting guidelines.

## 1. Hardware and Software Requirements

### Validator

#### 1.1 Minimum Hardware Requirements

- **CPU**: 8 cores, x86_64 architecture
- **RAM**: 32 GB RAM
- **Storage**: 2 TB NVMe SSD
- **Network**: 1 Gbps dedicated line
- **Operating System**: Ubuntu 22.04 LTS or higher

#### 1.2 Recommended Hardware Requirements

- **CPU**: 16 cores, x86_64 architecture
- **RAM**: 64 GB RAM
- **Storage**: 4 TB NVMe SSD
- **Network**: 5 Gbps dedicated line
- **Operating System**: Ubuntu 22.04 LTS or higher

### Supernode

#### 1.1 Minimum Hardware Requirements

- **CPU**: 8 cores, x86_64 architecture
- **RAM**: 16 GB RAM
- **Storage**: 1 TB NVMe SSD
- **Network**: 1 Gbps
- **Operating System**: Ubuntu 22.04 LTS or higher

#### 1.2 Recommended Hardware Requirements

- **CPU**: 16 cores, x86_64 architecture
- **RAM**: 64 GB RAM
- **Storage**: 4 TB NVMe SSD
- **Network**: 5 Gbps
- **Operating System**: Ubuntu 22.04 LTS or higher


### 1.3 Notes

- ARM-based processors (like Apple M1) are not recommended for production
- Hard drives (HDD) are not suitable due to I/O requirements
- SATA SSDs may underperform during high load

### 1.4 Software

- `pasteld` binary: Downloaded from official releases or compiled from source.
- Go 1.21 or later.
- Git.
- `jq`.
- `curl` or `wget`.
-  `gpg`

## 2. Setting Up the Validator Node

### 2.1. Base System Setup

1. **System Updates**
```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install build-essential jq curl git wget snap unzip gpg -y
```

2. **Security Basics**
```bash
# SSH Configuration
sudo vim /etc/ssh/sshd_config
# Set:
# PermitRootLogin no
# PasswordAuthentication no
# MaxAuthTries 3

# Firewall Setup
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 26656/tcp
sudo ufw enable
```

3. **Go Installation**
```bash
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
echo "export PATH=$PATH:$(go env GOPATH)/bin" >> ~/.profile
source ~/.profile
```

### 2.2. `pasteld` Binary Installation

- Download the appropriate binary from the official releases page on GitHub.
- Verify the checksum of the downloaded binary.
- Make the binary executable: `chmod +x pasteld`.
- Move the binary to an executable path (e.g., `/usr/local/bin`).

### 2.3. Node Initialization and Configuration

1. **Initialize the Node:**
```bash
pasteld init <moniker> --chain-id <chain-id>
```
	- Replace `<moniker>` with your validator's moniker.
	- Replace `<chain-id>` with the chain ID of your network.
1. **Copy Genesis File:**
    - Copy the downloaded `genesis.json` file to `$HOME/.pasteld/config` directory.
2. **Configure Node:**
    - Adjust parameters in `$HOME/.pasteld/config/config.toml`:
        - Update `persistent_peers` with the list of initial peers provided by the core team.
    - Adjust parameters in `$HOME/.pasteld/config/app.toml`:
        - Example: set `min-gas-prices`.
3. **Start the Node:**
```bash
pasteld start
```

## 3. Security Best Practices

### 3.1. Key Management

- Securely store `node_key.json` and `priv_validator_key.json`.
- Use hardware wallets or encrypted storage for private keys.
- Never expose your private keys.
- Use a strong password for keyring.
- Always back up your keys offline

### 3.2. Sentry Node Architecture

- Implement a sentry node architecture to mitigate DDoS attacks.
- Validators should only connect to trusted full nodes.
- Sentry nodes shield validator nodes from direct internet exposure.

```bash
# In sentry config.toml
pex = true
persistent_peers = "[validator-node-id]@[validator-private-ip]:26656"
private_peer_ids = "[validator-node-id]"

# In validator config.toml
pex = false
persistent_peers = "[sentry1-node-id]@[sentry1-private-ip]:26656,[sentry2-node-id]@[sentry2-private-ip]:26656"
private_peer_ids = ""
```

### 3.3. Firewall Configuration

- Configure the firewall to allow only necessary traffic.
- Restrict access to ports used by `pasteld` (e.g., port 26656).
- Use SSH key authentication.
- Disable root login

### 3.4. Regular Updates

- Keep the OS and software up-to-date with the latest patches.
- Regularly update the `pasteld` binary.

### 3.5. Monitoring

- Set up tools to monitor node performance and health.
- Set up alerting to be notified in case of issues

### 3.6. Access Control

- Limit access to authorized personnel.
- Use strong passwords and/or SSH keys.

### 3.7. Avoiding Publicly Accessible Nodes

- Avoid running validators directly on the public internet; use private networks and secure tunnels.

## 4. Validator Operations

### 4.1. Basic Commands

- **Check node status:**  `pasteld status`
- **Display validator node ID:**  `pasteld tendermint show-node-id`
- **Get validator information:**  `pasteld query staking validator <your_validator_address>`

### 4.2. Staking and Bonding

- **Delegate tokens to validator:** `pasteld tx staking delegate <validator_address> <amount> --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`
- **Unbond tokens from validator:** `pasteld tx staking unbond <validator_address> <amount> --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`
- **Redelegate tokens from one validator to another:** `pasteld tx staking redelegate <source_validator_address> <destination_validator_address> <amount> --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`

### 4.3. Governance

Actively participate in network governance.

  - **List all governance proposals:** `pasteld query gov proposals`
  - **Query specific proposal:** `pasteld query gov proposal <proposal_id>`
  - **Check voting status on proposal:** `pasteld query gov votes <proposal_id>`
  - **Vote on proposal:** `pasteld tx gov vote <proposal_id> <option> --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`

### 4.4. Distribution

Manage rewards and commissions

- **Withdraw rewards:** `pasteld tx distribution withdraw-rewards <validator_address> --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`
- **Withdraw all rewards and commissions:** `pasteld tx distribution withdraw-all-rewards --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`

### 4.5 Slashing

   - **Unjail validator if slashed:** `pasteld tx slashing unjail --from <key_name> --chain-id <chain_id> --gas-prices <gas_price>`

### 4.6 Validator Management

* **Edit validator data:**  `pasteld tx staking edit-validator --new-moniker <new_name> --website <new_website> --identity <keybase_id> --details <new_details> --chain-id <chain_id> --from <key_name> --gas-prices <gas_price>`
- **Show validator public key:**  `pasteld tendermint show-validator

### 4.7 Troubleshooting

- Consult official documentation for common issues and solutions.
- Contact the core team or other validators for assistance.

## 5. Security Recommendations

### 5.1. Hardware Security Module (HSM)

- Recommended: YubiHSM 2.
- Alternative: Ledger Nano S/X with Tendermint app.
- Follow HSM-specific documentation for key generation and usage.

### 5.2. System Hardening

```bash
# Disable root login
sudo passwd -l root

# Set up automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Secure shared memory
echo "tmpfs     /run/shm     tmpfs     defaults,noexec,nosuid     0     0" >> /etc/fstab

# Set up fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 5.3. Process Isolation

```bash
# Create dedicated user
sudo useradd -m -s /bin/bash validator
sudo usermod -aG sudo validator

# Set up process limits
sudo vim /etc/security/limits.conf
# Add:
# validator soft nofile 65535
```

## 6. Monitoring Setup

### 6.1 Node Monitoring

```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

# Create service file
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=validator
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### 6.2 Alerting Setup

1.  **Prometheus Configuration**

```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'validator'
    static_configs:
      - targets: ['localhost:26660']
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

2.  **Alert Rules**

```yaml
# /etc/prometheus/alerts.yml
groups:
- name: validator
  rules:
  - alert: ValidatorDown
    expr: up == 0
    for: 5m
    labels:
      severity: critical
  - alert: BlocksMissed
    expr: validator_missed_blocks > 10
    for: 10m
    labels:
      severity: warning
```

## 7. Maintenance Procedures

### 7.1. Regular Maintenance

1. **Daily Tasks**
    - Check validator status.
    - Monitor system resources.
    - Review logs for errors.
    - Verify synchronization status.

2. **Weekly Tasks**
    - Update security patches.
    - Back up validator keys.
    - Check disk usage.
    - Review performance metrics.

3. **Monthly Tasks**
    - Full system audit.
    - Review and update documentation.
    - Test backup restoration.
    - Check for software updates.

### 7.2. Emergency Procedures

- **Node Recovery**

```bash
# Quick status check
pasteld status

# Reset node (if needed)
pasteld unsafe-reset-all

# Restore from backup
cp backup/priv_validator_key.json ~/.[$binary]/config/
cp backup/node_key.json ~/.[$binary]/config/

# Restart service
sudo systemctl restart [binary]
```

- **Double-Sign Prevention**
    - Never run validator keys on multiple machines.
    - Always use recent snapshots for recovery.
    - Implement proper backup procedures.

## 8. Useful Commands

### 8.1. Node Management

```bash
# Check node status
pasteld status

# Check sync status
pasteld status 2>&1 | jq .sync_info

# Check validator status
pasteld query staking validator $(pasteld keys show validator --bech val -a)

# Check blocks signed
pasteld query slashing signing-info $(pasteld tendermint show-validator)
```

### 8.2. Common Operations

```bash
# Unjail validator
pasteld tx slashing unjail --from validator --chain-id [chain-id] --gas-prices [gas-price]

# Edit validator
pasteld tx staking edit-validator \
    --new-moniker [new-name] \
    --website [new-website] \
    --identity [keybase-id] \
    --details [new-details] \
    --chain-id [chain-id] \
    --from validator \
    --gas-prices [gas-price]

# Withdraw rewards
pasteld tx distribution withdraw-rewards $(pasteld keys show validator --bech val -a) \
    --from validator \
    --commission \
    --chain-id [chain-id] --gas-prices <gas_price>
```

## 9. Troubleshooting Guide

### 9.1. Common Issues

1. **Node Not Syncing**
    - Check network connectivity.
    - Verify genesis checksum.
    - Check disk space.
    - Review peer connections.
2. **Missing Blocks**
    - Check system resources.
    - Review network latency.
    - Verify time synchronization.
    - Check validator status.
3. **Slashing Events**
    - Document the incident.
    - Check for double-signing evidence.
    - Prepare recovery plan.
    - Contact chain support.

### 9.2. Support Resources

- Chain Documentation: [DOCS_URL]
- Community Forum: [FORUM_URL]
- Technical Support: [SUPPORT_URL]
- Emergency Contact: [EMERGENCY_CONTACT]

This manual provides essential information for operating a validator node on the Pastel Network. Follow the security best practices, stay informed about updates, and engage with the community to ensure the health and stability of the network.


---
(C) 2024 Pastel Network
