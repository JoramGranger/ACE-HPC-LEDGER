# Error  SSH Error due to -o BatchMode from kla-ac-prox-06 to kla-ac-prox-01
# Fix
### 1. First, let's get the host key for kla-ac-prox-01 and add it specifically with that hostname:
```
ssh-keyscan -H kla-ac-prox-01 >> /root/.ssh/known_hosts
```
### 2. Also add an entry using the explicit HostKeyAlias format:
```
ssh-keyscan -H 10.35.50.10 | sed 's/10.35.50.10/kla-ac-prox-01/' >> /root/.ssh/known_hosts
```

# Error  SSH Error due to -o BatchMode from kla-ac-prox-01 to kla-ac-prox-06
# Fix
### 1. Add kla-ac-prox-06's host key with the proper hostname alias:
```
ssh-keyscan -H kla-ac-prox-06 >> /root/.ssh/known_hosts
ssh-keyscan -H 10.35.50.16 | sed 's/10.35.50.16/kla-ac-prox-06/' >> /root/.ssh/known_hosts
```
### 2. 
```
ssh-keyscan -H 10.35.50.16 > /tmp/temp_keys
```