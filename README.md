# ROS Setups

Instructions and scripts for the installation of ROS on various platforms, such as Odroid and Intel Edison.

# Building the Edison Debian Image

## System Requirements

Ubuntu 14.04 LTS (Doesn't work on 16.04 LTS)

It takes about 4-5 hours build time depending on number of CPU cores.

## References

1. [How to Build Debian Image for Edison](https://communities.intel.com/thread/110217 "Intel")
2. [Building Debian Linux for Intel Edison](http://www.hackgnar.com/2016/02/building-debian-linux-for-intel-edison.html "Hackgnar")
3. [Building a Custom Debian Image for the Intel Edison](https://jakehewitt.github.io/custom-edison-image/ "JakeHewitt")

## Create a Directory for Building the Images

`cd ~`  
`mkdir -p ~/src/edison`  
`cd ~/src/edison`  

## Install Build Dependencies

`sudo apt-get -y install build-essential git diffstat gawk chrpath texinfo libtool gcc-multilib debootstrap u-boot-tools debian-archive-keyring python curl`

## Download Latest Source

`curl -O http://downloadmirror.intel.com/25028/eng/edison-src-ww25.5-15.tgz`

## Uncompress the Source Files

`tar xfvz edison-src-ww25.5-15.tgz`

## Get Started

Create two resource directories so we can rebuild the images without fetching support file twice

`mkdir bitbake_download_dir`  
`mkdir bitbake_sstate_dir`  

To take advantage of parallelization, change `parallel_make` and `bb_number_thread` to equal to the number of cores available.

`./meta-intel-edison/setup.sh --dl_dir=bitbake_download_dir --sstate_dir=bitbake_sstate_dir --deb_packages --parallel_make=40 --bb_number_thread=40`

## Setup Enviroment Variables

`cd out/linux64`  
`source poky/oe-init-build-env`  

## Edit Some Errors

1. paho-mqtt_3.1.bb  
   `sudo nano ~/src/edison/edison-src/linux64/poky/meta-intel-iot-middleware/recipes-connectivity/paho-mqtt/paho-mqtt_3.1.bb`⋅⋅
   and change SRC_URI from   `git://git.eclipse.org/gitroot/paho/org.eclipse.paho.mqtt.c.git`  
   to `git://github.com/eclipse/paho.mqtt.c.git`  

2. edison-images.bb  
   `sudo nano ~/src/edison/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-core/images/edison-images.bb`  
   Comment the following lines  
   `IMAGE_INSTALL += "iotkit-comm-js"`  
   `IMAGE_INSTALL += "iotkit-comm-c-dev"`  


## Building the Image (Part 1)

Now we are ready to build the image. I will take several hours, around 5 hours. (It takes 20 mins on a 40-core CPU :P)

run `bitbake edison-image`

## Building the Image (Part 2)

`cd ~/src/edison-src`  
`sudo nano meta-intel-edison/utils/create-debian-image.sh`  

1. Change `build_dir=$top_repo_dir/build`
   to `build_dir=$top_repo_dir/out/linux64/build`

2. Change ``fsize=$((`stat --printf="%s" toFlash/edison-image-edison.ext4` / 524288))`` to ``fsize=$((`stat --printf="%s" toFlash/edison-image-edison.ext4` / 524288 * 2)) ``

3. Change `$CHROOTCMD dpkg -i /tmp/deb/edison/kernel-image-3.10.17-poky-edison+_1.0-r2_i386.deb`  
   `$CHROOTCMD dpkg -i /tmp/deb/edison/kernel-3.10.17-poky-edison+_1.0-r2_i386.deb`  
   to `$CHROOTCMD dpkg -i /tmp/deb/edison/kernel-image-3.10.98-poky-edison+_1.0-r2_i386.deb`  `$CHROOTCMD dpkg -i /tmp/deb/edison/kernel-3.10.98-poky-edison+_1.0-r2_i386.deb`

Finally, we cleared all the mistakes in the scripts.

`sudo ./meta-intel-edison/utils/create-debian-image.sh`

The flashable image is build in `out/linux64/build/toFlash`
