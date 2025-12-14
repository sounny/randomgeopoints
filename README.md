# Random GeoPoints

A comprehensive, client-side web application for geospatial sampling and accuracy validation. Generate probability-based random samples, define validation protocols, and calculate accuracy metrics—all in your browser.

![Light/Dark Theme Toggle](docs/theme-toggle.gif)

## Features

### 🎯 Tab 1: Sample Design

- **Sampling Methods**:
  - **Simple Random** (Bounding Box or Polygon)
  - **Spatially Balanced** (Halton sequence for reduced clumping)
  - **Systematic Grid** (rectangular lattice)
  - **Hexagonal Lattice** (optimal spatial packing)
  - **Stratified** (proportional or equal allocation by strata)
  - **Cluster Sampling** (two-stage: random centers + points within radius)
  - **Random Plots** (square area units with configurable size)

- **AOI Definition**: Upload GeoJSON polygon or draw bounding box on map
- **Strata Support**: Upload strata file for stratified sampling
- **Power Analysis Chart**: Visual sample size vs. margin of error
- **Design Metadata**: All outputs include method, seed, and unit information

### 📋 Tab 2: Protocol Definition

Define your validation response design:
- Minimum Mapping Unit (MMU) rules
- Labeling confidence thresholds
- Edge case handling instructions
- Field data schema preview

Export protocol as text for field teams.

### 📊 Tab 3: Accuracy Analysis

Upload labeled validation results to calculate:
- **Overall Accuracy** with confidence intervals
- **Confusion Matrix** (map vs reference)
- **Class-level metrics** (User's Accuracy, Producer's Accuracy)

Based on Olofsson et al. (2014) methodology.

## Quick Start

1. Open `index.html` in any modern browser
2. Upload an AOI (GeoJSON polygon) or draw on map
3. Select sampling method and configure parameters
4. Click "Generate Sample"
5. Export as GeoJSON or CSV

## Theme Support

Toggle between **Light** and **Dark** modes using the button in the header. Your preference is saved automatically.

- **Light Mode**: Clean white interface with light CARTO basemap
- **Dark Mode**: Slate blue interface with dark CARTO basemap

## Technical Stack

- **Leaflet + Leaflet.draw**: Map visualization and drawing
- **Turf.js**: Geospatial operations
- **100% Client-side**: No server required, no data leaves your browser

## Development

```bash
# Lint check
npx htmlhint index.html

# Serve locally (optional)
npx serve .
```

## Scientific Background

This tool implements probability sampling designs as described in:

- **Olofsson et al. (2014)**: Good practices for accuracy assessment
- **Cochran (1977)**: Sampling Techniques
- **Stehman & Czaplewski (1998)**: Response design principles
- **Congalton & Green (2019)**: Accuracy assessment methods

### Key Principles

1. **Probability Sampling**: Every unit has a known, non-zero inclusion probability
2. **Reproducibility**: Seeded RNG ensures repeatable samples
3. **Spatial Balance**: Options to reduce clustering (Halton, Hex lattice)
4. **Design-Based Inference**: Outputs include metadata for proper variance estimation

## Color Scheme

The application uses a professional, earth-toned palette:

| Element         | Light            | Dark               |
|-----------------|------------------|--------------------|
| Primary         | Teal `#0d9488`   | Cyan `#14b8a6`     |
| Accent (Points) | Purple `#7c3aed` | Lavender `#a78bfa` |
| Strata          | Amber `#d97706`  | Orange `#f59e0b`   |

## File Structure

```
randomgeopoints/
├── index.html          # Main application (single file)
├── README.md           # This file
├── DEVELOPMENT_PLAN.md # Feature roadmap
└── references/         # Literature and documentation
```

## License

MIT

## Author

Dr. Sounny
