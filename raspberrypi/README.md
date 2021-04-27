# Raspberry Pi Video Swiss Army Knife

## Basic setup

1. Use Raspberry Pi Imager to install Raspberry Pi OS (default) on a MicroSD card
2. Insert the MicroSD card in to the Pi, and attach a keyboard/mouse (via USB) and a display (via HDMI using the Micro HDMI port next to the Micro USB power port). Power up the Pi using a high-quality power supply to the Micro USB power port.
3. As prompted by the GUI, set
  - location settings
  - password
  - overscan
  - wifi (if desired; ideally ethernet will *also* be connected)
4. As prompted by the GUI, update software; this may take a while. Allow the requested restart, and the display should now fill the screen.
5. Set display settings: From the "Raspberry" menu in the upper-left, select "Preferences" and "Screen Configuration." Right-click on the HDMI-1 screen and select your preferred screen resolution and refresh rates. Click the green checkmark icon to save them, and if everything looks good, close the application.
6. Enable SSH access: From the "Raspberry" menu, select "Preferences" and "Raspberry Pi Configuration." Under "Interfaces," enable SSH and click OK.
7. Open the Terminal application from the menubar. Run `sudo raspi-config`. Set the following:
  - System Options S2 (Audio): **HDMI 1**
  - System Options S4 (Hostname): As desired
  - Advanced Options A1 (Expand Filesystem): Select
  - Advanced Options A2 (GL Driver): **G2 GL**
  - Advanced Options A3 (Compositor): **No**
8. Allow the Pi to reboot.


## Dicaffeine (NDI receiver)

[Homepage](https://dicaffeine.com/about)

In terminal, run
```bash
wget -O - http://dicaffeine.com/repository/dicaffeine.key | sudo apt-key add -
echo "deb https://dicaffeine.com/repository/ buster main non-free" | sudo tee -a /etc/apt/sources.list.d/dicaffeine.list
sudo apt update
sudo apt install -y dicaffeine
```

Ensure you know your local IP address (which you can check using `hostname -I`). Open dicaffeine at that IP, either using the Pi's web browser, or another computer on the local network that can access the Pi.

The default password for Dicaffeine is `admin`; you can change this using the "System" tab at the top of the browser window.

On the "Player" tab, choose an NDI source under Main stream, and select "Built-in Audio Digital Stereo" under Audio. "Low resolution" is suggested (or use the Advanced properties); Dicaffeine does not seem able to handle e.g. 1080p30. Click "Save" to save setting, and then "Play."

Exit the player by clicking "Stop" if controlling from a remote computer, or bu right-clicking the Pi's mouse.


## OBS Ninja

[Homepage](https://obs.ninja)


## aaronpkbridge (RTMP receiver)

[Homepage](https://aaronparecki.com/2020/09/07/7/raspberry-pi-streaming-server)


## VLC


## omxplayer


## clock-8001

[Homepage](https://gitlab.com/Depili/clock-8001)


## Bitfocus Companion control

[Homepage](https://bitfocus.io/companion)
