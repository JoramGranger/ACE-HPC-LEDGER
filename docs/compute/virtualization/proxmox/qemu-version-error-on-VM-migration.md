# Qemu-version-error-on-VM-migration
## Fix
### 1. confirm versions on both source and destination
```
root@kla-ac-prox-06:~# qemu-system-x86_64 --version
QEMU emulator version 9.2.0 (pve-qemu-kvm_9.2.0-2)
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
root@kla-ac-prox-06:~# 
---------------------------------------------------------------------------
root@kla-ac-prox-07:~# qemu-system-x86_64 --version
QEMU emulator version 9.0.2 (pve-qemu-kvm_9.0.2-4)
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
root@kla-ac-prox-07:~# 
```

### 2. for non-subscription installation, adjust the repo
```
# Edit or create /etc/apt/sources.list.d/pve-no-subscription.list:
nano /etc/apt/sources.list.d/pve-no-subscription.list
# add the line below 
"deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription"
```

### 3. update version on destination to match source
```
- adjust the repo
apt update
apt install -y qemu-system
```

### 4. confirm version
```
pveversion
qemu-system-x86_64 --version
```