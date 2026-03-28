# ◻ JHMCS HUD

A single-file, zero-dependency AR Heads-Up Display running in mobile Chrome. Designed as a DIY Helmet-Mounted Cueing System, real camera feed, real ADS-B aircraft tracking, real sensor data, all in one HTML file.

Built for landscape mode on Android. Tested on Pixel 4a (Android 13) and Galaxy S7.

---

## What It Does

- **Live camera feed** as background with full HUD symbology overlay
- **Real-time ADS-B aircraft tracking** — diamonds appear on-screen where planes actually are in the sky
- **Tilt-compensated compass** — pitch the phone ±30° without the heading drifting
- **Edge-cued off-screen targets** — blinking arrows on screen edges point toward aircraft outside your FOV
- **Radar circle** with closest-aircraft type code and orbital bearing indicator
- **Template-based optical lock** — tap any object to track it frame-by-frame using SAD matching
- **Custom audio alerts** — load your own startup and new-contact sounds
- **GPS-driven speed (KTS) and altitude (FT)** readouts



## Requirements

- Android phone with Chrome (landscape orientation)
- Camera, GPS, and device orientation permissions
- Internet connection (for ADS-B data + Google Fonts)

---

## Quick Start

1. Open https://hmd.fhmd.workers.dev/ in       Chrome on your phone
2. (Optional) Select startup and contact alert sounds
3. Tap **INITIALIZE**
4. Point your phone at the sky

---

## HUD Elements

| Element | Position | Description |
|---|---|---|
| Crosshair | Center | Fixed reference point |
| Speed (KTS) | Left | GPS ground speed in knots |
| Altitude (FT) | Right | GPS altitude in feet |
| Compass tape | Bottom | Smooth heading tape with cardinal markers |
| Heading box | Bottom center | Current heading in degrees |
| Radar circle | Top left | ICAO type code of closest aircraft |
| Orbital square | Top left | Rotates around radar circle at bearing to closest target |
| Diamond ◇ | In-FOV | Aircraft within 35nm — position matches real-world location |
| Dim diamond + X + OOR | In-FOV | Aircraft 35–50nm — out of range indicator |
| Blinking triangle ▲ | Screen edge | Points toward off-screen aircraft, updates at 2Hz |
| Lock reticle | Tap target | 24×24 square with center dot, tracks locked object |
| ARM | Left | Arm status label |
| AC count | Top right | Number of tracked aircraft |
| GPS OK / SENS OK | Top right | Sensor status indicators |

---

## Changelog

### V2.0.0 — Pitch Fix + Full FOV Arrows
- **Fixed pitch axis** — replaced broken `PITCH_SIGN * gamma` with rotation-matrix-derived pitch: `pitch = asin(cos(β)·cos(γ))`. Works correctly in both portrait and landscape without any sign constant to tune.
- **Vertical FOV arrows** — off-screen arrows now trigger on all 4 screen edges (top, bottom, left, right). Previously only horizontal out-of-FOV was detected; aircraft above/below were silently y-clamped as diamonds instead of showing directional arrows.
- **Version label** on startup screen.

### V1.7 — Landscape Mode
- Landscape orientation lock with portrait "ROTATE DEVICE" overlay.
- Pitch axis switched from `beta - 90` (portrait) to `gamma`-based (landscape).
- Diamond y-position clamped to screen bounds so targets don't disappear off-canvas.
- Virtual FOV widened to `H_FOV = 110°`, `V_FOV = 80°` for less lateral sensitivity.
- Triangle edge-clamping expanded to all 4 edges (was stuck on left/right only).

### V1.6 — Audio System
- Two file pickers on init screen: startup sound + new contact sound.
- AudioContext unlocked on INITIALIZE tap (user gesture) to bypass autoplay policy.
- Audio buffers decoded immediately; `playBuf()` fires from timers and fetch callbacks without blocking.

### V1.5 — Radar Circle Intelligence
- Radar circle label shows ICAO type code of closest aircraft (A320→A32, B738→B73, etc.).
- Orbital square rotates around circle rim at absolute bearing to closest aircraft.
- New contact pulse: label + square flash at 2.5Hz for 5 seconds on new hex ID detection.

### V1.4 — ADS-B Aircraft Tracking
- `api.adsb.one` as primary source (CORS-open), `opensky-network.org` as fallback.
- Polls every 5 seconds, 50nm radius from GPS position.
- Projection math: haversine bearing + elevation angle → screen XY relative to phone heading/pitch.
- In-FOV: diamond symbol, slightly darker than HUD color.
- OOR (35–50nm): dimmer diamond + X above + OOR label.
- Off-FOV: filled triangle pinned to correct screen edge, points at aircraft, blinks at 2Hz.
- AC count debug counter in top right.

### V1.3 — Tilt-Compensated Compass
- Fixed compass heading drifting ~110° when pitching phone ±10°.
- Implemented full W3C rotation matrix tilt compensation using alpha, beta, and gamma.
- iOS `webkitCompassHeading` (already tilt-compensated) left untouched.

### V1.2 — Units + Lock Reticle
- Speed converted from KMH to KTS (`m/s × 1.944`).
- Altitude converted from meters to feet (`m × 3.281`).
- Lock reticle redesigned: full square border with solid center dot (was corner brackets).
- Template tracker using SAD (Sum of Absolute Differences): tap to capture 32×32px patch, searches ±48px each frame for best match, reticle lerps smoothly toward match position.

### V1.1 — Compass Fix
- Fixed bottom compass tape showing negative values (`-101` instead of `259`).
- Root cause: `fracOffset` starting at 0 instead of relative to center, plus JS `%` not wrapping negatives.
- Fixed double-firing sensor bug where both `deviceorientation` and `deviceorientationabsolute` fired simultaneously.

### V1.0 — Base HUD
- Camera feed (rear-facing, 1080p).
- Center crosshair.
- Speed and altitude boxes.
- ARM label.
- Radar circle with orbital square.
- GPS OK / SENS OK status indicators.
- Permission/init screen.



## Architecture

```
hud.html (single file)
├── Camera feed (getUserMedia, rear-facing)
├── HUD overlay (DOM + Canvas)
│   ├── Compass tape (canvas)
│   ├── Aircraft layer (canvas)
│   ├── Radar circle (SVG + DOM)
│   └── Static elements (DOM)
├── Sensors
│   ├── GPS (watchPosition → speed, alt, lat/lon)
│   ├── Orientation (deviceorientationabsolute → heading, pitch)
│   └── Tilt-compensated heading (W3C rotation matrix)
├── ADS-B engine
│   ├── api.adsb.one (primary, CORS-open)
│   ├── opensky-network.org (fallback)
│   └── Projection: haversine + elevation → screen XY
├── Template tracker (SAD patch matching)
└── Audio (AudioContext, user-provided files)
```

### Ready for expansion
- **WebSocket bridge** from passive radar backend → push targets into `aircraftList`
- **MediaPipe** object detection upgrade for lock reticle
- **MLAT layer** on top of ADS-B for non-squawking targets
- **Servo-driven dish** TWS scan mode feeding bearing data


