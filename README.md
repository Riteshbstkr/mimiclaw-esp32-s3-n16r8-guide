# MimiClaw on a Generic ESP32-S3 N16R8 (Windows 10)

A beginner-friendly record of installing [MimiClaw](https://github.com/memovai/mimiclaw) on a generic ESP32-S3 N16R8 board from Windows 10, using only a web browser.

> **Project status:** The board successfully booted MimiClaw and started its Wi-Fi setup portal. The project was intentionally stopped before AI-provider configuration. MimiClaw currently needs an API key from either OpenAI or Anthropic (Claude) for its AI responses; it does not run a powerful chat model entirely on the ESP32-S3.

## What this guide covers

* Generic ESP32-S3-WROOM-1 **N16R8** development board (16 MB flash, 8 MB PSRAM)
* Windows 10
* Browser-based flashing with [Espressif's ESP Tool](https://espressif.github.io/esptool-js/)
* MimiClaw v0.1.1
* Fixing a successful flash that initially did not show a `MimiClaw-...` Wi-Fi network

## Hardware used

The board used for this guide has two USB-C ports and an ESP32-S3-WROOM-1 N16R8 module.

| Label on board | Typical use |
| --- | --- |
| `USB` | Native USB / JTAG; usually the best port for browser flashing |
| `COM` | UART serial console / interactive command prompt |

Use a **data-capable** USB-C cable. A charging-only cable will not work.

## Before you start

1. Use Chrome or Edge on Windows.
2. Connect the board with the `USB` port first.
3. Open [ESP Tool](https://espressif.github.io/esptool-js/).
4. Click **Connect** and choose the ESP32-S3 serial device.

If Windows shows a port but the browser cannot connect, try another data cable and then try the other USB-C port. Do not erase the flash unless you deliberately want to remove the firmware and saved settings.

## Confirm the board

After connecting, ESP Tool reported:

```
Connected to device: ESP32-S3 (QFN56) (revision v0.2)
```

This confirms it is an ESP32-S3. The `N16R8` marking means 16 MB flash and 8 MB PSRAM, which meets MimiClaw's stated hardware requirement.

## Reliable flashing method

### Why use separate files?

Flashing one combined firmware image completed successfully but did not result in a visible MimiClaw Wi-Fi network on this generic board. Flashing the official release images at their individual addresses produced a normal boot and the setup access point.

Download the following MimiClaw **v0.1.1** release files from the project's Releases page:

| Flash address | File |
| ---: | --- |
| `0x0` | `bootloader-v0.1.1.bin` |
| `0x8000` | `partition-table-v0.1.1.bin` |
| `0xf000` | `ota_data_initial-v0.1.1.bin` |
| `0x20000` | `mimiclaw-v0.1.1.bin` |
| `0x420000` | `spiffs-v0.1.1.bin` |

### Flash in the browser

1. In ESP Tool, click **Connect**.
2. Keep Flash Mode, Flash Frequency, and Flash Size set to `keep`.
3. For each row above, enter the flash address, choose the matching file, then click **Add File** for the next row.
4. Do **not** select **Erase all**.
5. Click **Program** and wait for completion.
6. A good result ends with lines similar to:

   ```
   Hash of data verified.
   Leaving...
   Hard resetting via RTS pin...
   ```

7. Disconnect and reconnect the board, or press `RST` once.

## What a healthy boot looks like

On the serial console, the normal startup eventually shows a MimiClaw banner and lines similar to:

```
SPIFFS: total=11378081, used=7028
Type 'help' to get the list of commands
DHCP server assigned IP to a client
```

The early boot text below is normal, but by itself does not prove that MimiClaw has started:

```
ESP-ROM:esp32s3-20210327
rst:0x1 (POWERON_RESET),boot:0x8 (SPI_FAST_FLASH_BOOT)
entry 0x403c8948
```

For interactive commands, use the port labelled `COM` at 115200 baud. The `USB` port is normally used for native USB/JTAG and browser flashing.

## Wi-Fi onboarding

1. On a phone or computer, look for a Wi-Fi network named similar to **`MimiClaw-XXXX`**.
2. Join it.
3. Open **http://192.168.4.1** in a browser.
4. Enter your normal 2.4 GHz home Wi-Fi name and password.

The setup page also asks for a model provider and API key. At the time of writing, the released MimiClaw firmware supports only:

* OpenAI
* Anthropic (Claude)

It uses one of these cloud services as the language-model brain. ChatGPT Plus and a Claude chat subscription do not automatically include API use. Do not post an API key in an issue, a GitHub repository, screenshot, or chat message.

If you do not want to use either provider, stop at this point. The board and portal are working, but MimiClaw cannot answer messages without a supported AI provider.

## Troubleshooting

### Flashing finishes, but no MimiClaw Wi-Fi network appears

1. Wait 30 seconds after reset.
2. Verify you flashed all five files at the addresses in the table.
3. Re-flash the five separate files; do not rely only on a combined image.
4. Check the serial output. Repeating only the ROM boot lines means the main app has not yet provided useful startup information.
5. Try the alternate USB-C port for the serial console.

### The browser says no port was selected

Click **Connect**, then select the device in the Windows browser permission dialog. If no device appears, change the USB cable before changing anything else.

### I can see boot text but cannot type commands

Move the console cable to the port marked `COM`. The ESP32-S3 board's UART console is normally on that port.

### I joined the setup Wi-Fi but cannot open the page

Use exactly: [http://192.168.4.1](http://192.168.4.1)

Mobile devices may warn that the network has no internet. Keep the connection anyway; it is a local setup network.

## Useful references

* [MimiClaw source, requirements, and releases](https://github.com/memovai/mimiclaw)
* [ESP Tool browser flasher](https://espressif.github.io/esptool-js/)
* [Espressif ESP32-S3 documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/)

## Safety and privacy

* This is low-voltage USB hardware, but do not short the header pins while it is powered.
* Keep API keys private. Treat them like a password that can spend money.
* If a key is ever exposed, revoke it with the provider and create a new one.

## License

This guide is released under the MIT License. MimiClaw itself is a separate project; see its repository for its own license and terms.
