# HPC to domain
### add domain controller
```
echo "10.35.50.200 kla-ac-dc-04.ace.ac.ug kla-ac-dc-04" | sudo tee -a /etc/hosts
```
### install chrony
```
sudo dnf install -y chrony
sudo systemctl enable --now chronyd
chronyc tracking
```
### install freeipa
```
sudo dnf install -y ipa-client
```
### run freeipa
```
sudo ipa-client-install \
  --hostname=kla-ac-hpc-02.ace.ac.ug \
  --domain=ace.ac.ug \
  --server=kla-ac-dc-04.ace.ac.ug \
  --realm=ACE.AC.UG \
  --mkhomedir \
  --force-ntpd
```
### or
```
sudo ipa-client-install \
  --hostname=kla-ac-hpc-02.ace.ac.ug \
  --domain=ace.ac.ug \
  --server=kla-ac-dc-04.ace.ac.ug \
  --realm=ACE.AC.UG \
  --mkhomedir \
  --force-ntpd \
  --force-join
```