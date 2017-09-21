# Kioskify

Kioskify is a tool for configuring a Raspberry Pi as a stand-alone web kiosk. It aims to make it as easy as possible to set up your RPi to boot into a full screen web browser pointing to the URL of your choice.

## Using Kioskify

Kioskify is designed to be used with the *Jessie* or *Stretch* versions of the Raspbian operating system for the Raspberry Pi, so the first thing that you will need to do is get an SD Card and load Raspbian onto it. While the tool can in most cases be used on an existing installations it is recommended that you start with a clean install since you are far less likely to run into trouble that way. Instructions for downloading Raspbian and loading it onto an SD Card can be found at the [Raspberry Pi web site](https://www.raspberrypi.org/documentation/installation/installing-images/).

## Preparing your Raspberry Pi

If you have an existing, configured and networked RPi which you intend to use as a kiosk then for this stage all you need to do is copy the kioskify script onto that device and ensure that it is readable and executable by the **pi** user.

If you are starting with a fresh install then the easiest way to get Kioskify onto your Pi is to mount the SC Card on your laptop or desktop computer and copy the kioskify script onto the /boot volume.

If you are planning to set up the device over the network (which is often the easiest way) then at this stage you should also `touch` the `/boot/ssh` file on the SD card (the path to this file on your laptop will depend on where you mounted the SD card; on macOS for instance it will probably be `/Volumes/boot/ssh`).

If you plan to have your Raspberry Pi connect to your WiFi network then you can pre-configure the network at this stage by putting the network configuration into `/boot/wpa_supplicant.conf`. At the very minimum you need to include the **network** stanza containing the *ssid*. Hopefully you are are also using some sort of security on you WiFi network and in most cases this means that you'll also need to include the *psk* (pre-shared key) line with your network password. If you expect to ever use the regular desktop environment for the RPi (i.e. log in as a regualar user rather than run in kiosk mode) then you might also want to include the *ctrl_interface* and *update_config* lines which will allow the desktop WiFi tools to find new networks, prompt the user for passwords and generally reconfigure the WiFi. If the device is only ever going to be used as a kiosk device then you can leave these lines out.

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
        ssid="MyNetworkName"
        psk="MyNetworkPassword"
}```



## Running the Kioskify script

Once you have prepared your Raspberry Pi boot the device and log in to a command line as the **pi** user. If this is a newly set up device the password should be **raspberry**. Once logged in as **pi** run the kioskify script. If you copied it to the SD card as described in the previous section then all you need to do is type `/boot/kioskify` and hit return.

The kioskify script will ask you a series of fairly self-explanatory questions. If you want to you can skip the next few sections and just try it but in case you want more details below is some more information about each.

### Setting the hostname

By default every new Raspberry Pi has its hostname set to `raspberrypi`. Since the device will also by default advertise its name over Mutlicast DNS (mDNS) its easy to end up with lots of devices with the same hostname. In this case the mDNS system will add a number to the end of the hostname so that each device has a different name but that just means that you won't know the name of the device you are after. A much better option is to give each device a different hostname. Try using a name that describes either the function or location of the kiosk.

> Internally the new hostname is set using the `raspi-config` command. If you wish to change the hostname again at a later date you can either re-run the `kisokify` script, use the `raspi-config` command or manually edit the file `/etc/hostname` and reboot.

### Setting a password

By default a new Raspberry Pi has a user called **pi** with a password of **raspberry**. You really, really, really should change this. Really. If you have already changed the password to the **pi** user then you can just hit return to leave the password unchanged. If you do this and haven't changed the password then the script will ask you to confirm that you really don't want to set a better password.

The script will also do some cursory checks to see if the password has a moderate amount of complexity. Complexity is a combination of both having a mixture of cases, digits and punctuation and also having a decent length. You can have an all-lower-case password if you want but in that case you should make it rather longer. The script will prevent you from using dreadfully short or simple passwords but it's not going to save you from your own stupidity. Whole theses have been written on what makes a good password but suffice it to say that the longer and more varied the better.

> If you wish to change the password at a later time you may use the `passwd` command to do so.

### Setting an SSH key

Even if you set a good password it's still a bad form of authentication. A much better way to authenticate is using something called *public key cryptography*. The SSH (Secure SHell) program allows you to log in to your RPi over an encrypted connection using this sort of more secure authentication and this is **highly recommended**.

If you don't already have an SSH public key then you can easily create one on either Linux or macOS machines by opening up a command line terminal and running the `ssh-keygen` command. Typically your public keys will be stored in the `.ssh` subdirectory of your home directory in files called `id_rsa.pub`, `id_dsa.pub`, `id_ecdsa.pub` or similar, depending on the key type. A discussion of the differences, pros and cons of the different key types is beyond the scope of this document but it is sufficient to copy the contents of any one of these public key files in response to this question.

***NOTE:*** *It is critical that you provide your **public** key, not your **private** key. The key files that you want have the extension `.pub` on the end. The contents of the private key files need to be kept **secret** otherwise bad people will do bad things with your identity.*

If you provide the script with your public key then it will also ask you if you would like to disable password-based remote login to your Raspberry Pi. This is also **highly recommended**. Note that if you choose to leave the default password unchanged and you provide an SSH key then password-based SSH login will automatically be disabled.

> The SSH key is stored in the file `/home/pi/.ssh/authorized_keys`. You may add more keys or remove the provided key by manually editing this file. Note that if you have disabled password-based SSH login and you remove all the authorized keys then you may find yourself a little stuck, so be careful!

> Internally password-based authentication is disabled by editing the file `/etc/ssh/sshd_config` to include a line saying `PasswordAuthentication no`. You may manually edit this file to say `yes` if you need to reenable password authentication, although this is not recommended.

### Screen orientation

When setting up a kiosk display it is sometimes useful to have the screen in a different orientation to the default. Examples of this are having the screen in "portrait" mode (turned so that it is taller than it is wide) or reflected (for instance if you are back-projecting the screen). It is also sometimes useful to mount the display upside down (i.e. turned through 180 degrees).

Kioskify will ask if you want the screen in a non-standard orientation and if so it will ask how you want the display to be rotated or reflected.

> Internally the screen oritentation is set by editing the file `/boot/config.txt` to include a line saying `display_rotate=<value>`. Removing this line will return the display to its default orientation.

### Setting the URL of the page to be displayed

Finally, Kioskify will ask for the URL of the page to be displayed by the browser when it boots. If you do not provide a *scheme* then the script will assume that it is HTTP.

> The URL is stored in the file `/home/pi/KIOSK_URL`. You may edit this file at any time to change which page will be displayed. You will need to reboot the machine in order to have this change take effect.

## Internal scripts

In order to have the Raspberry Pi boot to a full screen web browser a number of configuration files are created or modified. Kioskify also installs a utility script to make it easier to reload the browser page. The following files are created or modified:

* /home/pi/.ssh/authorized_keys
* /home/pi/.config/lxsession/LXDE-pi/autostart
* /home/pi/reload-slideshow.sh
* /home/pi/start-slideshow.sh
* /home/pi/KIOSK_URL
* /etc/ssh/sshd_config
* /etc/lightdm/lightdm.conf
* /boot/config.txt

For system configuration files Kioskify will make a copy of the original file, in the same directory as the original, with the time that the script was run appended to the file name, so that if you want to revert to the old version you can simply replace the edited version with the original.
