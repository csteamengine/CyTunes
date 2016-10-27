# Rune Audio
This is a guide to setting up a music player using Rune Audio that also turns the RPi into a wifi router, so no external router is needed. 

## Rune Audio 
Rune audio by default, connects to your already existing router, and this allows you to connect to it from other computers. The beauty of WiFi 
Routers, is that even if they aren't connected to the internet, you can still connect to them and share files and other stuff with other computers
connected to the same router. In this case, I will be making the RPi into a WiFi router that will be disconnected from the internet. Rather than 
using the onboard wifi to connect to a wifi signal, we will be converting it to send out a WiFi signal that other devices connect to.

### Use Case
You will plug in the RPi, which will boot up into RuneAudio, then the RPi will automatically start its own WiFi sharing signal. The user would then
get on their other device(s) and connect to the RPi's wifi signal. The user would then be able to surf the music files on the RPi or any storage devices
connected to the RPi. Say for example you have a 1TB hard drive full of music connected to your RPi. When you boot the RPi into RuneAudio, it will
scan those music files automatically, start the wifi signal, and you can now view those music files from your other devices and play them. 

## Setup
#### *Warning -- This is for advanced users, if done incorrectly, could cause the RPi to fail at boot*
To set up RuneAudio, follow these extremely well documented [Instructions](http://www.runeaudio.com/documentation/quick-start/quick-start-guide/).

Once you have the RuneAudio img flashed to your micro SD card and the RPi booted into RuneAudio, follow these instructions to get a local Wifi signal going.

### Root Login
You will need to either connect a monitor and keyboard to the RPi, or SSH into the RPI. My personal favorite is SSH (Less Wires)
You will also need to know the IP address of the RPi (Instructions for which can be found in the link above.)
 - Windows
    - Download Putty and connect using the ip address of the RPi
 - Mac and Linux
    - In the terminal type
        ```
        ssh root@<ip address of the pi>
        ```
    - (where you replace <ip address of the pi> with the actual IP address of the RPi)
 - Username is `root` and password is `rune`
 
## Set up RPi as WiFi Access Point
[Original Instructions](https://frillip.com/using-your-raspberry-pi-3-as-a-wifi-access-point-with-hostapd/)
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
