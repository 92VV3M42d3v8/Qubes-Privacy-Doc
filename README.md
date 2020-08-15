A little Advanced Configuration for Qubes-OS use. (Might be buggy if you have WiFi connectivity and specific rules set on sys-firewall)

1. Creating Media for Qubes-OS

Attach USB to VM in which you've downloaded iso.

    qvm-usb attach <vmname> <sys-usb:2-10> 

This VM should be based on fedora and "Disks" should be in appVM shortcuts & Private volume should be atleast 7 GB. Iso file should be in Home folder itself and you should mount usb(by just clicking on it in file manager).
Open "Disks" utility and format USB with ext4 partition and overwrite. Name it reasonably and remember.
Now in AppVM terminal-
     
     df -h
     
Usually device is /dev/sda

    sudo umount /dev/sda
    
    sudo dd if=Qubes<iso-name> of=/dev/xvdi1 status=progress bs=1048576 && sync
    
This should create Installation media. Always install after testing media.    

During Install configuration, allow only Template installation (all 3). Rest everything disable (means don't create sys-net, anon-whonix, sys-usb, workvm etc)

2. After Installation

A. Creation of Default DispVM- In dom0

    [user@dom0 ~]$ qvm-create --template fedora-32 --label red fedora-32-dvm
    [user@dom0 ~]$ qvm-prefs fedora-32-dvm template_for_dispvms True
    [user@dom0 ~]$ qvm-features fedora-32-dvm appmenus-dispvm 1
    [user@dom0 ~]$ qubes-prefs default_dispvm fedora-32-dvm
    
B. sys-net creation

    qvm-create -C DispVM -l red sys-net
    qvm-prefs sys-net virt_mode hvm
    qvm-service sys-net meminfo-writer off
    qvm-pci attach --persistent sys-net2 dom0:00_1a.0
    qvm-prefs sys-net autostart true
    qvm-prefs sys-net netvm ''
    qvm-features sys-net appmenus-dispvm ''
    qvm-prefs sys-net provides_network true
    qvm-prefs sys-firewall netvm sys-net
    qubes-prefs clockvm sys-net
    
  4th step should be done with bdf of ethernet adapter.
  6th and 7th step is "none" not "default none" and prefer via sys-net Qubes settings.
  9th step should be skipped now. 10th step should be set in Qubes Global settings.


C. sys-firewall creation

    qvm-create -C DispVM -l red sys-firewall
    qvm-prefs sys-firewall autostart true
    qvm-prefs sys-firewall netvm sys-net
    qvm-prefs sys-firewall provides_network true
    qvm-features sys-firewall appmenus-dispvm ''


D. sys-usb Creation
    
    qvm-create -C DispVM -l red sys-usb
    qvm-prefs sys-usb virt_mode hvm
    qvm-service sys-usb meminfo-writer off
    qvm-pci a sys-usb dom0:03_00.0 --persistent -o no-strict-reset=true
    qvm-prefs sys-usb autostart true
    qvm-prefs sys-usb netvm ''
    qvm-features sys-usb appmenus-dispvm ''

    
    sudo nano /etc/qubes-rpc/policy/qubes.InputMouse
    
Add first line 

    sys-usb dom0 allow,user=root
    
Save and exit.

This VM should have no networking, HVM mode, no dispVM, Start automatically, included in backup. Don't hide USB controllers at system start-up.

Set some settings in global settings as you like.


E. sys-whonix creation

    sudo qubesctl state.sls qvm.whonix-ws-dvm
    
    
F. Default mgmt dvm creation

    sudo qubesctl state.sls qvm.default-mgmt-dvm


3. MAC alter (not needed is sys-* are disposable.)

Network Manager < Edit connections...< Wired connection 1< settings < Cloned MAC address< Random< Save.

4. Update dom0

       $ sudo qubes-dom0-update
    
5. Update Templates and tor browser.    

6. Using USB without device widget

       $ qvm-usb
    
       $ qvm-usb attach Vault sys-usb:2-10
    
       $ qvm-usb detach Vault sys-usb:2-10

7. Clone Debian template-
   
       qvm-clone debian-10 debian-X
       
8. Install SearX, Syncthing etc. in Clone template always.  

9. Using Block devices with cli-

       $ qvm-block
       
       $ qvm-block attach work sys-usb:sda
       
   This should be visible in "other locations" of work VM. Use it and unmount by right click and unmount option when you are done. 
   
   Then in dom0
   
       $ qvm-block detach work sys-usb:sda

10. Keyfiles for some applications-

        Brave browser- https://brave-browser-apt-release.s3.brave.com/brave-core.asc
        
        Chrome browser- https://dl.google.com/linux/linux_signing_key.pub
        
11. How to run AppImage

    To be run they first need to be marked executable with chmod +x AppImage , where <AppImage> is the file name of the AppImage, including its file extension) and then run with ./AppImage . Either that or clicked/double-clicked in one's file manager.
