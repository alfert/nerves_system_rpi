# Raspberry Pi Model Zero

This is the base Nerves System configuration for the Raspberry Pi Zero and
Raspberry Pi Zero W.

![Fritzing Raspberry Pi Zero image](assets/images/raspberry-pi-model-zero.png)
<br><sup>[Image credit](#fritzing)</sup>

| Feature              | Description                     |
| -------------------- | ------------------------------- |
| CPU                  | 1 GHz ARM1176JZF-S              |
| Memory               | 512 MB                          |
| Storage              | MicroSD                         |
| Linux kernel         | 4.4 w/ Raspberry Pi patches     |
| IEx terminal         | OTG USB serial port (`ttyGS0`). Can be changed to HDMI or UART. |
| GPIO, I2C, SPI       | Yes - Elixir ALE                |
| ADC                  | No                              |
| PWM                  | Yes, but no Elixir support      |
| UART                 | 1 available - `ttyAMA0`         |
| Camera               | Yes - via rpi-userland          |
| Ethernet             | Yes - via OTG USB port          |
| WiFi                 | Pi Zero W or IoT pHAT           |
| Bluetooth            | Not supported yet               |

## Supported OTG USB modes

The base image activates the `dwc2` overlay, which allows the Pi Zero to appear as a
device (aka gadget mode). When plugged into a host computer via the OTG port, the Pi
Zero will appear as a composite ethernet and serial device.

When a peripheral is plugged into the OTG port, the Pi Zero will act as USB host, with
somewhat reduced performance due to the `dwc_otg` driver used in other base systems like
the official `nerves_system_rpi`.

## Supported WiFi devices

The base image includes drivers for the Red Bear IoT pHAT and the onboard
Raspberry Pi Zero W wifi module (`brcmfmac` driver).

If you are using another WiFi module (for example, a USB module), you will
need to create a custom system image. Before doing this, check if the
[nerves_system_rpi](https://github.com/nerves-project/nerves_system_rpi) works
better for you. That image configures the USB port in host mode by default and
is probably more appropriate for your setup.

## Installation

Add nerves_system_rpi0 to your list of dependencies in mix.exs:

def deps do [{:"nerves_system_rpi0", github: "tmecklem/nerves_system_rpi0", tag: "v0.11.1", runtime: false}] end

[Image credit](#fritzing): This image is from the [Fritzing](http://fritzing.org/home/) parts library.


## Bluetooth Support

Preparation: Comment out all entries from the `checksum` entry within `nerves.exs`
to prevent that after a change of the mentioned files a new docker container
is constructed!

### Linux Kernel configuration
Inside the docker container, run `make linux-menuconfig` and do the following:
* enable BT subsystem (as module)
* enable BT HCI UART drivers and the BROADCOM Protocol support (also enables UART H4 support)
* enable BT HCI USB driver support

Save the generated config-file with `make linux-savedefconfig` and copy it from
`build/linux-....` to `/nerves/env/nerves_system_rpi0/linux-4.4.defconfig` to store
in the git directory on the host.

### Buildroot configuration
Run `make menuconfig` and configure
* `bluez-utils 5.x`
* `OBIX`, `CLI`, `GATT` and experimental plugins

Save the generated config file with `make savedefconfig`.

Leave the docker container and run `mix` from the host directory.
