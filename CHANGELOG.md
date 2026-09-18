# Changelog

Changes to the published dump. Each weekly commit updates the area files; this
file records changes to the repository itself, to the layout and to the schema.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## Unreleased

### Added

- Repository created during P0 with its layout, schema summary, license and
  index in place.
- `areas/` directory for per-area GeoJSON files, currently empty.
- `index.json` describing an empty dump: no areas, no networks, `generated` set
  to null.

### Notes

- No data has been published yet. The first area files are expected in P1, when
  the data pipeline and API are built.
- No release has been made. The monthly snapshot releases begin with the first
  published data.
