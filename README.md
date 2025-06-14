# Mini OS

Mini OS is a lightweight launcher for the Pico‑8 fantasy console. It runs on a small 128×128 ST7735 display attached to a Raspberry Pi and offers a simple menu for basic device tasks.

The interface provides a **Settings** screen with a **Display** submenu for changing brightness, font, color scheme and text size. Utilities show the current date, time, system information and network details. You can also reboot or shut down the Pi.

Built‑in games, the image gallery and note taking features have been removed to keep the focus on Pico‑8 and these small utilities.

The color scheme selector includes Mac&nbsp;Terminal palettes such as **Terminal Basic**, **Grass**, **Man Page**, **Novel**, **Ocean**, **Red Sands**, **Espresso**, **Homebrew** and **Pro** in addition to the default theme. Your last selected scheme is written to `settings.json` so it loads the same palette after a reboot.

## Setup on Raspberry Pi OS Lite (32-bit)

Install the required packages and enable the SPI interface:

```bash
sudo apt-get update
sudo apt-get install python3-pip python3-rpi.gpio fonts-dejavu-core
sudo pip3 install -r requirements.txt
sudo raspi-config nonint do_spi 0
```

Reboot after enabling SPI so the display can be accessed by the script.

Download Pico‑8 for Raspberry Pi from <https://www.lexaloffle.com/pico‑8.php> and extract the binary somewhere accessible, for example `/opt/pico8`. The launcher expects the command `pico8` to be on your `PATH` or available via the `PICO8_PATH` environment variable.

### Running Pico‑8

Once Pico‑8 is installed you can start it from the main menu. The launcher simply executes the `pico8` command with `-width 128 -height 128` so the output fits the small screen. Your preferred cartridge can be specified with `-run <cart>`.

## Pin Assignments

Mini OS uses BCM GPIO numbers. Connect the Waveshare 1.44" ST7735 display and buttons as follows:

### ST7735 Display
- **RST** - GPIO27
- **DC** - GPIO25
- **CS** - GPIO8
- **MOSI** - GPIO10
- **SCLK** - GPIO11
- **Backlight** - GPIO24

### Buttons and Joystick (active LOW)
- **KEY1** - GPIO21
- **KEY2** - GPIO20
- **KEY3** - GPIO16
- **Joystick Up** - GPIO6
- **Joystick Down** - GPIO19
- **Joystick Left** - GPIO5
- **Joystick Right** - GPIO26
- **Joystick Press** - GPIO13

## Running as a `systemd` Service

1. Copy `mini_os.service` to `/etc/systemd/system/` (or `~/.config/systemd/user/` for a user service).
2. Adjust the paths in the unit file if you place the project somewhere other than `/opt/mini_os`.
3. Reload systemd and enable the service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable mini_os.service
   sudo systemctl start mini_os.service
   ```

The service definition will start the program on boot and restart it automatically if it exits unexpectedly.

## NYT Top Stories

The menu can fetch headlines from the New York Times API. Copy `nyt_config.py.example` to `nyt_config.py` and add your API key. The file is in `.gitignore` so your key stays local.

## Web Interface

A lightweight web server can be started from the **Utilities** menu. It exposes
a simple browser based interface for viewing and updating settings. From the
`/settings` page you can change the display brightness, select a font and adjust
the text size. Wi-Fi can also be toggled on or off directly from the browser.

The server requires the Python packages listed in `requirements.txt`
(including **Flask** and **pexpect**). Install them with `pip3 install -r
requirements.txt` and then either select **Web Server** from the Utilities menu
or run `python3 utilities/web_server.py` manually. Once running, visit
`http://<Pi-IP>:8000` in your browser.

### Shell (`/shell`)

The web interface exposes a full interactive shell using WebSockets and a
browser-based terminal emulator. Type commands directly in the page and see
their output immediately, just like a normal terminal. Commands run as the same
user that started the server. If a command uses `sudo` and a password is
required, it can be entered within the terminal. Because anyone who can access
this page can execute commands on your Pi, enable the web server only on trusted
networks or behind a firewall.

### Interactive Shell

Opening `/shell` in a browser now presents a full terminal emulator powered by
WebSockets. The prompt is displayed on a black background with bright green
text and a blinking cursor to mimic the look of a native shell. Commands are
executed in a persistent Bash process so each one builds on the previous.
Type directly into the page and see output appear in real time. If a command
asks for a password (for example when using `sudo`) you can provide it right in
the terminal.

**Security Warning:** anyone who can access this page can run arbitrary commands
on your Pi. Only enable the web server on trusted networks and consider adding
additional authentication if it is exposed beyond localhost.

## Wi-Fi

Choose **Wi-Fi Setup** from the Settings menu to scan for nearby
networks. Select one and press **KEY3** to connect. Set the environment
variable `MINI_OS_WIFI_PASSWORD` before starting Mini OS so `nmcli` can
use it to connect without prompting for the password.

## Bluetooth

From **Settings** choose **Bluetooth** to open the bluetooth menu. The menu has
two options:

1. **Discover devices** – performs a scan and lists all nearby devices. Select
   a device with the joystick. Press **KEY1** to attempt a direct connection or
   **KEY2** to pair and connect (useful for phones that require confirming a
   passkey such as the iPhone 15 Pro Max).
2. **Pairing mode** – places the Mini‑OS into discoverable and pairable mode so
   other devices can initiate a connection. Pairing mode listens for incoming
   connections until one succeeds or the user exits the screen.

Bluetooth devices generally need to be paired before a connection will
succeed. Pairing establishes a trusted relationship and the plain `connect`
command often fails if the device was never paired. After pairing, you may
need to run `trust <MAC>` inside `bluetoothctl` so the device reconnects
automatically in the future.

Scanning now falls back to `bluetoothctl` if `hcitool` is unavailable and
connection failures will display the full output from `bluetoothctl` so you can
see exactly why a device did not connect. Each failure is also saved in the
`notes` directory as `btfail1.txt`, `btfail2.txt` and so on for later review.
While in pairing mode the system now watches the bluetooth service logs for
incoming connection attempts. If a remote device fails to connect, the last
log messages are written to `connectfail1.txt`, `connectfail2.txt`, etc. inside
the `notes` directory for troubleshooting.

The Bluetooth device list now uses a smaller font so long names fit on the
screen without running off the edge. Device names are displayed before their
Bluetooth addresses for quicker identification.
Long device names are wrapped onto multiple lines so the full text remains
readable.


The Utilities menu also includes **Shell**, which opens the on-screen keyboard
so you can type a command and execute it on the Pi. After a command is
submitted the keyboard hides so the output is easier to read. Press **KEY1** to
show the keyboard again and select a character, **KEY2** to delete the last
character and **KEY3** to run the command. Holding **KEY3** still exits back to
the menu.

There's also a lightweight **Console** option that keeps a persistent Bash
session running and logs all output to `logs/console.log`. It starts with the
keyboard hidden so more of the 128×128 display can show command output. The
bottom button hints are removed to maximize text area and a blinking cursor
shows where input will appear. Short presses last under one second while long
presses are held longer. The button functions are:

`1S` select highlighted character, `1L` backspace,
`2S` page characters, `2L` hide keyboard,
`3S` tab autocomplete and `3L` exit the console.
Press **KEY1** to reveal the keyboard when hidden.

