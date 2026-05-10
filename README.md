# homebridge-onkyo-26

Homebridge platform plugin for controlling Onkyo and Onkyo-compatible AV receivers as HomeKit TV accessories.

This fork is maintained at:

https://github.com/fredsocial/homebridge-onkyo-26

## What It Does

`homebridge-onkyo-26` exposes each configured receiver or receiver zone to HomeKit as a TV-style accessory. Depending on your receiver and configuration, HomeKit can control:

- Receiver power.
- Active input/source.
- Volume.
- Mute.
- Basic remote-control buttons such as arrows, select, back, play/pause, rewind, fast-forward, previous, next, and information/home.
- Optional extra volume control exposed as a Lightbulb dimmer or Fan speed service.
- Main zone and, where supported by the receiver, Zone 2.

The plugin talks to receivers over the Integra Serial Communication Protocol, also known as ISCP/eISCP. The bundled `eiscp` library and command database are included in this repository because the original library is no longer maintained on npm.

## Requirements

- Homebridge `>= 1.4.0`.
- Node.js `>= 14.19.1`.
- An Onkyo, Integra, or compatible receiver that supports network control through ISCP/eISCP.
- The receiver and Homebridge host must be on the same network.
- Network standby/network control should be enabled on the receiver if you want reliable power-on behavior.
- A reserved/static IP address for the receiver is strongly recommended.
- The receiver should already be controllable from the Onkyo mobile app or another network-control app before configuring this plugin.

## Supported Models

The model list comes from `eiscp/eiscp-commands.json` and is surfaced in `config.schema.json` for Homebridge UI configuration.

The default model is `TX-NR609`. If your exact model is not listed, try `TX-NR609` first. Many receivers share the same ISCP command set, so the closest matching model is often enough.

## Installation

### Install From GitHub

Until this fork is published on npm, install it directly from GitHub:

```bash
npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

On macOS, if global npm installs fail with `EACCES`, use:

```bash
sudo npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

If you are running Homebridge in Docker, install the plugin inside the Homebridge container, not on your Mac host.

### Replace The Original Plugin

If you previously installed `homebridge-onkyo`, remove it before installing this fork:

