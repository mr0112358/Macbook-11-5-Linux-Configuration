# Macbookpro 11,5 Linux Costomization
## Installation
We select Manjaro KDE Edition. 
### Making boot U-disk
* Reformat the U-disk to FAT.
* Unmount the disk
* Make the boot disk by `sudo dd if=name.iso of=/dev/sd*`

### Installing rEFInd
* Download [rEFInd](http://www.rodsbooks.com/refind/getting.html) 
* Reboot to Recovery Mode by holding `Command + R` at the beginning of boot.
* Open Terminal `cd` to the directory of rEFInd and type `./refind-install`
* `cd /efi/refind` and `vim refind.conf`. Uncomment the line `spoof_osx_version`
* Reboot your MAC and boot on the U-disk and install the system.
## Modification
### GPU issue
Since the MAC will automatically silence the Intel-GPU, so we spoofed by change the `refind.conf`. 

* To make it work, we still need to create Xorg config for i-GPU

`vim /etc/X11/xorg.conf.d/20-intel.conf`
~~~
Section "Device"
	Identifier "Intel Graphics"
	Driver "intel"
	BusID "PCI:0:2:0"
	Option "TearFree" "true"
	Option "AccelMethod" "glamor"
EndSection
~~~
* Clone gpu-switch and run it (from linux)
~~~
git clone https://github.com/0xbb/gpu-switch
cd gpu-switch
sudo ./gpu-switch -i # needs root. switches system to iGPU. needs reboot.
~~~

Now we have a working intel GPU now with proper HiDPI detection. We can also blacklist the AMD driver in `/etc/modprobe.d/blacklist.conf` by adding `blacklist amdgpu` to the end of the file.

* *unsolved: now we cannot boot in once we logged into OSX.* If so, we could use the boot U-disk and do the same above. See https://github.com/Dunedan/mbp-2016-linux/issues/6 for future solution.

### Modify the touchpad
The default driver `xf86-input-libinput` does not have a good support for multitouch. Thus we install the `x86-input-mtrack` in the AUR via `yaourt`.

Then we should change the configuration by
~~~
cd /usr/share/X11/xorg.conf.d/
vim 10-mtrack.conf	
~~~
Replace all of them by 
~~~
Section "InputClass"
    MatchIsTouchpad "on"
    Identifier "Touchpads"
    Driver "mtrack"
    Option "ConstantDeceleration" "3.8"
        Option          "Sensitivity" "1"
        Option          "FingerHigh" "5"
        Option          "FingerLow" "1" # 1
        Option          "IgnoreThumb" "false"
        Option          "ThumbRatio" "70"
        Option          "ThumbSize" "25"
        Option          "IgnorePalm" "true"
        Option          "TapButton1" "0"
        Option          "TapButton2" "0"
        Option          "TapButton3" "0"
        Option          "TapButton4" "0"
        Option          "ClickFinger1" "1"
        Option          "ClickFinger2" "3"
        Option          "ClickFinger3" "2"
        Option          "ButtonMoveEmulate" "false"
        Option          "ButtonIntegrated" "true"
        Option          "ClickTime" "25"
        Option          "BottomEdge" "30"
# Swipe to Drag
	Option          "SwipeLeftButton" "1"
        Option          "SwipeRightButton" "1"
        Option          "SwipeUpButton" "1"
        Option          "SwipeDownButton" "1"
        Option          "SwipeDistance" "1"
	Option		"SwipeClickTime" "0"
	Option		"SwipeSensitivity" "1000"

#        Option          "ScrollCoastDuration" "500"
#        Option          "ScrollCoastEnableSpeed" ".3"
        Option          "ScrollUpButton" "5"
        Option          "ScrollDownButton" "4"
        Option          "ScrollLeftButton" "7"
        Option          "ScrollRightButton" "6"
        Option          "ScrollDistance" "150"

	Option 		"Hold1Move1StationaryButton" "0"
EndSection
~~~
Rename it by `mv 10-mtrack.conf 50-mtrack.conf`
Log out and log in to activate it.

### Control the Screen Backlight
We first follow [acpilight](https://github.com/wavexx/acpilight/).

To do so, place a file in /etc/udev/rules.d/90-backlight.rules containing:
~~~
SUBSYSTEM=="backlight", ACTION=="add", \
  RUN+="/bin/chgrp video %S%p/brightness", \
  RUN+="/bin/chmod g+w %S%p/brightness"
~~~

The only reliable display backlight control is provided by the driver "apple_gmux". The sysfs interface is under `/sys/class/backlight/gmux_backlight`.

We add below script to custom shortcuts,

> Add `bright+` with action `echo 200 >  /sys/class/backlight/gmux_backlight/brightness`

> Add `bright-` with action `echo 100 >  /sys/class/backlight/gmux_backlight/brightness`
