---
name: wendy-entitlements
description: Use when creating, reviewing, or debugging `wendy.json` entitlements for Wendy apps, especially device access, networking, persistence, GPU, audio, camera, display, Bluetooth, USB, serial, I2C, GPIO, SPI, input, MCP, or admin, and for multi-service `isolation` and `frameworks`.
---

# Wendy Entitlements Workflow

Use this for any task involving `wendy.json` capabilities. Base recommendations on the runtime behavior in `wendy-agent/go/internal/agent/oci/entitlements.go` and validation in `wendy-agent/go/internal/shared/appconfig/appconfig.go`.

## Start from app needs

Ask what the app actually touches, then choose the smallest entitlement set:

- Server, WebRTC, callbacks, debugger, or LAN-visible API: `network`.
- NVIDIA/Jetson inference or CUDA: `gpu`.
- Microphone, speaker, ALSA, PipeWire, PulseAudio compatibility: `audio`.
- V4L2 cameras or USB webcams: `camera`.
- Draw to a locally-attached monitor as a Wayland client: `display`.
- Model cache, database, user uploads, generated files, or persistent app state: `persist`.
- BlueZ Bluetooth access: `bluetooth`.
- Raw USB devices: `usb`.
- USB serial tty (servo buses, sensors, microcontrollers): `serial`.
- I2C bus devices: `i2c`.
- GPIO chips: `gpio`.
- SPI devices: `spi`.
- HID/input event devices: `input`.
- Expose the container as an MCP server: `mcp`.
- Full unauthenticated local device control (trusted first-party only): `admin`.

Prefer no entitlement when the app does not need the host resource.

ROS 2 apps use the top-level `frameworks.ros2` block plus `network` (host) and, for intra-host zero-copy, `isolation` — not a dedicated entitlement. See [wendy-ros2] and the multi-service section below.

## Supported entitlements

Use this table as the starting point, then verify against the local repo when changing runtime behavior:

| Type | Required keys | Common optional keys | Runtime effect |
| --- | --- | --- | --- |
| `network` | none | `mode` as `host`, `host-admin`, or `none` | Defaults to host networking when `mode` is empty; `host` removes the network namespace and mounts host DNS config; `host-admin` does the same and additionally adds `CAP_NET_ADMIN`. Plain `host` no longer grants `CAP_NET_ADMIN`. |
| `gpu` | none | none | Adds NVIDIA device nodes, NVIDIA env vars (`NVIDIA_VISIBLE_DEVICES=all`), and relies on CDI for full Jetson/NVIDIA library/device wiring. On Raspberry Pi it instead binds `/dev/vcio` (VideoCore mailbox) for board telemetry. |
| `audio` | none | none | Adds audio group, mounts `/dev/snd`, allows sound devices, and mounts PipeWire/Pulse sockets when present. |
| `camera` | none | `mode`, `allowlist` | Canonical V4L2/camera entitlement; allows major 81 and bind-mounts host `/dev` for live camera hotplug. `mode`/`allowlist` are accepted only for backward-compatibility with the deprecated `video` entitlement and have no runtime effect today. |
| `video` | none | `mode`, `allowlist` | Deprecated compatibility alias for `camera`; prefer `camera` in new configs. |
| `display` | none | none | Presents to a locally-attached monitor as a Wayland client: mounts `/dev/dri`, adds `video`+`render` groups, and mounts the WendyOS compositor's Wayland socket (`WAYLAND_DISPLAY`, `XDG_RUNTIME_DIR`). At most one per app; requires a display-enabled WendyOS image (headless images accept it but render nothing). |
| `persist` | `name`, `path` | none | Creates/binds `/var/lib/wendy/volumes/<name>` to the container `path`; volume names are a shared namespace across apps. |
| `bluetooth` | none | `mode` | Uses a filtered `xdg-dbus-proxy` socket for BlueZ; sets `DBUS_SYSTEM_BUS_ADDRESS`. Do not assume unrestricted host D-Bus access. `mode` is accepted; docs describe `bluez` (D-Bus profiles) vs `kernel` (raw HCI, extra caps), but confirm the target agent version before relying on `kernel`. |
| `usb` | none | none | Mounts `/dev/bus/usb` and allows USB character devices. |
| `serial` | `device` | none | Binds the USB serial tty `/dev/<device>` at its exact major:minor and adds the `dialout` group. `device` is a bare node name matching `ttyACM<n>` or `ttyUSB<n>` (USB only — on-board UARTs like `ttyAMA`/`ttyS` are not supported), not a path. |
| `i2c` | `device` | none | Binds `/dev/<device>` and allows I2C devices. `device` must be in bare `i2c-N` form (e.g. `i2c-1`); the validator rejects a `/dev/`-prefixed path. |
| `gpio` | none | `pins` | Mounts existing `/dev/gpiochip0` through `/dev/gpiochip7`; `pins` are documentation/validation, access is chip-level. |
| `spi` | none | none | Mounts existing `/dev/spidev*` nodes and adds the host `spi` group when present. |
| `input` | none | none | Adds input group (GID 105), mounts `/dev/input`, and allows input event devices (major 13). |
| `mcp` | `port` | none | Registers the container as an MCP server on `port` (1–65535); recorded on container label `sh.wendy/mcp.port`. At most one per app. Accepted by `wendy json validate` even though it may not appear in `wendy json schema` output. |
| `admin` | none | none | Bind-mounts the agent's unix control socket into the container (`WENDY_AGENT_SOCKET`), granting full **unauthenticated** local control of the device (start/stop/remove apps, read all device data). This mount is the entire trust boundary — grant only to fully-trusted first-party apps. At most one per app; never reachable off-device. |

