# ESPHome Polytropic Heat Pump

ESPHome configuration for a Polytropic pool heat pump exposed to Home Assistant.

## Repository contents

- `polytropic-esphome-prod.yaml`: main ESPHome configuration.
- `polytropic-esphome-mqtt.yaml`: MQTT-discovery variant for Home Assistant.
- `docs/`: vendor and Modbus reference material, Home Assistant snippets, and 3D enclosure files.

## 3D printable case

The `docs/` directory also contains the 3D files for the enclosure used to house the controller.

- `docs/top_case_heatpump_control_pool.stl`: printable top half of the case.
- `docs/bottom_case_heatpump_control_pool.stl`: printable bottom half of the case.
- `docs/rs485-modbus-heating-pump-pool.stl`: full enclosure mesh for viewing or slicing as a single reference model.
- `docs/rs485-modbus-heating-pump-pool.step`: CAD source file if you want to inspect or adapt the enclosure geometry.

If you want to reprint the case as designed, print the top and bottom STL files. The STEP file is useful if you need to modify cutouts, mounting points, or clearances for your own hardware.

## Validation

This repository is validated automatically with GitHub Actions on each push to `main`, on pull requests, and on manual dispatch.

The workflow installs ESPHome `2026.4.5`, discovers all tracked YAML files that contain an `esphome:` block, creates temporary dummy `secrets.yaml` files next to each config, and runs `esphome config` on every detected configuration.

Today that effectively validates:

```sh
esphome config polytropic-esphome-prod.yaml
esphome config polytropic-esphome-mqtt.yaml
```

## Local validation

ESPHome requires a `secrets.yaml` file even for syntax validation. Keep your real secrets out of the repository.

Example with a temporary validation directory:

```sh
tmpdir=$(mktemp -d) && \
cp polytropic-esphome-prod.yaml "$tmpdir/" && \
cp secrets.local.yaml "$tmpdir/secrets.yaml" && \
esphome config "$tmpdir/polytropic-esphome-prod.yaml"
```

For the MQTT variant:

```sh
tmpdir=$(mktemp -d) && \
cp polytropic-esphome-mqtt.yaml "$tmpdir/" && \
cp secrets.local.yaml "$tmpdir/secrets.yaml" && \
esphome config "$tmpdir/polytropic-esphome-mqtt.yaml"
```

`secrets.local.yaml` is intentionally local-only and gitignored.

## MQTT variant

`polytropic-esphome-mqtt.yaml` replaces the native ESPHome API transport with MQTT discovery for Home Assistant.

It expects these additional secrets:

- `mqtt_broker`
- `mqtt_username`
- `mqtt_password`

The MQTT variant keeps `discover_ip: true` so ESPHome tools can still find the node IP over MQTT, and uses Home Assistant MQTT discovery to publish entities.

## Flash / upload

Once your real `secrets.yaml` is available locally, typical commands are:

```sh
esphome run polytropic-esphome-prod.yaml
esphome upload polytropic-esphome-prod.yaml
```

## Home Assistant wrapper

ESPHome must keep the standard `HEAT_PUMP` water heater mode, so Home Assistant will still consider the native operation mode to be `heat_pump`.

If you want a friendlier UI label such as `Smart`, use the companion snippet in `docs/home-assistant-smart-mode.yaml`.

It provides:

- a template select with `Off`, `Smart`, `Eco`, and `Boost`
- a template sensor that mirrors the current mode with friendly labels

This does not rename the native water heater mode inside Home Assistant's built-in water heater card. It adds a wrapper control and a readable status entity next to it.

## Home Assistant water flow guard

If you want Home Assistant to block operation when there is no water circulation, use `docs/home-assistant-water-flow-guard.yaml`.

It adds automations that:

- force the water heater back to `off` if water flow disappears while it is running
- force it back to `off` if someone tries to start it while `Water Flow Switch` is inactive
- create a persistent notification when a start is blocked for missing water flow
