
# using UFW and Wireguard remote VPS

setup UFW rules
reference: https://wiki.archlinux.org/title/Uncomplicated_Firewall#Forward_policy

edit /etc/ufw/before.rules
```
-A ufw-before-forward -i wg0 -j ACCEPT
-A ufw-before-forward -o wg0 -j ACCEPT
```

uncomment in /etc/ufw/sysctl.conf
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1
