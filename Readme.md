# How to create a Personal Voice Assistant for Home Assistant using Raspberry Pi Zero 2 W and a ReSpeaker 2Mic HAT with On-device Wakeword, Volume Ducking, Multi-Room Audio, Bluetooth Speaker for Audio output and ability to execute TTS actions from Home Assistant automations.

## Pre-requisites:
* Raspberry Pi Zero 2 W with a compatible 64GB microSD Card
* A microSD Card reader for Mac / PC to install Pi OS on the card
* ReSpeaker 2Mic HAT (https://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/)
* Home Assistant OS device (for installing snapcast-server add-on)
* A Voice Pipeline already setup in Home Assistant (you can follow https://www.youtube.com/watch?v=Vp_AaXQLtac but skip installing openWakeWord add-on as it's not needed in this case)

## Raspberry Pi OS Installation:
1. Download https://downloads.raspberrypi.org/imager/imager_latest.dmg
2. Select 'Raspberry Pi Zero 2 W' as the device and Raspberry Pi OS Lite (64-bit) image under 'Raspberry Pi OS (other)' as OS.

<img width="792" alt="Screenshot 2024-11-11 at 5 21 18 AM" src="https://github.com/user-attachments/assets/30427f2f-112b-4007-9abe-f93c98c28da7">
 
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
  vlc
```
6. Clone the Wyoming Satellite Github repo to the Pi.
```
sudo git clone https://github.com/rhasspy/wyoming-satellite.git
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
15. Navigate to the home directory.
```
cd ~
```
16. Clone Wyoming OpenWakeWord Github repo to the Pi.
```
sudo git clone https://github.com/rhasspy/wyoming-openwakeword.git
```
17. Install Wyoming OpenWakeWord.
```
cd wyoming-openwakeword/
sudo script/setup
```
18. Create a directory for storing your custom wakeword (Download .tflite file from https://github.com/fwartner/home-assistant-wakewords-collection).
```
sudo mkdir custom
```
19. Set directory permissions to allow uploading the wakeword from the host machine (Mac / PC).
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/custom
```
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/wyoming_openwakeword/models/
```
20. Upload the custom wakeword from Mac / PC (Replace the username and hostname in all the commands with the actual username and hostname used during Raspberry Pi OS Installation).
```
scp /path/to/wakeword/wakeword.tflite username@hostname.local:/home/username/wyoming-openwakeword/custom/wakeword.tflite
```
```
scp /path/to/wakeword/wakeword.tflite username@hostname.local:/home/username/wyoming-openwakeword/wyoming_openwakeword/models/
```
### Replace /path/to/wakeword/ with the actual path of the file on your host machine and wakeword.tflite with the actual custom wakeword filename.

21. Make the newly uploaded custom wakeword file executable.
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/custom/wakeword.tflite
```
```
sudo chmod -R 777 /home/username/wyoming-openwakeword/wyoming_openwakeword/models/wakeword.tflite
```
22. Create Wyoming OpenWakeWord service.
```
sudo systemctl edit --force --full wyoming-openwakeword.service
```
23. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
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
### Replace the wakewordnamehere phrase in the preload-model section with the custom wakeword name or use the standard ones such as 'ok_nabu', 'hey_jarvis' or 'hey_mycroft' (Skip step 18 and 19 if using a standard wakeword).

24. Press the Control and X keys on your keyboard.
25. Press the Y key when prompted to save changes and then Press Enter to save and exit.
26. Navigate to Home Directory.
```
cd ~
```
27. Access the shell as root user.
```
sudo -s
```
28. Create a Virtual Python Environment and update all the dependencies necessary to use the Wyoming Satellite service.
```
cd wyoming-satellite/examples
python3 -m venv --system-site-packages .venv
.venv/bin/pip3 install --upgrade pip
.venv/bin/pip3 install --upgrade wheel setuptools
.venv/bin/pip3 install 'wyoming==1.5.4'
```
29. Exit root user shell.
```
exit
```
30. Navigate to the examples directory inside the cloned Wyoming Satellite repo.
```
cd ~/wyoming-satellite/examples
```
31. Check the setup for Led Service for the ReSpeaker 2Mic HAT.
```
sudo .venv/bin/python3 2mic_service.py --help
```
32. Create a service for the LEDs on the 2Mic HAT.
```
sudo systemctl edit --force --full 2mic_leds.service
```
33. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
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
34. Press the Control and X keys on your keyboard.
35. Press the Y key when prompted to save changes and then Press Enter to save and exit.
36. Reboot the Pi.
```
sudo reboot -h now
```
37. SSH back into the Pi.
```
ssh username@hostname.local
```
38. Access the /etc/pulse/client.conf file and change 'autospawn = yes' to 'autospawn = no'.
```
sudo nano /etc/pulse/client.conf
```
39. Press the Control and X keys on your keyboard.
40. Press the Y key when prompted to save changes and then Press Enter to save and exit.
41. Give non root bluetooth privileges.
```
sudo usermod -G bluetooth -a username
```
42. Edit the /lib/systemd/system/bluetooth.service file.
```
sudo nano /lib/systemd/system/bluetooth.service
```
43. Change the below line from:
```
ExecStart=/usr/libexec/bluetooth/bluetoothd
```
to:
```
ExecStart=/usr/libexec/bluetooth/bluetoothd --plugin=a2dp
```
44. Press the Control and X keys on your keyboard.
45. Press the Y key when prompted to save changes and then Press Enter to save and exit.
46. Reload Systemd Daemon.
```
sudo systemctl daemon-reload
```
47. Restart Bluetooth.
```
sudo systemctl restart bluetooth
```
48. Edit the /etc/bluetooth/main.conf bluetooth config file.
```
sudo nano /etc/bluetooth/main.conf
```
49. Add the below line at the end of the file:
```
ControllerMode = bredr
```
50. Press the Control and X keys on your keyboard.
51. Press the Y key when prompted to save changes and then Press Enter to save and exit.
52. Disable Pulse Audio Services.
```
sudo systemctl --global disable pulseaudio.service pulseaudio.socket
```
53. Create the Pulseaudio service.
```
sudo systemctl edit --force --full pulseaudio.service
```
54. Clear the existing contents and paste the following:
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
55. Enable the Pulseaudio service.
```
sudo systemctl --system enable pulseaudio.service
```
56. Start the Pulseaudio service.
```
sudo systemctl --system start pulseaudio.service
```
57. Give non-root necessary user permissions for Pulseaudio.
```
sudo sed -i '/^pulse-access:/ s/$/root,pi,snapclient,pulse,username/' /etc/group
```
58. Reboot the Pi.
```
sudo reboot -h now
```
59. SSH back into the Pi.
```
ssh username@hostname.local
```
60. Check available audio output sources (skip this and the steps 61 and 62 if opting for Bluetooth speaker as audio output).
```
sudo pactl list sinks
```
61. Change output to activate audio jack on the 2Mic HAT.
```
sudo pactl set-sink-port 1 "analog-output-headphones"
```
62. Test the audio output from the speaker / headphones attached to the audio jack.
```
sudo paplay /usr/share/sounds/alsa/Front_Center.wav
```
63. Edit the /etc/pulse/system.pa file to enable Volume Ducking.
```
sudo nano /etc/pulse/system.pa
```
64. Add the below lines at the end of the file:
```
load-module module-bluetooth-discover

### Enable Volume Ducking
load-module module-role-ducking trigger_roles=announce,phone,notification,event ducking_roles=any_role volume=33%
```
65. Press the Control and X keys on your keyboard.
66. Press the Y key when prompted to save changes and then Press Enter to save and exit.
67. Reboot the Pi.
```
sudo reboot -h now
```
68. SSH back into the Pi.
```
ssh username@hostname.local
```
69. Clone the Wyoming Satellite Enhancements Github repo.
```
sudo git clone https://github.com/FutureProofHomes/wyoming-enhancements.git
```
70. Navigate to the Snapcast directory.
```
cd wyoming-enhancements/snapcast/
```
71. Download Snapclient for enabling Multi-Room Audio.
```
sudo wget https://github.com/badaix/snapcast/releases/download/v0.29.0/snapclient_0.29.0-1_armhf_bookworm_with-pulse.deb
```
72. Install Snapclient.
```
sudo apt install ./snapclient_0.29.0-1_armhf_bookworm_with-pulse.deb
```
73. Edit the /etc/default/snapclient file
```
sudo nano /etc/default/snapclient
```
74. Replace the last line with the below (Replace 192.168.xx.xx with the actual IP of your Home Assistant instance):
```
SNAPCLIENT_OPTS="-h 192.168.xx.xx --player pulse:property=media.role=music --sampleformat 44100:16:*"
```
75. Disable and edit the Snapclient service.
```
sudo systemctl disable snapclient.service
```
```
sudo systemctl edit --force --full snapclient.service
```
76. Replace the contents of the file with the below:
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
77. Reboot the Pi.
```
sudo reboot -h now
```
78. SSH back into the Pi.
```
ssh username@hostname.local
```
79. Create the Wyoming Satellite service.
```
sudo systemctl edit --force --full wyoming-satellite.service
```
80. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
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
    --detection-command '/home/username/wyoming-enhancements/snapcast/scripts/awake.sh' \
    --tts-stop-command '/home/username/wyoming-enhancements/snapcast/scripts/done.sh' \
    --error-command '/home/username/wyoming-enhancements/snapcast/scripts/done.sh' \
    --awake-wav sounds/awake.wav \
    --done-wav sounds/done.wav \
    --timer-finished-wav sounds/timer_finished.wav \
    --timer-finished-wav-repeat 3 1
WorkingDirectory=/home/username/wyoming-satellite
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```
### Replace Satellite name in the name section and the wakeword name in preload model section with your custom wakeword name or use the standard ones such as 'ok_nabu', 'hey_jarvis' or 'hey_mycroft' (Skip step 18 and 19 if using a standard wakeword). The wakeword name should be the same as the one used while creating the OpenWakeWord service

81. Press the Control and X keys on your keyboard.
82. Press the Y key when prompted to save changes and then Press Enter to save and exit.
83. Create the VLC TTS Service.
```
sudo systemctl edit --force --full vlc-tts.service
```
84. Paste the following (replace the username in the below with the actual username used during Raspberry Pi OS Installation):
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
### Replace the password with your a password of choice in the telnet-password section.

85. Press the Control and X keys on your keyboard.
86. Press the Y key when prompted to save changes and then Press Enter to save and exit.
87. Make a directory for custom sounds.
```
sudo mkdir ~/custom-sounds
```
88. Set directory permissions.
```
sudo chmod -R 777 ~/custom-sounds
```
89. Navigate to the Wyoming Satellite sounds directory.
```
cd /home/username/wyoming-satellite/sounds
```
90. Copy the sounds files to the custom sounds directory.
```
sudo cp -R done.wav /home/username/custom-sounds/
```
```
sudo cp -R done.wav /home/username/custom-sounds/stt-stop.wav
```
91. Navigate to the custom sounds directory.
```
cd /home/username/custom-sounds
```
92. Make audio files executable to allow access by the Wyoming Satellite service.
```
sudo chmod -R 777 done.wav stt-stop.wav
```
93. Make scripts executable for to allow access by the Wyoming Satellite service.
```
sudo chmod -R 777 /home/username/wyoming-enhancements/snapcast/scripts/awake.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/snapcast/scripts/stt-stop.sh
```
```
sudo chmod -R 777 /home/username/wyoming-enhancements/snapcast/scripts/done.sh
```
```
sudo chmod -R 777 /home/username/wyoming-satellite/script/run
```
94. Edit the done.sh script to point to the correct custom sound directory
```
sudo nano /home/username/wyoming-enhancements/snapcast/scripts/done.sh
```
95. Replace 'pi' in the below line with the actual username used during Raspberry Pi OS Installation.
```
paplay --property=media.role=notification /home/pi/custom-sounds/done.wav 
```
96. Edit the stt-stop.sh script to point to the correct custom sound directory
```
sudo nano /home/username/wyoming-enhancements/snapcast/scripts/stt-stop.sh
```
97. Replace 'pi' in the last line with the actual username used during Raspberry Pi OS Installation.
```
paplay --property=media.role=notification /home/pi/custom-sounds/stt-stop.wav
```
98. Enable all the created services.
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
99. Reload the Systemd daemon.
```
sudo systemctl daemon-reload
```
100. Check the status of all the newly created services.
```
sudo systemctl status wyoming-satellite.service wyoming-openwakeword.service 2mic_leds.service pulseaudio.service snapclient.service vlc-tts.service bluetooth.service
```
### Do not worry if the log from pulseaudio.service and vlc-tts.service look like below as they'll work just fine.

<img width="1310" alt="pulse" src="https://github.com/user-attachments/assets/a569930a-88f7-4624-97cf-45280ba7eaf9">
<img width="1296" alt="vlc" src="https://github.com/user-attachments/assets/0da10dc6-dfe1-461d-8d59-660b5b7b052d">

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
2.
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
9. Paste the below (Replace the devicemacid in the last line with the actual MAC ID of the bluetooth speaker):
```
#!/bin/bash
pulseaudio --start
bluetoothctl power on
bluetoothctl connect devicemacid
```
11. Press the Control and X keys on your keyboard.
12. Press the Y key when prompted to save changes and then Press Enter to save and exit.
13. Make the script executable.
```
sudo chmod -R 777 bluetooth_startup.sh
```
14. Create a cronjob to run the script on startup.
```
sudo crontab -e
```
15. Add the below lines at the end:
```
@reboot sleep 30 && /home/username/bluetooth_startup.sh
```
16. Press the Control and X keys on your keyboard.
17. Press the Y key when prompted to save changes and then Press Enter to save and exit.
18. Reboot the Pi.
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

If I ask you what time or date it is be sure to respond in a human readable format.

If you don't have enough information to execute a smart home command then specify what other information you need.

If you successfully execute a smart home comand, do not respond at all.

If the user wants to control a device, do it.

Keep your responses short and concise.
```

## Credits:
This tutorial is just a combination of the separate ones made by Github users rhasspy (https://github.com/rhasspy/wyoming-satellite) and FutureProofHomes (https://github.com/FutureProofHomes/wyoming-enhancements).
