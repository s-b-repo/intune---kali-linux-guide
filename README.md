### intune---kali-linux-guide


### Install Microsoft Intune app for Ubuntu Desktop

Run the following commands in a command line to manually install the Microsoft Intune app and its dependencies on your device.

Install Curl.
 

    sudo apt install curl gpg

Install the Microsoft package signing key.


    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg

    sudo install -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/
    
    rm microsoft.gpg

Add and update Microsoft Linux Repository to the system repository list.
Bash

    sudo apt update

Install the Intune app.

    sudo apt install intune-portal

Reboot your device.

Install the missing Qt utilities used by xdg-mime:

    sudo apt install qt5-qmake qtbase5-dev qttools5-dev-tools

Then try again:

    xdg-mime default intune-portal.desktop x-scheme-handler/oneauth
