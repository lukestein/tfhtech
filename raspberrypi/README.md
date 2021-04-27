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
6. Open the Terminal application from the menubar. Run `sudo raspi-config`. Set the following:
  - System Options S2 (Audio): **HDMI 1**
  - System Options S4 (Hostname): As desired
  - Advanced Options A1 (Expand Filesystem): Select
  - Advanced Options A2 (GL Driver): **G2 GL**
  - Advanced Options A3 (Compositor): **No**


## Dicaffeine (NDI receiver)

[Homepage](https://dicaffeine.com/about)


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
