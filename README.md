Syncthing in Qubes-OS

1. First of all, create a clone of debian template (if you haven't created yet)

       qvm-clone debian-10 debian-X
    
2. Install curl

       sudo apt-get install curl

3. Provide network to this template by sys-firewall.

4. Add the release PGP key

       curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
       
5. Add the "stable" channel to your APT sources:

       echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
       
6. Set network to none.

7. Update and install syncthing:

       sudo apt-get update
       
       sudo apt-get install syncthing
       
8. Shut down template, (see below) create AppVM based on it. Start using Syncthing.

Note: Ususally it's a good idea to set Maximum storage size to larger size for new template as you might need larger sharing storage sizes before creating AppVM.
