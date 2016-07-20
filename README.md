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
## Install Mopidy

## Number 2
