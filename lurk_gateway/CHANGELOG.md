# Changelog

## Unreleased

## 0.3.3

- Cloud: MQTT keepalive drops from 30s to 15s. The broker declares a hub dead
  at 1.5x keepalive before publishing the last-will, so an unplugged unit now
  shows as offline in 22.5s instead of 45s.
- LAN API: the role in the retained auth document's `members[]` entry now
  overrides the edge token's `role` claim, so a role change made in the app
  reaches the LAN with the next document instead of the next token.

## 0.3.2

- Add-on: the optional `serial` no longer ships with a null default, which
  Home Assistant Supervisor rejected on real Raspberry Pi units.
- Station: Supervisor validation errors redact the one-time enrollment token.
- Station: the box check reads the whole `/health` answer; a body that arrived
  after the headers was dropped and reported as "this box says it is ?".

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
