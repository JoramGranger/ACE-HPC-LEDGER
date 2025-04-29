## Error: Unable to negotiate with 10.35.1.235 port 22: no matching host key type found. Their offer: ssh-rsa
```
joram@KAIRO:~/Documents/RDCT/ace-hpc-ledger$ ssh critical@10.35.1.235
Unable to negotiate with 10.35.1.235 port 22: no matching host key type found. Their offer: ssh-rsa
```
## Fix
```
ssh -o HostKeyAlgorithms=+ssh-rsa critical@10.35.1.235
```
## Error: Unable to negotiate with 10.35.1.235 port 22: no matching cipher found. Their offer: aes256-cbc,3des-cbc
```
joram@KAIRO:~/Documents/RDCT/ace-hpc-ledger$ ssh -o HostKeyAlgorithms=+ssh-rsa critical@10.35.1.235
Unable to negotiate with 10.35.1.235 port 22: no matching cipher found. Their offer: aes256-cbc,3des-cbc
```

## Fix
```
ssh -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes256-cbc,3des-cbc critical@10.35.1.235
```