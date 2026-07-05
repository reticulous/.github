# Welcome to Reticulous!

**Reticulous** implements the reticulum mesh networking stack for ESP32 embedded systems/boards. It comes with a rich set of tools to do LXMF messenging and Nomad micron mesh browsing and implements some things (such as rnsh remote management) not normally found on reticulum nodes on embedded systems.

Reticulous is build on top of the brand new [**Spangap**](https://githib.com/spangap) *Device Application Framework*. In fact, it was developed in a parallel with it, yielding both a highly capable reticulum node and a demonstration of what Spangap can do. (In a nutshell, Spangap provides layers on top of the ESP32's RTOS/ESP-IDF that cover everything from facilities for more powerful multitasking, via a modern web-interface, a powerful CLI, SSH, all the way to making mobile-phone-style apps for small LCD screens, so makers of embedded firmware can start with a rich environment all ready to go.)

Back to Reticulous, which at the moment comes with ready-made hardware support modules (Spangap calls them 'straddles') for the **Heltec-v4 LoRa board** and the **LilyGO T-Deck Plus**. Without any special hardware support, the generic version will have no mesh radio, but will still do reticulum mesh over WiFi and let you use the system via a browser or the command line interface, accessible via serial and SSH. It should run on any ESP32-S3 hardware provided it has some amount of PSRAM and flash - 4MB or more of PSRAM and 8MB or more of flash would be ideal. Support for the many other ESP32-S3 boards with LoRa radios and/or LCD/touch screens can be built and is forthcoming.

## Getting started

At this very early stage of their development, both Spangap and Reticulous still lack some of the documentation they will have soon. But to get you started if you can't wait for everything to be finished, here's a very quick description of how to build and flash the software.

The procedure below should work for Linux, Mac and possibly also for Windows Subsystem for Linux (the latter untested as of now). Let's say you are in your homedir on a Mac and want to build the Reticulous mesh networking software that relies on Spangap and run it on the LilyGo T-Deck Plus device connected to port `/dev/cu.usbmodem1101` on your Mac. You'll first need to install the `spangap` command line tool somewhere that is in your `$PATH`. Let's say we put it in `~/bin` which is already in your `$PATH` and you'd like the workspace directory to be called `tdeck`, directly below your homedir. In a terminal window, enter:

```sh
curl -o ~/bin/spangap https://raw.githubusercontent.com/spangap/spangap/spangap && \
chmod a+x ~/bin/spangap && \
spangap init tdeck && \
cd tdeck && \
spangap monitor /dev/cu.usbmodem1101
```

(If system doesn't seem to know about spangap, enter `rehash` and/or try last three lines again from a new terminal window.)

Leave `spangap monitor` running, it will show you the device log output and once we're done will allow you to switch to CLI mode by simply typing a command.

Now open another terminal window and do:

```sh
cd tdeck && \
spangap build reticulous/reticulous --with spangap/hw-tdeck && \
spangap flash
```

*(Replace that `hw-tdeck` with `hw-heltecv4` to build for that board, or leave off `--with ...` to get the generic build that works on any ESP32-S3 with at least 4MB of PSRAM and 8MB of flash.)*
