Here’s a Markdown (`.md`) version of the steps to configure SNMPv3 on Rocky Linux while disabling SNMPv1 and SNMPv2c. This can be saved as a file (e.g., `snmpv3-rocky-linux.md`) for documentation or reference.

```markdown
# Configuring SNMPv3 on Rocky Linux with SNMPv1/v2c Disabled

This guide outlines how to set up SNMPv3 on Rocky Linux with authentication and encryption, while ensuring SNMPv1 and SNMPv2c are disabled for security. Rocky Linux uses the Net-SNMP package, and this configuration enforces SNMPv3-only access.

## Prerequisites
- Root or sudo access to the Rocky Linux system.
- SNMP packages installed:
  ```bash
  sudo dnf install net-snmp net-snmp-utils -y
  ```

## Steps

### 1. Stop the SNMP Service
Stop `snmpd` to make changes safely:
```bash
sudo systemctl stop snmpd
```

### 2. Create an SNMPv3 User
Set up a secure SNMPv3 user (e.g., `snmpuser`) with SHA authentication and AES encryption:
- Use the helper tool:
  ```bash
  sudo net-snmp-create-v3-user
  ```
  Example prompts:
  ```
  Enter a SNMPv3 user name to create: snmpuser
  Enter authentication pass-phrase: myAuthPass123
  Enter encryption pass-phrase: myPrivPass123
  ```
  - This adds a line to `/var/lib/net-snmp/snmpd.conf`, e.g.:
    ```
    createUser snmpuser SHA "myAuthPass123" AES "myPrivPass123"
    ```

- Alternatively, manually append to `/var/lib/net-snmp/snmpd.conf`:
  ```bash
  sudo sh -c 'echo "createUser snmpuser SHA \"myAuthPass123\" AES \"myPrivPass123\"" >> /var/lib/net-snmp/snmpd.conf'
  ```

### 3. Edit the Main SNMP Configuration
Configure `/etc/snmp/snmpd.conf` for SNMPv3-only access:
```bash
sudo vi /etc/snmp/snmpd.conf
```

Replace contents with:
```
###########################################################################
#
# snmpd.conf
# Configuration file for the Net-SNMP agent ('snmpd')
#
###########################################################################
# SECTION: System Information Setup
#

# sysLocation: Physical location of the system
#   Setting this disables SNMP SET on sysLocation.0
sysLocation    ACE Kampala
sysContact     support@ace-bioinformatics.org
sysServices    72

###########################################################################
# SECTION: Agent Operating Mode
#
#   Defines how the agent operates when running

# master: Enable AgentX master agent support
master  agentx

# agentaddress: Listening address and port
#   Matches Ubuntu: all IPv4 interfaces (udp:161) and IPv6 localhost (udp6:[::1]:161)
agentaddress  udp:161,udp6:[::1]:161

###########################################################################
# SECTION: Access Control Setup
#
#   Defines who can access the SNMP agent

# Views
#   Restrict access to system and host resources
view   systemonly  included   .1.3.6.1.2.1.1      # System group
view   systemonly  included   .1.3.6.1.2.1.25.1   # Host resources group

# SNMPv1/v2c: Explicitly disabled by omitting rocommunity/rwcommunity
#   No community strings defined to ensure v1/v2c rejection

# SNMPv3 user access
#   Read-only user with authentication and privacy, restricted to systemonly view
rouser authPrivUser authpriv -V systemonly

# Include additional configuration files
includeDir /etc/snmp/snmpd.conf.d
```
- **Notes**:
  - No `rocommunity` or `rwcommunity` lines ensures SNMPv1/v2c are disabled.
  - `rouser snmpuser authpriv -V systemonly` restricts access to SNMPv3 with authentication, encryption, and the `systemonly` view.

### 4. Set File Permissions
Secure the configuration files:
```bash
sudo chmod 600 /etc/snmp/snmpd.conf
sudo chmod 600 /var/lib/net-snmp/snmpd.conf
sudo chown root:root /etc/snmp/snmpd.conf
sudo chown root:root /var/lib/net-snmp/snmpd.conf
```

### 5. Start and Enable SNMP Service
Start and enable `snmpd`:
```bash
sudo systemctl start snmpd
sudo systemctl enable snmpd
```

Verify it’s running:
```bash
sudo systemctl status snmpd
```
- Check for `Active: active (running)`. If it fails, investigate port conflicts:
  ```bash
  sudo netstat -tulnp | grep :161
  ```

### 6. Test SNMPv3
Verify SNMPv3 works:
```bash
snmpwalk -v3 -l authPriv -u snmpuser -a SHA -A "myAuthPass123" -x AES -X "myPrivPass123" 127.0.0.1
```
- Expected output: OIDs like `SNMPv2-MIB::sysName.0 = STRING: rocky-server`.

### 7. Verify SNMPv1/v2c Are Disabled
Test SNMPv1:
```bash
snmpwalk -v1 -c public 127.0.0.1
```
Test SNMPv2c:
```bash
snmpwalk -v2c -c public 127.0.0.1
```
- **Expected**: Both fail (e.g., `Timeout: No Response`), confirming v1/v2c are disabled.

### 8. Firewall Configuration (Optional)
If not using localhost, allow UDP 161:
```bash
sudo firewall-cmd --add-port=161/udp --permanent
sudo firewall-cmd --reload
```

### 9. Debugging (If Needed)
Check logs if issues arise:
```bash
sudo journalctl -u snmpd -n 50
```
Run in debug mode:
```bash
sudo snmpd -f -Lsd -Dall
```

## Why This Disables v1/v2c
- SNMPv1/v2c require community strings (e.g., `rocommunity public`).
- By omitting these and only defining an SNMPv3 user with `authpriv`, `snmpd` rejects v1/v2c requests.

## Final Configuration Summary
- **User**: `snmpuser`
- **Auth**: SHA with `myAuthPass123`
- **Priv**: AES with `myPrivPass123`
- **Access**: Read-only, limited to `systemonly` view
- **v1/v2c**: Disabled (no community strings)

## Troubleshooting
- **Port Conflict**: Adjust `agentAddress` or stop conflicting processes.
- **SELinux**: Test with `sudo setenforce 0` if binding fails, then revert with `sudo setenforce 1`.

For further assistance, check logs or adjust settings as needed!
```

### Usage
- Save this as `snmpv3-rocky-linux.md`.
- View it in a Markdown viewer (e.g., VS Code, GitHub) or convert to HTML/PDF with tools like `pandoc`.
- Replace placeholders (e.g., `myAuthPass123`, `myPrivPass123`) with your actual passphrases.

Let me know if you’d like adjustments or additional sections!