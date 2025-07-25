### intune---kali-linux-guide


### Install Microsoft Intune app for Ubuntu Desktop

Run the following commands in a command line to manually install the Microsoft Intune app and its dependencies on your device.

Install Curl.
 

    sudo apt install curl gpg

Add the Repository Key: 
Microsoft signs its packages. You need to download and add their GPG key to verify the packages.

    curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor > /tmp/microsoft.gpg
    sudo install -o root -g root -m 644 /tmp/microsoft.gpg /etc/apt/trusted.gpg.d/
    sudo rm /tmp/microsoft.gpg

Add the Repository: 
Create a new list file for the repository. Although it's for Ubuntu 20.04, you can add it. Be aware of potential dependency issues.

    echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/20.04/prod focal main" | sudo tee /etc/apt/sources.list.d/microsoft-prod.list

Add and update Microsoft Linux Repository to the system repository list.

    sudo apt update

add debian repos for dependancies to /etc/apt/sources.list

use nala to install or fix broken install

    sudo nano /etc/apt/sources.list

    deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

    deb http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware

    sudo apt update


fix intune webview manually

    sudo apt install libgtk-3-0t64 libwebkit2gtk-4.0-dev



Install the Intune app.

    sudo apt install intune-portal


Run with Interactive Mode: Even if dependencies aren't perfectly resolved, trying to run the portal interactively might bypass some background service issues:

    intune-portal --interactive

Reboot your device.

Install the missing Qt utilities used by xdg-mime:

    sudo apt install qt5-qmake qtbase5-dev qttools5-dev-tools

Then try again:

    xdg-mime default intune-portal.desktop x-scheme-handler/oneauth
