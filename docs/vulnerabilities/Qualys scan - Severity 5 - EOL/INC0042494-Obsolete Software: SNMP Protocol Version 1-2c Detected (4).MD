# INC0042494 Qualys scan - Severity 5 - EOL/Obsolete Software: SNMP Protocol Version 1/2c Detected (4)
### details
```
affected hosts - 10.35.1.120, 10.35.1.121, 10.35.1.122, 10.35.1.123,   
```

## fix
### 1. enable snmp v3 / disable snmp v1/2c
- log into the web ui for each of the affeced hosts
- navigate to iDRAC settings on the left pane
- dropdown to Network
- in the right pane, navigate to services tab, (top of pane)
- scroll to SNMP agent
- confirm the "Enabled" attribute is checked.
- under SNMP Protocol, check SNMP V3 option
- scroll to the bottom
- click apply

### 2. configure snmp v3
- navigate to iDRAC settings on the left pane
- dropdown to User Authentication
- click on the userID for a given user
- in right pane, under user configuration, check configure user
- in the bottom right, click next
- on user configuration page,
----------------------------------------------------------------
- under the general section
- check "enable user"
- provide username and password
----------------------------------------------------------------
- under snmp v3 section
- check "enable snmpv3"
- authentication type, "SHA"
- privacy type "AES"
----------------------------------------------------------------
- under iDRAC User Privileges section
- select Administrator for roles

### 3. confirm configurations (nmap)
- snmpwalk -v 3 -u <snmp-user> -l authPriv -a MD5 -A <password> -x AES -X <password> 10.35.1.120 1.3.6.1.2.1.1




