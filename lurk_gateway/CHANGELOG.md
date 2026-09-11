# Changelog

## 0.3.17

- `home_state` now answers with sensors and binary sensors as well as the
  controllable domains, each with its name and room, read live from Home
  Assistant. Every tool call is logged by name.
- A `/v1/state` send that loses the race with its own eviction ends the
  connection quietly instead of logging a traceback.

## 0.3.16

- The voice relay answers Lurk's tool calls on the box: `home_state` from the
  LAN cache and the structure, `control_device` through the same path as the
  LAN control API, with the member's role enforced. The phone never sees the
  call (`docs/contract-voice.md` › Relay).
- A `/v1/state` socket evicted by its own handset's reconnect no longer logs a
  traceback: a send after the close now ends that connection as a plain
  disconnect.

## 0.3.15

- `WS /v1/voice`: the hub relays a phone's Gemini Live session on the LAN.
  Edge JWT bearer, a `start` frame carrying the minted socket URL, then an
  opaque relay — the hub never reads a frame. See `docs/contract-voice.md`.

## 0.3.14

- Build: `BUILD_VERSION` has no default in the Dockerfile. It was pinned at
  0.3.2 and eleven releases stale, so a locally built image was labelled
  `io.hass.version=0.3.2`. CI passes the tag and the Supervisor passes
  config.yaml's version, so the default was only ever wrong.
- Nothing changes for a unit: the published image already carried the right
  label. This is the first release cut by `scripts/release.sh`, and proving
  that path is the reason it exists.

## 0.3.13

- Updates: a failure always carries a reason. `str()` of an httpx timeout is
  the empty string, so the payload dropped the field and the cloud recorded
  `failed` with no error at all — which is how both of the updates that timed
  out on the old 20-second budget arrived, and why neither could be diagnosed
  without reading the box. Falls back to the exception's type name.

## 0.3.12

- Updates: the self-update asks Core to install the add-on's update entity, and
  Core does not answer until the image is pulled and the container replaced.
  That ran on the HA client's 20s read timeout, so a slow pull reported
  `failed` for an update that was still running — and clearing the in-progress
  flag killed the resume that would have sent the real result. The install now
  gets the same 20-minute budget the Supervisor call had.
- Cloud: `agent_version` is what the Supervisor reports installed, not a
  version declared in this source tree. The cloud's "is there an update" is
  that value against the store's, so the two have to come from the same place;
  a hand-maintained copy drifting either hides an update or offers one forever.

## 0.3.3 — 0.3.11

Getting a delivered unit to update itself from the app, with nothing typed on
the box. Nine releases because each fix could only be proved by publishing one
and watching a real unit take it.

- Updates: the cloud's "update now" never ran — the downlink handed the update
  id to a callback that took no arguments, and the listener's handler guard
  swallowed the TypeError, so every cloud-triggered update logged a failure and
  did nothing.
- Updates: every pass reloads the add-on store before comparing versions. The
  Supervisor's `update_available` is measured against its cached copy of the
  store, which refreshes on its own slow schedule, so a release published
  minutes earlier was invisible and both the daily self-check and the cloud's
  "update now" reported success having installed nothing.
- Updates: the self-update asked the Supervisor to update the slug `self`,
  which addresses this add-on for reads but does not exist in the store an
  update resolves against — every attempt failed with "App self does not exist
  in the store". The real slug is read from the add-on's own info.
- Updates: the ask then moved to Home Assistant. The Supervisor refuses an
  add-on's request to update itself ("App <slug> can't update itself!") because
  the update kills the container mid-request; the same ask from Core is
  allowed. Falls back to the Supervisor when Core has no matching entity.
- Updates: an update may carry `scope: "addon"`, which updates this add-on only
  and leaves Home Assistant OS and Core alone. The scope is persisted, so the
  restart the update itself causes resumes the same narrow pass instead of
  widening into a reboot, and the resume does not re-announce `started`.
- Updates: `auto_update` is switched off for this add-on at startup. A unit
  moves when its owner asks, not on Home Assistant's own schedule.
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
