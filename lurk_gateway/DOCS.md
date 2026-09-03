# Lurk Gateway

Local Home Assistant manager and the single channel to the Lurk cloud.

## Enrollment

Nothing is typed here in the normal case. The factory writes a one-time
**enrollment token** into this add-on's options. On first boot the add-on
generates a private key that never leaves the box, sends a signing request
with the unit's hardware serial, and receives a certificate signed by the Lurk
Device CA. From then on that certificate is the unit's identity: it
authenticates the hub to the broker (mutual TLS) and it is what the Lurk app
verifies when it connects over the local network.

The token is burned by the cloud on use. There is nothing to rotate, recover
or retype afterwards. The panel shows "Enrolling…" while the first boot is
working and keeps retrying if the network is not up yet.

The identity lives in `/addon_configs/lurk_gateway` (mounted as `/config`
inside the add-on): `hub.key`, `hub.crt`, `ca.pem`, `identity.json`, and
`registry.json` -- the login the Supervisor pulls this add-on's image with.
The cloud derives that login from the certificate key, so a reissue changes
it and a suspended or retired unit can no longer pull. The add-on writes it
into the Supervisor at enrollment and again on every start. Everything here
survives add-on updates and an uninstall. Only removing the add-on's
configuration explicitly, or reflashing the operating system, deletes it.

A token the cloud refuses on an already-enrolled unit is dropped at once
(tokens are one-time, so retrying could never succeed); the panel says so and
the unit keeps running on its existing identity until support issues a new
token.

The **claim code** on the welcome card is a different secret, and it decides
who OWNS the unit. It belongs in the customer's app, never on this screen.
Ownership is separate from enrollment: a hub can be online, updatable and
supportable while belonging to nobody, and it publishes no household data
until somebody claims it.

## The Home Assistant login

A factory-built unit is onboarded at the station with one Home Assistant
owner account, `lurk`, and a random password. Nobody at the factory keeps
it: the hub receives it once, inside the enrollment response, kept in
`/config/ha-owner.json` (readable by root only), and **immediately sets a
new random password, deletes every other Home Assistant login and every
account other than `lurk`**, then reports the new password to the cloud on
its first broker connection. Whatever the station -- or anyone on the
factory floor -- knew or created is dead seconds after enrollment. Whoever
owns the unit in the Lurk app can see the current login there and sign in
to Home Assistant directly.

On release the add-on does the same as the owner: a fresh random password,
handed to the cloud, and every other login and account removed, so the
previous household keeps nothing. If Home Assistant refuses, the release
still completes and the failure is reported to the cloud like any other
leftover. A unit provisioned by hand has no such file and nothing is rotated.

**Lost welcome card.** Support issues a new claim code from the fleet page
and prints a new card; the certificate and the enrollment are untouched, and
the old code stops working. Nothing is typed on the box.

## Recovery (only if the identity is gone)

After a full reflash the add-on has no key and no certificate. Support issues
a fresh enrollment token for this serial from the fleet page; the "Recovery"
card accepts it and the add-on enrolls again exactly as it did at the factory.
The unit keeps its owner — the certificate changed, the ownership record in
the cloud did not.

## When the cloud refuses the unit

If the broker refuses the certificate the panel says so. The add-on cannot
tell suspended from retired from reissued, and none of those is something it
can fix on its own: it keeps trying at a slow cadence and leaves the remedy to
whoever reads the fleet page. A serial mismatch (a backup restored onto a
different board) quarantines the stored identity and waits for a token.

## Release

Releasing hands the unit to somebody else. The cloud detaches every member,
publishes an empty auth document, and sends a one-shot reset. The add-on then
deletes the previous household's Home Assistant data — integrations (and with
them the Zigbee network and any camera configuration), areas, floors, labels,
people, automations, scenes and scripts — and forgets its own caches. The key
and certificate stay: same unit, new owner. The next owner claims it with the
same welcome card.

**What a release does not do.** An add-on cannot factory-reset Home Assistant
OS. It clears everything reachable through Home Assistant's own API and
nothing below it — no disk wipe, no OS-level state, no other add-ons' data.
That covers a private resale between people who broadly trust each other. A
unit going to a stranger, or leaving a business, should be reflashed.

Two ways to run it:

- **The Lurk app** — the owner removes the hub, and the cloud commands the
  reset. Use this whenever the unit is reachable.
- **The Lurk panel in Home Assistant** — reaching it takes a Home Assistant
  admin session on the machine itself, which is the closest thing to a
  physical button hold that software offers. It is deliberately unreachable
  from the LAN: a credential good enough to turn on a light must not also be
  good enough to erase the house.