```bash
npm uninstall -g homebridge-onkyo
npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

The package name is `homebridge-onkyo-26`, but the Homebridge platform alias remains `Onkyo`, so your config should still use:

```json
"platform": "Onkyo"
```

### Homebridge UI Search

Homebridge's Plugins search page searches npm. A GitHub-only plugin will not appear in search until it is published to npm.

To make it searchable in Homebridge UI, publish this package to npm as `homebridge-onkyo-26`, then search for that name in the Homebridge Plugins page.

## Configuration

Add this platform to the `platforms` array in your Homebridge `config.json`.

### Minimal Configuration

```json
{
  "platform": "Onkyo",
  "receivers": [
    {
      "name": "Receiver",
      "ip_address": "10.0.0.46",
      "model": "TX-NR609"
    }
  ]
}
```

### Full Example

```json
{
  "platform": "Onkyo",
  "receivers": [
    {
      "name": "Receiver",
      "ip_address": "10.0.0.46",
      "model": "TX-NR609",
      "zone": "main",
      "poll_status_interval": "3000",
      "default_input": "net",
      "default_volume": 10,
      "max_volume": 40,
      "map_volume_100": false,
      "volume_type": "none",
      "filter_inputs": true,
      "inputs": [
        {
          "input_name": "dvd",
          "display_name": "Blu-ray"
        },
        {
          "input_name": "video2",
          "display_name": "Switch"
        },
        {
          "input_name": "video6",
          "display_name": "Apple TV"
        },
        {
          "input_name": "cd",
          "display_name": "TV/CD"
        }
      ]
    }
  ]
}
```

### Multiple Receivers Or Zones

You can add multiple objects to `receivers`. Each entry can represent a separate receiver or a supported zone on the same receiver.

```json
{
  "platform": "Onkyo",
  "receivers": [
    {
      "name": "Living Room Receiver",
      "ip_address": "10.0.0.46",
      "model": "TX-NR609",
      "zone": "main"
    },
    {
      "name": "Patio Speakers",
      "ip_address": "10.0.0.46",
      "model": "TX-NR609",
      "zone": "zone2"
    }
  ]
}
```

## Configuration Reference

| Field | Required | Default | Description |
| --- | --- | --- | --- |
| `platform` | Yes | `Onkyo` | Must be exactly `Onkyo`. |
| `receivers` | Yes | none | Array of receiver or receiver-zone configurations. |
| `name` | Yes | `Receiver` | Name shown in HomeKit. |
| `ip_address` | Yes | `10.0.0.46` in the UI schema | Receiver IP address. Use a reserved/static address. |
| `model` | Yes | `TX-NR609` | Receiver model or closest compatible model. |
| `zone` | No | `main` | Zone to control. Supported schema values are `main`, `zone2`, `zone3`, and `zone4`; runtime command mapping is implemented for `main` and `zone2`. |
| `poll_status_interval` | No | `0` | Status polling interval. Polling is enabled only when the parsed value is greater than `10` and less than `100000`. The current runtime multiplies this value by `1000` when creating pollers. |
| `default_input` | No | none | Input to select after powering on through HomeKit. Must be a valid receiver input label such as `net`, `dvd`, `video2`, or `cd`. |
| `default_volume` | No | none | Receiver volume to set after powering on through HomeKit. This is the receiver's actual volume number, not a HomeKit percentage. |
| `max_volume` | No | `30` | Maximum receiver volume allowed when controlling volume from HomeKit. |
| `map_volume_100` | No | `false` | When `true`, maps HomeKit's 0-100 volume scale to `max_volume`. |
| `inputs` | No | all supported model inputs | Optional input mappings. Use this to rename inputs in HomeKit. |
| `filter_inputs` | No | `false` | When `true`, only inputs listed in `inputs` are exposed to HomeKit. |
| `volume_type` | No | `none` | Optional extra volume service. Use `dimmer`, `speed`, or `none`. |

## Inputs

The plugin builds the available input list from the selected model's command set. You can optionally rename inputs for HomeKit.

```json
"inputs": [
  {
    "input_name": "video6",
    "display_name": "Apple TV"
  }
]
```

With `filter_inputs: false`, the plugin exposes all inputs supported by the selected model and applies your custom names where they match.

With `filter_inputs: true`, the plugin exposes only the inputs listed in `inputs`.

Common input names include:

- `dvd`
- `bd`
- `cbl`
- `sat`
- `game`
- `video1` through `video7`
- `cd`
- `tv`
- `tuner`
- `fm`
- `am`
- `net`
- `network`
- `usb`
- `phono`

For receiver-specific input support, check the output from the bundled eISCP examples or inspect `eiscp/eiscp-commands.json`.

## Volume Behavior

### Standard TV Speaker Volume

The plugin always creates a HomeKit `TelevisionSpeaker` service linked to the TV accessory. It supports:

- Absolute volume.
- Relative volume up/down.
- Mute on/off.

### `max_volume`

When `map_volume_100` is `false`, HomeKit volume values are treated as actual receiver volume numbers and capped at `max_volume`.

Example:

```json
"max_volume": 40,
"map_volume_100": false
```

If HomeKit tries to set volume above `40`, the plugin sends `40`.

### `map_volume_100`

When `map_volume_100` is `true`, HomeKit volume is treated as a percentage of `max_volume`.

Example:

```json
"max_volume": 40,
"map_volume_100": true
```

- HomeKit `100` sends receiver volume `40`.
- HomeKit `50` sends receiver volume `20`.
- HomeKit `25` sends receiver volume `10`.

### Extra Volume Service

`volume_type` can expose a second control surface for volume:

- `none`: no extra volume service.
- `dimmer`: exposes volume as a Lightbulb brightness control.
- `speed`: exposes volume as a Fan rotation speed control.

These extra services can be useful for automations, scenes, Siri, or Alexa integrations, but HomeKit may describe them as lights or fans because that is how the service is represented.

## Power-On Defaults

When HomeKit powers on a receiver, the plugin can optionally send:

- `default_volume`
- `default_input`

These defaults apply only when the receiver is powered on through HomeKit. If the receiver is powered on from the physical knob, remote control, or Onkyo app, the plugin may not apply those startup defaults.

## Polling

By default, polling is disabled.

When enabled, polling asks the receiver for current power, input, mute, and volume state. This helps HomeKit stay in sync if the receiver is controlled outside HomeKit.

Example:

```json
"poll_status_interval": "3000"
```

Important: the current runtime treats this value as a number and then multiplies it by `1000` before creating pollers. Keep that in mind when tuning polling frequency.

## HomeKit Notes

- The receiver is exposed as an external HomeKit TV accessory.
- You may need to pair the accessory in HomeKit after the plugin starts.
- Input names shown in HomeKit come from the receiver command database unless overridden with `inputs`.
- If you remove or rename accessories, Homebridge cached accessories may need to be cleared from the Homebridge UI.

## Troubleshooting

### Plugin Does Not Appear In Homebridge Search

If installed only from GitHub, this is expected. Homebridge plugin search uses npm.

Install from the Homebridge terminal:

```bash
npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

