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
sudo apt-get remove --purge snmpd snmp

# Reinstall SNMP
sudo apt-get update
sudo apt-get install snmpd snmp
````

### 5. configure snmp with v3 only
````
# Stop SNMP service before configuring
sudo systemctl stop snmpd

# Create a new configuration file
sudo mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.original
sudo nano /etc/snmp/snmpd.conf
````
##### content for the new snmp.conf
````
# Basic system information
sysLocation    ACE Kampala
sysContact     support@ace-bioinformatics.org
sysServices    72

# Agent configuration
master  agentx

# Listen only on localhost for security (change to your management IP if needed)
agentaddress  udp:127.0.0.1:161

# Explicitly disable SNMPv1 and SNMPv2c
disableSnmpv1  yes
disableSnmpv2c yes

# Define a view for SNMPv3 access
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1

# Include additional configuration files if needed
includeDir /etc/snmp/snmpd.conf.d
````

### 6. create an snmp user
````
# Create the SNMPv3 user with specified credentials
sudo net-snmp-config --create-snmpv3-user -a "F@bl35-Cl0ud5-5t@mp5" -x "F@bl35-Cl0ud5-5t@mp5" -X AES -A SHA s_snmp

# Add the user to the configuration file
sudo nano /etc/snmp/snmpd.conf
````
##### content to be added to snmpd.conf
````
# SNMPv3 user access
rouser s_snmp authpriv -V systemonly
````

### 7. start and test snmpv3
````
# Start the SNMP service
sudo systemctl start snmpd
sudo systemctl enable snmpd

# Test the SNMPv3 access
snmpwalk -v3 -l authPriv -u s_snmp -a SHA -A "F@bl35-Cl0ud5-5t@mp5" -x AES -X "F@bl35-Cl0ud5-5t@mp5" localhost
````

### 8. verify that snmp v1 and v2c are disabled
````
# These should fail with timeout or authentication errors
snmpwalk -v1 -c public localhost
snmpwalk -v2c -c public localhost
````

#### snmp user
````
name: s_snmp
````

#### user

````
a. Ensure that ICER-Uganda backups are up to date

b.  SSH into the linux servers using administrator credentials

c. Uninstall SNMP  from all servers and remove the installation directory for SNMP
yum remove net-snmp* OR apt remove net-snmp*
sudo rm -r /etc/snmp/

d. Re-install SNMP and its dependencies
yum -y install net-snmp net-snmp-utils

For Ubuntu/Debian
apt install snmpd snmp libsnmp-dev -y

e. Stop SNMP service
systemctl  stop snmpd

f. Configure SNMPv3 by setting the snmp user, authentication algorithm, encryption algorithm and Credentials.
net-snmp-create-v3-user -ro -A XXXXXXXX -a SHA -X XXXXX -x AES s_snmp

g. Start snmp service
systemctl  start snmpd

h. Test SNMPv3 connection
snmpwalk -u s_snmp -A XXXXX -a SHA -X XXXXX -x AES -l authPriv 127.0.0.1 -v3


9. Enable SNMP service
systemctl  enable snmpd
````

