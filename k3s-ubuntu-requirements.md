# Ubuntu - Install k3s requirements

Update system
```
sudo apt update && sudo apt -y upgrade
[ -f /var/run/reboot-required ] && sudo reboot -f
```

Install packages requirements
```
sudo apt install git
```
