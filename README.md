# ESPHome on Air Cube (over Thread)

Testing esphome support for the wonderful project at https://github.com/StuckAtPrototype/AirCube
As this project uses an ESP32-H2, only Bluetooth and Zigbee/Thread radios are supported.

This repo aims to make the Air Cube work over Thread using the esphome api.

# Quick Config
> [!NOTE]
> The key difference with this config is the need to provide the thread TLV - which you usually don't need to do with wifi enabled ESP devices.
> This config is also using a forked implementation of the [ens160](https://esphome.io/components/sensor/ens160/) component as the upstream component does not support the ens161.
> Once the air cube boots up, the firmware needs about 3 minutes to calibrate before it starts updating its color based on the environment.
 
```yaml
packages:
  air_cube: github://imondrag/esphome-air-cube/packages/core-air-cube.yaml
esphome:
  friendly_name: Air Cube
logger:
  level: INFO
api:
  encryption:
    key: !secret api_key
ota:
  - platform: esphome
    password: !secret ota_password
# Thread Setup
network:
  enable_ipv6: true
openthread:
  tlv: !secret thread_tlv
```

# Getting started

Assumed Knowledge:
- You've setup an ESPHome device on Home assistant

You'll need:
- Air cube
- Thread Border Router (ie: ZBT-2, Google Streamer, Apple TV / Homepod)
  - May also need a mobile phone running Home Assistant companion app to fetch the Thread credentials
- IPv6 enabled network
- Home Assistant with ESPHome Builder

# Finding your Thread TLV

You can use the guide linked below, but I didn't have any luck finding the TLV using the Home Assistant UI.
HOWEVER, I found the TLV in the HA file system at `/config/.storage/thread.datasets`. 

> [!NOTE]
> You may need to force a sync from the mobile app.
> Go to Settings > Companion app > Troubleshooting > Sync Thread credentials 

You can then use the command line to print the TLV. It's probably easiest to use the CLI provided with the [VS Code add-on](https://github.com/hassio-addons/addon-vscode/blob/main/vscode/DOCS.md).

```
$ cat /config/.storage/thread.datasets 
{
  "version": 1,
  "minor_version": 4,
  "key": "thread.datasets",
  "data": {
    "datasets": [
      {
        "created": "<snip>",
        "id": "<snip>",
        "preferred_border_agent_id": "<snip>",
        "preferred_extended_address": "<snip>",
        "source": "Google",
        "tlv": "<snip, but this is the really long string that you need>"
      }
    ],
    "preferred_dataset": "<snip>"
  }
}
```

Once you have the TLV, you should be able to flash the device locally and then restart it. It should appear as an adoptable esphome device once it boots up again!

# Home Assistant
Home assistant UI after adoption:
<p align="center">
  <img width="674" height="839" alt="image" src="https://github.com/user-attachments/assets/fe3bb9a3-669d-4c20-a74e-e7d946bcd80d" />
</p>

# Resources
- HA guide to get thread working on ESP32-H2: https://community.home-assistant.io/t/thread-working-on-esp32-h2-example-project/903960
