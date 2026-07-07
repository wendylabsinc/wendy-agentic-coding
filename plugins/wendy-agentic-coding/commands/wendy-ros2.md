---
description: Build, deploy, or debug a ROS 2 app on WendyOS (frameworks.ros2, RMW/DDS, isolation, wendy device ros2).
argument-hint: [app description, wendy.json path, or ROS 2 issue]
---

Use the `wendy-ros2` skill. Configure, deploy, or debug ROS 2 for:

`$ARGUMENTS`

Base recommendations on the released `wendy.json` schema (`wendy json schema`), the `frameworks.ros2` runtime behavior in `wendy-agent`, and the `wendy device ros2` CLI. Prefer host networking for DDS discovery and `isolation: shared-ipc` for intra-host zero-copy; do not add `frameworks.ros2` to apps that talk to an external robot with their own DDS config.
