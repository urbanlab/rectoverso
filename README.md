# Recto-Verso

- [Presentation](#presentation)
- [Installation](#installation)
  * [Requirements](#requirements)
  * [Main server](#main-server)
    1. [Server requirements](#server-requirements)
    1. [Installing the server app](#installing-the-server-app)
    1. [Update the code server side](#update-the-code-server-side)
  * [Secondary devices](#secondary-devices)
    1. [System installation](#system-installation)
    1. [Autostart rectoverso](#autostart-rectoverso)
    1. [Rotate the screen](#rotate-the-screen)
    1. [Rotate the touchscreen](#rotate-the-touchscreen)
    1. [Automatic system updates](#automatic-system-updates)
- [Usage](#usage)
  * [Application configuration](#application-configuration)
    1. [Languages](#languages)
       - [Change application language](#change-application-language)
       - [Add a language](#add-a-language)
    1. [Toggle verbosity](#toggle-verbosity)
    1. [Choosing a different set of pictures](#choosing-a-different-set-of-pictures)
    1. [Create a new dataset](#create-a-new-dataset)
- [Frequent problems](#frequent-problems)
  * [My raspberry won't start](#my-raspberry-wont-start)
- [Get the old version of the game (using node.js)](#get-the-old-version-of-the-game-using-nodejs)


## Presentation

Recto-Verso is a prototype consisting mainly of two big devices. You can see here one of them :

![rectoverso_borne.jpg](https://photos.erasme.org/images/2020/10/02/rectoverso_borne.jpg)

Each of these two devices has a touchscreen :

![rectoverso_affichage_principal.jpg](https://photos.erasme.org/images/2020/10/02/rectoverso_affichage_principal.jpg)

Using the touchscreen, a user can play a card game named _Concentration_
([english desciption](https://en.wikipedia.org/wiki/Concentration_(card_game)),
[french description](https://fr.wikipedia.org/wiki/Memory_(jeu))).

Both players start simultaneously, and the first who finds all pairs of cards wins.


## Installation

### Requirements

Technically, you can have as many players as you want but this would require some minor tweaks in the code...
and each player needs a touchscreen. So two players are perfectly fine.
 
This implies at least one computer with two touchscreens. Each screen would display a certain browser page
generated by the server on the computer.

***Remark :*** One computer and two screens are the minimal requirements but in the case of our devices
we put a computer with its touchscreen in both machines.


### Main server

#### Server requirements

You need a server which have apache2, PHP (5 or 7), sqlite and all plugins connecting them each other. This
changes greatly depending on your operating system. You might want *git* too to update safely the application. 


#### Installing the server app

Go to your server file system (most of the time something like `/var/www/html`) and clone the repository.

```sh
# Browse until you're in the correct directory.
cd /path/to/your/www/

# And clone the repository there using git.
git clone https://github.com/urbanlab/rectoverso.git
```

And remember to make sure your server has the rights to read and write the data folder.


#### Update the code server side

If you want to apply some changes on the server, just update the depot using *git*.

```sh
# Browse until you're in the correct directory.
cd /path/to/your/www/rectoverso

# And clone the repository there using git.
git pull
```

### Secondary devices

#### System installation

We are going to install two ***raspberry pi*** in each playable device. They only need to display a web page.

For instance, we choose to use ***raspbian*** as our OS. We must proceed this way :   

1. [download](https://www.raspberrypi.org/downloads/raspberry-pi-os/) the last version of ***raspbian OS***.You can
choose whatever version of raspbian you want. The *Lite* may be a bit... too light and require additional actions while
the desktop version has all you need (although do not take the one with all recommended software).
1. Create a disk image on the SD Card using a common software like *gnome-disk-utility* (on Ubuntu, type
`sudo apt install gnome-disk-utility` to install it and `gnome-disks` to start it)
- Turn on your raspberry (while its SD card is in) and follow the instructions (choose the language, keyboard, 
Wi-Fi network, install the last updates...)

#### Autostart rectoverso

Create a file somewhere and give it an evident name like `rectoverso_autostart.sh` and place it in an obvious and
easily accessible emplacement like the desktop. Now, write this inside (*remember to modify the url !*).

```sh
#!/usr/bin/env bash
sleep 10 # wait few seconds for network connection.
DISPLAY=:0 chromium-browser --kiosk url/to/access/rectoverso/play.php
```

Once the file is written, give it execution rights, then create a cronjob :

```sh
crontab -e
```

The precedent command will open a file where you have to add an extra line to its end :

```log
@reboot /path/to/your/rectoverso_autostart.sh > /home/pi/Desktop/cron_logs.txt  2>&1
``` 

That will launch the command at each startup of the *pi*.


####  Rotate the screen

The way to solve this problem changes depending on which *raspberry* you have.

***Raspberry 3***


```bash
# Open the configuration file in a text editor.
sudo nano /boot/config.txt
```


```yaml
# Check if the option display_hdmi_rotate exists.
# - if yes, change it to 3 (or to whatever rotation you need)
# - if no, just copy this line in the text.
# The number increments for each rotation of 90° you do to the screen counter-clockwise (or the trigonometric way).

# Here a rotation of 90° clockwise
display_hdmi_rotate=3
```

Once you added/modified the option in the config file, reboot tour system.

```sh
sudo reboot
```

#### Rotate the touchscreen

```bash
# Install the calibrator 
sudo apt install xinput-calibrator

# Open the file to rotate the touchscreen
sudo nano /usr/share/X11/xorg.conf.d/40-libinput.conf
```

Look for the block of text concerning your touchscreen and add this line :

```text
Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "TransformationMatrix" "0 -1 1 1 0 0 0 0 1"
```

Adding the option `TransformationMatrix` rotates your touchscreen 90° clockwise.


#### Automatic system updates

Create a file somewhere like `update_system.sh` on your desktop and make it executable.

```bash
#!/usr/bin/env/ bash
sudo apt update && sudo apt upgrade -y
``` 

Add a crontab to execute it at each startup.

## Frequent problems

### My raspberry won't start

The *pi* might need to be connected to a screen. Connect one to your *pi* and turn it on. 

### Force restart rectoverso using touchscreen

You have two different ways to force a refresh of the page. Click 5 times in a row on either :
- the block containing the scores (during a game)  
- the block containing the text in a modalbox (at startup, the texte saying *waiting for other players*)

## Usage

### Application configuration

#### Languages

##### Change application language

Two languages are currently available : french and english. You can modify the application language by connecting to the
server and modify the `configuration.json` file in the project root.

Here is an example of the file :

```json
{"chosen_dataset_sub-folder":"images_rectoverso","dataset_directory":"images_sets","language":"fr","number_of_players":"2","verbose_mode":"false"}
```

Modify the value of the language field to `"en"` or `"fr"` as you prefer.

##### Add a language

All translations are available in a file named `translations.json` on the server. Add your translation(s) to this file,
you can use them later in the `configuration.json`.

#### Toggle verbosity

You can force the application client-side to be way more verbose by modifying the verbosity mode in the
`configuration.json` on the server.

#### Choosing a different set of pictures

Once again, the solution is in the `configuration.json` file. Especially two fields :
- *chosen_dataset_sub-folder*
- *dataset_directory*

`dataset_directory` s the folder containing all your datasets (an absolute path if you can but a relative one might
work) whereas chosen_dataset_sub-folder` is the folder name of your dataset.

 
#### Create a new dataset

A *rectoverso* dataset is a collection of pictures placed as pairs in sub-folders. ***You need at least 9 pairs.***

Imagine we would like to create a sub-folder of pictures of european capital cities. YOu have to respect this tree
structure.

- images_capitals
  * Paris
    - 0.jpg
    - 1.jpg
  * Bruxelles
    - 0.jpg
    - 1.jpg
  * Dublin
    - 0.jpg
    - 1.jpg
  * ...
  * ...
  * ...
    

***N.B. :*** All common picture formats and extensions are accepted (*.jpeg, *.png, *.gif, etc...).

***N.B. :*** You can use more than 9 pairs. If you do so, the server will randomly pick 9 of them, and all players
will play with the same cards.


## Get the old version of the game (using node.js)

The precedent version of the application was written with node.js but we couldn't manage to make it work
again. If you want to access this code, [check the saved v1-branch](https://github.com/urbanlab/rectoverso/tree/sauvegarde-V1).