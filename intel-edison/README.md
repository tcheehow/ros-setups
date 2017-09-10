# Intel Edison Setup

# Before you Start

Before starting with the installation it's a good idea to boot the Edison straight out of the box to make sure it's working. This way we can make sure we have a functional board before proceeding and we won't be mistakenly blaming setup issues if something is wrong here.

Connect one USB cable to the cosole port and then start your temrminal app (see next section for more information on this). Once you are connected plug in the second USB cable for power and after 15 seconds you should see the system booting. If you want to login the user name is root (no password).

# Flash Debian

Download jubilinux from http://www.jubilinux.org/

If Windows is used, dfu-util is required. Download the latest version from this page:
http://dfu-util.sourceforge.net/releases/

Make sure you have the console USB cable in place and use it so you know when the installation has finished. You MUST NOT remove power before it’s done or it could be bricked. If you don't have a console connection make sure you wait 2 minutes at the end of the installation as it instructs. During this time it is completing the installion which shoudln't be interrupted. If you don't get any update on your console after this message is displayed restart your console terminal connection.

Connect to the console with 115000 8N1, for example:

`screen /dev/USB0 115200 8N1`

and login as root (password: edison)


## Post Debian Install
After Debian has been installed you will end up with the following partitions:

```
Filesystem       Size  Used Avail Use% Mounted on
rootfs           1.4G  813M  503M  62% /
/dev/root        1.4G  813M  503M  62% /
devtmpfs         480M     0  480M   0% /dev
tmpfs             97M  292K   96M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            193M     0  193M   0% /run/shm
tmpfs            481M     0  481M   0% /tmp
/dev/mmcblk0p7    32M  5.3M   27M  17% /boot
/dev/mmcblk0p10  1.3G  2.0M  1.3G   1% /home
```

## Post ROS Install
Once ROS is installed there won't be much space left on the home partition. 

```
Filesystem       Size  Used Avail Use% Mounted on
rootfs           1.4G  1.1G  194M  86% /
/dev/root        1.4G  1.1G  194M  86% /
devtmpfs         480M     0  480M   0% /dev
tmpfs             97M  304K   96M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            193M     0  193M   0% /run/shm
tmpfs            481M  6.6M  474M   2% /tmp
/dev/mmcblk0p7    32M  5.3M   27M  17% /boot
/dev/mmcblk0p10  1.3G  381M  910M  30% /home
```

# Post Installation Steps

## Freeing up Space on the Home Partition

You will need more space on the home partition. Run the following commands:

`sudo su`

`mv /usr/share /opt/.edison/`  
`ln -s /opt/.edison/ /usr/share`  

`exit`

## Creating Catkin PACKAGE

`mkdir ~/catkin_ws/src`
`cd ~/catkin_ws/`
`catkin_make`

If facing CMAKE error, `cd ~/catkin_ws` and `sudo chown -R edison:users .`

## Install AIR, MAVROS_EXTRAS, GEOMETRY

```
git clone https://github.com/tcheehow/air.git
git clone -b edison https://github.com/tcheehow/mavros.git
git clone -b indigo-devel https://github.com/ros/geometry.git
```

## Wifi

Run `sudo cp /etc/network/interfaces /etc/network/interfaces.home`
Run `sudo cp /etc/network/interfaces /etc/network/interfaces.home`

Run `wpa_passphrase your-ssid your-wifi-password` to generate pka.
`cd /etc/network`
Edit both /etc/network/interfaces.home and /etc/network/interfaces.work
- Change wpa-ssid
- Change wpa-pka
- Comment out `auto usb0` plus the three lines that follow it (interface definition)
- Uncomment `auto wlan0`
- Save
Run: `ifup wlan0`

If you want to use a static IP then your config will look something like this:
```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo  
iface lo inet loopback

auto wlan0  
iface wlan0 inet dhcp  
    wpa-ssid ExampleWifi
    wpa-psk 81088ba3b4b387ea4d22a4ad369ffa42f4966d2f3d61f6c65cdc001460239dc4
post-up iwconfig wlan0 power off #disable edison power management    
```

Create script `sudo nano ~/homenet.sh` and `sudo nano ~/worknet.sh`
Make script executable `sudo chmod +x ~/homenet.sh` and `sudo chmod +x ~/worknet.sh`

Filling the following in the script
<pre><code>
#!/bin/bash
# Change Network to Home Network
cp /etc/network/interfaces.home /etc/network/interfaces
echo "Disable wlan0"
ifdown wlan0
echo "Re-enable wlan0"
ifup wlan0
</code></pre>

For the remaining steps you may wish to login via ssh instead.

## Update

Add the following to the sources list (/etc/apt/sources.list)

```
deb http://ftp.sg.debian.org/debian jessie main contrib non-free
#deb-src http://http.debian.net/debian jessie main contrib non-free

deb http://ftp.sg.debian.org/debian jessie-updates main contrib non-free
#deb-src http://http.debian.net/debian jessie-updates main contrib non-free

deb http://security.debian.org/ jessie/updates main contrib non-free
#deb-src http://security.debian.org/ jessie/updates main contrib non-free

#deb http://ubilinux.org/edison wheezy main

deb http://ftp.sg.debian.org/debian jessie-backports main
```

