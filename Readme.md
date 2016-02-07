# Linux with Awesome WM on ASUS Zenbook UX305FA

## Introduction
This guide will focus on using Awesome WM as its window manager. I compiled many
guides from all over the Internet as well as some scripts I wrote to make this 
laptop work much better.

## Installation
Instalation with Ubuntu 15.10 works with UEFI enabled. I installed with full 
disk encyption with LVM with no problem. Just make sure you did a disk clone or 
at least backup windows product key.

## General

### Screen Brightness

Fn+f5 and Fn+f6 buttons are not recognized by `xev`. So we are going to use the 
empty Fn+f3 and Fn+f4 buttons to adjust screen brightness. First, we install 
xbacklight.

    sudo apt-get install xbacklight

Add the following code into rc.lua file. #69 and #70 is found through xev.

    awful.key({ }, "#70",      function () awful.util.spawn("xbacklight -inc 10") end),
    awful.key({ }, "#69",      function () awful.util.spawn("xbacklight -dec 10") end),

### Audio

There are two audio output, one is HDMI and another is Conexant. I find it 
easier to just blacklist HDMI and leave out Conexant which we are going to use.

    sudo touch /etc/modprobe.d/intel.conf
    sudo cat "options snd_hda_intel enable=0,1" >> /etc/modprobe.d/intel.conf

To use Fn+f10, Fn+f11 and Fn+12 buttons for mute, down volume and up volume
respectively, we are going to need `amixer` and `pactl`. Alsamixer is already
installed by default, so we need to install pulseaudio volume control.

    sudo apt-get install pavucontrol
    
The keycodes are found by `xev`. The code for rc.lua are the following:

    awful.key({ }, "#121",
        function ()
            awful.util.spawn("amixer -D pulse sset Master toggle")
        end),
    awful.key({ }, "#123",        
        function ()
	    awful.util.spawn("amixer -D pulse sset Master 10%+")
        end),
    awful.key({ }, "#122",
        function ()
	    awful.util.spawn("amixer -D pulse sset Master 10%-")
        end),

Another thing of note is the speaker volume can be very low. In this case, we
can use pavucontrol to increase the volume up to 150%. This should be used
sparingly as I noticed crackling when moving the volume more than 100%.

    awful.key({ altkey }, "Up",
        function ()
            awful.util.spawn("pactl set-sink-volume 0 +10%")
        end),
    awful.key({ altkey }, "Down",
        function ()
            awful.util.spawn("pactl set-sink-volume 0 -10%")
        end),

### Touchpad
As an Awesome WM user, touchpad is rarely used. But when we do, we want three 
finger tap to simulate middle button click and palm rejection when the touchad
is switched on. Put the following into rc.lua 
file.

    run_once("synclient TapButton3=2 FingerHigh=10 FingerLow=5")

Set up palm rejection with `synclient` as well:

    run_once("synclient AreaLeftEdge=500")
    run_once("synclient AreaRightEdge=2500")

To use Fn+f9 button to toggle the touchpad. We need to use `xinput`. 

    xinput enable 12   # Enable touchpad
    xinput disable 12  # Disable touchpad
    
Unfortunately, there is no toggle command that I know of. So to use Fn+f9 button
to toggle, use the following bash script.

```bash
#!/bin/bash
FILE=touchpad_enabled

if [ -f $FILE ]
then
    xinput disable 12
    rm $FILE
else
    xinput enable 12
    touch $FILE
fi
```

Id number 12 is found by

    xinput list | grep 'Touchpad'
    
To use the bash script inside rc.lua, we need to use os.execute instead.

```
awful.key({ }, "#199",
    function () os.execute("sh ~/<PATH_TO_SCRIPT>/toggle_touchpad.sh")
end),
```


### Suspend/Resume
Suspend and resume seems to work out of box. Closing the lid suspends the 
laptop. You can manually suspend and hibernate through terminal.

    systemctl suspend
    sudo systemctl hibernate

### Firefox

To make webpages render at an appropriate size, navigate to about:config
and set layout.css.devPixelsPerPx to 1.25. This increases the size by 25%.


## Credits

 - https://wiki.archlinux.org/index.php/ASUS_Zenbook_UX305
 - https://github.com/thezerobit/asus-zenbook-ux305fa/blob/master/Readme.md
 - https://www.reddit.com/r/linux/comments/3ia8ta/review_of_ubuntu_on_asus_ux305fa/
