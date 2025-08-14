# Random Geo Points

A single-page, client-side web app for generating random geographic points inside a user-defined area of interest (AOI). It runs entirely in the browser and requires no server.

## Features

- **AOI input**: Define a bounding box or upload a GeoJSON polygon/multipolygon.
- **Sampling methods**:
  - Uniform in bounding box
  - Uniform in polygon
  - Stratified grid with jitter
  - Poisson disk / blue-noise (minimum spacing in meters)
  - Hexagonal lattice
- **Controls**: Set number of points, random seed (repeatable), grid cell size and Poisson minimum distance.
- **Visualization**: Points appear on a Leaflet map for quick inspection.
- **Export**: Download results as GeoJSON or CSV or copy GeoJSON to the clipboard.
- **100% client-side**: No data leaves the browser.

## Development

Open `index.html` in any modern browser. To perform a basic HTML lint check:

```bash
npx htmlhint index.html
```

## License

MIT