Or publish `homebridge-onkyo-26` to npm and install/search from the Homebridge UI.

### Permission Denied During Global Install

On macOS, this usually means npm cannot write to `/usr/local/lib/node_modules`.

Use:

```bash
sudo npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

If you are using Docker, install inside the container instead.

### Receiver Does Not Respond

Check:

- Receiver is powered or network standby is enabled.
- Receiver IP address is correct.
- Receiver has a reserved/static IP address.
- Homebridge and receiver are on the same network.
- The receiver can be controlled from the Onkyo mobile app.
- The selected `model` supports the input or zone you are using.

### No Inputs Or Wrong Inputs

Try:

- Confirming the selected `model`.
- Using `TX-NR609` as a fallback model.
- Setting `filter_inputs` to `false` to expose all supported inputs.
- Verifying that each `input_name` matches the receiver's input label.

### Child Bridge Fails After Updating

Restart the child bridge or full Homebridge process after installing updates.

If Homebridge still loads an old copy, uninstall and reinstall:

```bash
npm uninstall -g homebridge-onkyo
npm uninstall -g homebridge-onkyo-26
npm install -g git+https://github.com/fredsocial/homebridge-onkyo-26.git
```

### Debugging eISCP Commands

The bundled eISCP examples live in:

```text
eiscp/examples
```

After installing the package, locate the installed plugin directory and run the examples from there. Example:

```bash
cd /path/to/homebridge-onkyo-26/eiscp/examples
node 3.js
```

The examples can show available commands and input names for your receiver.

## Development

Clone the repository:

```bash
git clone https://github.com/fredsocial/homebridge-onkyo-26.git
cd homebridge-onkyo-26
npm install
```

Run checks:

```bash
npm test
```

Package dry run:

```bash
npm pack --dry-run
```

## Publishing To npm

The package name is already set to `homebridge-onkyo-26`.

Log in:

```bash
npm login
```

Publish:

```bash
npm publish
```

After publishing, Homebridge UI should be able to find `homebridge-onkyo-26` from the Plugins search page.

If npm reports a local cache ownership error, fix the ownership of your npm cache directory. On macOS or Linux, this is usually:

```bash
sudo chown -R "$(id -u):$(id -g)" ~/.npm
```

## Fork Notes

This fork includes compatibility updates for newer Homebridge API shapes:

- Uses the current Homebridge accessory category location, with fallback for older API shapes.
- Uses compatible read-permission lookup for HomeKit characteristics.
- Uses `TX-NR609` as the default receiver model.
- Keeps `map_volume_100` disabled by default unless explicitly enabled.
- Uses `max_volume` default `30`.
- Uses `filter_inputs` default `false`.

## License And Credits

This plugin is licensed under the ISC license.

It is based on earlier `homebridge-onkyo` and `homebridge-onkyo-avr` work. The bundled eISCP library has its own license in `eiscp/LICENSE`; the original eISCP project is:

https://github.com/untitledlt/eiscp.js
