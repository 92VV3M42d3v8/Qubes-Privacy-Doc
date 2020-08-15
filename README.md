Syncthing in Qubes-OS

1. First of all, create a clone of debian template (if you haven't created yet)

       qvm-clone debian-10 debian-X
    
2. Run debian-X terminal. Also run a fresh disposable appVM to download key from syncthing.

3. In this dispVM go to https://syncthing.net/release-key.txt and copy this key and create a syncthing.pubkey file in template debian- X by pasting it. In debian-X:

       $ nano syncthing.pubkey

4. Add key to keyring-

       sudo apt-key add syncthing.pubkey

       
5. Add the "stable" channel to your APT sources:

       echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
       

6. Update and install syncthing:

       sudo apt-get update
       
       sudo apt-get install syncthing
       
7. Shut down template, (see below) create AppVM based on it. Start using Syncthing.

Note: Ususally it's a good idea to set Maximum storage size to larger size for new template as you might need larger sharing storage sizes before creating AppVM.
