# Proxmox_NAT_Patch
Proxmox patch to create firewall NAT rules for web UI
Support for Proxmox 8 and simple English readme
original: https://github.com/Code-Exec/Proxmox_NAT_Patch

**1. Install of patch **

get installed pve-firewall version
> apt show pve-firewall

        Package: pve-firewall
        Version: 5.0.3
        Priority: optional

replace the Firewall.pm according to the version from the dir to /usr/share/perl5/PVE/Firewall.pm
Example

        VERSION=$(dpkg-query -W -f='${Version}\n' pve-firewall)
        curl -o /tmp/Firewall-pm -L https://github.com/chiakistar/Proxmox_NAT_Patch/raw/master/pve-firewall%20$VERSION/Firewall.pm
        cp /usr/share/perl5/PVE/Firewall.pm /usr/share/perl5/PVE/Firewall.pm.orig
        cp /tmp/Firewall.pm /usr/share/perl5/PVE/Firewall.pm.orig
        systemctl restart pve-firewall


**2. external interface** 

External interface is hardcoded into the /usr/share/perl5/PVE/Firewall.pm`

        my $ext_if = 'vmbr0'; #external interface

**3. network configuration**
Works with vmbr0 as external Interface with an public IP and vmbr1 as internal bridge.

Needed change to the vmbr1

                post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
                post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
                post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
                
example:

 /etc/network/interfaces

        auto vmbr1
        #private sub network
        iface vmbr1 inet static
                address  10.10.10.1
                netmask  255.255.255.0
                bridge-ports none
                bridge-stp off
                bridge-fd 1

                post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
                post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
                post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
** Configuration **

create new firewall rule and write into the comment NAT
NAT in:
source nat (aka port forwarding)  is used when direction in

![Sample_NAT_in](https://github.com/Code-Exec/Proxmox_NAT_Patch/blob/master/img/Sample_NAT_in.PNG)
Forward Port 822 to 10.10.10.107:22

Пример NAT out:
MASQUERADING DNAT is used when direction out
![Sample_NAT_out](https://github.com/Code-Exec/Proxmox_NAT_Patch/blob/master/img/Sample_NAT_out.PNG)

Allow DNAT from 10.10.10.105 to 123.123.123.123 Port 443

To allow DNAT to all destination from an internal host only write the VM-IP into source and left all out
Direction:out
Action: accept
Source: 10.10.10.1 on vmbr1
