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

## Stream receivers

### Aaron Parecki's PiBridge (RTMP receiver)

[Homepage](https://aaronparecki.com/2020/09/07/7/raspberry-pi-streaming-server)

In terminal, run
```bash
sudo apt update
sudo apt install omxplayer nginx libnginx-mod-rtmp
sudo usermod -aG video www-data
```
Note that nginx will fail to set up if it cannot control port 80 for its webserver, although once set up, we can disable nginx's web server on port 80 so that other apps (e.g, Dicaffeine) can use port 80 instead.

To create an RTMP server in nginx, edit the main nginx config file `sudo nano /etc/nginx/nginx.conf` adding at the bottom (and savings your changes):
```
rtmp {
  server {
    listen 1935;

    application live {
      # Enable livestreaming
      live on;

      # Disable recording
      record off;

      # Allow only this machine to play back the stream
      allow play 127.0.0.1;
      deny play all;

      # Start omxplayer and play the stream out over HDMI
      exec omxplayer -o hdmi rtmp://127.0.0.1:1935/live/$name;
    }
  }
}
```

Test the config using `sudo nginx -t` and restart nginx using `sudo nginx -s reload`

Ensure you know your local IP address (which you can check using `hostname -I`). You should then be able to use the RTMP URL `rtmp://YOUR_IP_ADDRESS/live` with any stream key.

Note that the cursor will show on top of streamed video. (This is not true if you are using a terminal-only Pi as suggested in Aaron Parecki's original instructions.) We can hide the cursor when it's not moving using the `unclutter` package
```bash
sudo apt install unclutter
sudo service lightdm restart
```

Change nginx's default webserver from port 80 to another port (e.g., 8080) using `sudo nano /etc/nginx/sites-available/default` and changing the two "listen" lines under `server {` from "80" to "8080". Test the config using `sudo nginx -t` and restart nginx using `sudo nginx -s reload`



### Dicaffeine (NDI receiver)

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

If you choose "Autorun after start" (and then save), the Pi will boot directly into a running Dicaffeine player rather than the desktop environment.


### OBS Ninja

[Homepage](https://obs.ninja)


## Video players

### VLC


### omxplayer


## Other utilities

### clock-8001

[Homepage](https://gitlab.com/Depili/clock-8001)


## Bitfocus Companion control

[Homepage](https://bitfocus.io/companion)
