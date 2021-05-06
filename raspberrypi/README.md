# Raspberry Pi Video Swiss Army Knife

My goal was to set up a single Raspberry Pi 4 to receive/play video out to HDMI from a variety of sources. I'm drawing on a number of online tutorials, and even what works now may not continue to work in the future.

## Basic setup

1. Use Raspberry Pi Imager to install Raspberry Pi OS (default) on a MicroSD card.
2. Insert the MicroSD card into the Pi, and attach a keyboard/mouse (via USB) and a display (using the Micro HDMI port closer to the Micro USB power port). Power up the Pi using a high-quality power supply to the Micro USB power port.
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
  - Display Options D1 (Resolution): As desired (I get decent performance with 1920x1080 30Hz)
  - Display Options D4 (Screen Blanking): **No**
  - Advanced Options A1 (Expand Filesystem): Select
  - Advanced Options A2 (GL Driver): **G2 GL**
  - Advanced Options A3 (Compositor): **No**
8. Allow the Pi to reboot.


### Getting your IP address

Ensure you know your Pi's local IP address (e.g., 192.168.0.140) which you can check using `hostname -I`, or [various other ways](https://www.raspberrypi.org/documentation/remote-access/ip-address.md). You probably want to set up a static IP address using your router settings.

Especially if you will be regularly accessing the Raspberry Pi remotely via ssh, you may want to set up [passwordless SSH access](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md) on the Pi, and/or a [`~/.ssh/config` file](https://linuxize.com/post/using-the-ssh-config-file/) on the computer you will be using to access it.


### Optional additional setup

1. If you would like the default pi user account to always require a password to `sudo` commands, edit `/etc/sudoers.d/010_pi-nopasswd` and change `NOPASSWD` to `PASSWD`.

## Stream receivers

### Aaron Parecki's PiBridge (RTMP receiver)

[Homepage](https://aaronparecki.com/2020/09/07/7/raspberry-pi-streaming-server)

In terminal, run:
```bash
sudo apt update
sudo apt install omxplayer nginx libnginx-mod-rtmp
sudo usermod -aG video www-data
```

Note that nginx will fail to set up if a previously-installed app already controls port 80. However, once set up, we can change nginx's web server to a different port so that other apps (e.g, Dicaffeine) can use port 80 instead.

To create an RTMP server, edit the main nginx config file `/etc/nginx/nginx.conf` using e.g.,
```bash
sudo nano /etc/nginx/nginx.conf
```
adding at the bottom (and savings your changes):
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

If you prefer, you can set up a restreamer rather than a player by replacing the `exec omxplayer` line with one or more lines containing e.g., `push rtmp://a.rtmp.youtube.com/live2/xxxx-xxxx-xxxx-xxxx-xxxx;`

Test the config using:
```bash
sudo nginx -t
```
and restart nginx using:
```bash
sudo nginx -s reload
```

Ensure you know your local IP address (e.g., 192.168.0.140, which you can check using `hostname -I`). You should then be able to use the RTMP URL `rtmp://192.168.0.140/live` with any stream key. Access from outside your local network should require you to set up port forwarding on your router, with **potentially serious security implications**; at a minimum, you may want to change nginx to use a non-standard port, and change the application name to a long string other than "live."

Note that by default the cursor will show on top of streamed video. (This would not be true if you were using a terminal-only Pi as suggested in Aaron Parecki's original instructions.) We can hide the cursor when it's not moving using the `unclutter` package:
```bash
sudo apt install unclutter
sudo service lightdm restart
```

If you wish to change nginx's default webserver from port 80 to another port (e.g., so that we can install Dicaffeine), edit `/etc/nginx/sites-available/default` using e.g.,
```bash
sudo nano /etc/nginx/sites-available/default
```
and change the two "listen" lines under `server {` from "80" to any unused port (e.g., "81"):
```
server {
        listen 81 default_server;
        listen [::]:81 default_server;
```

Test the config using
```bash
sudo nginx -t
```
and restart nginx using
```bash
sudo nginx -s reload
```



### Dicaffeine (NDI receiver)

[Homepage](https://dicaffeine.com/about)

In terminal, run
```bash
wget -O - http://dicaffeine.com/repository/dicaffeine.key | sudo apt-key add -
echo "deb https://dicaffeine.com/repository/ buster main non-free" | sudo tee -a /etc/apt/sources.list.d/dicaffeine.list
sudo apt update
sudo apt install -y dicaffeine
```

Ensure you know your local IP address (e.g., 192.168.0.140, which you can check using `hostname -I`). Open Dicaffeine at that IP using either the Pi's web browser, or another computer on the local network that can access the Pi.

The default password for Dicaffeine is `admin`; you can change this using the "System" tab at the top of the browser window.

On the "Player" tab, choose an NDI source under Main stream, and select "Built-in Audio Digital Stereo" under Audio. "Low resolution" is suggested (or use the Advanced properties); Dicaffeine does not seem able to handle e.g. 1080p30. Click "Save" to save setting, and then "Play."

Exit the player by clicking "Stop" if controlling from a remote computer, or by right-clicking the Pi's mouse.

If you choose "Autorun after start" (and then save), the Pi will boot directly into a running Dicaffeine player rather than the desktop environment.


### OBS Ninja

[Homepage](https://obs.ninja)

We need to set a few Chromium flags to ensure the Pi relies on hardware-accelerated decoding to the degree possible. Open the Chromium browser, and enable the following options:
* `chrome://flags/#ignore-gpu-blocklist`
* `chrome://flags/#enable-accelerated-video-decode`
* `chrome://flags/#enable-gpu-rasterization`

Restart the browser. You can check hardware acceleration status by browsing to `chrome://gpu`, although some sources suggest that Chromium may falsely claim to be using hardware acceleration.

As a simple example, direct guests to join an OBSN room at e.g., <https://obs.ninja/?room=ROOMNAME&pw=ROOMPASSWORD> or obfuscate the password using <https://invite.cam>. You can control this "room" (including sending audio/video to your guest that will not be captured by the Pi) at <https://obs.ninja/?director=ROOMNAME&pw=ROOMPASSWORD>.

Ensure you know your local IP address (e.g., 192.168.0.140, which you can check using `hostname -I`). You can now open a fullscreen OBS viewer remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "chromium-browser --kiosk --display=:0 --autoplay-policy=no-user-gesture-required \"https://obs.ninja/?scene=0&room=ROOMNAME&password=ROOMPASSWORD&codec=h264&nocursor&height=720\""
```

You can close this browser window remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "pkill -o chromium"
```

## Video players

### VLC

VLC should already be installed by default, but you can confirm by running:
```bash
sudo apt install vlc
```

Ensure you know your local IP address (e.g., 192.168.0.140, which you can check using `hostname -I`). You can now open a fullscreen VLC player remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "cvlc --one-instance -I http --http-port 8080 --http-password testpassword --no-xlib --aout=alsa --no-video-title --repeat /opt/vc/src/hello_pi/hello_video/test.h264"
```

The `one-instance` flag helps ensure that any subsequent commands run in the same VLC instance. [Many command line flags](https://wiki.videolan.org/VLC_command-line_help) are available.


You can close this player by asking it to "play" `vlc://quit`, or remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "cvlc --one-instance --no-xlib --aout=adummy vlc://quit"
```


### omxplayer

omxplayer should already be installed by default, but you can confirm by running:
```bash
sudo apt install omxplayer
```

Ensure you know your local IP address (e.g., 192.168.0.140, which you can check using `hostname -I`). You can now open a fullscreen omxplayer player remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "omxplayer --loop /home/pi/media/fallingstars_1080p.mp4"
```

You can close this player remotely via SSH using e.g.,
```bash
ssh pi@192.168.0.140 "pkill omxplayer"
```



## Other utilities

### clock-8001

[Homepage](https://gitlab.com/Depili/clock-8001)

*To be added*

## Bitfocus Companion control

[Homepage](https://bitfocus.io/companion)

*To be added*
