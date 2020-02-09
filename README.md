How to run a private Searx instance in Qubes-OS

This tutorial is a initial Documentation to run a Private Searx instance in Qubes-OS.
( This is work in progress, which performed well in my testings.)

1. Run Terminal in Template Debian-10 and install dependacies here.

2. Installing Dependancies-

     Run following command in terminal-
     
       sudo apt-get install git build-essential libxslt-dev python-dev python-virtualenv python-babel zlib1g-dev libffi-dev libssl-dev
     
3. Create a Qubes AppVM based on Debian-10 and networking through Default(Sys-Firewall). Do not provide Network.

4. Setup installation directory

    Now open termianl in newly created VM, we can move to the directory that we'll install Searx in:

       cd /usr/local/

    Next, we'll use git to download a copy of the Searx source code here:

       sudo git clone https://github.com/asciimoo/searx.git

5. See Note below for Settings Alteration.

6. Now we're going to create a new user for Searx to use and assign it directory privileges:

       sudo useradd searx -d /usr/local/searx
    
       sudo chown searx:searx -R /usr/local/searx

7. Build Searx

   Now we can begin building Searx.

   First, lets move to the directory we created when downloading the source code:

       cd searx/

   Next, we'll switch to our newly created user:

       sudo -u searx -i

   Once we're logged in, we can configure and activate the Searx virtual environment. This allows Searx to run within its own   environment so we can ensure that it runs properly without restrictions. Enter the following commands to activate the environment:

       virtualenv -p python3 venv3
   
       . ./venv3/bin/activate

       pip3 install -r requirements.txt

8. Launch Searx

   Now we can launch the main Searx program with Python:

       python3 searx/webapp.py

   Searx will continue to run until the terminal window is closed. (You'll probably want to get around this and allow it to run indefinitly. This can be done by running the application in the background.

   Press CTRL + C to stop the current instance from running and then enter the command below:)(untested)

       nohup python searx/webapp.py &

9. Now run a browser in same VM and open  http://127.0.0.1:8888
   
10. Now shut down this App VM.

11. Now again start the App VM terminal and run following command

       sudo gedit /rw/config/rc.local
   
   Delete everything and Paste following-
   
       cd /usr/local/searx
   
       . ./venv3/bin/activate
   
       python3 searx/webapp.py
   
10. Now again open browser and reach http://127.0.0.1:8888
    
    Hooray our own private Searx instance running.
     
If you run VPN you can set VPN VM as NetVM of this App VM.
     

Note:

   If you want to modify Settings.yml file, You have to do it after step 4. For that-
   
    sudo gedit /usr/local/searx/searx/settings.yml
    
   Alter settings as you want like altering Secretkey, default-theme etc. save it and exit. 
