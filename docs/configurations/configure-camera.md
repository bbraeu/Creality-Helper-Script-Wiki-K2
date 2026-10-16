# Camera Support for Fluidd, Mainsail and Mobileraker — K2

The K2 has a built-in camera that uses a proprietary WebRTC protocol on port 8000 (`webrtc_local`). go2rtc connects to it directly and re-serves the stream to Fluidd, Mainsail and the Mobileraker app. Optionally, an additional **MJPEG** stream is created for clients that cannot use WebRTC.

!!! success "Camera fix"
    The camera feed shows automatically in both the Fluidd and Mainsail dashboards after boot, with no manual steps and no dependency between the two interfaces.

---

## How It Works

The K2's `webrtc_local` service sends RTP using a payload type that it never lists in its own SDP answer. Without accounting for this, a generic WebRTC client (including a default go2rtc connection) ends up with the actual video data routed to an internal receiver that nothing is consuming, so the connection reports "connected" while no video frame ever renders.

go2rtc has an official handler for exactly this quirk, added in v1.9.10, via the `#format=creality` source modifier:

```yaml
streams:
  k2plus:
    - "webrtc:http://127.0.0.1:8000/call/webrtc_local#format=creality"
```

!!! note "Stream name `k2plus`"
    The internal go2rtc stream is still called `k2plus` on every K2-family printer. The name is kept for compatibility with existing installs, nginx rules and URLs — it is not a sign that K2 Plus settings are being used.

The solution consists of:

1. **go2rtc** — ARM binary (latest release, downloaded to `/mnt/UDISK/helper-script/go2rtc`), connects directly to `webrtc_local` on port 8000 using `#format=creality`
2. **Nginx proxy** — routes `/go2rtc/` on ports 4408 (Fluidd) and 4409 (Mainsail) to go2rtc's API on port 1984
3. **camera_watchdog.py** — monitors the stream and reconnects if the producer drops
4. **Startup service** — `/etc/rc.d/S99camera`, started from `rc.local` on boot
5. **Moonraker webcam entries** — shared by Fluidd, Mainsail and Mobileraker
6. **Optional MJPEG stream** `k2plus_mjpeg` — see below

An earlier version of this fix used a hand-rolled Python relay (`k2rtc.py`) plus a hidden keepalive iframe and auto-reload JavaScript in Fluidd. Those workarounds are no longer needed now that go2rtc's `#format=creality` handler addresses the root cause directly, and have been removed.

Full credit for the original WebRTC discovery goes to [DnG-Crafts](https://github.com/DnG-Crafts/K2-Camera), and to [AlexxIT/go2rtc](https://github.com/AlexxIT/go2rtc) for the official Creality format handler (see [go2rtc#2024](https://github.com/AlexxIT/go2rtc/issues/2024) for background on the underlying firmware quirk).

---

## Installation

Install from the helper script:

```sh
sh /mnt/UDISK/helper-script/helper.sh
```

Select **11) Camera Support (Fluidd, Mainsail, Mobileraker)**.

The installer asks:

1. **Continue?** — a reboot is required after the install for the camera service to start.
2. **Add MJPEG camera? [y/n]** — answer `y` if you use Mobileraker (free version, notification snapshots) or want to watch the camera remotely through OctoEverywhere.
3. **Install ffmpeg 5 from Entware? [y/n]** — only asked when [Entware](../helper-script/entware-k2.md) is installed and no ffmpeg 5 is found. A *live* MJPEG stream needs **ffmpeg 5 or newer**: go2rtc refuses the K2's stock ffmpeg 4.3 for transcoding (`ffmpeg: unsupported version: 4.3.6` in `/tmp/go2rtc.log`). Entware provides ffmpeg 5.1 (about 15 MB).

    Without ffmpeg 5 the MJPEG camera is registered in **snapshot mode** instead (`mjpegstreamer-adaptive`, about 1 frame per second). Snapshots work with the stock ffmpeg, so notifications and the free Mobileraker version still get a picture.

At the end it offers to reboot.

---

## Webcams Registered in Moonraker

