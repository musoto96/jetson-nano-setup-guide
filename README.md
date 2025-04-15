# Jetson-nano setup notes for desktop or headless mode.

---

<br>

### Introduction.
This is a complete setup guide for the Nvidia Jetson nano system, it is mainly written as a personal reminder for future setups but also as a guide for anyone in need of a similar setup.

<br>

###  Objective
The goal is to get the Jetson nano running headless, powering it via the usb cable and operating it with a host machine either through ssh or a VNC server for a virtual desktop.

<br>

### Requirements.
1. The guide assumes you have a capable micro-sd card flashed with a Jetson Nano developer kit image
2. A jumper for the *J-48 pin*, and a compatible barrel-jack power supply (only for the initial setup)
3. The peripherals to operate the Jetson Nano in stand-alone mode (only for the initial configuration)
4. A usb cable with data capabilities as we will configure it for headless mode and operate it via the host-machine

<br>

###  What this document covers.
1. Fixing the dekstop sharing window error.
2. VNC setup.

<br>

---

<br>

**Installing the base system**
Attach a screen, mouse, keyboard, put a jumper in the *J-48 pin* (in front of barrel-jack) and turn on the Jetson Nano. Proceed with the (graphical) installation normally.

After the instalation of the system it would be recommended to do 
```
sudo apt update
sudo apt upgrade
```

<br>

**Fixing the desktop sharing window error**
Thanks to this article: https://www.hackster.io/news/getting-started-with-the-nvidia-jetson-nano-developer-kit-43aa7c298797

At this point if you tried opening the desktop sharing options you would get an error after a long pause. To fix this we need to edit the schema for the Vino binary in the following way:
```
sudo vim /usr/share/glib-2.0/schemas/org.gnome.Vino.gschema.xml
```

and append the following key, (or add it at the beginning before the first key for "prompt-enabled" begins)
```
    <key name='enabled' type='b'>
      <summary>Enable remote access to the desktop</summary>
      <description>
        If true, allows remote access to the desktop via the RFB 
        protocol. Users on remote machines may then connect to the 
        desktop using a VNC viewer.
      </description>
      <default>false</default>
    </key>
```

then compile the schemas with 
```
sudo glib-compile-schemas /usr/share/glib-2.0/schemas
```

<br>

**VNC Setup.**
At this point we can follow the README-vnc.txt which is included along with other files in the system documentation and should be mounted automatically under `/media/username/L4-README`.

From `README-vnc.txt` we see the following steps:
```
sudo apt update
sudo apt install vino

# Enable the VNC server to start each time you log in
sudo ln -s ../vino-server.service /usr/lib/systemd/user/graphical-session.target.wants

# Configure the VNC server
gsettings set org.gnome.Vino prompt-enabled false
gsettings set org.gnome.Vino require-encryption false

# Set a password to access the VNC server
# Replace thepassword with your desired password
gsettings set org.gnome.Vino authentication-methods "['vnc']"
gsettings set org.gnome.Vino vnc-password $(echo -n 'thepassword'|base64)

# Reboot the system so that the settings take effect
sudo reboot
```

**NB**: The VNC server is only available after you have logged in to Jetson locally. If you wish VNC to be available automatically, use the GUI **System settings** application to enable automatic login.

The desktop resolution is typically determined by the capabilities of the display that is attached to Jetson. If no display is attached, a default resolution of 640x480 is selected. To use a different resolution, edit `/etc/X11/xorg.conf` and append the following lines:
```
Section "Screen"
   Identifier    "Default Screen"
   Monitor       "Configured Monitor"
   Device        "Tegra0"
   SubSection "Display"
       Depth    24
       Virtual 1280 800 # Modify the resolution by editing these values
   EndSubSection
EndSection
```


To get more (and possibly updated) information please refer to your local `README-vnc.txt`.

<br>

At this point we should be able to access the remote desktop. Install a vnc client on the host machine and access the Jetson. 

If not connected to a network plug in the usb cable from the Jetson to the host machine. The ip for the Jetson is `192.168.55.1` and `192.168.55.100` for the host by default, try pinging it from your host, if there is no response check that the host machine is connected to the new interface that was created when plugging the jetson to the host via usb.

Shutdown the Jetson and remove all peripherals including the J-48 jumper, and only connect the usb to the host machine. You should be able to access your VNC server now.