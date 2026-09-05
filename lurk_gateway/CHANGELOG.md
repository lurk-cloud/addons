# Changelog

## 0.3.2

- Add-on: the optional `serial` no longer ships with a null default, which
  Home Assistant Supervisor rejected on real Raspberry Pi units.
- Station: Supervisor validation errors redact the one-time enrollment token.

## 0.3.1

- Station: Supervisor calls that take longer than ten seconds (image install,
  restart) no longer time out in Home Assistant's relay.

## 0.3.0

- Image pulled from `registry.lurk.site` with a per-unit credential the hub
  registers itself at enrollment; the Supervisor updates the add-on automatically.
- The Home Assistant owner password is rotated and all sessions and extra users
  are purged the moment the unit enrolls, and again on release.
- The Ingress bind accepts only the Supervisor; a stale enrollment token is
  dropped on the first refusal.

## 0.2.0

- Factory station script: onboarding, mint, enrollment and card in one run.
- The Home Assistant owner login travels with the enrollment and is rotated on release.

## 0.1.0

- First milestone: activation, MQTT sync, fleet updates, local API, Ingress panel.
