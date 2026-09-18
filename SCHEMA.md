# Published network schema

What one network looks like in a published area file.

**The normative definition is in the [spec](https://github.com/WiFiShare/spec)
repository**, together with the JSON Schemas and the test fixtures. This page is
a readable summary of it for people consuming the dump. If the two disagree,
`spec` is right. The field list is not final: the project is at P0 and nothing
has been published yet, so expect it to change until the first dump ships in P1.

## Shape

Each area file is a GeoJSON `FeatureCollection`. Each `Feature` is one network:

- `geometry` is a `Point`, in WGS84, longitude first, as GeoJSON requires.
- `properties` describes the network.

## Verification changes what is published

This is the most important thing to understand about the data, so it comes
before the field table.

| | Community-found | Owner-verified |
| --- | --- | --- |
| How it got here | Someone's app saw it while walking past | The network's owner shared it deliberately |
| Position | Reduced to a geohash-7 cell, about 150 m | Exact |
| BSSID | **Never published** | Published |

A community-found network is published only as "there is a network with this
name somewhere in this 150 m cell". That is enough to find and join it, and not
enough to place it at an address. Rule 2 of the project's three privacy rules is
that it publishes less than it collects, and this is where that rule does most of
its work.

Two exclusions happen before publication and so are never visible in the data:

- Networks whose SSID ends in `_nomap` or `_optout` are never collected at all.
- A network seen more than 1 km apart is treated as mobile and is never
  published, at any precision.

Private password-protected networks are not collected unless their owner shares
them deliberately, so a password-protected network in the dump is there because
its owner put it there.

## Fields

`V` marks a field that appears only on owner-verified networks.

| Field | Type | | Meaning |
| --- | --- | --- | --- |
| `geometry.coordinates` | `[lon, lat]` | | Exact position for a verified network. For a community-found network, a position derived from the geohash-7 cell only, so it is not a measurement of where the router is. |
| `properties.ssid` | string | | The network name, as broadcast. |
| `properties.verified` | boolean | | `true` if the owner verified the network. Decides everything in the table above. |
| `properties.geohash` | string | | The geohash cell the position stands for: 7 characters for a community-found network, and the cell is the real precision. |
| `properties.precision_m` | number | | Nominal position precision in metres. About 150 for a community-found network. Read this before treating the coordinates as exact. |
| `properties.bssid` | string | V | The access point's hardware address. Present only for owner-verified networks. Never present for a community-found one. |

Fields beyond these — how a network is secured, whether joining it needs a
captive portal or a code, what kind of venue it is, and what the apps need in
order to offer a one-tap join — are being worked out in `spec` and are not
settled yet. They are not listed here so that this page does not describe
something that has not been agreed. Read the schemas in `spec` for the current
state.

## Reading the data safely

- Do not treat a community-found coordinate as a location. It is a cell centre.
  Use `precision_m` and `geohash`, not the point, when precision matters.
- Do not assume `bssid` is present. Code that requires it will fail on almost
  every feature, because almost every feature is community-found.
- Do not join this data against another database in a way that would restore the
  precision the filter removed, or that would re-identify a person. The three
  privacy rules apply to what you build on the dump as much as to the project
  itself, and the ODbL's share-alike terms apply to the result.
