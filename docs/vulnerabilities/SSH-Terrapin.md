# terapin vulnerability
### 1. backup current ssh config
```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```
###  2: Add mitigation to SSH config
```
sudo tee -a /etc/ssh/sshd_config << 'EOF'

# Terrapin vulnerability (CVE-2023-48795) mitigation
Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1
KexAlgorithms diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521

EOF

```

### Step 3: Test SSH config syntax
```
sudo sshd -t
```

### Step 4: Restart SSH service
```
sudo systemctl restart sshd
```

### Step 5: Verify SSH is running
```
sudo systemctl status sshd
```

### Step 6: Test connection
```
ssh localhost
```

### Step 7: check vulnerable algorithms
```
# Test if server accepts vulnerable ChaCha20 cipher
ssh -c chacha20-poly1305@openssh.com critical@10.35.50.22

# Test if server accepts vulnerable ETM MACs
ssh -m umac-128-etm@openssh.com critical@10.35.50.22

# Test combination of vulnerable algorithms
ssh -c chacha20-poly1305@openssh.com -m umac-128-etm@openssh.com critical@10.35.50.22
```

### step 8. final test
```
ssh -c aes128-ctr -m hmac-sha2-256 critical@10.35.50.200
```
