# RaspberryPi-VPN
<h2>Build Your Own Virtual Private Network With A Raspberry Pi 3</h2>

<h3>Reasons to build a VPN if you are long in time and short in money</h3>

- You are tired of paying for VPN
- You are connected to a public network and you want to make your connection secure (I'm not sure how secure this connection could get)
- You need to connect to your home network remotely
- You are travelling to a filtered internet zone (There is no guarantee that this method would unbound your filtered connection)
- You already have a **Raspberry Pi 3** and you want to do something cool with it :)

When we talk about Raspberry pi package, we normally talk about these items.
1. Raspberry Pi board
1. MicroSD card
1. HDMI cable
1. Keyboard (and a mouse)  

<h3> How to install Raspbian on a MicroSD?</h3>  

There are several different operating systems you can install and use. See the full list of operating systems [here](https://www.raspberrypi.org/downloads/). For the Simplicity of process for us and for the RPi's CPU we recommand you to download these two items.  

1. Raspberry Pi Imager (for [Windows](https://downloads.raspberrypi.org/imager/imager.exe), for [macOS](https://downloads.raspberrypi.org/imager/imager.dmg), for [Ubuntu](https://downloads.raspberrypi.org/imager/imager_amd64.deb))
1. Rasbian Lite ([Download Link](https://www.raspberrypi.org/downloads/raspbian/))  

Install the imager and install the Rasbian using the imager to the microsd card.

<h3>First Startup</h3>

The fist time you start a RPi it would ask for login information. The default username and password is:

    Username: pi
    Password: raspberry

SSH connection is normally set to disabled by default. If you want to access to Raspbian easily without being have to use a TV and a keyboard you need to enable SSH connection first.  

<h4>Find your IP address</h4>

    ifconfig

is the command you write in the terminal to see your IP (inet). Write down the netmask and broadcast just in case. We are going to need it in the future.

<h4>Enable SSH</h4>  

    sudo raspi-config  
    
will take you to the **Configuration Tool**. Then go to **Interfacing Option** and enable SSH.

<h4> Set Static IP</h4>

Since we are warmed up with writing commands in terminal, let's set a static IP and disconnect the HDMI.   
Write this command to open the **dhcpcd.conf** file.  

    sudo nano /etc/dhcpcd.conf  
    
Now, you need to uncomment this part if you are using **eth0** connection. We recommend you not to use wifi to connect your Raspberry Pi to internet.

```   
interface eth0
static ip_address=<IP>/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
static routers=<GW>
static domain_name_servers=<GW> 8.8.8.8 fd51:42f8:caae:d92e::1

profile static_eth0
static ip_address=<IP>/24
static routers= <GW>
static domain_name_servers= <GW>
```

Where \<IP> is the IP address you saved before and \<GW> is in fact the Gateway or the IP to access your modem (normally 192.168.0.1)  
Make sure to put correct information. Reboot your RPi. Now you can disconnect your HTMI cable and connect to RPi with your computer.

<h3> Connect To RPi Via SSH </h3>

Raspberry Pi can be accessed remotely since you enabled SSH.   
Download and install [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)   
Open Putty put your RPi IP address. The port should be 22. Connect to RPi.

It's time to change your RPi password and choose a stronger one. The command below will let you change the password.

    passwd
    
Now you are ready to make your first VPN on RPi.

<h3>Get A Dynamic DNS Hostname</h3>

There are several services for it. I use [No-IP](https://www.noip.com/). You can make up to three free hostnames. Just remember that you have to extend your hostname every month or it would expire.  
A typical format for a hostname is: name.hopto.org. Write it down somewhere. You are gonna need it.  

<h3>Download And Install PiVPN</h3>

- Update first with this command:   

```
    sudo apt-get update
```

- Go to your modem (probably 192.168.0.1) and make sure the static IP for Raspberry Pi is reserved (static) there   
- Open Putty and connect to your RPi. Write this command to install pivpn.

```
    curl -L https://install.pivpn.io | bash
```
    
Now we dig into the questions it asks.   
- Are you Using DHCP Reservation on your Router/DHCP Server?   
    Answer yes  
- Choose local user: pi   
- WireGuard or OpenVPN? OpenVPN  (select by pressing **Space**)  
- UDP or TCP? I recommend UDP to be faster and a bit more loss  
- Port? Choose a different port such as 1990 or 21021 (any 4 or 5 digit number is good)  
- DNS Provider? Chose google  
- Would you like to add a custom search domain? You don't need to  
- Will clients use a Public IP or DNS Name to connect to your server? Put the hostname you made in No-IP (<name>.hopto.org)  
- Enable features for OpenVPN 2.4 or higher? Recommended  
- Desired size of your certificate: choose 256 if you don't want to access sensitive information :P   
- Unattended upgrades: Yes  
- Reboot now? If you want to use pivpn right away  

<h4> Make Clients </h4>  

- Make one client for each device you want to connect: `pivpn add`
- Choose a name that lets you manage clients easier (Hesam.Phone, Sara.Ipad, etc)  
- Choose not easy to guess passwords
- You can see the connected clients by writing this command: `pivpn -c`
- Just write `pivpn` to see all possible commands

<h4>Get .ovpn Client Files</h4>

There are several methods to access files and folders inside a Raspberry Pi. Since RPi has Python installed, we go with SimpleHTTPSever.  
1. Write this command  

    ```
    cd ovpns  
    python -m SimpleHTTPServer 8080  
    ```
    
2. Open a web browser and write this in the address bar:  

    ```
    <RPi IP>:8080  
    ```
    The RPi IP is the static IP you set for your RPi. (Example: 192.168.1.15:8080)  
    
You should see the .ovpn client files you made. Click on them and they are now downloded to your PC   
Once done hit **Ctrl + c** to stop RPi sharing the directory.  

<h3> Connect To VPN </h3>  

- Your Raspberry Pi is on and pivpn is running. Use the command `<pivpn -d>` just to make sure that you are done with the RPi and everything is in order with the VPN server you just made.  
- You need to download and install OpenVPN on your PC, Laptop or Phone. You can find it in this [Link](https://openvpn.net/community-downloads/)  
- Once it's installed, open **OpenVPN GUI** (go to the icon in the taskbar) and import the .ovpn file. Right click on the icon again and connect.  

**Important note:** You may not be able to connect to VPN if you are connected to the same modem as the RPi is connected. To test if your VPN works, connect to a different modem or use hotspot on your phone connected to mobile data.  

If you want to make a local network for gaming with your friends, this method should work.

    
    







