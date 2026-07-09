# Welcome to Reticulous!

**Reticulous** implements the reticulum mesh networking stack for ESP32 embedded systems/boards. It comes with a rich set of tools to do LXMF messaging and Nomad micron mesh browsing and implements some things (such as rnsh remote management) not normally found on reticulum nodes on embedded systems.

Reticulous is built on top of the brand new [**Spangap**](https://github.com/spangap) *Device Application Framework*. In fact, it was developed in parallel with it, yielding both a highly capable reticulum node and a demonstration of what Spangap can do. (In a nutshell, Spangap provides layers on top of the ESP32's RTOS/ESP-IDF that cover everything from facilities for more powerful multitasking, via a modern web-interface, a powerful CLI, SSH, all the way to making mobile-phone-style apps for small LCD screens, so makers of embedded firmware can start with a rich environment all ready to go.)

Back to Reticulous, which at the moment comes with ready-made hardware support modules (Spangap calls them 'straddles') for the **Heltec-v4 LoRa board** and the **LilyGO T-Deck Plus**. Without any special hardware support, the generic version will have no mesh radio, but will still do reticulum mesh over WiFi and let you use the system via a browser or the command line interface, accessible via serial and SSH. It should run on any ESP32-S3 hardware provided it has some amount of PSRAM and flash - 4MB or more of PSRAM and 8MB or more of flash would be ideal. Support for the many other ESP32-S3 boards with LoRa radios and/or LCD/touch screens can be built and is forthcoming.

## Project Overview

Everything is packaged as *straddles* — Spangap's self-contained modules, each bundling an ESP-IDF firmware component and (where relevant) a browser app. A Reticulous build layers the mesh stack and its apps on top of the Spangap platform straddles, which provide the runtime, networking, web UI and LCD shell.

**The Spangap platform (see the [Spangap overview](https://github.com/spangap) for detail):**

- **`spangap`** — the build system and CLI: resolves dependencies, builds inside Docker, generates the boot glue, flashes and monitors.
- **`spangap-core`** — the base runtime: **`init`** (boot dispatcher), **`storage`** (device↔browser value store), **`its`** (the poll-free inter-task IPC everything rides on), **`fs`** (flash/SD filesystem), **`cli`**, **`auth`**, **`cron`**, **`logging`**, **`power-management`**, **`memory`** and **`ota`**.
- **`spangap-net`** — WiFi/IP: **`net`**, **`tls`**, **`ntp`**, **`mdns`**, plus optional remote-access services (**`acme`** certs, **`duckdns`**, **`upnp`**, **`wg`** WireGuard, **`sshd`**).
- **`spangap-web`** — the **`web`** HTTPS/WebDAV server, **`webrtc`** DataChannel plumbing and the browser-shell SPA that folds every straddle's UI into one web app.
- **`spangap-lcd`** — the phone-style LVGL UI (launcher, Settings/Log/CLI apps, VT100 terminal) that Reticulous draws its screens on.

**The Reticulous straddles:**

- **`reticulous`** — the buildable itself. It carries no firmware or board code, but ships the browser SPA, the LittleFS data image and the LCD icons, and pulls in the whole family below plus the Spangap platform. You build it *with* a board.
- **`rns`** — the Reticulum stack, centered on the `rnsd` task that owns the entire protocol engine: identity, destinations, the path table, the `Transport` state machine, `Link`s, `Resource`s and reliable in-order `Channel`s. It has no radio or IP of its own — interfaces plug in over `its`, and apps reach it through a byte-array C API.
- **`iface-tcp`** — Reticulum over plain TCP/IP (outbound peers plus an inbound server), HDLC-framed to interop with desktop Reticulum.
- **`iface-lora`** — Reticulum over LoRa radios via RadioLib (SX126x / SX127x / SX128x / LR11x0 / LR2021), up to four on one SPI bus.
- **`iface-espnow`** — Reticulum over Espressif ESP-NOW using the long-range PHY, one RNS packet per frame.
- **`iface-auto`** — zero-config AutoInterface over the local WiFi LAN, wire-compatible with upstream Reticulum's `AutoInterface`.
- **`lxmf`** — the LXMF messaging mailbox: sends/receives signed messages, holds up to four identities, pays/enforces PoW stamps, and interoperates wire-for-wire with Sideband, NomadNet and MeshChat.
- **`nomad`** — a Nomad Network "text web" client for browsing Micron pages and files hosted on `nomadnetwork.node` destinations over Reticulum Links.
- **`rnsh`** — a Reticulum remote shell (server *and* client) that bridges an already-encrypted mesh Channel into the device `cli` — the mesh-native analogue of SSH.
- **`maps`** — an offline slippy-map viewer on the LCD, blitting pre-baked, GPS-centered map tiles from SD (not RNS-specific; it ships here as a feature straddle).

**Board support** (the `--with` target, provided as Spangap-org straddles):

- **`spangap/hw-tdeck`** — LilyGO T-Deck Plus: LoRa (SX1262), 320×240 LCD, QWERTY keyboard, trackball, GNSS and microSD.
- **`spangap/hw-heltecv4`** — Heltec V4 LoRa board (headless).

## Getting started

### Flashing

It is really early days. Some stuff doesn't work yet., some stuff looks ugly. You're literally one of the very first users. If you just want to flash a unit, click a link below:

* [T-Deck Plus](https://reticulous.net/flasher?build=tdeck)
* [Heltec V4](https://reticulous.net/flasher?build=heltecv4)
* [Generic - No LoRa](https://reticulous.net/flasher?build=generic)

### Building

But much more interesting than just flashing your device is to install `spangap` and compile for yourself.

At this very early stage of their development, both Spangap and Reticulous still lack some of the documentation they will have soon. But to get you started if you can't wait for everything to be finished, here's a very quick description of how to build and flash the software. To do this, you must have `docker` installed and running, and you must have `python`, `git` and `curl`. Nothing will be installed on your system outside of the spangap project directory you initialize. 

The procedure below should work for Linux, Mac and possibly also for Windows Subsystem for Linux (the latter untested as of now). Let's say you are in your homedir on a Mac and want to build the Reticulous mesh networking software that relies on Spangap and run it on the LilyGo T-Deck Plus device connected to port `/dev/cu.usbmodem2101` on your Mac. You'll first need to install `spangap`. For this, you do:

```sh
curl -fsSL https://spangap.org/install.sh | sh
```

Then create the workspace and start a serial monitor: 

```sh
spangap init reticulous && \
cd reticulous && \
spangap monitor /dev/cu.usbmodem2101
```

Leave `spangap monitor` running, it will show you the device log output and once we're done will allow you to switch to CLI mode by simply typing a command.

Now open another terminal window and do:

```sh
cd reticulous && \
spangap build reticulous/reticulous --with spangap/hw-tdeck && \
spangap flash
```

*(Replace that `hw-tdeck` with `hw-heltecv4` to build for that board, or leave off `--with ...` to get the generic build that works on any ESP32-S3 with at least 4MB of PSRAM and 8MB of flash.)*