`network.ports` is accepted by config validation, but the runtime path in `entitlements.go` currently acts on `mode`. Do not claim port mapping behavior without tracing the caller that consumes `ports`.

`build` and `audio.allowCdiDevSnd` appear in the upstream `wendy-agent` source and docs but are **not accepted by the released CLI validator** yet — treat them as unreleased and do not add them to a `wendy.json` you intend to deploy today.

## Example patterns

Web server:

```json
{
  "appId": "api-server",
  "platform": "wendyos",
  "entitlements": [
    { "type": "network", "mode": "host" }
  ],
  "readiness": {
    "tcpSocket": { "port": 8000 },
    "timeoutSeconds": 30
  }
}
```

Camera + audio app:

```json
{
  "appId": "camera-assistant",
  "platform": "wendyos",
  "entitlements": [
    { "type": "network", "mode": "host" },
    { "type": "camera" },
    { "type": "audio" }
  ]
}
```

Jetson GPU app with persistent model cache:

```json
{
  "appId": "vision-inference",
  "platform": "wendyos",
  "entitlements": [
    { "type": "network", "mode": "host" },
    { "type": "gpu" },
    { "type": "camera" },
    { "type": "persist", "name": "vision-models", "path": "/models" }
  ]
}
```

I2C sensor app:

```json
{
  "appId": "sensor-reader",
  "platform": "wendyos",
  "entitlements": [
    { "type": "i2c", "device": "i2c-1" },
    { "type": "persist", "name": "sensor-data", "path": "/data" }
  ]
}
```

## Multi-service isolation and frameworks

These are top-level `wendy.json` keys (not entitlements) that apply to apps with a `services` map. See [wendy-project-setup] for the full multi-service shape.

- `isolation` (top-level string) controls namespace sharing across services:
  - `shared-ipc` — shares the network and IPC namespaces plus `/dev/shm` across all services, enabling zero-copy/shared-memory transports (e.g. CycloneDDS intra-host). Host networking alone does not share IPC or `/dev/shm`.
  - `shared-network` — shares only the network (and UTS) namespaces; no shared `/dev/shm`.
  - `isolated` — per-service CNI networking with an assigned IP and an injected `/etc/hosts` so service names resolve.
  The value is not enum-validated by the CLI, so a typo is silently inert; keep it to the three values above.
- `frameworks.ros2` (top-level and per-service) injects ROS 2 environment into every service container: `ROS_DOMAIN_ID`, `RMW_IMPLEMENTATION`, and (for CycloneDDS) a `CYCLONEDDS_URI`, plus `ROS_LOCALHOST_ONLY=1`. Because it forces localhost-only DDS and overrides `CYCLONEDDS_URI`, use it for intra-host ROS 2 graphs — **not** for apps that must reach an external robot over the physical network with their own DDS config. See [wendy-ros2].

## Review checklist

- `appId` is present.
- Entitlement names are current; prefer `camera` over `video`.
- Entitlement types are limited to the released set; do not add `build` or `audio.allowCdiDevSnd` (unreleased).
- `persist` entries include both `name` and an absolute container `path`.
- `i2c` entries include a device name without a leading `/dev/` (bare `i2c-N`); `serial` devices are bare `ttyACM<n>`/`ttyUSB<n>`.
- `mcp` entries include a `port`; `display`/`admin`/`mcp` appear at most once per app.
- `admin` is present only on fully-trusted first-party apps.
- Device-heavy apps include only the devices they actually use.
- GPU issues are checked across app image, device type, CDI, agent logs, and app fallback behavior before blaming the entitlement alone.
- For ROS 2 apps, DDS discovery needs `network` host mode; intra-host zero-copy needs `isolation: shared-ipc`; `frameworks.ros2` is not used when talking to an external robot with app-managed DDS.
- JSON examples remain valid JSON; do not put comments inside `wendy.json`.
