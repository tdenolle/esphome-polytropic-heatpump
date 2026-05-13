# ESPHome Polytropic Heat Pump

ESPHome configuration for a Polytropic pool heat pump exposed to Home Assistant.

## Repository contents

- `polytropic-esphome-prod.yaml`: main ESPHome configuration.
- `docs/`: vendor and Modbus reference material.

## Validation

This repository is validated automatically with GitHub Actions on each push to `main`, on pull requests, and on manual dispatch.

The workflow installs ESPHome `2026.4.5`, discovers all tracked YAML files that contain an `esphome:` block, creates temporary dummy `secrets.yaml` files next to each config, and runs `esphome config` on every detected configuration.

Today that effectively validates:

```sh
esphome config polytropic-esphome-prod.yaml
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

`secrets.local.yaml` is intentionally local-only and gitignored.

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
