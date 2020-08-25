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
Method 1 (don't use)(first see method 2 and then use this method accordingly)

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

Method 2:

If they are to be based on Minimal fedora-

    qvm-run -u root network-mini xterm


network-mini template= ( Don't know if less is required.)

    dnf install pciutils less psmisc qubes-core-agent-networking iproute qubes-core-agent-network-manager network-manager-applet notification-daemon gnome-keyring 
    

usb-mini template=

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus qubes-usb-proxy qubes-input-proxy-sender qubes-gpg-split oathtool gedit keepassxc qubes-core-agent-passwordless-root polkit pycairo
    
  
Create DispVM based on network-mini and usb-mini and use them as base for sys-net, sys-firewall, sys-usb.
       
       https://www.qubes-os.org/doc/disposablevm-customization/#creating-a-new-disposablevm-template


vpn-mini template=

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus gedit openvpn iptables qubes-core-agent-networking qubes-core-agent-passwordless-root polkit


fedora-mega =

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus qubes-core-agent-networking firefox gedit qubes-core-agent-passwordless-root polkit qubes-pdf-converter qubes-img-converter pycairo pulseaudio-qubes


fedora-extreme = (I prefer full debian template clone modified instead of fedora-extreme)

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus qubes-core-agent-networking firefox gedit qubes-core-agent-passwordless-root polkit qubes-pdf-converter qubes-img-converter pycairo gimp pulseaudio-qubes uget
    
   Also intall vlc, searx, syncthing, keybase in it. 
   
 Vlc-
 
    $> su -
    #> dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    #> dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    #> dnf install vlc

 Searx-
 
    https://github.com/92VV3M42d3v8/Qubes-Privacy-Doc/tree/Searx-instance--Private
 
 Syncthing-
 
    https://github.com/92VV3M42d3v8/Qubes-Privacy-Doc/tree/Syncthing

 Keybase-

    https://prerelease.keybase.io/keybase_amd64.deb 

E. sys-whonix creation

    sudo qubesctl state.sls qvm.whonix-ws-dvm
    
    
F. Default mgmt dvm creation

    sudo qubesctl state.sls qvm.default-mgmt-dvm


3. MAC alter (not needed is sys-* are disposable.)

Network Manager < Edit connections...< Wired connection 1< settings < Cloned MAC address< Random< Save.

4. Update dom0

       $ sudo qubes-dom0-update
    
5. Update Templates and tor browser. (first see 15)   

6. Using USB without device widget

       $ qvm-usb
    
       $ qvm-usb attach Vault sys-usb:2-10
    
       $ qvm-usb detach Vault sys-usb:2-10

7. Clone Debian template-
   
       qvm-clone debian-10 debian-X
       
8. Install SearX, Syncthing, uget etc. in Clone template always.  

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


12. sys-net, sys-firewall = network-mini

    sys-usb, vault, gpg-vm, StorageVM = usb-mini
    
    DispVM, Bank, mailvm (tutanota) = fedora-mega
    
    SurfVM, MediaVM, Syncthing vm, keybase = fedora-extreme/ debian
    
    VpnVM (proxy) = vpn-mini


13. Sample Qubes Qrexec policy 

        $ cd /etc/qubes/policy.d/
        sudo nano 30-user.policy

        *                    *  @anyvm                Vault                  deny
        *                    *  @anyvm                Bank                   deny
        qubes.ClipboardPaste *  dom0                  @anyvm                 ask
        qubes.ClipboardPaste *  Vault                 @anyvm                 ask
        *                    *  Vault                 @anyvm                 deny
        qubes.ClipboardPaste *  @dispvm:fedora-32-dvm Storage                ask
        qubes.VMShell        *  @anyvm                @anyvm                 deny
        qubes.VMExec         *  @anyvm                @anyvm                 deny
        qubes.VMSExecGUI     *  @anyvm                @anyvm                 deny
        qubes.UpdatesProxy   *  @type:TemplateVM      @default               allow target=sys-whonix
        qubes.Filecopy       *  @anyvm                Storage                ask
        qubes.Filecopy       *  @anyvm                @default               ask
        *                    *  Bank                  @anyvm                 deny
        qubes.Filecopy       *  @anyvm                @anyvm                 deny
        qubes.ClipboardPaste *  @anyvm                @anyvm                 deny
        qubes.OpenInVM       *  @Storage              @dispvm:debian-10-dvm  allow
        qubes.OpenInVM       *  @anyvm                @dispvm:debian-10-dvm  ask
        qubes.OpenInVM       *  @anyvm                @anyvm                 deny
        *                    *  @anyvm                Storage                deny
        *                    *  Storage               @anyvm                 deny
        *                    *  @dispvm:debian-10-dvm @anyvm                 deny
        
  
  If Any person among readers have a great policy file to share, please share with me.   


14. How to open every(many file types) file from a VM like Storage VM to dispVM by default:

    Create a file in StorageVM
   
        cd ~/.local/share/applications
        sudo gedit browser_vm.desktop
      
   Type following in Editor-
       
        [Desktop Entry]
        Encoding=UTF-8
        Name=BrowserVM
        Exec=qvm-open-in-vm @dispvm:debian-10-dvm %u
        Terminal=false
        X-MultipleArgs=false
        Type=Application
        Categories=Network;WebBrowser;
        MimeType=x-scheme-handler/unknown;x-scheme-handler/about;text/html;text/xml;application/xhtml+xml;application/xml;application/vnd.mozilla.xul+xml;application/rss+xml;application/rdf+xml;image/gif;image/jpeg;image/png;x-scheme-handler/http;x-scheme-handler/https;application/pdf;text/plain
        
   Save with ctrl+x then y then enter.
   
   Set it as your default browser-
   
        user@Storage:~$ xdg-settings set default-web-browser browser_vm.desktop
        
   Change any pdf or text or jpg default application (from file property) to browser_vm.desktop    
     

15. Onionized Repositories

        https://www.whonix.org/wiki/Onionizing_Repositories
        https://www.qubes-os.org/news/2019/04/17/tor-onion-services-available-again/
        
16. Split-GPG 

    1.Keybase

        [user@dom0 ~]$ sudo qubes-dom0-update qubes-gpg-split-dom0
        
        [user@usb-mini ~]$ sudo dnf install qubes-gpg-split
        
        [user@usb-mini ~]$ gpg2 --gen-key
        
        [user@gpg-vm ~]$ gpg2 -K
        
        [user@gpg-vm ~]$ echo "export QUBES_GPG_AUTOACCEPT=120" >> ~/.profile
        
        [user@keybase ~]$ sudo bash
        
        [root@keybase ~]$  echo "gpg-vm" > /rw/config/gpg-split-domain
        
        [user@keybase ~]$ keybase config set gpg.command /usr/bin/qubes-gpg-client-wrapper
        
        [user@keybase ~]$ keybase pgp select
        
    2. https://www.qubes-os.org/doc/split-gpg/#using-thunderbird--enigmail-with-split-gpg
    
    3. https://www.qubes-os.org/doc/split-gpg/#importing-public-keys
    
    4. https://www.qubes-os.org/doc/split-gpg/#advanced-using-split-gpg-with-subkeys
    
 
 17.    https://github.com/rustybird/qubes-split-dm-crypt
 
 
 18.    https://github.com/Qubes-Community/Contents/blob/master/docs/configuration/http-proxy.md
 
 19. 
