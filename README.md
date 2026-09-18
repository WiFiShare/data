# WiFiShare public dump

The public dump of the WiFiShare database: free and open Wi-Fi networks that
people and venues have shared, as GeoJSON files split by area.

## Status

Early development, started September 2026. **No data has been published yet.**
This repository is the scaffolding for the dump: the layout, the schema and the
license are fixed first, the data follows in P1. `index.json` currently
describes an empty dump and `areas/` is empty.

## Directory layout

Networks are split into per-area files, one file per geohash-5 cell, nested
under the geohash-2 and geohash-3 prefixes of that same geohash:

```
areas/<gh2>/<gh3>/<gh5>.geojson
```

The three path segments are prefixes of one geohash, not three separate values.
`<gh3>` always starts with `<gh2>`, and `<gh5>` always starts with `<gh3>`. The
nesting exists so that no single directory ends up with tens of thousands of
entries.

A geohash-5 cell is about 4.9 km across.

Worked example. Piazza Maggiore in Bologna is at 44.4938, 11.3426. Its geohash
is `srbj45g...`, so:

| Segment | Value |
| --- | --- |
| geohash-2 | `sr` |
| geohash-3 | `srb` |
| geohash-5 | `srbj4` |

and the file for that area is:

```
areas/sr/srb/srbj4.geojson
```

## Finding the file for a coordinate

1. Encode the coordinate as a geohash of at least 5 characters.
2. Take the first 5 characters. That is the file name, `<gh5>.geojson`.
3. Take the first 2 and the first 3 characters of that same geohash. Those are
   the two directories above it.
4. Open `areas/<gh2>/<gh3>/<gh5>.geojson`. If the file does not exist, nothing
   is published for that area.

Check the eight neighbouring cells too if you are searching near a cell edge. A
network just across the boundary lives in the neighbour's file.

`index.json` lists the areas that exist, so you can fetch that first instead of
probing for files. Its shape is:

```json
{"schema": "wifishare.index/1", "generated": null, "network_count": 0, "area_count": 0, "areas": {}}
```

`generated` is null and the counts are zero while the dump is empty.

## What is in a file

Each file is a GeoJSON `FeatureCollection`. Each `Feature` is one network, with
a `Point` geometry and properties describing how to find and join it. The fields
are listed in [SCHEMA.md](SCHEMA.md). The normative definition lives in the
[spec](https://github.com/WiFiShare/spec) repository, together with the JSON
Schemas and the test fixtures; if this repository and `spec` disagree, `spec` is
right.

Two things are worth stating here rather than only in the schema:

- **BSSIDs of community-found networks are never published.** A network found by
  someone walking past is published without its BSSID, and with its position
  reduced to a geohash-7 cell, about 150 m. Only networks verified by their
  owner are published with an exact position and with BSSIDs.
- **Mobile networks are never published.** A network seen more than 1 km apart
  is treated as a phone hotspot or a vehicle and is dropped, not published at a
  coarser precision.

Networks whose SSID ends in `_nomap` or `_optout` are never collected, so they
never reach this repository. Private password-protected networks are not
collected at all unless their owner shares them deliberately.

## Cadence

| What | When | How |
| --- | --- | --- |
| Area files | Weekly | Committed by a bot |
| Snapshot | Monthly | Published as a GitHub release |

Use the monthly release if you want a stable, citable version. Use the weekly
files if you want the current state.

## History is squashed monthly

Git history in this repository is squashed once a month, at the snapshot. This
is deliberate: a removed network must not stay readable in an old commit forever.
If someone renames their network to `_nomap`, or asks for it to be taken out, the
removal has to be real, and history that keeps a copy would defeat that.

The practical consequences for you:

- Commit hashes in this repository are not stable across a squash. Do not pin to
  one.
- `git pull` may refuse to fast-forward after a squash. Re-clone, or reset to
  the remote branch.
- If you need a fixed version to refer to, use a monthly release, not a commit.

## Reporting a problem in the data

Open a `Network problem` issue on this repository: a network that no longer
exists, one that cannot be joined, a wrong position, or a private network that
should not be listed. Do not include a BSSID or a password; issues are public.

If the network is yours and you want it left out for good, you do not need an
issue and you do not need an account. Rename it so its SSID ends in `_nomap` or
`_optout` and it will never be collected. Once an app is released it will also
offer one-tap removal.

## License

This dump is licensed under the
[Open Database License (ODbL) 1.0](LICENSE). You are free to share, adapt and
use it, including commercially, as long as you attribute it, keep any adapted
database under the same license, and do not use technical measures to restrict
others' use of it.

Attribution line to use:

```
Contains data from WiFiShare, available under the Open Database License (ODbL) 1.0.
```

The individual contents of the database, taken on their own, are not claimed
under ODbL. The license text in [LICENSE](LICENSE) governs; the summary above is
not a substitute for it.
