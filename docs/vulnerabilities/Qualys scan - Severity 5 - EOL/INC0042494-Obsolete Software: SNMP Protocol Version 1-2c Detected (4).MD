# INC0042494 Qualys scan - Severity 5 - EOL/Obsolete Software: SNMP Protocol Version 1/2c Detected (4)
### details
```
affected hosts - 10.35.1.120, 10.35.1.121, 10.35.1.122, 10.35.1.123,   
```

## fix
- log into the web ui for each of the affeced hosts
- navigate to iDRAC settings 
- dropdown to Network
- in the right pane, navigate to services tab, (top of pane)
- scroll to SNMP agent
- confirm the "Enabled" attribute is checked.
- under SNMP Protocol, check SNMP V3 option
- scroll to the bottom
- click apply
