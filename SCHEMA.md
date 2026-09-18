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

`V` marks a field that appears only on owner-verified networks. The normative
version is
[`schemas/network.schema.json`](https://github.com/WiFiShare/spec/blob/main/schemas/network.schema.json).

| Field | Type | | Meaning |
| --- | --- | --- | --- |
| `geometry.coordinates` | `[lon, lat]` | | Exact position for a verified network. For a community-found network, the centre of the geohash-7 cell, so it is not a measurement of where the router is. |
| `properties.id` | string | | A random 12-character id. It is not derived from the BSSID, the name or the position, and it is not reused. |
| `properties.ssid` | string | | The network name, as broadcast. |
| `properties.security` | `open`, `owe` or `shared` | | `shared` means the owner published a credential for an encrypted network. |
| `properties.captive_portal` | `unknown`, `detected` or `none` | | Whether joining is likely to land on a login page. |
| `properties.verification` | `community` or `owner-verified` | | Decides everything in the table above. |
| `properties.cell` | string | | The geohash-7 cell the network sits in. For a community-found network this cell, not the point, is the real precision. |
| `properties.precision_m` | `150` or `10` | | Nominal position precision in metres. Read this before treating the coordinates as exact. |
| `properties.first_seen`, `properties.last_seen` | `YYYY-MM-DD` | | Day precision only, never a time of day. |
| `properties.reports` | object | | `{ "works": n, "fails": n }`, how people found it. |
| `properties.bssids` | array of string | V | The access point hardware addresses. Never present for a community-found network. |
| `properties.venue` | object | V | `{ "name": ..., "kind": ... }` as given by the owner. |
| `properties.credential` | object | V | Present only when the owner chose to publish it: `{ "type": ..., "secret": ..., "note": ... }`. |

## Reading the data safely

- Do not treat a community-found coordinate as a location. It is a cell centre.
  Use `precision_m` and `geohash`, not the point, when precision matters.
- Do not assume `bssid` is present. Code that requires it will fail on almost
  every feature, because almost every feature is community-found.
- Do not join this data against another database in a way that would restore the
  precision the filter removed, or that would re-identify a person. The three
  privacy rules apply to what you build on the dump as much as to the project
  itself, and the ODbL's share-alike terms apply to the result.
