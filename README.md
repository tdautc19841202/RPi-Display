# RPi-Display
2.8" TFT-Display (240x320) with Touchpanel for Raspberry Pi.

Shop: [RPi-Display](http://www.watterott.com/en/RPi-Display)

![RPi-Display](https://raw.github.com/watterott/RPi-Display/master/img/rpi-display.jpg)


## SD-Card Image
Ready to run SD-Card image (Raspbian/Debian).
Copy the image to a SD-Card with [dd](http://en.wikipedia.org/wiki/Dd_%28Unix%29) under Linux or [Win32-Disk-Imager](http://sourceforge.net/projects/win32diskimager/) under Windows.
Further infos can be found [here](https://github.com/notro/fbtft/wiki/FBTFT-shield-image).

* [Raspbian-FBTFT-RPi-Display](http://tronnes.org/fbtft/download.html)
* Mirror: [Raspbian-FBTFT-RPi-Display](http://www.watterott.net/fbtft/2014-01-07-wheezy-raspbian-2014-02-03-fbtft-RPi-Display.zip)


## Linux Driver Installation

There is a [Linux Framebuffer driver (FBTFT)](https://github.com/notro/fbtft/wiki) available for the display (MI0283QT-9) and the ADS7846 touch controller has also a Linux device driver.

* [Install FBTFT](https://github.com/notro/fbtft/wiki#install) Framebuffer:

    Install rpi-update for the kernel update:
    ```
    $ sudo wget https://raw.github.com/Hexxeh/rpi-update/master/rpi-update -O /usr/bin/rpi-update
    $ sudo chmod +x /usr/bin/rpi-update
    ```

    Remove or comment out the SPI blacklist line (spi-bcm2708):
    ```
    $ sudo nano /etc/modprobe.d/raspi-blacklist.conf
    ```

    Start kernel update:
    ```
    $ sudo REPO_URI=https://github.com/notro/rpi-firmware rpi-update
    $ sudo shutdown -r now
    ```

* Install Touchscreen Tools:

    ```
    $ sudo apt-get install xinput evtest
    ```

* [Activate Framebuffer](https://github.com/notro/fbtft/wiki#enable-driver):

    ```
    $ sudo modprobe fbtft_device name=mi0283qt-9a cs=0 gpios=reset:23,led:18 rotate=90 speed=16000000
    ```

    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    fbtft_device name=mi0283qt-9a cs=0 gpios=reset:23,led:18 rotate=90 speed=16000000
    ```

    *Note: For a higher speed than 16MHz the display has to be connected directly to the Raspberry Pi or with wires not longer than 5cm.*

* [Activate Touchpanel](https://github.com/notro/fbtft/wiki/Touchpanel#watterott-mi0283qt-9a):

    ```
    $ sudo modprobe ads7846_device model=7846 cs=1 gpio_pendown=25 speed=2000000 keep_vref_on=1 swap_xy=1 pressure_max=255 x_plate_ohms=60 x_min=250 x_max=3780 y_min=160 y_max=3930
    ```

    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    ads7846_device model=7846 cs=1 gpio_pendown=25 speed=2000000 keep_vref_on=1 swap_xy=1 pressure_max=255 x_plate_ohms=60 x_min=250 x_max=3780 y_min=160 y_max=3930
    ```

* Enable for Console:

    ```
    $ con2fbmap 1 1
    ```

    To make it permanent (on Debian) add to the file ```/boot/cmdline.txt``` at the end of the line the following kernel argument: ```fbcon=map:10 fbcon=font:VGA8x8```

* Enable for X-Window-System:

    ```
    $ FRAMEBUFFER=/dev/fb1 startx & 
    ```

    *...wait till X-Window-System starts up...*

    ```
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axis Inversion' 1 0
    ```

    *...to stop X-Window-System run:*

    ```
    $ sudo pkill x
    ```

    To make it permanent (on Debian) see [autostart x](https://github.com/notro/fbtft/wiki#wiki-make-it-permanent-debian) and [xinput axis inversion](https://github.com/notro/fbtft/wiki/Touchpanel#wiki--xinput---make-it-permanent).

* Video Test:

    ```
    $ sudo apt-get install mplayer
    $ wget http://download.blender.org/peach/bigbuckbunny_movies/BigBuckBunny_320x180.mp4
    $ mplayer -vo fbdev2:/dev/fb1 -vf scale=320:-3 BigBuckBunny_320x180.mp4
    ```

    *Note: The video file is about 60MB big.*


## Display Connection

```
Display  Raspberry Pi
---------------------
LCD-LED  GPIO18
LCD-RST  GPIO23
LCD-CS   GPIO8  (CE0)
ADS-CS   GPIO7  (CE1)
ADS-IRQ  GPIO25
MISO     GPIO9  (MISO)
MOSI     GPIO10 (MOSI)
SCK      GPIO11 (SCK)
```


## Optional Switch or LDR (Light-Dependent-Resistor)

There are pads for an optional tactile switch or LDR sensor on the PCB with a connection to **GPIO22** of the Raspberry Pi.
* Switch:
  * GPIO22 = 0 -> switch pressed
  * GPIO22 = 1 -> switch not pressed
* LDR (VT93N1):
  * GPIO22 = 0 -> bright light
  * GPIO22 = 1 -> low light

**How to read the pin state?**
```
$ sudo -i
$ echo 22 > /sys/class/gpio/export
$ echo in > /sys/class/gpio/gpio22/direction
$ exit
$ cat /sys/class/gpio/gpio22/value
```
