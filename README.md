# Qubes-Privacy-Doc
It is a repository for documentation for running various Privacy Services in Qubes-OS

How to Run VPN in Qubes-OS

Create a App VM based on Fedora-30 with Sys-Firewall as networking and providing Network.

Now open a Browser in Disposable VM. Login to your VPN website and Download openvpn configuration files. Copy these files in Newly created App VM.

Now open a terminal in App VM and run following command

sudo mkdir /rw/config/vpn

Now run-

sudo gedit /rw/config/vpn/openvpn-client.ovpn

Now open one of the configuration files in Text editor and copy its content into Newly opened openvpn-client.ovpn

Search for auth-user-pass in theses lines and change it into

auth-user-pass pass.txt

Search for redirect-gateway def1 also and if not present write it there as it is in a new line.

Also add these three lines 

script-security 2

up 'qubes-vpn-handler.sh up'

down 'qubes-vpn-handler.sh down'

If there are already existing lines starting with up and down, Remove them.

Save this file and Exit.

Now again in App VM terminal run following command-

sudo gedit /rw/config/vpn/pass.txt

Type in text editor-

username

password

Save and Exit.

Now again in App VM terminal

sudo gedit /rw/config/vpn/qubes-vpn-handler.sh

Add following text-

#!/bin/bash
set -e
export PATH="$PATH:/usr/sbin:/sbin"
  
case "$1" in
  
up)
# To override DHCP DNS, assign DNS addresses to 'vpn_dns' env variable before calling this script;
# Format is 'X.X.X.X  Y.Y.Y.Y [...]'
if [[ -z "$vpn_dns" ]] ; then
    # Parses DHCP foreign_option_* vars to automatically set DNS address translation:
    for optionname in ${!foreign_option_*} ; do
        option="${!optionname}"
        unset fops; fops=($option)
        if [ ${fops[1]} == "DNS" ] ; then vpn_dns="$vpn_dns ${fops[2]}" ; fi
    done
fi
  
iptables -t nat -F PR-QBS
if [[ -n "$vpn_dns" ]] ; then
    # Set DNS address translation in firewall:
    for addr in $vpn_dns; do
        iptables -t nat -A PR-QBS -i vif+ -p udp --dport 53 -j DNAT --to $addr
        iptables -t nat -A PR-QBS -i vif+ -p tcp --dport 53 -j DNAT --to $addr
    done
    su - -c 'notify-send "$(hostname): LINK IS UP." --icon=network-idle' user
else
    su - -c 'notify-send "$(hostname): LINK UP, NO DNS!" --icon=dialog-error' user
fi
  
;;
down)
su - -c 'notify-send "$(hostname): LINK IS DOWN !" --icon=dialog-error' user
;;
esac


The above Script should be copied from Blame.

Add these two lines just beneath first line

vpn_dns="X.X.X.X"

addr="X.X.X.X"

Replace X.X.X.X with following addresses written in front of VPN name in these two lines-

NordVPN - 103.86.99.99 or 103.86.96.96

IVPN - 10.0.254.2 or 10.0.254.3

Perfect-Privacy - DNS address of VPN configuration file written after Remote

Mullvad - 10.8.0.1 or 10.14.0.1

Similarly you can ask your provider about these.

Save and Exit.

Now again in terminal run

sudo chmod +x /rw/config/vpn/qubes-vpn-handler.sh

sudo gedit /rw/config/qubes-firewall-user-script

Delete everything and Add following-

#!/bin/bash
#    Block forwarding of connections through upstream network device
#    (in case the vpn tunnel breaks):
iptables -I FORWARD -o eth0 -j DROP
iptables -I FORWARD -i eth0 -j DROP
ip6tables -I FORWARD -o eth0 -j DROP
ip6tables -I FORWARD -i eth0 -j DROP
   
#    Block all outgoing traffic
iptables -P OUTPUT DROP
iptables -F OUTPUT
iptables -I OUTPUT -o lo -j ACCEPT
   
#    Add the `qvpn` group to system, if it doesn't already exist
if ! grep -q "^qvpn:" /etc/group ; then
     groupadd -rf qvpn
     sync
fi
sleep 2s
   
#    Allow traffic from the `qvpn` group to the uplink interface (eth0);
#    Our VPN client will run with group `qvpn`.
iptables -I OUTPUT -p all -o eth0 -m owner --gid-owner qvpn -j ACCEPT


Again Above script should be copied from Blame. Save and exit.

In Terminal

sudo chmod +x /rw/config/qubes-firewall-user-script

sudo gedit /rw/config/rc.local

Delete everything and add

#!/bin/bash
VPN_CLIENT='openvpn'
VPN_OPTIONS='--cd /rw/config/vpn/ --config openvpn-client.ovpn --daemon'
   
su - -c 'notify-send "$(hostname): Starting $VPN_CLIENT..." --icon=network-idle' user
groupadd -rf qvpn ; sleep 2s
sg qvpn -c "$VPN_CLIENT $VPN_OPTIONS"

Again this script from Blame. Save and Exit.

In terminal

sudo chmod +x /rw/config/rc.local

Exit terminal.

Open Settings of this AppVM. Reach Firewall Settings. Choose Limit Networking and add Rules.

For rules open your choosen configuration file and look for "remote". Address written there should be added in these rules. Save and exit.

Now open terminal in Dom0


    qvm-firewall VPNVM list
    
    qvm-firewall VPNVM del --rule-no [icmp_rule_#]
    
    qvm-firewall VPNVM add --before [last_drop_rule_#] drop proto=icmp
    
    Now you can verify by running the list command again. The rules should be in this order: accept -> the IP addresses of the VPN servers, accept -> dns, drop -> icmp, drop

Exit.

Restart your VPN VM.
