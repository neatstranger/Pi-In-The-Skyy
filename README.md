# Pi In The Skyy 

A PoE powered camera based on the Raspberry Pi Compute Module 4.
<br>

<img src="/Assets/PCB-Front.jpg" alt="drawing" width="400"/> <img src="/Assets/PCB-Back.jpg" alt="drawing" width="425"/> 

----

## Specs Overview
- Dual 22 Pin 0.5mm Pitch Camera Inputs
- Passive PoE In (8-60V)
- HDMI / UART Connectors For Troubleshooting
- Micro SD Card Slot


## Enclosure
The main body of the enclosure is designed to be 3D printed out of ASA, but PLA/PETG would work just as well. Acrylic dome found on Amazon.

<img src="/Assets/Housing-3D-View.png" alt="drawing" width="800"/> 


## Camera Mounting Hole Compatability
As far as I can tell, any of the first party v1, v2, and v3 cameras should work with the existing mounting plate. HQ/GS will need a new plate.


## Flexible Printed Circuit (FPC) Cable
You will need a 22 pin to 15 pin adapter cable. This is the same cable you would need to attach a first party camera to a RPi Zero, CM4IO board, or RPi5
[15 Way 1mm Spacing to 22 Way 0.5mm Spacing](https://www.pishop.us/product/camera-cable-for-raspberry-pi-5/)


## Power In
**Passive** POE IN 8-60V        Positive(3,6) (4,5)  Negative(7,8) (1,2) <br>
I have tons of passive PoE injectors from Ubiquiti equipment laying around, so I grabbed one like this:
[PoE+ Ubiquiti Injector](https://www.bhphotovideo.com/c/product/1737953-REG/ubiquiti_networks_u_poe_at_802_3at_supported_poe_injector.html/?ap=y&ap=y&smp=y&smp=y&lsft=BI%3A5451&gad_source=1&gclid=Cj0KCQjwwuG1BhCnARIsAFWBUC2Srk6iD-r-BquvTg5gMVnUvmvvXXQUebGMEcndWcXucgtnzZV4jrsaAs08EALw_wcB)

All of the pairs don't have to exactly match, this one is +(4,5) , -(7,8). As long as you don't cross the expected negative and positive it should work just fine. Most PoE injectors on Amazon or otherwise will list the pinout in the description. 


Make sure you purchase one that is passive though. 
Active PoE requires components on the baord to negotiate power delivery. Which this carrier board doesn't have. We are just pulling the voltage from the center tap of the magnetics.



## Complete Part List
- [Raspberry Pi CM4](https://rpilocator.com/?cat=CM4)
- [v1, v2, v3 Camera Module](https://www.pishop.us/product/raspberry-pi-camera-module-3-wide/)
- [15 to 22 Pin Camera Cable](https://www.pishop.us/product/camera-cable-for-raspberry-pi-5/)
- Carrier Board has no source, but production files are ready to order from jlcpcb.com (Cost is ~$250 / 5 Boards)
- 3D Printed Case
- [100mm Acrylic Dome](https://www.amazon.com/dp/B07L6GLTNP?psc=1&ref=product_details)
- Various Fasteners and Heat Set Inserts. M3 for Most of Assembly, M2 For Camera Module
- [Gland For Cable Passthrough](https://www.mcmaster.com/5302N121/)


## Systemd Service for Streaming `/etc/systemd/system/camera-stream.service`
```
[Unit]
Description=open-source-nvr
Wants=network-online.target
After=network-online.target

[Service]
User=<user>
Group=<user>
Type=simple
ExecStart=/bin/sh /var/camstart.sh

[Install]
WantedBy=multi-user.target
```
^^^ Make sure to replace <user> with your login account.

## `/var/camstart.sh`
```
#!/bin/sh
rpicam-vid -t 0 --width 1920 --height 1080 --inline --nopreview --rotation 180 -o - | cvlc stream:///dev/stdin --sout '#rtp{sdp=rtsp://:8554/stream1}' :demux=h264

```


## After Creating both Files
```
sudo chmod +x /var/camstart.sh # make the service executable
sudo systemctl daemon-reload #reload the system file
sudo systemctl start camera-stream #start the camera stream
sudo systemctl enable camera-stream
```
