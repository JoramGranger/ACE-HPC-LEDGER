# Qualys scan - Severity 5-  EOL/Obsolete Software: SNMP Protocol Version 1/2c Detected (19)

## Explanation
````
use of obsolete versions of SNMP (Simple Network Management Protocol) - specifically SNMPv1 and SNMPv2c - on multiple Ubuntu/Linux systems in your network.
the servers are 10.35.1.121, 10.35.32.10, 10.35.50.34, 10.35.50.109, 10.35.50.21, 10.25.50.27, 10.35.50.28, 10.35.50.[71-83]
````

## Fix
### 1. confirm snmp is running
````
sudo systemctl status snmpd
````
### 2. check running snmp version by checking snmp config file
````
sudo cat /etc/snmp/snmpd.conf
````

### 3. backup current configuration
````
# Create a backup directory
sudo mkdir -p /root/snmp_backup_$(date +%Y%m%d)

# Back up your configuration files
sudo cp -a /etc/snmp /root/snmp_backup_$(date +%Y%m%d)/
sudo cp -a /var/lib/snmp /root/snmp_backup_$(date +%Y%m%d)/
````

### 4. Stop, Remove and Uninstall SNMP
````
# Stop the SNMP service
sudo systemctl stop snmpd

# Remove the current SNMP packages while keeping configuration files
# ubuntu
sudo apt-get remove --purge snmpd snmp
# centos
dnf remove net-snmp net-snmp-utils net-snmp-libs

# Reinstall SNMP
# ubuntu
sudo apt-get update
sudo apt-get install snmpd snmp libsnmp-base snmp-mibs-downloader
# centos
dnf install net-snmp net-snmp-utils
# rocky
sudo dnf install net-snmp-devel snmpd snmp libsnmp-base snmp-mibs-downloader
````

### 5. configure snmp with v3 only
````
# Stop SNMP service before configuring
sudo systemctl stop snmpd

# Create a new configuration files
sudo mkdir -p /etc/snmp/snmpd.conf.d
sudo mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.original
sudo nano /etc/snmp/snmpd.conf
````
##### content for the new snmpd.conf
````
###########################################################################
#
# snmpd.conf
# Configuration file for the Net-SNMP agent ('snmpd')
#
###########################################################################
# SECTION: System Information Setup
#

# System location and contact information
sysLocation    ACE Kampala
sysContact     support@ace-bioinformatics.org

# sysservices value
sysServices    72

###########################################################################
# SECTION: Agent Operating Mode
#

# Run as a master agent (AgentX)
master  agentx

# Listen on standard SNMP ports
agentaddress  udp:161,udp6:[::1]:161

###########################################################################
# SECTION: Access Control Setup
#

# Define views - limit to system information only
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1

# SNMPv1/SNMPv2c community access (disabled)
#rocommunity  public default -V systemonly
#rocommunity6 public default -V systemonly

# SNMPv3 user access configuration
# Note: User creation should be in /var/lib/snmp/snmpd.conf:
# createUser s_snmp SHA-512 authpassphrase AES privpassphrase

# Configure read-only access for SNMPv3 user with auth+privacy
rouser s_snmp authpriv -V systemonly

# Include additional configuration files
includeDir /etc/snmp/snmpd.conf.d
````

### 6. create an snmp user as per config
````
# Create the SNMPv3 user with specified credentials
sudo net-snmp-config --create-snmpv3-user -a "F@bl35-Cl0ud5-5t@mp5" -x "F@bl35-Cl0ud5-5t@mp5" -X AES -A SHA s_snmp
````
### or
```
sudo net-snmp-config --create-snmpv3-user \
  -ro \
  -A SHA -a 'F@bl35-Cl0ud5-5t@mp5' \
  -X AES -x 'F@bl35-Cl0ud5-5t@mp5' \
  s_snmp
```

### 7. start and test snmpv3 on the host its self
````
# Start the SNMP service
sudo systemctl start snmpd
sudo systemctl enable snmpd

# Test the SNMPv3 access
snmpwalk -v3 -l authPriv -u s_snmp -a SHA -A "F@bl35-Cl0ud5-5t@mp5" -x AES -X "F@bl35-Cl0ud5-5t@mp5" localhost
````

### 8. verify that snmp v1 and v2c are disabled on the host itself
````
# These should fail with timeout or authentication errors
snmpwalk -v1 -c public localhost
snmpwalk -v2c -c public localhost
````

### important
```
- check firewall
sudo ufw status
```
```
- if inactive
# Enable the firewall
sudo ufw enable

# Allow SNMP on UDP port 161 (standard SNMP)
sudo ufw allow 161/udp

# Allow SNMP on TCP port 161 (if needed)
sudo ufw allow 161/tcp

# Verify the rules were added
sudo ufw status
```
