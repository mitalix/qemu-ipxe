# qemu and ipxe


Working config:

Qemu with basic network bridge and dnsmasq over ipxe

```
cat /etc/qemu/bridge.conf
```
allow br0



```
sudo ip link add name br0 type bridge    
sudo ip addr add 172.20.0.1/16 dev br0    
sudo ip link set br0 up    
```

Install dnsmasq:
```
sudo pacman -Ss dnsmasq
```
Check the status and run ...
```
systemctl status dnsmasq  
sudo systemctl start dnsmasq  
```
Open another window to view the continuous logging
```
journalctl -fu dnsmasq.service  
```

Through trial and error, these options just worked together in /etc/dnsmasq.conf: 
```
$ grep -v ^# /etc/dnsmasq.conf | strings
```  
```
interface=br0  
bind-interfaces  
dhcp-range=172.20.0.2,172.20.0.20  
enable-tftp  
tftp-root=/srv/tftp  
log-dhcp  
pxe-service=x86PC,"PXELINUX (BIOS)",boot/pxelinux  
tftp-secure  
```
```
$ qemu-system-x86_64 -nic bridge,br=br0,model=virtio-net-pci -m 1G
```
in another window:  <br>
```
$ vncviewer :5900
```
>To get the kernel and initramfs, see this post to download and copy those files into the tftp tree below
> https://github.com/mitalix/qemu-debian-buster

```
$ tree /srv/tftp/  
/srv/tftp/  
├── boot  
    ├── initramfs-linux.img  
    ├── pxelinux.0  
    ├── pxelinux.cfg  
    │   └── default  
    └── vmlinuz-linux  
```
$ cat /srv/tftp/boot/pxelinux.cfg/default   
```
DEFAULT linux  
        SAY Now booting the kernel from SYSLINUX ...  
LABEL linux  
        KERNEL vmlinuz-linux  
        initrd initramfs-linux.img  
        SYSAPPEND 3  
```

