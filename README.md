

# HackberryPi5 - Kali on SSD 

This is the hardware I used:

<img src="https://m.media-amazon.com/images/I/61gL8zmcDIL._AC_SL1500_.jpg" width="200" />

[Waveshare PCIe to M.2 Adapter Board (E) For Raspberry Pi 5, With Cooling Fan, Compatible With 2242/2230 NVMe Protocol M.2 Solid State Drive, 8Gbps Data Rate, High-Speed Reading/Writing, Pi 5 NVMe](https://www.amazon.it/dp/B0DBZ6PWF6)

<img src="https://m.media-amazon.com/images/I/613+hkU87eL._AC_SL1500_.jpg" width="200" />

[Waveshare SK M2 NVME 2242 256GB Solid State Drive, 3D TLC Flash Memory, High-Speed Reading/Writing Compatible With Windows/Raspberry Pi OS/CentOS/Ubuntu/Linux](https://www.amazon.it/dp/B0D5CJPKN9)

<img src="https://m.media-amazon.com/images/I/71NHG5ox2cL._AC_SL1500_.jpg" width="200" />

[ELUTENG M.2 NVMe to USB Adapter, USB 3.1 Card 10Gbit/s, USB to M.2 NVMe SSD Card Reader USB SSD Adapter 10Gbps for M Key SSD Support M.2 2230 2242 2260 2280](https://www.amazon.it/dp/B0CB7MGWGQ)


## Summary
1. Check RPi5 EProm settings and change EProm to enable boot from SSD
2. Install NVMe Hat and SSD
3. Install Kali onto SSD and Configure it on the PC before installing in RPi5
4. Boot Kali and some next steps
	1. Connect to Wifi - Nothing to do here I just make sure I can connect to my network at home.
	2. Update and Upgrade Kali
	3. Autostart Bluetooth services
	4. Increase Swap size
5. Auto-connect Bluetooth Speakers
6. Add SDR
7. SatDump
8. SDR++


## 1. Check RPi5 EProm Settings
1. Using the Raspberry Pi Imager install Raspberry PI OS (64Bit) onto a 32Gb SD card.  
2. If you are using this on the HackberryPi5 then remember to add ONLY the following line to the /boot/config.txt file on your SD card.
```
dtoverlay=vc4-kms-dpi-hyperpixel4sq
```
3. Next follow the steps from [here](https://community.volumio.com/t/guide-prepare-raspberry-pi-for-boot-from-usb-nvme/65700). The next steps are a summary of what I did.

 - 3.1 Boot from the SD Card into Raspberry Pi OS
  
  - 3.2 Checked EEProm version `vcgencmd bootloader_version`

  - 3.3 Edit this file /etc/default/rpi-eeprom-update
    
```
sudo nano /etc/default/rpi-eeprom-update
```
	Changed 
        FIRMWARE_RELEASE_STATUS="default" 
	- to - 
		FIRMWARE_RELEASE_STATUS="latest" 

  - 3.4 Run `sudo rpi-eeprom-update -a`
    - Reboot

4. Check Boot order by running this `sudo rpi-eeprom-config`
	
- Mine already had a boot order of 
`
BOOT_ORDER=0xf461
`
	
5. Next I had to add "PCIE_PROBE=1" to the bottom of the file on screen.
```
sudo -E rpi-eeprom-config --edit
```
It should liike this:

<img src="https://community.volumio.com/uploads/default/original/3X/6/9/69b2c14bd2a62d777153b4c2e4e5f6a81fcab272.png" width="500">

6. Save your changes (Ctrl-O) and exit (Ctrl-X)
7. Reboot

## 2. Install Hat and SSD into HPi5
This is what mine looks like

<img src="https://github.com/lunved/HPi5/blob/main/images/NVMe%20hat%20and%20SSD.jpg" width="500">


## 3. Install Kali onto SSD	
1. Put SSD into NVMe to USB adapter and using Raspverry Pi Imager write [Kali](https://www.kali.org/get-kali/#kali-arm) to the device

2. When I completed writing and verifying the image I had to remove the USB device and reisert it.  After that I can edit the config.txt file

3. Remember to add the line to the `config.txt` file on your SD card.
```
dtoverlay=vc4-kms-dpi-hyperpixel4sq
```
4. Remove the other dtoverlay lines.  (If you do not remove the other lines the console will work but NOT the GUI) I just comment them out like bellow wherever I find them:
```
#dtoverlay=vc4-fkms-v3d
```

## 4. Boot Kali and next steps
Once I have booted into kali I do the following:
1. Connect to wifi - Nothing to do here I just make sure I can connect to my network at home.
2. Update and Upgrade
```
sudo apt update && sudo apt -y upgrade
```
If you get an error with something like Error "release file is not yet valid" you need to [set your timezone](https://linuxconfig.org/how-to-set-time-on-kali-linux) correctly.

3. I then make sure the bluetooth autostarts. Open a terminal and type the following `sudo systemctl enable bluetooth` and then `sudo systemctl start bluetooth`
4. Open Settings -> Bluetooth Manager and search for your Bluetooth speakers (The onboard ones)  Mine are `XWF-M18-M28-M38`. You can then "Pair", "Trust" and "Connect" the bluetooth speakers.  If this works you should see your speakers connect in the top right hand corner.

4. Increase Swapsize I could not find the `dphys-swapfile` command so I had to install it.  Instructions were found [here](https://kalitut.com/raspberry-pi-swapping/)
```
sudo apt-get install dphys-swapfile
```
Check the file /etc/dphys-swapfile and change the CONF_MAXSWAP to a size that suits you.  I made mine 4096.  If that file does not exist run the following:
```
sudo dphys-swapfile setup
```
after you have edited the file you can enable the service to have it autostart using this command:
```
sudo systemctl enable dphys-swapfile
```
and then start the service with this command 
```
sudo systemctl start dphys-swapfile
```

If you want to check if the swap file is running you can simply run:
```
swapon -s
```
You can also run `free -m` to see your devices memory and swap.



## 5. AutoConnect Bluetooth speakers
To do this you need a script that will run everytime the system boots.  This is what I did.

First I had to find out the MAC address of my speakers (headset).  To do this I simply clicked on the Bluetooth icon in the top right hand corner of the screen.  That opens up a Bluetooth Devices application and the only device I had there was something called `XWF-M18-M28-M38` - The headset.  Underneath that you will see the mac address displayed (E.g. 3x:xx:xx:xx:xx:x2).

1. I created a `/home/kali/startup.sh` file with the following contents:
```
sleep 30
echo "connect 3x:xx:xx:xx:xx:x2" | bluetoothctl 
```

2. I then made that script executable by running the following:
```
sudo chmod +x  ~/startup.sh
```

(**NOTE**: you can test that this works but rebooting and then from a terminal simply executing ~/startup.sh - So the bluetooth daemon starts automatically and then when I ran ~/startup.sh from the command line my speakers connect [Obviously after the 30 sec delay.] )

3. Make the ~/startup.sh script run at every reboot.  Simply add a crontab entry.  To edit the crontab run this (you may get asked which editor you prefer to use):
```
crontab -e
```
- add the following line to the bottom of crontab:
```
@reboot sh /home/kali/startup.sh 
```

Now reboot and wait - you can login but you do not have to.  You will hear the Bluetooth speakers connecting after 30 seconds.


## 6. Add SDR

These are the SDR dongles I have tested with and currently use.

<img src="https://github.com/lunved/HPi5/blob/main/images/SDR%20Dongles.jpg" width="500">

I found [this link - A newbie’s guide to Software Defined Radios on Kali Linux | Part 1: Fun with FM radios](https://medium.com/poka-techblog/a-newbies-guide-to-software-defined-radios-on-kali-linux-part-1-fun-with-fm-radios-3a3589b78608) really helpful

Everything up to the Tuning to an FM Station worked perfectly.

However when I tried to install `gqrx` I started getting errors.

I did the following to get it working.
```
sudo apt update
sudo apt dist-upgrade
```

I still had errors trying to install `gqrx`  and the errors has something to do with `xtrx-dkms`  I found something online saying that we may not even need that  - But I have not confirmed this to be 100% accurate yet.  What I did do though is remove both dkms and xtrx-dkms packages.
```
sudo apt purge xtrx-dkms
sudo apt purge dkms
```

Then when I ran 
```
udo apt install gqrx-sdr
```
the install completed successfully.

However when I ran `sudo gqrx` I started getting problems again.  Issues like PulesAudio: connection refused!

I checked that pulseaudio was installed and started as a daemon:
```
pulseaudio -D
```
If you are having issues with pulseaudio [this link helped  - Help Fixing pulseaudio install](https://forums.kali.org/archived/showthread.php?17921-Help-Fixing-pulseaudio-install)

I also found out that you **DON'T START GQRX AS SUDO**.  After that `gqrx` started perfectly and I was able to tune into a normal FM station.

**NOTE:** My` nooelec nesdr Smart` does not go below 24.000 kHz ... However I unplugged  it and inserted my RTL-SDR BLOG v4 into the usb and it worked without any problems ... I can even tune down to the Lower Ham bands and pick up 40m with the correct antenna.

Listening to 20m CW video coming soon.

## 7. SatDump compiling

**NOTE:** I have not tested SatDump yet as I still have to build the antenna.

But [These instructions - GitHub - SatDump/SatDump: A generic satellite data processing software.](https://github.com/SatDump/SatDump#linux) on how to include the dependencies and how to build the application worked flawlessly for me.


## 8. SDR++

<video src="https://github.com/lunved/HPi5/blob/main/images/SDR%20FM%20Stations.mp4" width="320" height="240" controls></video>

I manually went through [these instructions](https://www.instructables.com/Building-SDR-From-Source-Code-on-a-Raspberry-PI-4-/) with some subtle changes.

The changes I had to make were:
- Replaced libvolk2-dev with libvolk-dev when installing the dependencies
- ran the cmake from the SDRPlusPlus directory and no from inside build or CMakeFiles

```
sudo apt update

sudo apt install -y build-essential cmake git libfftw3-dev libglfw3-dev libglew-dev libvolk-dev libsoapysdr-dev libairspyhf-dev libairspy-dev libiio-dev libad9361-dev librtaudio-dev libhackrf-dev librtlsdr-dev libbladerf-dev liblimesuite-dev p7zip-full wget

git clone https://github.com/AlexandreRouma/SDRPlusPlus

cd SDRPlusPlus

sudo mkdir build
cd build
sudo mkdir CMakeFiles

cd ..

cmake -DOPT_BUILD_RTL_SDR_SOURCE=ON

make

sudo make install
```

And Voila!!! I now have SDR++ running on Kali onm a Hackberry Pi5.  To start it you simply have to run:
```
sdrpp
```

---

# TODO:

 1. Build a satellite receiving antenna from [here](https://www.a-centauri.com/articoli/noaa-poes-satellites-reception)
 2. Not sure now ... brush up on my CW?

