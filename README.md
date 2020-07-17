Creating Media for Qubes-OS

Attach USB and then in dom0 terminal-
     
     df -h
     
Usually device is /dev/xvdi1

    sudo umount /dev/xvdi1
    
    sudo mkfs.ext4 /dev/xvdi1
    
    sudo dd if=Qubes<iso-name> of=/dev/xvdi1 status=progress bs=1048576 && sync
    
This should create Installation media.    



sys-usb Creation


dom0
    
    qvm-pci
    
    qvm-pci a sys-usb dom0:03_00.0 --persistent -o no-strict-reset=true
    
    sudo nano /etc/qubes-rpc/policy/qubes.InputMouse
    
Add first line 

    sys-usb dom0 allow
    
Save and exit.

This VM should have no networking, HVM mode, no dispVM, Start automatically, included in backup. Don't hide USB controllers at system start-up.


MAC alter

Network Manager < Edit connections...< Wired connection 1< settings < Cloned MAC address< Random< Save.

Update dom0

    $ sudo qubes-dom0-update
    
Update Templates and tor browser.    
