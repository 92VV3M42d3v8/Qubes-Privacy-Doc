How to run a private Searx instance in Qubes-OS

This tutorial is a initial Documentation to run a Private Searx instance in Qubes-OS.
( This is work in progress, which performed well in my testings.)( Recently tested on debian-11-minimal with some issues)

1. Run Terminal in Template Debian-11-minimal and install Packages here.

2. Installing Packages-

     Run following command in terminal as root-
     
       sudo apt-get install python3-dev python3-babel python3-venv uwsgi uwsgi-plugin-python3 git build-essential libxslt1-dev zlib1g-dev libffi-dev libssl-dev shellcheck

     
3. Create a Qubes AppVM based on above template and networking through VPN. Following commands need to be run as root.

4. Create user
    
       sudo -H useradd --shell /bin/bash --system --home-dir "/usr/local/searx" searx
      
       sudo -H mkdir "/usr/local/searx"
      
       sudo -H chown -R "searx:searx" "/usr/local/searx"
       
5. Install searx and dependancies
     
       $ sudo -H -u searx -i
       (searx)$ git clone "https://github.com/searx/searx.git" "/usr/local/searx/searx-src"
       (searx)$ python3 -m venv "/usr/local/searx/searx-pyenv"
       (searx)$ echo ". /usr/local/searx/searx-pyenv/bin/activate" >>  "/usr/local/searx/.profile"
       
   Now press ctrl+D and exit from searx bash session.
   
6. Now again enter searx bash
       
       sudo -H -u searx -i
       pip install -U pip
       pip install -U setuptools
       pip install -U wheel
       pip install -U pyyaml
       cd "/usr/local/searx/searx-src"
       pip install -e .
   
   Now exit this searx bash too.

7. Now as root user alter settings.yml
        
        $ sudo nano /usr/local/searx/searx-src/searx/settings.yml
        
   Alter ultrasecretkey at least.
        
        
8. Final touch before running instance

       $ sudo -H -u searx -i
       (searx)$ cd /usr/local/searx/searx-src
       (searx)$ export SEARX_SETTINGS_PATH="/usr/local/searx/searx-src/searx/settings.yml"
       

9. Launch Searx

   Now we can launch the main Searx program with Python:

       (searx)$ python searx/webapp.py

   Searx will continue to run until the terminal window is closed. (You'll probably want to get around this and allow it to run indefinitly. This can be done by running the application in the background.

   Press CTRL + C to stop the current instance from running.
   
9. Now run a browser in same VM and open  http://127.0.0.1:8888

10. Now run following command( This is obsolete at the momemt) (rc.local part is not working in debian-11, I will test further later but there is a workaround for now. See 11) Tried part is following which is not working yet, don't know reason.

        sudo gedit /rw/config/rc.local
   
    Delete everything and Paste following-
   
        #!/bin/bash
   
        sudo -H useradd --shell /bin/bash --system --home-dir "/usr/local/searx" searx
        sudo -H -u searx -i
        (searx)$ cd /usr/local/searx/searx-src
        (searx)$ python searx/webapp.py

    Make it executable.
    
    
11. Restart VM 
    open sudo terminal and run following in order-
          
        sudo -H useradd --shell /bin/bash --system --home-dir "/usr/local/searx" searx
        sudo -H -u searx -i
        (searx)$ cd /usr/local/searx/searx-src
        (searx)$ python searx/webapp.py
         
12. reach http://127.0.0.1:8888
    
    Hooray our own private Searx instance running.
     
If you run VPN you can set VPN VM as NetVM of this App VM.
     

Note:
    
   Alter settings as you want like altering Secretkey, default-theme (logicodev-dark), instance name, outgoing request timeout (5,10), enabled plugins like HTTPS rewrite, Tracker URL remover etc. save it and exit. 
   
   
