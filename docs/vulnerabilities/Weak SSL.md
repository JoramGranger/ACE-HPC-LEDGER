# Weak SSL 
### 1. investigate
```
nmap -sS -p- 10.35.50.32 --open
```
### result
```                
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-02 12:33 EAT
Nmap scan report for 10.35.50.32
Host is up (0.018s latency).
Not shown: 65519 closed tcp ports (reset), 10 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
6160/tcp  open  ecmp
6162/tcp  open  patrol-coll
8181/tcp  open  intermapper
8182/tcp  open  vmware-fdm
25000/tcp open  icl-twobase1

Nmap done: 1 IP address (1 host up) scanned in 9.94 seconds
```