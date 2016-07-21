# CyTunes
Wireless Raspberry Pi + IPad music system for Iowa State tailgates

CyTunes is a mopidy based Raspberry Pi music server that can be accessed with any MPD client or web browser.
For this project, I set it up to be accessible offline from an IPad using the MPad app. 

Essentially, I turned the Pi into a wireless router so that sends out a wifi signal. Using mopidy, the pi then turns into a mpd and html server. The IPad can then connect to the Wifi signal, and the MPad app does the rest by detecting the mopidy server automatically.

## Pros:
  - No internet access required which is perfect for tailgates
  - Can play local music files or from external hard drives/usb drives
  - Wirelessly connected to IPad using wifi hosted by the Pi
  - SSH connection to pi with laptop or IPad using same wifi
  - Multiple connections possible with multiple devices
  - Library of songs viewable from phone or tablet

## Cons:
  - Can't use Pi's wifi to connect to internet anymore so no Spotify

# Steps
## Install Mopidy  -  [Or follow these instructions](https://mopidy.readthedocs.io/en/latest/installation/raspberrypi/)
### Install the newest version of Raspbian, write it to an SD card, and boot up the Pi
  
### Open up terminal for the following commands.
  
### Add the archives GPG key
      wget -q -O - https://apt.mopidy.com/mopidy.gpg | sudo apt-key add -
      
### Pull down the Mopidy Repo   -- You may need to get the most up to date repo
      sudo wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/jessie.list

### Install Mopidy and all dependencies
      sudo apt-get update
      sudo apt-get install mopidy

### Set some config values
#### Create and edit a config file
      sudo nano .config/mopidy/mopidy.conf
#### Add these lines
      [mpd]
      hostname = ::

      [spotify]
      username = alice
      password = mysecret
      
#### To check current config run
      mopidy config
#### Any of the values printed out above can be copied into the file we edited above and changed.
#### To change the directory the music is pulled from, add this to the config file
      [local]
      media_dir = <Your_Directory_Name>
#### Set mopidy to run at boot
```
sudo dpkg-reconfigure mopidy
```
  - and set mopidy to run at boot by selecting yes

## Set Pi up as access point -- [Or follow these instructions](https://frillip.com/using-your-raspberry-pi-3-as-a-wifi-access-point-with-hostapd/)
### Install Hostapd and DNSMASQ
      sudo apt-get install dnsmasq hostapd
### Configure the interfaces
#### dhcpcd.conf
      sudo nano /etc/dhcpcd.conf
and add the following to the bottom
      denyinterfaces wlan0 
#### interfaces
      sudo nano /etc/network/interfaces
edit the wlan0 section to this
```
allow-hotplug wlan0  
iface wlan0 inet static  
    address 172.24.1.1
    netmask 255.255.255.0
    network 172.24.1.0
    broadcast 172.24.1.255
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```
#### hostapd.conf -- Create this file 
```
sudo nano /etc/hostapd/hostapd.conf
```
  - And add the following (You can edit the password and ssid values to that of your choosing.)
```
# This is the name of the WiFi interface we configured above
interface=wlan0

# Use the nl80211 driver with the brcmfmac driver
driver=nl80211

# This is the name of the network
ssid=Pi3-AP

# Use the 2.4GHz band
hw_mode=g

# Use channel 6
channel=6

# Enable 802.11n
ieee80211n=1

# Enable WMM
wmm_enabled=1

# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]

# Accept all MAC addresses
macaddr_acl=0

# Use WPA authentication
auth_algs=1

# Require clients to know the network name
ignore_broadcast_ssid=0

# Use WPA2
wpa=2

# Use a pre-shared key
wpa_key_mgmt=WPA-PSK

# The network passphrase
wpa_passphrase=raspberry

# Use AES, instead of TKIP
rsn_pairwise=CCMP
```
#### To check if it is working run
```
sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
```
  - The above will create a wifi signal that your computer or phone can connect to, but it will not share the internet connection.

#### edit config file location
```
sudo nano /etc/default/hostapd
```
  - and find the line 
```
#DAEMON_CONF="" 
```
  - and replace it with 
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
#### Configure DNSMASQ
  - copy the dnsmasq config file to save it

```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig  
sudo nano /etc/dnsmasq.conf  
```
  - paste the following into the file
```
interface=wlan0      
listen-address=172.24.1.1 
bind-interfaces      
server=8.8.8.8       
domain-needed        
bogus-priv            
dhcp-range=172.24.1.50,172.24.1.150,12h 
```
#### sysctl.conf
```
sudo nano /etc/sysctl.conf
```
  - and remove the # from the beginning of the line containing `net.ipv4.ip_forward=1`. 

#### Share internet connection
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
#### Edit rc.local
```
sudo nano /etc/rc.local
```
and just above the line containing `exit 0` add
```
iptables-restore < /etc/iptables.ipv4.nat  
```

### Now just start the services
```
sudo service hostapd start  
sudo service dnsmasq start 
```
OR
```
sudo reboot
```
## Notes
  - When the pi reboots, mopidy and the wifi signal should now both start automatically.
  - To use mopidy
    - Connect to the Pi's wifi signal
    - Either download the app MPod or MPad
    - or
    - Go to the servers address in the url of a web browser.
  - You may need to install a gstreamer library in order for mopidy to recognize .m4a files and other extensions.
  - If you plan on not having the Pi connected to an ethernet, remember to download all the packages and extensions you need before     hand.
    - For example, I am using mine as a tailgate music player, so I have no internet connection. The wifi is being used purely as a
      means of connecting two computers.
