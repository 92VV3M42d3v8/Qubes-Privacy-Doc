A little Advanced Configuration for Qubes-OS use. (Might be buggy if you have WiFi connectivity and specific rules set on sys-firewall)( I have just recently switched from Windows to directly Qubes, with no knowledge of linux)

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

During Install configuration, allow Template installation (all 3).(Qubes don't provide minimal template yet so allow here to create sys-*, disp-vm and templates over tor)

2. After Installation (see 13)

My templates= t-mgmt, t-network, t-secure, t-vpn, debian-X, fedora-minimal(with nano), fedora, debian, whonix-gw, whonix-ws. (all offline)

My DVM-templates= default-mgmt-dvm, debian-X-dvm(default), fedora-dvm (online), t-network-dvm, t-secure-dvm, whonix-ws-dvm (online), BaseVM* ( only two are online)

My Appvm= Vault(offline), Mailvm, storage(offline), GPG(offline), sync, sys-whonix, keybasethunder, Bank, Nord, ivpn

My Disp-Appvm= sys-net, sys-firewall(online), sys-usb, fileopen, HardCore, Surf, Multimedia

My AppVM which are templates for DispVM= BaseVM

{ fileopen is created with (qvm-create -C DispVM -l yellow fileopen) base debian-X-dvm.}

{ BaseVM is appvm based on debian-X in which I modified firefox, installed searx and allowed it to be template for dispvm from advanced tab of settings. Then I made it default dispVM and created all named online disposable vm for different purpose from it and after creating them I changed default dispvm to offline one i.e. debian-X-dvm. Like in above eg. Hardcore is based on BaseVM and it's also created like fileopen.}

Some appvms arise out of requirements too.

Method 1 (don't use)(first see method 2 and then use this method accordingly)

A. Creation of Default DispVM- In dom0

    [user@dom0 ~]$ qvm-create --template debian-X --label red debian-X-dvm
    [user@dom0 ~]$ qvm-prefs debian-X-dvm template_for_dispvms True
    [user@dom0 ~]$ qvm-features debian-X-dvm appmenus-dispvm 1
    [user@dom0 ~]$ qubes-prefs default_dispvm debian-X-dvm
    
   Here debian-X is clone of debian template (modified later). Set networking to none.

B. sys-net creation

    qvm-create -C DispVM -l red sys-net
    qvm-prefs sys-net virt_mode hvm
    qvm-service sys-net meminfo-writer off
    qvm-pci attach --persistent sys-net dom0:22_00.0
    qvm-prefs sys-net autostart true
    qvm-prefs sys-net netvm ''
    qvm-features sys-net appmenus-dispvm ''
    qvm-prefs sys-net provides_network true
    qvm-prefs sys-firewall netvm sys-net
    qubes-prefs clockvm sys-net
    
  4th step should be done with bdf of ethernet adapter.
  6th and 7th step is "none" not "default none" and prefer via sys-net Qubes settings.
  9th step should be skipped now. 10th step should be set in Qubes Global settings.
  appmenus-dispvm entry creates an entry in qubes dropdown menu.


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
    qvm-pci a sys-usb dom0:28_00.0 --persistent -o no-strict-reset=true
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

    qvm-run -u root t-mgmt xterm

t-mgmt template=

    dnf install qubes-core-agent-passwordless-root qubes-mgmt-salt-vm-connector

t-network template= 

    dnf install psmisc qubes-core-agent-networking iproute qubes-core-agent-network-manager network-manager-applet 
    

t-secure template= (see 12 below first)

    dnf install psmisc gnome-keyring qubes-core-agent-nautilus qubes-usb-proxy qubes-input-proxy-sender qubes-gpg-split oathtool keepassxc qubes-core-agent-passwordless-root polkit pycairo
    
  
t-secure template Volume size should be increased to 40-50 GB.  
  
Create DispVM based on t-network and t-secure and use them as base for sys-net, sys-firewall, sys-usb.
       
       https://www.qubes-os.org/doc/disposablevm-customization/#creating-a-new-disposablevm-template


You have to create t-network-dvm and t-secure-dvm and then have to follow method 1. Method 1 pick default dispvm, so always change default dispvm to as requirement before creating sys-* based on that.


t-vpn template=

    dnf install psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus gedit openvpn iptables qubes-core-agent-networking


fedora-mega = (I prefer fedora-32)

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus qubes-core-agent-networking firefox gedit qubes-core-agent-passwordless-root polkit qubes-pdf-converter qubes-img-converter pycairo pulseaudio-qubes


fedora-extreme = (I prefer full debian template clone modified instead of fedora-extreme)

    dnf install pciutils less psmisc notification-daemon gnome-keyring qubes-core-agent-nautilus qubes-core-agent-networking firefox gedit qubes-core-agent-passwordless-root polkit qubes-pdf-converter qubes-img-converter pycairo gimp pulseaudio-qubes uget
    
   Also intall vlc, searx, syncthing, keybase, uget in it. 
   
 Vlc-
 
    $> su -
    #> dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    #> dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    #> dnf install vlc

  In Debian-
  
    sudo apt-get install vlc


 Searx-
 
    https://github.com/92VV3M42d3v8/Qubes-Privacy-Doc/tree/Searx-instance--Private
 
 Syncthing-
 
    https://github.com/92VV3M42d3v8/Qubes-Privacy-Doc/tree/Syncthing

 Keybase-

    https://prerelease.keybase.io/keybase_amd64.deb 

 I use debian-X for extreme version which include searx, syncthing, keybase, vlc, uget, ublock-origin, https-eveywhere.
 
    sudo apt-get install xul-ext-ublock-origin
    
    sudo apt-get install webext-https-everywhere

E. sys-whonix creation

    sudo qubesctl state.sls qvm.whonix-ws-dvm
    
    
F. Default mgmt dvm creation

    sudo qubesctl state.sls qvm.default-mgmt-dvm


3. MAC alter (not needed is sys-* are disposable.)

Network Manager < Edit connections...< Wired connection 1< settings < Cloned MAC address< Random< Save.

4. Update dom0

       $ sudo qubes-dom0-update
    
5. Update Templates and tor browser. (first see 13)   

6. Clone Debian template-
   
       qvm-clone debian-10 debian-X
       
7. Using Block devices with cli-

       $ qvm-block
       
       $ qvm-block attach work sys-usb:sda
       
   This should be visible in "other locations" of work VM. Use it and unmount by right click and unmount option when you are done. 
   
   Then in dom0
   
       $ qvm-block detach work sys-usb:sda

8. Keyfiles for some applications-

        Brave browser- https://brave-browser-apt-release.s3.brave.com/brave-core.asc
        
        Chrome browser- https://dl.google.com/linux/linux_signing_key.pub
        
9. How to run AppImage

    To be run they first need to be marked executable with chmod +x AppImage , where <AppImage> is the file name of the AppImage, including its file extension) and then run with ./AppImage . Either that or clicked/double-clicked in one's file manager.


10. A. Templates for different App and DispVM

    sys-net, sys-firewall = t-network-dvm
    
    sys-usb = t-secure-dvm
    
    fileopen = debian-X-dvm
    
    sys-whonix = whonix-gw
    
    t-network-dvm = t-network
    
    t-secure-dvm = t-secure
    
    whonix-ws-dvm = whonix-ws
    
    debian-X-dvm = debian-X
    
    vault, gpg-vm, StorageVM = t-secure
    
    hardcore, Multimedia, Surf = BaseVM
    
    Sync, keybasethunder = debian-X
    
    VpnVM (proxy) = t-vpn
    
    fedora-dvm, mailvm, Bank = fedora
    
    default-mgmt-dvm = t-mgmt
    
    B. Networking Structure (not always same, I prefer to change vpn, appvm names, and networking for them)
    
     All templates are offline.
     
     Among DVM-templates fedora-dvm is via sys-firewall and whonix-dvm via sys-whonix.
     
     Bank, Nord, ivpn, mailvm via sys-firewall
     
     Rest appvm are either offline or passing via vpn.
     

11. Sample Qubes Qrexec policy ( WIP- Changes frequently)

        $ cd /etc/qubes/policy.d/
        sudo nano 30-user.policy

        *                    *  @anyvm                Hardcore               deny
        *                    *  Hardcore              @anyvm                 deny
        *                    *  @anyvm                Vault                  deny
        *                    *  @anyvm                Bank                   deny
        *                    *  Bank                  @anyvm                 deny
        qubes.VMShell        *  @anyvm                @anyvm                 deny
        qubes.VMExec         *  @anyvm                @anyvm                 deny
        qubes.VMSExecGUI     *  @anyvm                @anyvm                 deny
        qubes.ClipboardPaste *  dom0                  @anyvm                 ask
        qubes.ClipboardPaste *  Vault                 @anyvm                 ask
        *                    *  Vault                 @anyvm                 deny
        qubes.ClipboardPaste *  @anyvm                Storage                ask
        qubes.ClipboardPaste *  @anyvm                @anyvm                 deny
        qubes.UpdatesProxy   *  @type:TemplateVM      @default               allow target=sys-whonix
        qubes.Filecopy       *  @anyvm                Storage                ask
        qubes.Filecopy       *  @anyvm                @default               ask
        qubes.Filecopy       *  @anyvm                @anyvm                 deny
        qubes.OpenInVM       *  Storage               fileopen               allow
        qubes.OpenInVM       *  @anyvm                @anyvm                 deny
        *                    *  @anyvm                Storage                deny
        *                    *  Storage               @anyvm                 deny
        *                    *  Surf                  @anyvm                 deny
        *                    *  @anyvm                Surf                   deny
        *                    *  sync                  @anyvm                 deny
        *                    *  @anyvm                sync                   deny
        *                    *  Multimedia            @anyvm                 deny
        *                    *  @anyvm                Multimedia             deny
        *                    *  fileopen              @anyvm                 deny
        *                    *  @dispvm:debian-X-dvm  @anyvm                 deny
        
  
  If Any person among readers have a great policy file to share, please share with me. I want to create a policy file with last line 
  
       *    *    @anyvm     @anyvm   ask/deny
       
   but VMRootShell is problem.


12. How to open every(many file types) file from a VM like Storage VM to dispVM by default:

    Create a file in StorageVM ( below instuctions are based on t-secure without passwordless root.)
   
        bash-5.0# mkdir /home/user/.local/share/applications
                  nano /home/user/.local/share/applications/browser-dvm.desktop      
      
   Type following in Editor-
       
        [Desktop Entry]
        Encoding=UTF-8
        Name=BrowserVM
        Exec=qvm-open-in-vm fileopen %u
        Terminal=false
        X-MultipleArgs=false
        Type=Application
        Categories=Network;WebBrowser;
        MimeType=x-scheme-handler/unknown;x-scheme-handler/about;text/html;text/xml;application/xhtml+xml;application/xml;application/vnd.mozilla.xul+xml;application/rss+xml;application/rdf+xml;image/gif;image/jpeg;image/png;x-scheme-handler/http;x-scheme-handler/https;application/pdf;text/plain
        
   Save with ctrl+x then y then enter.
   
   Set it as your default browser-
   
        bash-5.0# xdg-settings set default-web-browser browser-dvm.desktop
        
   Change any pdf or text or jpg default application (from file property) to browser-dvm.desktop    
     

13. Onionized Repositories

        https://www.whonix.org/wiki/Onionizing_Repositories
        https://www.qubes-os.org/news/2019/04/17/tor-onion-services-available-again/
        
14. Split-GPG 

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
    
 
 15.    https://github.com/rustybird/qubes-split-dm-crypt
 
        https://github.com/Qubes-Community/Contents/blob/master/docs/configuration/http-proxy.md
 
        https://www.qubes-os.org/doc/disposablevm-customization/#customization-of-disposablevm
 
        https://tails.boum.org/doc/about/features/index.en.html
 
        https://0xacab.org/jvoisin/mat2
        
        https://www.whonix.org/wiki/Linux_Kernel_Runtime_Guard_LKRG/Qubes
