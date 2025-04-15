# SSH key error during proxmox migration
## Explanation
```
The "Host key verification failed" error during VM migration in Proxmox indicates an SSH key mismatch or misconfiguration between the cluster nodes, often due to the new node’s SSH keys not being properly synchronized or recognized.
```
## Fix
### Step 1. Update Cluster Certificates: On the new node. This forces a refresh of the SSH keys and certificates across the cluster
```
pvecm updatecerts -f
```
### Step 2. Restart the pve-cluster and pveproxy services:
```
systemctl restart pve-cluster pveproxy
```
### Step 3. Verify SSH Known Hosts: The error suggests the source node doesn’t trust the new node’s host key. On the source node, manually connect to the new node to update the known_hosts file:
```
ssh -o 'HostKeyAlias=kla-ac-prox-01' root@10.35.50.10
```
### Step 4. Check /etc/pve/priv/known_hosts: Ensure the new node’s key is in the cluster’s shared known_hosts file. On the source node, run:
```
ssh-keyscan -t rsa kla-ac-prox-01 >> /etc/pve/priv/known_hosts
```
### Step 5. (optional). If prompted for a password, copy the SSH key:
```
ssh-copy-id -o 'HostKeyAlias=kla-ac-prox-01' root@10.35.50.10
```
### Step 6. Restart Services on All Nodes: After making changes, restart services on all nodes:
```
systemctl restart pve-cluster pveproxy
```

# Error 2: 
```
# /usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-06' root@10.35.50.16 /bin/true
2025-04-15 11:04:06 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2025-04-15 11:04:06 @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
2025-04-15 11:04:06 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2025-04-15 11:04:06 IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
2025-04-15 11:04:06 Someone could be eavesdropping on you right now (man-in-the-middle attack)!
2025-04-15 11:04:06 It is also possible that a host key has just been changed.
2025-04-15 11:04:06 The fingerprint for the RSA key sent by the remote host is
2025-04-15 11:04:06 SHA256:lG1XqiS448oYGRbL/8Vklp6K1NWyKLoWM2CAGZwjUTY.
2025-04-15 11:04:06 Please contact your system administrator.
2025-04-15 11:04:06 Add correct host key in /root/.ssh/known_hosts to get rid of this message.
2025-04-15 11:04:06 Offending RSA key in /etc/ssh/ssh_known_hosts:12
2025-04-15 11:04:06   remove with:
2025-04-15 11:04:06   ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-06"
2025-04-15 11:04:06 Host key for kla-ac-prox-06 has changed and you have requested strict checking.
2025-04-15 11:04:06 Host key verification failed.
2025-04-15 11:04:06 ERROR: migration aborted (duration 00:00:00): Can't connect to destination address using public key
TASK ERROR: migration aborted"
```
# Fix
### step 1. Remove Conflicting Keys on Source Nodes. 
- On each source node (kla-ac-prox-01, 02, 04, 05), remove the stale key for kla-ac-prox-06 as suggested by the error. Run: This clears outdated keys in both /etc/ssh/ssh_known_hosts (system-wide) and /root/.ssh/known_hosts (user-specific) to prevent conflicts.
```
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "kla-ac-prox-06" &&
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "10.35.50.16" &&
ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-06" &&
ssh-keygen -f "/root/.ssh/known_hosts" -R "10.35.50.16"
```
### step 2. Update Cluster Shared Known Hosts:
- Since Proxmox uses /etc/pve/priv/known_hosts for cluster migrations, ensure kla-ac-prox-06’s current key is correctly registered. On any node (e.g., kla-ac-prox-01), run:
```
ssh-keyscan -t rsa kla-ac-prox-06 >> /etc/pve/priv/known_hosts &&
ssh-keyscan -t rsa 10.35.50.16 >> /etc/pve/priv/known_hosts
```

# Fix 2