The webcams are stored in Moonraker (absolute URLs with your printer's IP, so Mobileraker can map them through OctoEverywhere). They use port **4408** — the stock Fluidd server, which always exists; port 4409 only exists when Mainsail is installed:

| Name | Service | Stream URL | Snapshot URL |
|---|---|---|---|
| **K2 Camera** | `webrtc-go2rtc` | `http://<printer-ip>:4408/go2rtc/` | `http://<printer-ip>:4408/go2rtc/api/frame.jpeg?src=k2plus` |
| **K2 Camera MJPEG** (live, ffmpeg 5) | `mjpegstreamer` | `http://<printer-ip>:4408/go2rtc/api/stream.mjpeg?src=k2plus_mjpeg` | `http://<printer-ip>:4408/go2rtc/api/frame.jpeg?src=k2plus` |
| **K2 Camera MJPEG** (snapshot mode) | `mjpegstreamer-adaptive` | `http://<printer-ip>:4408/go2rtc/api/frame.jpeg?src=k2plus` | same |

!!! tip "Camera installed with an older helper version?"
    Older versions registered the webcams on port 4409, so the camera stayed black when Mainsail was not installed. Update the helper script and run Camera Support again.

### The optional MJPEG stream

In live mode, `k2plus_mjpeg` is an extra go2rtc stream that transcodes the WebRTC/H.264 video (1280×720, 15 fps from the camera) to MJPEG with ffmpeg 5:

```yaml
  k2plus_mjpeg:
    - "ffmpeg:k2plus#video=mjpeg#raw=-r 10"

ffmpeg:
  bin: /opt/bin/ffmpeg
```

- 10 frames per second at the camera's resolution
- ffmpeg only runs **while someone is watching** the MJPEG stream — it costs no CPU otherwise
- If CPU load while printing is too high, lower the `-r` value or add `#width=640` in `/mnt/UDISK/helper-script/go2rtc.yaml` (re-running the installer overwrites the file)

---

## After Installation

The camera appears automatically in both Fluidd and Mainsail after boot:

1. Printer boots up
2. After ~60–90 seconds go2rtc connects to the K2 camera
3. The camera feed shows in both Fluidd and Mainsail independently — opening one does not affect the other

!!! note "First open after boot"
    The camera takes 60–90 seconds after boot to become available, since the startup service waits for the rest of the printer's services to come up first.

---

## Mobileraker

- Add the printer in the Mobileraker app **by IP address**, not by hostname. The webcam URLs contain the IP, and Mobileraker only maps them through OctoEverywhere for remote viewing when the host matches the printer address it knows.
- **K2 Camera** (go2rtc WebRTC) is shown by Mobileraker to **supporters (paid subscription) only**.
- **K2 Camera MJPEG** works for **everyone**, and MJPEG is the only kind of webcam Mobileraker Companion can take **notification snapshots** from.
- Remotely via OctoEverywhere, **WebRTC does not work, MJPEG does**.
- Mobileraker's own `webrtc-creality` camera mode does not work on the K2 ([mobileraker#452](https://github.com/Clon1998/mobileraker/issues/452), open).
- Webcams via Obico are not supported by Mobileraker.

See [Mobileraker Companion](../helper-script/mobileraker-companion-k2.md) for push notifications.

---

## Direct Camera Access

Via go2rtc's own web interface:

```
http://<printer-ip>:1984/stream.html?src=k2plus&mode=webrtc
```

MJPEG stream (if installed) and a single snapshot:

```
http://<printer-ip>:4408/go2rtc/api/stream.mjpeg?src=k2plus_mjpeg
http://<printer-ip>:4408/go2rtc/api/frame.jpeg?src=k2plus
```

## Troubleshooting

| Symptom | Check |
|---|---|
| No picture at all | Run `curl -o /tmp/t.jpg "http://127.0.0.1:1984/api/frame.jpeg?src=k2plus"` on the printer. If that works, the camera chain is fine — check the webcam URLs (port 4408) in Fluidd → Settings → Cameras. |
| Webcam URLs use `:4409` but Mainsail is not installed | Update the helper and run Camera Support again (or edit the URLs to `:4408`). |
| MJPEG camera black, `/tmp/go2rtc.log` shows `ffmpeg: unsupported version` | ffmpeg 5 is needed — install Entware, run Camera Support again and accept the ffmpeg install. |
| Camera missing right after boot | Normal for 60–90 s; check that go2rtc runs (`ps w`) and look at `/tmp/go2rtc.log`. |

---

## Fixed: Stream Not Found on Port 4408

Older versions of the nginx rule for `/go2rtc/api/ws` on port 4408 forwarded `?src=k2plus?src=...` when the client sent its own query string, so go2rtc answered "stream not found". The rule now appends the client's query with `&`. Existing installs are repaired when Camera Support is installed again. Reinstalling also no longer adds a duplicate script to Fluidd's `index.html`, and removing Camera Support removes it.

---

## Known Limitations

- **Startup delay** — the camera takes 60–90 seconds after boot to appear
- **Firmware updates** — a firmware update may remove the nginx and rc.local changes. Re-run the camera install after any firmware update.

---

## Credits

- [DnG-Crafts](https://github.com/DnG-Crafts/K2-Camera) — Original K2 camera WebRTC discovery
- [AlexxIT/go2rtc](https://github.com/AlexxIT/go2rtc) — Stream conversion software and the official Creality format handler
