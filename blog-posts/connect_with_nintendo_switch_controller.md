# Instructions on how to connect the Nintendo Switch Pro controller
In this document, you can find, pair, connect and read the input event signals from the controller. This would be the first step to build a game-pad controlled robot arm.

## Connecting from bluetooth
We will need `bluez` to make a bluetooth connection to the device. 
```bash
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

Run
```bash
bluetoothctl
```
Once in the `bluetoothctl` suite, run following commands. They are self-explanatory
```
power on
scan on
```
A list of discoverable devices will appear with their corresponding MAC address. Look for the name of the controller, mine is "LIC PRO CONTROLLER". Say the MAC is `40:EC:99:A5:93:20`. We continue by
```
scan off
pair 40:EC:99:A5:93:20
trust 40:EC:99:A5:93:20
connect 40:EC:99:A5:93:20
```
Make sure the connection is approved.


## Kernel module `hid-nintento` and daemon `joycond`
If you didn't install `dkms` and `linux-headers` then go ahead and install these packages first:
```bash
sudo pacman -S dkms linux-lts-headers
```
I used the `linux-lts` so I installed the `linux-lts-headers`. If you are not using the LTS version, install the non-LTS kernel headers.

Let's start with installing the kernel module. The installation instruction is right on the module's [github](https://github.com/nicman23/dkms-hid-nintendo). You might need to change it a bit.
```bash
git clone https://github.com/nicman23/dkms-hid-nintendo
cd dkms-hid-nintendo
sudo dkms add .
```
After the last command, the text output will let you know the version of the `hid-nintendo` being installed. At the moment of this writing, I have `nintendo-3.1` version installed. This version number goes into the next command.
```bash
sudo dkms build nintendo -v 3.1
sudo dkms install nintendo -v 3.1
```

For `joycond`, follow the instruction [here](https://github.com/DanielOgorchock/joycond). We need `libevdev`. I have installed `libinput` which depends on `libevdev`. If these packages are not on your system yet, install them. Otherwise, go ahead with `joycond`:
```bash
sudo pacman -S libevdev
git clone git@github.com:DanielOgorchock/joycond.git
cd joycond
sudo make install
sudo systemctl enable --now joycond
sudo systemctl start --now joycond
```

## Quick check using [Gamepad\_Tester](https://gamepad-tester.com/)
Just go to [`https://gamepad-tester.com/`](https://gamepad-tester.com/) while the controller is connected. Trying out the buttons on the controller and see if the website can register the controller's signals.

## Test communication with `evdev`
I followed the instructions in [here](https://core-electronics.com.au/tutorials/using-usb-and-bluetooth-controllers-with-python.html). You should read the article carefully, it's full with useful information. I listed the commands I ran here.

Verify controller connection. The devices and the Linux system communicate by reading/writing the device files in `/dev`. By `cat`ing the correct file, which is of the formal `/dev/input/eventXXX`, we can confirm that we receive the game controller's signals. If you only connect the controller to the computer recently, then the correct device file will be the one with maximum `XXX`. Mine is `30`, yours will be different.
```bash
cat /dev/input/event30
```
Now, if you tap buttons on your controller, you will see a bunch of nonsense characters in the terminal. If that is the case, you confirmed the communication between the controller and the computer.

## Decode the signals from the controller
Install the python package `evdev`
```bash
pip install evdev
```

The package `evdev` comes with a script named `evtest.py` that allows the decode of various general input devices. Now, unlike `cat`, the script will spit out the codes of the buttons pressed and values of the analog joysticks. 
```bash
python <PYTHONSITEPATH>/evdev/evtest.py

ID  Device               Name                                Phys                                Uniq
------------------------------------------------------------------------------------------------------------------
0   /dev/input/event30   Lic Pro Controller                  40:ec:99:a5:93:20                   30:31:7d:2f:1d:5b

Select devices [0-0]: 0
Listening for events (press ctrl-c to exit) ...
time 1612684892.627957 type 3 (EV_ABS), code 16   (ABS_HAT0X), value -1
time 1612684892.627957 --------- SYN_REPORT --------
time 1612684892.777951 type 3 (EV_ABS), code 16   (ABS_HAT0X), value 0
time 1612684892.777951 --------- SYN_REPORT --------
time 1612684893.317963 type 3 (EV_ABS), code 16   (ABS_HAT0X), value 1
time 1612684893.317963 --------- SYN_REPORT --------
time 1612684893.422946 type 3 (EV_ABS), code 16   (ABS_HAT0X), value 0
time 1612684893.422946 --------- SYN_REPORT --------
time 1612684893.992971 type 3 (EV_ABS), code 1    (ABS_Y), value 37663
```

The `PYTHONSITEPATH` can be found in the list of `sys.path` in
```bash
python -m site
```

Save the battery of the controller by disconnect its bluetooth.
```bash
bluetoothctl
disconnect 
```

