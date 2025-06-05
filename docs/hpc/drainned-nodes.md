# Nodes kla-ac-cpu-[04,16-17] are drain at ACE Uganda.
## solution
### 1. find out why they are drained
```
sinfo -R | grep kla-ac-cpu
---------------------------------------------------------------------
[critical@kla-ac-hpc-02 ~]$ sinfo -R | grep kla-ac-cpu
Kill task failed     root      2025-06-02T22:09:38 kla-ac-cpu-[04,16]
Kill task failed     root      2025-06-03T02:15:27 kla-ac-cpu-17
---------------------------------------------------------------------
Nodes kla-ac-cpu-04, kla-ac-cpu-16, and kla-ac-cpu-17 were drained due to task kill failures, likely from jobs that couldn’t be terminated cleanly.
```

### 2. force resume to undrain the nodes
```
sudo scontrol update NodeName=kla-ac-cpu-[04,16-17] State=RESUME Reason="Clearing drain"
```
### 3. reboot nodes
```
for node in kla-ac-cpu-04 kla-ac-cpu-16 kla-ac-cpu-17; do
  sudo scontrol reboot $node
done
```

### 4. confirm status 
```
sinfo
```