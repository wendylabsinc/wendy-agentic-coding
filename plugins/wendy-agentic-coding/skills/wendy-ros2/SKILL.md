---
name: wendy-ros2
description: Use when building, deploying, or debugging ROS 2 apps on WendyOS — the `frameworks.ros2` config, RMW/DDS middleware choice (CycloneDDS, Fast DDS, Connext, Gurum), multi-node discovery and shared-memory transport, `isolation: shared-ipc`, or the `wendy device ros2` CLI (nodes, topics, echo, hz, bag, param, graph, doctor).
---

# Wendy ROS 2 Workflow

WendyOS runs real upstream ROS 2 from stock containers — Humble by default, any RMW you choose. There is no `ros2` entitlement: ROS 2 is configured with the top-level `frameworks.ros2` block, host networking for DDS discovery, and (optionally) `isolation: shared-ipc` for zero-copy transport. Use [wendy-entitlements] for the surrounding capabilities and [wendy-project-setup] for the multi-service shape.

## Configure ROS 2 with `frameworks.ros2`

`frameworks` is a top-level key (and may also appear per-service, overriding the group value):

```json
{
  "appId": "sh.wendy.examples.ros2",
  "platform": "linux",
  "version": "1.0.0",
  "isolation": "shared-ipc",
  "frameworks": {
    "ros2": { "domainId": 42, "rmw": "rmw_cyclonedds_cpp", "distro": "humble" }
  },
  "services": {
    "talker": { "context": "./talker" },
    "listener": { "context": "./listener", "dependsOn": ["talker"] }
  }
}
```

Keys (all optional):

- `domainId`: integer `0`–`232`. When omitted, a stable hash of `appId` in `[0, 232]` is used, so every service in the app shares one domain.
- `rmw`: middleware. Short names `cyclonedds` (default), `fastrtps`/`fastdds`, `connextdds`, `gurumdds`, or full identifiers `rmw_cyclonedds_cpp`, `rmw_fastrtps_cpp`, `rmw_connextdds`, `rmw_gurumdds_cpp`.
- `distro`: e.g. `humble` (default), `jazzy`.

At runtime the agent injects into every service container: `ROS_DOMAIN_ID`, `RMW_IMPLEMENTATION`, `ROS_LOCALHOST_ONLY=1`, and — for CycloneDDS — a `CYCLONEDDS_URI` (with shared memory disabled). It also tags the app so `wendy device ros2` can attach a CLI sidecar in the same domain.

### When NOT to use `frameworks.ros2`

Because it forces `ROS_LOCALHOST_ONLY=1` and overrides `CYCLONEDDS_URI`, `frameworks.ros2` is for **intra-host** ROS 2 graphs (nodes cooperating on one device). Do **not** add it to an app that must reach an **external robot** over the physical network with its own DDS configuration (e.g. a Unitree Go2 on `192.168.123.0/24` using an app-supplied `cyclonedds.xml`): the injected localhost-only setting and overriding URI will cut the app off from the robot. Such an app should keep `network` host mode and manage DDS itself, with no `frameworks.ros2`.

## Networking and transport

Two independent layers — pick based on where DDS peers live:

- **Discovery / multicast / reaching other hosts or robots:** add a `network` entitlement in `host` mode to each service that speaks DDS (in a Compose companion, `network_mode: host` becomes a host-network entitlement). Host networking alone is enough for discovery.
- **Zero-copy intra-host transport (CycloneDDS shared memory):** set top-level `isolation: "shared-ipc"`, which shares the network and IPC namespaces plus `/dev/shm` across services. Host networking does not share IPC or `/dev/shm`, so it cannot provide zero-copy on its own.

## Build a ROS 2 service

Use the official upstream ROS image and install nodes + the matching RMW with `apt`. No colcon/rclpy build step is required for prebuilt nodes:

```dockerfile
FROM ros:humble
RUN apt-get update && apt-get install -y --no-install-recommends \
      ros-humble-demo-nodes-cpp ros-humble-rmw-cyclonedds-cpp \
  && rm -rf /var/lib/apt/lists/*
CMD ["bash", "-lc", "source /opt/ros/humble/setup.bash && exec ros2 run demo_nodes_cpp talker"]
```

Give each service its own `context` directory and Dockerfile; order startup with `dependsOn`. `wendy run` streams `[serviceName]`-prefixed logs and stops services in reverse dependency order on Ctrl+C.

## Inspect and debug a live ROS 2 system

`wendy device ros2 <cmd>` runs `ros2` inside an agent-managed sidecar over gRPC — no SSH, no `source setup.bash`, no extra ports. Add `--device <hostname>`; most subcommands accept `--domain <n>` to override the app's `ROS_DOMAIN_ID`.

```bash
# Graph
wendy device ros2 nodes
wendy device ros2 topics --all
wendy device ros2 topic info /chatter
wendy device ros2 services
wendy device ros2 graph --format dot

# Live data
wendy device ros2 echo /chatter --count 10
wendy device ros2 hz /chatter

# Parameters
wendy device ros2 params --node /talker
wendy device ros2 param get /talker use_sim_time
wendy device ros2 param set /talker use_sim_time true

# Services
wendy device ros2 call /reset std_srvs/srv/Empty

# rosbags (recorded on the device)
wendy device ros2 bag record /camera/image_raw /imu --output drive-test
wendy device ros2 bag list
wendy device ros2 bag download drive-test ./local-bags

# Health + escape hatch
wendy device ros2 doctor
wendy device ros2 --domain 42 exec node info /talker
```

`exec` passes any raw `ros2` subcommand through to the sidecar. Use `--json` for machine-readable output where supported.

## Checklist

- `frameworks.ros2` is present for intra-host graphs; absent for apps talking to an external robot with their own DDS.
- Each DDS-speaking service has a `network` host entitlement; add `isolation: "shared-ipc"` only when you need zero-copy/shared memory.
- `rmw` is one of the accepted names; `distro` matches the base image (`ros:<distro>`).
- Validate with `wendy json validate` before `wendy run`.
- Debug live systems with `wendy device ros2`, not SSH.
