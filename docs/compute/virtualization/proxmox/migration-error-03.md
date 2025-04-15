# Proxmox Cluster SSH Key Refresh Commands

## Overview

These commands resolve SSH host key verification issues between Proxmox nodes by systematically clearing old keys and adding fresh ones for both IP addresses and hostnames.

## Commands for kla-ac-prox-06

Run these commands on `kla-ac-prox-06` to refresh SSH keys for all other nodes in the cluster:

### Step 1: Clear Existing Keys

```bash
# Remove keys for kla-ac-prox-01
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-01" &&
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.10" &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-01" 2>/dev/null &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.10" 2>/dev/null

# Remove keys for kla-ac-prox-02
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-02" && 
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.12" &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-02" 2>/dev/null &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.12" 2>/dev/null

# Remove keys for kla-ac-prox-04
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-04" && 
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.14" &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-04" 2>/dev/null &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.14" 2>/dev/null

# Remove keys for kla-ac-prox-05
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-05" && 
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.15" &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-05" 2>/dev/null &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.15" 2>/dev/null
```

### Step 2: Add Fresh Keys

```bash
# Add keys for kla-ac-prox-01
ssh-keyscan -H 10.35.50.10 >> /root/.ssh/known_hosts &&
ssh-keyscan -H 10.35.50.10 | sed 's/10.35.50.10/kla-ac-prox-01/' >> /root/.ssh/known_hosts &&
ssh-keyscan -H 10.35.50.10 >> /etc/ssh/ssh_known_hosts 2>/dev/null &&
ssh-keyscan -H 10.35.50.10 | sed 's/10.35.50.10/kla-ac-prox-01/' >> /etc/ssh/ssh_known_hosts 2>/dev/null

# Add keys for kla-ac-prox-02
ssh-keyscan -H 10.35.50.12 >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.12 | sed 's/10.35.50.12/kla-ac-prox-02/' >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.12 >> /etc/ssh/ssh_known_hosts 2>/dev/null
ssh-keyscan -H 10.35.50.12 | sed 's/10.35.50.12/kla-ac-prox-02/' >> /etc/ssh/ssh_known_hosts 2>/dev/null

# Add keys for kla-ac-prox-04
ssh-keyscan -H 10.35.50.14 >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.14 | sed 's/10.35.50.14/kla-ac-prox-04/' >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.14 >> /etc/ssh/ssh_known_hosts 2>/dev/null
ssh-keyscan -H 10.35.50.14 | sed 's/10.35.50.14/kla-ac-prox-04/' >> /etc/ssh/ssh_known_hosts 2>/dev/null

# Add keys for kla-ac-prox-05
ssh-keyscan -H 10.35.50.15 >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.15 | sed 's/10.35.50.15/kla-ac-prox-05/' >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.15 >> /etc/ssh/ssh_known_hosts 2>/dev/null
ssh-keyscan -H 10.35.50.15 | sed 's/10.35.50.15/kla-ac-prox-05/' >> /etc/ssh/ssh_known_hosts 2>/dev/null
```

## Commands for Other Nodes

Run these commands on each other node (kla-ac-prox-01, kla-ac-prox-02, kla-ac-prox-04, kla-ac-prox-05) to refresh SSH keys for kla-ac-prox-06:

### Step 1: Clear Existing Keys

```bash
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-06"
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.16"
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-06" 2>/dev/null
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.16" 2>/dev/null
```

### Step 2: Add Fresh Keys

```bash
ssh-keyscan -H 10.35.50.16 >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.16 | sed 's/10.35.50.16/kla-ac-prox-06/' >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.16 >> /etc/ssh/ssh_known_hosts 2>/dev/null
ssh-keyscan -H 10.35.50.16 | sed 's/10.35.50.16/kla-ac-prox-06/' >> /etc/ssh/ssh_known_hosts 2>/dev/null
```

## Testing SSH Connectivity

After applying these changes, test the SSH connectivity between nodes:

### From kla-ac-prox-06 to other nodes:

```bash
# Test connection to kla-ac-prox-01
/usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-01' root@10.35.50.10 /bin/true

# Test connection to kla-ac-prox-02
/usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-02' root@10.35.50.12 /bin/true

# Test connection to kla-ac-prox-04
/usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-04' root@10.35.50.14 /bin/true

# Test connection to kla-ac-prox-05
/usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-05' root@10.35.50.15 /bin/true
```

### From other nodes to kla-ac-prox-06:

```bash
/usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-06' root@10.35.50.16 /bin/true
```

## How This Works

1. The `ssh-keygen -f ... -R ...` commands remove any existing host keys
2. The `ssh-keyscan` commands fetch fresh host keys from the servers
3. The `sed` commands create duplicate entries with the hostname alias
4. Keys are added to both user and system-wide known hosts files

These commands ensure that both IP-based and hostname-based SSH connections work properly, which is essential for Proxmox migration operations that use the `HostKeyAlias` parameter.

---

*Created: April 14, 2025*