```
apt-get -y update
apt-get -f install
apt-get -y upgrade
```

## Locales
```
apt-get install locales
dpkg-reconfigure locales # Select only en_US.UTF8 and select None as the default on the confirmation page that follows.
update-locale
```
Update the `/etc/default/locale` file and ensure `LANG=en_US.UTF-8` and it uncommented out. Add `LC_ALL=C`. Then reboot.

Note that if you receive warning messages about missing or wrong languages this is likely to be due to the locale being forwarded when using SSH. Either ignore them or complete this step via the serial console by commenting out the SendEnv LANG LC_* line in the local /etc/ssh/ssh_config file on your machine (not the Edison).

## Timezone

`sudo apt-get install ntp`

`sudo nano /etc/ntp.conf` and change `server 0.debian.pool.ntp.org` to `server 0.sg.pool.ntp.org`

`sudo dpkg-reconfigure tzdata`

## Tools
```
apt-get -y install git
apt-get -y install sudo less
```

## User

Always login as root due to permission issues for serial and i2c.

## Add host
`nano /etc/hosts` and add below localhost `127.0.0.1 edison`

Login as root to continue.

# ROS/MAVROS Installation

As ROS packages for the Edison/Ubilinux don't exist we will have to build it from source. This process will take about 1.5 hours but most of it is just waiting for it to build.

A script has been writen to automate the building and installation of ROS. Current testing has been copy-pasting line by line to the console. Willing testers are encouraged to try out running the script:

```
git clone https://github.com/tcheehow/ros-setups
cd ros-setups/intel-edison/
./install_ros.sh
```

If all went well you should have a ROS installation. Hook your Edison up to the Pixhawk and run a test. See this page for instructions: https://pixhawk.org/peripherals/onboard_computers/intel_edison

# Install Edison MRAA Libraries

Follow the instructions here to install the latest swig for mraa http://swig.org/svn.html

To build swig, `apt-get install bison automake autoconf build-essential g++`.

Follow the instructions on https://learn.sparkfun.com/tutorials/installing-libmraa-on-ubilinux-for-edison

If encounter `Unknown Cmake command "target_include_directories"`, modify mraa source code in accordance to the following commit:
https://github.com/Drunkar/mraa/commit/8c1891013a6665ac35d33ff00e13f1e3db3d53f5

In short,

in /mraa/src/python/python2/CMakeLists.txt
```
-    target_include_directories(${SWIG_MODULE_python2-mraa_REAL_NAME}
-      PUBLIC
+    set_property(TARGET ${SWIG_MODULE_python2-mraa_REAL_NAME}
+      APPEND PROPERTY INCLUDE_DIRECTORIES
```

in /mraa/src/python/python3/CMakeLists.txt
```
-    target_include_directories(${SWIG_MODULE_python3-mraa_REAL_NAME}
-      PUBLIC
+    set_property(TARGET ${SWIG_MODULE_python3-mraa_REAL_NAME}
+      APPEND PROPERTY INCLUDE_DIRECTORIES
```


# Setting I2C Permission with udev rules

```
sudo usermode -aG i2c edison
```
Reboot the edison for udev rules to take effect.

# Setting Serial Permission with udev rules

```
sudo usermode -aG dialout edison
```
Reboot the edison for udev rules to take effect.

# Python Flight App

Once you have a functional ROS setup you can *very carefully* perform an offboard flight using the setpoint_demo.py script. This script assumes that you have already successfully run `roslaunch mavros px4.launch`.

WARNING WARNING: Make sure you can take control via RC transmitter at any time, things can go quite wrong. Also be aware that there isn't any velocity control currently and the multirotor will use max velocity at times. Read the code before you fly so you know what to expect.

Launch the demo by running:

`./setpoint_demo.py`

and once it is running activate offboard control on your RC transmitter.

## Freeing up Space on the Root Partition

Once again we will remove unneeded files from the root partition. You can delete the files in root's home directory (that's /root) or move them to the home partition.

## Mounting the Edison as a mass storage using Win-sshfs

**Note this is for Window Users only!**

1. Download Dokan 0.7.4 from here : https://github.com/dokan-dev/dokany/releases?after=v0.8.0

2. Download Win-sshfs 1.5.12.8 release from: https://github.com/Foreveryone-cz/win-sshfs/releases

3. Install Dokan 0.7.4 (restart is required after installation)

   - If Visual C++ Redistributable Packages for Visual Studio 2013 (x86) is not installed,

     **Download the x86 redistributable installer!**

   - Install Visual C++ Redistributable Packages for Visual Studio 2013 (x86).

4. Unpack the Win-sshfs 1.5.12.8 zip file and run Win-sshfs

5. Fill Win-sshfs according to the picture.

   ![win-sshfs](C:\Users\AIR LAB\Documents\win-sshfs.png)

6. Replace Host with the IP address of your Edison

7. Click mount. The Edison should show as a removable drive on your computer.
