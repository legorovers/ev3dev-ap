# ev3dev Access Point

This repo contains instructions and an Ansible script to set up an ev3dev image
that will make a Lego Mindstorms EV3 robot and a Netgear WBNA1100 WiFi dongle
act as a standalone access point that a tablet or laptop can connect to.

This is useful for demos or classroom activities where the overhead of having
to connect to a local WiFi network is too much.  Obviously, neither the  EV3 or
the tablet/latop can connect to the internet when this approach is used --
although the EV3 can make use of the micro-USB cable CDC connection to do this
when necessary.

Firstly here are some manual steps to create the base SD card image and to set
up the CDC network connection.  The rest is automatically provisioned using an
Ansible script.  Please note that since Ansible does not officially support
running from Windows, that these are focused on running from Mac OS X.


## SD Card Setup

You will need a suitable Micro SD card.  The EV3 supports SDHC version 2.0 (max
32 GB).  SDHC v2 means that it can manage transfer rates of up to 25MB/s which
is essentially Class 10.  8GB is more than enough space.

The manual steps are largely based on following the Mac version of the
ev3dev.org Getting Started instructions from http://www.ev3dev.org/docs/getting-started/ .

   1. Download the image file
      https://github.com/ev3dev/ev3dev/releases/download/ev3dev-jessie-2015-12-30/ev3-ev3dev-jessie-2015-12-30.img.zip

   2. Copy to the SD card (on a Mac) following http://www.ev3dev.org/docs/tutorials/writing-sd-card-image-mac-command-line/

        diskutil list

        # plug in card

        diskutil list

        # carefully check which disk number corresponds to the card
        # TRIPLE CHECK that the disk number is correct before running this!
        # if you have an external drive, for example, it may be disk2!!!
        sudo diskutil eraseDisk FAT32 EV3_BOOT MBRFormat /dev/disk1

        # carefully check which disk number corresponds to the card and unmount
        # it using commands something like the following
        diskutil unmountDisk /dev/disk1

        # TRIPLE CHECK that the disk number is correct before running this!
        # if you have an external drive, for example, it may be rdisk2!!!
        sudo dd if=~/Downloads/ev3-ev3dev-jessie-2015-12-30.img of=/dev/rdisk1 bs=4m

   3. Eject card, insert into EV3 and power on using centre button


## Connect the EV3

Set up networking over the micro-USB cable, following
http://www.ev3dev.org/docs/tutorials/connecting-to-the-internet-via-usb/?tabs-0=mac-os-x

   4. Wait for lights to go green and plug the micro-USB cable into the EV3 and
   the USB into your Mac and go to the Network in Preferences

        (+) Add the EV3+ev3dev Interface and give it the Service Name ev3dev
        [OK]

   5. Now enable Internet Sharing for the EV3+ev3dev interface and [Start] as
   described in step (4) of the ev3dev Mac instructions

      You may not have to do this every time, as your Mac will remember if you
      use the same USB port.

   6. Now configure the EV3's connection manager

        Wireless and Networks -> All Network Connections -> Wired
        -> Connect
        -> Connect automatically [*]

   7. If the robot does not show its State as Online, then you may need to
   reboot your Mac.  It appears that internet sharing in OS X can be very
   flakey and sometimes bootp or the DNS service don't come up or are not
   accessible (see http://apple.stackexchange.com/a/126229 for more info).

      Later versions of Mac OS X may also have trouble here.  See
      https://github.com/ev3dev/ev3dev/issues/471 for workarounds.

   8. Check the IP address shown in the top-left of the brickman UI and then,
   in a Terminal window on the Mac

        ssh robot@ev3dev.local
        password: maker
        sudo reboot

      (Press `<ENTER>`, `~` then `.` to close the ssh session.)

      You need to do this to add the remote host's key to your `known_hosts` file
      and for the system to spot that the logical volume has been extended.

Note that if you see `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`
it's probably because you've previously ssh'd into a different SD card image
at the same IP address.  Delete the line from your `~/.ssh/known_hosts` file
as it suggests.


## Install Ansible

Install ansible on your desktop machine using http://docs.ansible.com/intro_installation.html#latest-releases-via-pip

   1. Install/Update Pip (Globally)

        curl -OL https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py
        sudo python ez_setup.py

        curl -OL https://raw.github.com/pypa/pip/master/contrib/get-pip.py
        sudo python get-pip.py

   2. Install Ansible

        sudo pip install ansible

   3. Install sshpass

        curl http://downloads.sourceforge.net/project/sshpass/sshpass/1.05/sshpass-1.05.tar.gz -Lo sshpass.tar.gz
        tar xzf sshpass.tar.gz && cd sshpass-1.05
        ./configure && make
        sudo make install
        cd .. && rm -fr sshpass*

These are one off steps, and if you have done them before you won't need to
repeat them.


## Setup Access Point

Configure and run the ansible script to set up your ev3 as an access point.

   1. Edit the SSID and password vars in the `ev3dev.yml` file

   2. Run

        ansible-playbook -i hosts ev3dev.yml


Note that if you see `SSH encountered an unknown error during the connection.`
it's probably because you've previously ssh'd into a different SD card image
at the same IP address.  Run with `-vvvv`, as it suggests, to check and then
delete that line from your `~/.ssh/known_hosts` file.

If you need to modify the wifi SSID, password, country or channel after install
then edit the `hostapd` configuration and restart it

    vi /etc/hostapd/hostapd.conf
    service hostapd restart


## Wireless Adaptors


sudo vi /etc/udev/rules.d/70-persistent-net.rules
wlan1 wlan0
