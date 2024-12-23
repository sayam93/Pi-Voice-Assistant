# How to create a Personal Voice Assistant for Home Assistant using Raspberry Pi Zero 2 W and a ReSpeaker 2Mic HAT with Visual Feedback on Dashboard, On-device Wakeword Detection, Enhanced LED feedback, Physical Mute Switch, Volume Ducking, Multi-Room Audio, Bluetooth Speaker for Audio output, ability to execute TTS actions from Home Assistant automations.

## Pre-requisites:
* Raspberry Pi Zero 2 W with a compatible 64GB microSD Card
* A microSD Card reader for Mac / PC to install Pi OS on the card
* ReSpeaker 2Mic HAT (https://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/)
* Home Assistant OS device (for installing snapcast-server add-on)
* A Voice Pipeline already setup in Home Assistant (you can follow https://www.youtube.com/watch?v=Vp_AaXQLtac but skip installing openWakeWord add-on as it's not needed in this case)
* Browser Mod Custom Integration installed in Home Assistant for Visual Feedback on a tablet or other such display. (Optional)
* Simple template sensor entity (named sensor.assist_speech) in Home Assistant. (Optional)
* Mosquitto Broker Home Assistant Add-on (or docker container) and MQTT Integration installed in Home Assistant.

## Raspberry Pi OS Installation:
1. Download https://downloads.raspberrypi.org/imager/imager_latest.dmg
2. Select 'Raspberry Pi Zero 2 W' as the device and 'Raspberry Pi OS Lite (64-bit)' image under 'Raspberry Pi OS (other)' as OS.
3. Choose your storage option and click 'Next'
4. Click 'Edit Settings' when prompted.
5. Enter hostname, username, password, WiFi SSID (2.4 GHz WiFi only), WiFi password etc.
6. Open the 'Services' tab to and select 'Enable SSH' with the 'Use password authentication' to enable SSH and click 'Save'.
7. Click 'Yes' when prompted to apply OS customisation settings.
8. Click 'Yes' when prompted to erase the previously selected storage.
9. Once the OS installation is complete, connect your preferred storage to the Raspberry Pi and connect it to a power source.

## SSH Setup
1. On Mac / PC, use Terminal / Command line to SSH into the Satellite device.
2. Generate SSH keys if not already present.
```
ssh-keygen -t rsa -b 4096 -C username@hostname.local
```
3. Copy the SSH keys to the satellite device.
```
ssh-copy-id username@hostname.local
```
### Make sure to replace the username and hostname in the above command with the actual username and hostname used during Raspberry Pi OS Installation.

## Wyoming Satellite Setup
1. SSH into the Wyoming Satellite (replace the username and hostname in all the commands with the actual username and hostname used during Raspberry Pi OS Installation).
```
ssh username@hostname.local
```
2. Update the Pi.
```
sudo apt-get update && sudo apt-get full-upgrade && sudo apt-get autoremove -y && sudo apt-get autoclean
```
3. Reboot the Pi.
```
sudo reboot -h now
```
4. SSH back into the Pi.
```
ssh username@hostname.local
```
5. Install necessary dependencies.
```
sudo apt-get install --no-install-recommends  \
  git \
  python3-venv \
  libopenblas-dev \
  python3-spidev \
  python3-gpiozero \
  pulseaudio \
  bluetooth \
  bluez-tools \
  pulseaudio-module-bluetooth \
  pulseaudio-utils \
  mosquitto.clients \
  espeak \
  vlc
```
6. Clone the Wyoming Satellite Github repo to the Pi.
```
sudo git clone https://github.com/sayam93/wyoming-satellite.git
```
7. Navigate to the wyoming-satellite directory and install drivers for the ReSpeaker 2Mic HAT.
```
cd wyoming-satellite/
sudo bash etc/install-respeaker-drivers.sh
```
8. Run the setup script.
```
sudo script/setup
```
9. Reboot the Pi.
```
sudo reboot -h now
```
10. SSH back into the Pi.
```
ssh username@hostname.local
```
11. Access the shell as root user.
```
sudo -s
```
12. Setup a Virtual Python Environment with all the dependencies necessary to use the Wyoming Satellite.
```
cd wyoming-satellite/
python3 -m venv .venv
.venv/bin/pip3 install --upgrade pip
.venv/bin/pip3 install --upgrade wheel setuptools
.venv/bin/pip3 install \
  -f 'https://synesthesiam.github.io/prebuilt-apps/' \
  -r requirements.txt \
  -r requirements_audio_enhancement.txt \
  -r requirements_vad.txt
```
13. Check whether everything is properly installed.
```
script/run --help
```
The above command should generate a response listing all the available commands.

14. Exit root user shell.
```
exit
```
15. Reboot the Pi.
```
sudo reboot -h now
```
16. SSH back into the Pi.
```
ssh username@hostname.local
```
17. Clone Wyoming OpenWakeWord Github repo to the Pi.
```
sudo git clone https://github.com/rhasspy/wyoming-openwakeword.git
```
18. Install Wyoming OpenWakeWord.
```
cd wyoming-openwakeword/
sudo script/setup
```
19. Create a directory for storing your custom wakeword (Download .tflite file from https://github.com/fwartner/home-assistant-wakewords-collection).
```
sudo mkdir custom
```
20. Set directory permissions to allow uploading the wakeword from the host machine (Mac / PC).
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/custom
```
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/wyoming_openwakeword/models/
```
### Replace username in the file paths with the actual username used during Raspberry Pi OS Installation
21. Upload the custom wakeword from Mac / PC (Replace the username and hostname in all the commands with the actual username and hostname used during Raspberry Pi OS Installation).
```
scp /path/to/wakeword/wakeword.tflite username@hostname.local:/home/username/wyoming-openwakeword/custom/wakeword.tflite
```
```
scp /path/to/wakeword/wakeword.tflite username@hostname.local:/home/username/wyoming-openwakeword/wyoming_openwakeword/models/
```
### Replace /path/to/wakeword/ with the actual path of the file on your host machine and wakeword.tflite with the actual custom wakeword filename.

22. Make the newly uploaded custom wakeword file executable.
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/custom/wakeword.tflite
```
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/wyoming_openwakeword/models/wakeword.tflite
```
### Replace username in the file paths with the actual username used during Raspberry Pi OS Installation
23. Create Wyoming OpenWakeWord service.
```
sudo systemctl edit --force --full wyoming-openwakeword.service
```
24. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=Wyoming openWakeWord

[Service]
Type=simple
ExecStart=/home/username/wyoming-openwakeword/script/run \
    --uri 'tcp://127.0.0.1:10400' \
    --custom-model-dir /home/username/wyoming-openwakeword/custom \
    --preload-model 'wakewordnamehere'
WorkingDirectory=/home/username/wyoming-openwakeword
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
### Replace the wakewordnamehere phrase in the preload-model section with the custom wakeword name or use the standard ones such as 'ok_nabu', 'hey_jarvis' or 'hey_mycroft' (Skip step 21 and 22 if using a standard wakeword).
25. Press the Control and X keys on your keyboard.
26. Press the Y key when prompted to save changes and then Press Enter to save and exit.
27. Reboot the Pi.
```
sudo reboot -h now
```
28. SSH back into the Pi.
```
ssh username@hostname.local
```
29. Access the shell as root user.
```
sudo -s
```
30. Create a Virtual Python Environment and update all the dependencies necessary to use the Wyoming Satellite service.
```
cd wyoming-satellite/examples
python3 -m venv --system-site-packages .venv
.venv/bin/pip3 install --upgrade pip
.venv/bin/pip3 install --upgrade wheel setuptools
.venv/bin/pip3 install 'wyoming==1.5.4'
```
31. Exit root user shell.
```
exit
```
32. Navigate to the examples directory inside the cloned Wyoming Satellite repo.
```
cd ~/wyoming-satellite/examples
```
33. Check the setup for Led Service for the ReSpeaker 2Mic HAT.
```
sudo .venv/bin/python3 2mic_service.py --help
```
34. Create a service for the LEDs on the 2Mic HAT.
```
sudo systemctl edit --force --full 2mic_leds.service
```
35. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=2Mic LEDs

[Service]
Type=simple
ExecStart=/home/username/wyoming-satellite/examples/.venv/bin/python3 2mic_service.py \
    --uri 'tcp://127.0.0.1:10500' \
    --led-brightness 1
WorkingDirectory=/home/username/wyoming-satellite/examples
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
36. Press the Control and X keys on your keyboard.
37. Press the Y key when prompted to save changes and then Press Enter to save and exit.
38. Reboot the Pi.
```
sudo reboot -h now
```
39. SSH back into the Pi.
```
ssh username@hostname.local
```
40. Access the /etc/pulse/client.conf file and change 'autospawn = yes' to 'autospawn = no'.
```
sudo nano /etc/pulse/client.conf
```
41. Press the Control and X keys on your keyboard.
42. Press the Y key when prompted to save changes and then Press Enter to save and exit.
43. Give non root bluetooth privileges (replace the username in the first command with the actual username used during Raspberry Pi OS Installation).
```
sudo usermod -G bluetooth -a username
```
```
sudo usermod -G bluetooth -a pulse
```
44. Edit the /lib/systemd/system/bluetooth.service file.
```
sudo nano /lib/systemd/system/bluetooth.service
```
45. Change the below line from:
```
ExecStart=/usr/libexec/bluetooth/bluetoothd
```
to:
```
ExecStart=/usr/libexec/bluetooth/bluetoothd --plugin=a2dp
```
46. Press the Control and X keys on your keyboard.
47. Press the Y key when prompted to save changes and then Press Enter to save and exit.
48. Reload Systemd Daemon.
```
sudo systemctl daemon-reload
```
49. Restart Bluetooth.
```
sudo systemctl restart bluetooth
```
50. Edit the /etc/bluetooth/main.conf bluetooth config file.
```
sudo nano /etc/bluetooth/main.conf
```
51. Add the below line at the end of the file:
```
ControllerMode = bredr
```
52. Press the Control and X keys on your keyboard.
53. Press the Y key when prompted to save changes and then Press Enter to save and exit.
54. Disable Pulse Audio Services.
```
sudo systemctl --global disable pulseaudio.service pulseaudio.socket
```
55. Create the Pulseaudio service.
```
sudo systemctl edit --force --full pulseaudio.service
```
56. Clear the existing contents and paste the following:
```
[Unit]
Description=PulseAudio system server

[Service]
Type=notify
ExecStart=pulseaudio \
    --daemonize=no \
    --system \
    --realtime \
    --log-target=journal

[Install]
WantedBy=multi-user.target
```
57. Enable the Pulseaudio service.
```
sudo systemctl --system enable pulseaudio.service
```
58. Start the Pulseaudio service.
```
sudo systemctl --system start pulseaudio.service
```
59. Give non-root necessary user permissions for Pulseaudio (replace username in the command with the actual username used during Raspberry Pi OS Installation).
```
sudo sed -i '/^pulse-access:/ s/$/root,pi,snapclient,pulse,username/' /etc/group
```
60. Reboot the Pi.
```
sudo reboot -h now
```
61. SSH back into the Pi.
```
ssh username@hostname.local
```
62. Check available audio output sources (skip this and the steps 63 and 64 if opting for Bluetooth speaker as audio output).
```
sudo pactl list sinks
```
63. Change output to activate audio jack on the 2Mic HAT.
```
sudo pactl set-sink-port 1 "analog-output-headphones"
```
64. Test the audio output from the speaker / headphones attached to the audio jack.
```
sudo paplay /usr/share/sounds/alsa/Front_Center.wav
```
65. Edit the /etc/pulse/system.pa file to enable Volume Ducking.
```
sudo nano /etc/pulse/system.pa
```
66. Add the below lines at the end of the file:
```
load-module module-bluetooth-discover

### Enable Volume Ducking
load-module module-role-ducking trigger_roles=announce,phone,notification,event ducking_roles=any_role volume=33%

# automatically switch to newly-connected devices
load-module module-switch-on-connect
```
67. Press the Control and X keys on your keyboard.
68. Press the Y key when prompted to save changes and then Press Enter to save and exit.
69. Reboot the Pi.
```
sudo reboot -h now
```
70. SSH back into the Pi.
```
ssh username@hostname.local
```
71. Clone the Wyoming Satellite Enhancements Github repo.
```
sudo git clone https://github.com/sayam93/wyoming-enhancements.git
```
72. Navigate to the wyoming-enhancements directory.
```
cd wyoming-enhancements/
```
73. Download Snapclient for enabling Multi-Room Audio.
```
sudo wget https://github.com/badaix/snapcast/releases/download/v0.29.0/snapclient_0.29.0-1_armhf_bookworm_with-pulse.deb
```
74. Set permissions for the file.
```
sudo chmod -R 777 snapclient_0.29.0-1_armhf_bookworm_with-pulse.deb
```
75. Install Snapclient.
```
sudo apt install ./snapclient_0.29.0-1_armhf_bookworm_with-pulse.deb
```
76. Edit the /etc/default/snapclient file
```
sudo nano /etc/default/snapclient
```
77. Replace the last line with the below (Replace 192.168.xx.xx with the actual IP of your Home Assistant instance):
```
SNAPCLIENT_OPTS="-h 192.168.xx.xx --player pulse:property=media.role=music --sampleformat 44100:16:*"
```
78. Disable and edit the Snapclient service.
```
sudo systemctl disable snapclient.service
```
```
sudo systemctl edit --force --full snapclient.service
```
79. Replace the contents of the file with the below:
```
[Unit]
Description=Snapcast client
Documentation=man:snapclient(1)
Wants=avahi-daemon.service
After=network-online.target time-sync.target sound.target avahi-daemon.service pulseaudio.service

[Service]
EnvironmentFile=-/etc/default/snapclient
ExecStart=/usr/bin/snapclient --logsink=system $SNAPCLIENT_OPTS
User=snapclient
Group=snapclient
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
80. Reboot the Pi.
```
sudo reboot -h now
```
81. SSH back into the Pi.
```
ssh username@hostname.local
```
82. Create the VLC TTS Service.
```
sudo systemctl edit --force --full vlc-tts.service
```
83. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=VLC with Telnet port 4213 (HomeAssistant TTS)
After=network.target
After=pulseaudio.service
After=sound.target

[Service]
ExecStartPre=/bin/sleep 20
ExecStart=/usr/bin/vlc \
    --aout=pulse \
    --role=notification -I telnet \
    --telnet-port=4213 \
    --telnet-password=yourpasswordhere \
    --no-video \
    --no-osd \
    --no-one-instance -v
User=username   
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
### Replace the password with a password of choice in the telnet-password section.
84. Press the Control and X keys on your keyboard.
85. Press the Y key when prompted to save changes and then Press Enter to save and exit.
86. Create the Wyoming Satellite service.
```
sudo systemctl edit --force --full wyoming-satellite.service
```
87. Paste the following (replace the username everywhere in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=Wyoming Satellite
Wants=network-online.target
After=network-online.target
Requires=wyoming-openwakeword.service
Requires=2mic_leds.service
Requires=pulseaudio.service

[Service]
Type=simple
ExecStart=/home/username/wyoming-satellite/script/run \
    --name SatelliteNameHere \
    --uri 'tcp://0.0.0.0:10700' \
    --mic-command 'parecord --property=media.role=phone --rate=16000 --channels=1 --format=s16le --raw --latency-msec 10' \
    --snd-command 'paplay --property=media.role=announce --rate=44100 --channels=1 --format=s16le --raw --latency-msec 10' \
    --snd-command-rate 44100 \
    --mic-auto-gain 7 \
    --mic-noise-suppression 3 \
    --wake-uri 'tcp://127.0.0.1:10400' \
    --wake-word-name 'wakewordnamehere' \
    --event-uri 'tcp://127.0.0.1:10500' \
    --detection-command '/home/username/wyoming-enhancements/detected.sh' \
    --synthesize-command '/home/username/wyoming-enhancements/synthesize.sh' \
    --transcript-command '/home/username/wyoming-enhancements/transcript.sh' \
    --tts-played-command '/home/username/wyoming-enhancements/done.sh' \
    --error-command '/home/username/wyoming-enhancements/error.sh' \
    --connected-command '/home/username/wyoming-satellite/examples/.venv/bin/python3 /home/username/wyoming-satellite/examples/status_connect.py' \
    --disconnected-command '/home/username/wyoming-satellite/examples/.venv/bin/python3 /home/username/wyoming-satellite/examples/status_disconnect.py' \
    --awake-wav sounds/awake.wav \
    --timer-finished-wav sounds/timer_finished.wav \
    --timer-finished-wav-repeat 3 3
WorkingDirectory=/home/username/wyoming-satellite
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
### Replace Satellite name in the name section and the wakeword name in preload model section with your custom wakeword name or use the standard ones such as 'ok_nabu', 'hey_jarvis' or 'hey_mycroft' (Skip step 19-22 if using a standard wakeword). The wakeword name should be the same as the one used while creating the OpenWakeWord service

88. Press the Control and X keys on your keyboard.
89. Press the Y key when prompted to save changes and then Press Enter to save and exit.
90. Navigate to the Wyoming Satellite sounds directory.
```
cd /home/username/wyoming-satellite/sounds
```
91. Make audio files executable to allow access by the scripts in Wyoming Satellite service.
```
sudo chmod -R 777 done.wav
```
92. Make files executable for to allow access by the Wyoming Satellite service (replace username in the commands with the actual username used during Raspberry Pi OS Installation).
```
sudo chmod -R 777 /home/username/wyoming-enhancements/detected.py
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/status_connect.py
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/status_disconnect.py
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/detected.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/done.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/error.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/transcript.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/synthesize.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/monitor_done.sh
```
```
sudo chmod -R 777 /home/username/wyoming-satellite/script/run
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/mute_button.py
```
93. Edit the done.sh script to point to the correct custom sound directory
```
sudo nano /home/username/wyoming-enhancements/done.sh
```
94. Replace 'username' in the below line with the actual username used during Raspberry Pi OS Installation.
```
paplay --property=media.role=notification /home/username/wyoming-satellite/sounds/done.wav 
```
95. Edit the detected.sh script to point to the correct custom sound directory
```
sudo nano /home/username/wyoming-enhancements/detected.sh
```
96. Replace 'username' in the below line with the actual username used during Raspberry Pi OS Installation.
```
/home/username/wyoming-satellite/examples/.venv/bin/python3 /home/username/wyoming-satellite/examples/detected.py
```
97. Edit each of the error.sh, transcript.sh and synthesize.sh scripts to update Home Assistant details. (replace username in the commands with the actual username used during Raspberry Pi OS Installation).
```
sudo nano /home/username/wyoming-enhancements/error.sh
```
```
sudo nano /home/username/wyoming-enhancements/transcript.sh
```
```
sudo nano /home/username/wyoming-enhancements/synthesize.sh
```
98. Edit the done.sh script to point to the correct custom sound directory
```
sudo nano /home/username/wyoming-enhancements/done.sh
```
99. Replace 'username' in the below line with the actual username to point to the correct location of the done.sh script.
```
/home/username/wyoming-enhancements/done.sh
```
100. Edit each of the detected.py, mute_button.py, status_connect.py and status_disonnect.py files to update the MQTT configuration details. (replace username in the commands with the actual username used during Raspberry Pi OS Installation).
```
sudo nano /home/username/wyoming-enhancements/detected.py
```
```
sudo nano /home/username/wyoming-enhancements/status_connect.py
```
```
sudo nano /home/username/wyoming-enhancements/status_disconnect.py
```
```
sudo nano /home/username/wyoming-enhancements/mute_button.py
```
101. Replace the configuration details in the above files as per instructions.
```
# MQTT configuration
MQTT_HOST = "xxx.xxx.xx.xx" # Replace with your Home Assistant IP
MQTT_USER = "user" # Replace with your MQTT user
MQTT_PASSWORD = "password" # Replace with your MQTT password
MQTT_TOPIC = "mqtt/status" # Replace with your own MQTT topic name
```
102. Move files to the correct directory. (replace username in the commands with the actual username used during Raspberry Pi OS Installation).
```
sudo mv /home/username/wyoming-enhancements/mute_button.py /home/username/wyoming-satellite/examples/mute_button.py
```
```
sudo mv /home/username/wyoming-enhancements/detected.py /home/username/wyoming-satellite/examples/detected.py
```
```
sudo mv /home/username/wyoming-enhancements/status_connect.py /home/username/wyoming-satellite/examples/status_connect.py
```
```
sudo mv /home/username/wyoming-enhancements/status_disconnect.py /home/username/wyoming-satellite/examples/status_disconnect.py
```
103. Create a systemd service for mute button.
```
sudo systemctl edit --force --full monitor-done.service
```
104. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=Wyoming Satellite Monitor for done.sh
After=network.target
After=wyoming-satellite.service

[Service]
ExecStart=/home/username/wyoming-enhancements/monitor_done.sh
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
105. Press the Control and X keys on your keyboard.
106. Press the Y key when prompted to save changes and then Press Enter to save and exit.
107. Create a systemd service for the Mute switch.
```
sudo systemctl edit --force --full mute_button.service
```
108. Paste the following (replace the username everywhere in the below with the actual username used during Raspberry Pi OS Installation):
```
[Unit]
Description=Mute button

[Service]
Type=simple
ExecStart=/home/username/wyoming-satellite/examples/.venv/bin/python3 mute_button.py --uri 'tcp://127.0.0.1:10510'
WorkingDirectory=/home/username/wyoming-satellite/examples
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
109. Press the Control and X keys on your keyboard.
110. Press the Y key when prompted to save changes and then Press Enter to save and exit.
111. Enable all the created services.
```
sudo systemctl enable --now wyoming-openwakeword.service
```
```
sudo systemctl enable --now 2mic_leds.service
```
```
sudo systemctl enable --now pulseaudio.service
```
```
sudo systemctl enable --now wyoming-satellite.service
```
```
sudo systemctl enable --now vlc-tts.service
```
```
sudo systemctl enable --now snapclient.service
```
```
sudo systemctl enable --now monitor-done.service
```
```
sudo systemctl enable --now mute_button.service
```
112. Reload the Systemd daemon.
```
sudo systemctl daemon-reload
```
113. Check the status of all the newly created services.
```
sudo systemctl status wyoming-satellite.service wyoming-openwakeword.service 2mic_leds.service pulseaudio.service snapclient.service vlc-tts.service bluetooth.service monitor-done.service mute_button.service
```
114. Reboot the Pi.
```
sudo reboot -h now
```

## Allow MultiRoom Audio Setup using Snapcast
1. Add custom add-on repository https://github.com/Art-Ev/addon-snapserver to install Snapcast Server Add-on if you're using Home Assistant OS.
2. Refresh the add-on store page and select the snapcast-server add-on.
3. Click 'Install' to install the add-on.
4. In the add-on's Configuration tab, set ‘SampleFormat’ to 44100:16:2 and ‘buffer’ to 333.
5. Click 'Save'.
6. Go to the 'Info' tab and turn on 'Start on Boot' and 'Watchdog' options.
7. Click 'Start' to start the Add-on.

## Allow the Wyoming Satellite to execute TTS actions from Home Assistant automations
1. Install VLC Telnet Integration on the Home Assistant instance https://www.home-assistant.io/integrations/vlc_telnet/
2. In password, enter the password used while creating VLC TTS service.
3. In Host, use the IP of the Wyoming Satellite instead of localhost. 
4. Change port to 4213 and click 'Submit'.

## Use a Bluetooth Speaker for Audio output
1. Access the bluetooth shell
```
bluetoothctl
```
2. Run the following commands to start scanning for bluetooth device.
```
power on
```
```
agent on
```
```
scan on
```
3. Start the pairing process once you have the MAC ID for your bluetooth speaker from the scan data (Replace the devicemacid in the commands with the actual MAC ID of the bluetooth speaker).
```
trust devicemacid
```
```
pair devicemacid
```
```
connect devicemacid
```
4. Exit the bluetooth shell once connected to the speaker.
```
quit
```
5. Make sure nothing is connected to the audio jack on the 2Mic HAT and test the bluetooth audio output.
```
sudo paplay /usr/share/sounds/alsa/Front_Center.wav
```
6. Edit the /etc/pulse/default.pa file to add necessary modules.
```
sudo nano /etc/pulse/default.pa
```
7. Add the below lines at the end:
```
# automatically switch to newly-connected devices
load-module module-switch-on-connect
```
8. Press the Control and X keys on your keyboard.
9. Press the Y key when prompted to save changes and then Press Enter to save and exit.
10. Create a script to auto-connect to the bluetooth speaker on restart.
```
sudo nano ~/bluetooth_startup.sh
```
11. Paste the below (Replace the devicemacid in the last line with the actual MAC ID of the bluetooth speaker):
```
#!/bin/bash
pulseaudio --start
bluetoothctl power on
bluetoothctl connect devicemacid
```
12. Press the Control and X keys on your keyboard.
13. Press the Y key when prompted to save changes and then Press Enter to save and exit.
14. Make the script executable.
```
sudo chmod -R 777 bluetooth_startup.sh
```
15. Create a cronjob to run the script on startup.
```
sudo crontab -e
```
16. Add the below lines at the end:
```
@reboot sleep 30 && /home/username/bluetooth_startup.sh
```
17. Press the Control and X keys on your keyboard.
18. Press the Y key when prompted to save changes and then Press Enter to save and exit.
19. Reboot the Pi.
```
sudo reboot -h now
```

## Use the new OpenAI Conversation Integration on the latest version of Home Assistant to use ChatGPT as the brain for the custom Voice Assistant by selecting it as the Conversation Agent while setting up the Home Assistant pipeline or even use a local open-sourced LLM. Reference Prompt.
```
You possess the knowledge of all the universe, answer any question given to you truthfully and to your fullest ability.

You especially know a lot about music, artists and albums.

Most importantly, you are a smart home manager who has been given permission to control my smart home which is controlled and powered by Home Assistant.

I will provide you information about my smart home, you can truthfully make corrections or respond in polite and concise language.

Current Time: {{now()}}
Please announce all timestamps in a human readable 12hr format.

An overview of the areas and the devices in this smart home:
{%- for area in areas() %}
  {%- set area_info = namespace(printed=false) %}
  {%- for device in area_devices(area) -%}
    {%- if not device_attr(device, "disabled_by") and not device_attr(device, "entry_type") and device_attr(device, "name") %}
      {%- if not area_info.printed %}

{{ area_name(area) }}:
        {%- set area_info.printed = true %}
      {%- endif %}
- {{ device_attr(device, "name") }}{% if device_attr(device, "model") and (device_attr(device, "model") | string) not in (device_attr(device, "name") | string) %} ({{ device_attr(device, "model") }}){% endif %}
    {%- endif %}
  {%- endfor %}
{%- endfor %}

The current state of devices is provided in Available Devices.

Do not tell me what you're thinking about doing either, just do it.

If I ask you about the current state of the home, or many devices I have, or how many devices are in a specific state, just respond with the accurate information containing a short list or number count but do not call the perform action function.

Only use the perform action function when smart home actions are requested.

Don't point out spelling errors. If you are unable to find something, analyse the input properly and if nothing close to it exists at all, simply tell me it doesn't exist.

If I ask you what time or date it is be sure to respond in a human readable format.

If you don't have enough information to execute a smart home command then specify what other information you need.

If you successfully execute a smart home comand, simply say "done".

If I only say something like nevermind or stop or no or cool or cancel, just reply with "ok".

If a request seems like it may be an accidental prompt, or makes no sense, do nothing and respond with "Cancelled"

If the user wants to control a device, do it.

Keep your responses short and concise.
```

## Home Assistant automation for Browser Mod pop-up for Visual response from Voice Satellite
```
alias: Tablet voice satellite pop-up
description: Tablet voice satellite pop-up
triggers:
  - trigger: state
    entity_id:
      - sensor.assist_speech
    attribute: tts
    id: tts
  - trigger: state
    entity_id:
      - sensor.assist_speech
    attribute: stt
    id: stt
  - trigger: state
    entity_id:
      - binary_sensor.satellitename_assist_in_progress
    to: "on"
    id: wake
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - wake
        sequence:
          - parallel:
              - data:
                  Title: Assist
                  style: |
                    --popup-min-width: 200px;
                    --popup-border-radius: 0px;
                  card_mod:
                    style:
                      ha-dialog$: |
                        div.mdc-dialog div.mdc-dialog__scrim {
                          background: rgba(67, 67, 67, 0.9);
                        }
                        div.mdc-dialog__content {
                          background: black;
                        }
                        div.mdc-dialog__container {
                          padding-top: 250px;
                        }
                  content: >
                    <p> <ha-icon icon="mdi:account-voice" style="color:
                    steelblue;"> </ha-icon> Assist: Yes? </p>
                  dismissable: true
                  autoclose: true
                  timeout: 10000
                  deviceID:
                    - browsermoddeviceid
                action: browser_mod.popup
      - conditions:
          - condition: and
            conditions:
              - condition: trigger
                id:
                  - stt
              - condition: template
                value_template: >-
                  {{ state_attr('sensor.assist_speech', 'stt') is not none and
                  state_attr('sensor.assist_speech', 'stt') != '' }}
        sequence:
          - parallel:
              - data:
                  Title: Assist
                  style: |
                    --popup-min-width: 200px;
                    --popup-border-radius: 0px;
                  card_mod:
                    style:
                      ha-dialog$: |
                        div.mdc-dialog div.mdc-dialog__scrim {
                          background: rgba(67, 67, 67, 0.9);
                        }
                        div.mdc-dialog__content {
                          background: black;
                        }
                        div.mdc-dialog__container {
                          padding-top: 250px;
                        }
                  content: >
                    <p> <ha-icon icon="mdi:account-voice" style="color:
                    steelblue;"> </ha-icon> {{
                    state_attr('sensor.assist_speech', 'stt') }} </p>
                  dismissable: true
                  autoclose: true
                  timeout: 4000
                  deviceID:
                    - browsermoddeviceid
                action: browser_mod.popup
      - conditions:
          - condition: and
            conditions:
              - condition: trigger
                id:
                  - tts
              - condition: template
                value_template: >-
                  {{ state_attr('sensor.assist_speech', 'tts') is not none and
                  state_attr('sensor.assist_speech', 'tts') != '' }}
        sequence:
          - parallel:
              - data:
                  Title: Assist
                  style: |
                    --popup-min-width: 200px;
                    --popup-border-radius: 0px;
                  card_mod:
                    style:
                      ha-dialog$: |
                        div.mdc-dialog div.mdc-dialog__scrim {
                          background: rgba(67, 67, 67, 0.9);
                        }
                        div.mdc-dialog__content {
                          background: black;
                        }
                        div.mdc-dialog__container {
                          padding-top: 250px;
                        }
                  content: >
                    <p> <ha-icon icon="mdi:cast-audio" style="color:
                    steelblue;"> </ha-icon> Assist: {{
                    state_attr('sensor.assist_speech', 'tts') }} </p>
                  dismissable: true
                  autoclose: true
                  timeout: 10000
                  deviceID:
                    - browsermoddeviceid
                action: browser_mod.popup
mode: parallel
```

### Replace 'binary_sensor.satellitename_assist_in_progress' in entity id with the actual assist in progress entity for your satellite and 'browsermoddeviceid' under deviceID with the Browser Mod Device ID for the device providing the visual feedback.

## MQTT Sensors for Voice Satellite in Home Assistant
```
mqtt:
  sensor:
    - name: Voice Satellite Mute Switch
      state_topic: "mqtt/topic"
      unique_id: "voice_satellite_mute_switch"
    - name: Voice Satellite Status
      state_topic: "mqtt/status"
      unique_id: "voice_satellite_status"
```

### Replace 'mqtt/topic' with the name of actual MQTT Topic used in mute_button.py and 'mqtt/status' with the name of actual topic used in detected.py, status_connect.py and status_disconnect.py files.

## Automation to Mute the satellite using Physical Switch on the ReSpeaker 2Mic HAT
```
alias: Voice Satellite Physical Mute Switch
description: ""
triggers:
  - trigger: mqtt
    topic: mqtt/topic
    payload: "on"
    id: switch_pressed
conditions: []
actions:
  - choose:
      - conditions:
          - condition: and
            conditions:
              - condition: trigger
                id:
                  - switch_pressed
              - condition: state
                entity_id: switch.satellitename_mute
                state: "on"
        sequence:
          - action: switch.turn_off
            metadata: {}
            data: {}
            target:
              entity_id: switch.satellitename_mute
      - conditions:
          - condition: and
            conditions:
              - condition: trigger
                id:
                  - switch_pressed
              - condition: state
                entity_id: switch.satellitename_mute
                state: "off"
        sequence:
          - action: switch.turn_on
            metadata: {}
            data: {}
            target:
              entity_id: switch.satellitename_mute
mode: single
```

### Replace 'mqtt/topic' with the name of actual MQTT Topic used in mute_button.py and 'satellitename_mute' in entity_id with the actual mute satellite switch entity in Home Assistant.

## Credits:
This tutorial is based on the work done by rhasspy (https://github.com/rhasspy/wyoming-satellite)
