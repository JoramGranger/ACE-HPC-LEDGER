# Proxmox Node Migration Troubleshooting Guide

## Overview
This guide addresses common issues when migrating VMs between Proxmox nodes in a cluster environment, particularly focusing on SSH host key verification errors and QEMU version compatibility problems.

## SSH Host Key Verification Issues

### Symptoms
- Error message: `Host key verification failed`
- Error message: `ERROR: migration aborted: Can't connect to destination address using public key`

### Solution: Configure Host Resolution and SSH Keys

1. **Update `/etc/hosts` on ALL nodes**
   
   Edit the hosts file on each node:
   ```bash
   nano /etc/hosts
   ```
   
   Add entries for all cluster nodes:
   ```
   127.0.0.1       localhost
   
   # Proxmox Cluster Nodes
   10.35.50.10     kla-ac-prox-01
   10.35.50.XX     kla-ac-prox-02
   10.35.50.XX     kla-ac-prox-03
   10.35.50.XX     kla-ac-prox-04
   10.35.50.XX     kla-ac-prox-05
   10.35.50.16     kla-ac-prox-06
   ```

2. **Verify hostname resolution works**
   ```bash
   ping kla-ac-prox-01
   ping kla-ac-prox-06
   ```

3. **Clear existing host keys (if necessary)**
   ```bash
   ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-01"
   ssh-keygen -f "/root/.ssh/known_hosts" -R "kla-ac-prox-06"
   # Repeat for other nodes as needed
   ```

4. **Add host keys with proper hostnames**
   
   Run on each node for all other nodes:
   ```bash
   ssh-keyscan -H kla-ac-prox-01 >> /root/.ssh/known_hosts
   ssh-keyscan -H kla-ac-prox-02 >> /root/.ssh/known_hosts
   ssh-keyscan -H kla-ac-prox-03 >> /root/.ssh/known_hosts
   ssh-keyscan -H kla-ac-prox-04 >> /root/.ssh/known_hosts
   ssh-keyscan -H kla-ac-prox-05 >> /root/.ssh/known_hosts
   ssh-keyscan -H kla-ac-prox-06 >> /root/.ssh/known_hosts
   ```

5. **Test SSH connections with HostKeyAlias**
   ```bash
   /usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-01' root@10.35.50.10 /bin/true
   /usr/bin/ssh -e none -o 'BatchMode=yes' -o 'HostKeyAlias=kla-ac-prox-06' root@10.35.50.16 /bin/true
   ```

## QEMU Version Compatibility Issues

### Symptoms
- Error message: `Installed QEMU version '8.0.2' is too old to run machine type 'pc-i440fx-8.1+pve0'`

### Solution Options

1. **Upgrade the older node (recommended long-term)**
   ```bash
   apt update
   apt dist-upgrade
   ```
   Reboot the node after upgrading.

2. **Change VM machine type (quicker solution)**
   - Shut down the VM
   - Edit VM configuration
   - Change machine type to an older version (e.g., "pc-i440fx-8.0")
   - Start the VM
   - Try migration again

## Additional Troubleshooting Steps

### Check Cluster SSH Key Synchronization
```bash
pvecm updatekeys
```

### Update Proxmox Certificates
```bash
pvecm updatecerts --force
```

### Check Proxmox Cluster Status
```bash
pvecm status
```

### Check Corosync Configuration
```bash
cat /etc/corosync/corosync.conf
```

## Best Practices

1. **Keep all nodes on the same Proxmox version**