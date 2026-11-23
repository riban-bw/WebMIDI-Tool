# Web MIDI Monitor

A self-contained, single-page web app for sending and receiving MIDI in the browser. It logs incoming and outgoing messages with timestamps, raw bytes and a human-readable interpretation, and lets you send raw bytes (hex or decimal), build messages from a form, fire off common preset messages, or play an on-screen keyboard.

Everything lives in one file, `index.html`. There is no build step, no dependencies and no network access.

<img width="1855" height="921" alt="image" src="https://github.com/user-attachments/assets/9991825e-9bfe-4a12-81e8-bc6c2d0db972" />

## Requirements

- A browser with the [Web MIDI API](https://developer.mozilla.org/en-US/docs/Web/API/Web_MIDI_API): Chrome, Edge or Opera work best. Firefox supports Web MIDI but may require granting a site permission. Safari does not support Web MIDI.
- A secure context: open the file directly (`file://`), or serve it from `localhost` or over `https://`.

## Getting started

1. Open `index.html` in your browser, either by double-clicking it or by serving the folder:
   ```sh
   python3 -m http.server 8000
   # then visit http://localhost:8000/
   ```
2. The page requests MIDI access on load. Allow the browser's permission prompt.
3. To send or receive System Exclusive (SysEx) messages, tick **Enable SysEx**. The page reconnects and the browser asks for the additional SysEx permission.

If access is denied or unavailable, the status in the header explains why. Click **Connect** / **Reconnect** to try again.

## Ports

- **Inputs:** every available input is listed with a checkbox. Ticked inputs are monitored; untick one to ignore it. Devices that are plugged in or removed appear automatically, and the change is logged.
- **Outputs:** every available output is listed with a checkbox. Sent messages go to each checked output that is currently connected. On first use the first output is checked; after that your choices are remembered.

## The log

Each row shows:

| Column | Meaning |
| --- | --- |
| `#` | Sequence number |
| Time | Clock time (`HH:MM:SS.mmm`) or seconds since the last clear, selectable in the toolbar |
| Δ ms | Time since the previous row |
| Dir | `IN` (received), `OUT` (sent) or `SYS` (connection events and errors) |
| Port | Port name |
| Raw (hex) | The message bytes. Hover to see them in decimal |
| Type | Message type, e.g. Note On, Control Change, SysEx |
| Ch | MIDI channel (1–16) for channel messages |
| Interpretation | Decoded details |

Interpretation includes:

- Note names (middle C = 60 = C4). A Note On with velocity 0 is shown as a Note Off.
- Controller names (Modulation, Volume, Sustain, RPN/NRPN, channel mode messages, …) and on/off state for switch controllers.
- Pitch bend as a signed value (−8192…+8191) and a percentage.
- Program change (0-based and 1-based numbering), aftertouch and channel pressure.
- MTC quarter frames, Song Position (with bar/beat), Song Select, Tune Request.
- Real-time messages: Clock, Start, Continue, Stop, Active Sensing, Reset.
- SysEx: manufacturer name (including common 3-byte IDs) and decoding of common Universal messages, such as Identity Request/Reply, GM/GM2 on/off, Master Volume/Balance, MMC commands, MTC Full Frame and ACK/NAK.

### Toolbar

- **Pause / Resume:** stop adding rows. Messages are still counted.
- **Clear:** empty the log and reset counters and relative time.
- **Export CSV:** download the current log, with ISO timestamps and high-resolution times.
- **Autoscroll:** keep the newest row in view.
- **Hide clock / Hide active sensing:** drop the frequent `F8` and `FE` messages, which are hidden by default. The number dropped is shown as "filtered" in the counters.
- **Log sent:** include outgoing messages in the log.
- **Time mode:** clock time or seconds since the last clear.
- **Max rows:** how many rows to keep. Older rows are discarded.
- **Filter rows:** show only rows containing the given text, e.g. `note on`, `ch3`, `F0`, or a port name.

Click any row to copy its bytes into the **Send raw** box so you can resend or edit it.

## Sending

### Send raw

Type bytes and press **Enter** or click **Send**.

- **Hex** mode accepts many styles: `90 3C 7F`, `0x90, 0x3C, 0x7F`, `903C7F`, `90-3C-7F`.
- **Decimal** mode: `144 60 127`.
- Several messages can be sent at once, e.g. `90 3C 7F 90 40 7F 90 43 7F` for a chord.
- Text after `//` is ignored, so you can annotate entries.
- ↑ / ↓ steps through previously sent entries. History is saved in the browser.

Input is validated before sending, with specific errors for missing data bytes, stray data bytes, unterminated SysEx, undefined status bytes, or SysEx without permission.

### Message builder

Choose a message type (Note On/Off, Control Change, Program Change, Poly Aftertouch, Channel Pressure, Pitch Bend), a channel and values. A live preview shows the bytes and their interpretation. **Send** transmits the message; **→ Raw** copies it into the raw box for editing. For pitch bend, double-click the slider to recentre it.

### Preset messages

One-click buttons for common messages. Hover to see the exact bytes; **Shift-click** copies them to the raw box instead of sending.

- **Channel** (uses the builder's channel): Note On/Off C4, All Notes Off, All Sound Off, Reset Controllers, Sustain On/Off, Volume, Pan Center, Mod Wheel 0, Pitch Bend Center, Program 0, and **PANIC**, which sends All Sound Off and All Notes Off on all 16 channels.
- **System Real-Time / Common:** Start, Continue, Stop, Timing Clock, Active Sensing, System Reset, Song Position 0, Song Select 0, Tune Request.
- **SysEx** (requires SysEx permission): Identity Request, GM System On/Off, GM2 System On, Roland GS Reset, Yamaha XG On, Master Volume max, MMC Play/Stop.

### Keyboard

A two-octave clickable keyboard with its own channel and velocity settings. Use **◀ Oct / Oct ▶** to shift the range. Click and drag for glissando. Incoming notes on any channel light up the matching keys in green.

- **Velocity by position** — velocity is set by where on the key you click: soft near the top, up to 127 at the bottom. A small band at each end of the key gives the minimum / maximum, so you don't need to hit the very edge. The Vel box is disabled and shows the last velocity sent; your fixed velocity is restored when the option is turned off. Keys are shaded to hint at this.
- **Full width** — moves the keyboard to a strip across the bottom of the window and shows as many octaves as fit (up to the full MIDI range), adjusting as the window is resized.

## Stored settings

The page saves the SysEx preference, the checked outputs and the raw-send history in `localStorage`. Clear your site data to reset them.

## Troubleshooting

- **"Web MIDI is not supported":** use a Chromium-based browser.
- **No ports listed:** check that the device is connected and not opened exclusively by another application (some drivers on Windows allow only one client at a time), then click **Reconnect**.
- **SysEx won't send:** tick **Enable SysEx** and accept the permission prompt. If you previously denied it, re-enable MIDI/SysEx permission in the browser's site settings.
- **Nothing appears when the device sends clock:** clock and active sensing are hidden by default; untick the filters in the toolbar.
