# GLOBUS SETUP AT ACE
### 1. prepare a VM and install ubuntu
```
details
- hostname: kla-ac-gbs-01.ace.ac.ug
- ip address: 10.35.50.37
- name servers: 10.35.50.200, 10.35.50.100, 8.8.8.8, 8.8.4.4
- domain:  ace.ac.ug
```
### 2. system update
```
sudo apt update
sudo apt upgrade
```

### 3. Install globus server package
```
sudo apt install globus-connect-server54
```

### 4. Install globus cli (optional)
```
# sudo apt update && sudo apt install -y python3-pip
# python3 -m pip install --upgrade globus-cli
```

### 5. create the endpoint
```
globus-connect-server endpoint setup "ACE UGANDA HPC" \
    --organization "African Center of Excellence in Bioinformatics - Uganda" \
    --owner jkiyemba@ace.ac.ug \
    --contact-email ace.ac.ug@gmail.com

```


