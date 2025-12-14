# Random GeoPoints

A comprehensive, client-side web application for **scientifically rigorous geospatial sampling and accuracy validation**. Generate probability-based random samples, define validation protocols, and calculate Olofsson-style accuracy metrics—all in your browser.

![Random GeoPoints Screenshot](docs/screenshot.png)

## ✨ What's New (December 2024)

- 🎨 **Dark/Light Theme Toggle** with persistent preference
- 📊 **Cochran Sample Size Calculator** - determine optimal n from expected accuracy and margin of error
- 🗺️ **Strata Legend with Color Map** - visual per-stratum allocation
- 🎯 **True Stratified Sampling** - proportional and equal allocation by area (Cochran 1977)
- 📐 **Nearest Neighbor Index (NNI)** - real Clark & Evans spatial pattern analysis
- 📋 **Design Weights** - inclusion probability and weight on every output point
- 💡 **Scientific Info Tooltips** - hover for methodology citations

## Features

### 🎯 Tab 1: Sample Design

**Sampling Methods** (all probability-based):
| Method                          | Description                 | Use Case                    |
|---------------------------------|-----------------------------|-----------------------------|
| **Simple Random (BBox)**        | Uniform within bounding box | Quick baseline              |
| **Simple Random (Polygon)**     | Uniform within AOI polygon  | Standard reference          |
| **Spatially Balanced (Halton)** | Low-discrepancy sequence    | Reduced clumping            |
| **Stratified (Proportional)**   | n_h ∝ area of stratum h     | Optimize overall accuracy   |
| **Stratified (Equal)**          | Same n per stratum          | Rare class representation   |
| **Systematic Grid**             | Rectangular lattice         | Complete coverage           |
| **Hexagonal Lattice**           | Optimal packing geometry    | Field logistics             |
| **Cluster Sampling**            | Two-stage: centers + radius | Cost-effective remote areas |
| **Random Plots**                | Square area units           | Forest inventory            |

**Smart Features**:
- **Sample Size Calculator**: Enter expected accuracy (%) and target margin of error → get recommended n
- **Power Analysis Chart**: Visualize MOE trade-offs at different sample sizes
- **Strata Legend**: Upload strata GeoJSON → see color-coded proportions on map
- **NNI Diagnostic**: Real-time spatial pattern feedback (Clustered/Random/Dispersed)

### 📋 Tab 2: Protocol Definition

Define your response design (Stehman & Czaplewski 1998):
- Minimum Mapping Unit (MMU) rules
- Labeling confidence thresholds  
- Edge case handling instructions
- Field data schema preview

Export protocol as text for field teams.

### 📊 Tab 3: Accuracy Analysis

Upload labeled validation results (CSV or GeoJSON) to calculate:
- **Overall Accuracy** with standard error
- **Confusion Matrix** (map vs reference)
- **User's Accuracy** per class
- **Producer's Accuracy** per class

Based on **Olofsson et al. (2014)** methodology.

## Quick Start

1. Open `index.html` in any modern browser (no server needed)
2. Upload an AOI (GeoJSON polygon) or draw bounding box on map
3. *Optional*: Upload strata file for stratified sampling
4. Select sampling method and configure parameters
5. Click **Generate Sample**
6. Export as GeoJSON (with metadata) or CSV

## GeoJSON Output Format

```json
{
  "type": "FeatureCollection",
  "metadata": {
    "generatedAt": "2024-12-14T12:00:00Z",
    "seed": "12345",
    "method": "stratified-prop",
    "targetN": 100,
    "actualN": 100,
    "aoiAreaKm2": "542.30"
  },
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-73.5, 45.2] },
      "properties": {
        "id": 1,
        "method": "stratified-prop",
        "inclusion_prob": "0.010000",
        "weight": "100.00",
        "stratum": "Forest",
        "stratum_weight": "0.4523"
      }
    }
  ]
}
```

## Theme Support

Toggle between **Light** and **Dark** modes using the header button. Your preference is saved to localStorage.

| Mode  | Interface          | Basemap           |
|-------|--------------------|-------------------|
| Light | Clean white panels | CARTO Positron    |
| Dark  | Slate blue panels  | CARTO Dark Matter |

## Technical Stack

- **Leaflet + Leaflet.draw**: Interactive mapping and AOI drawing
- **Turf.js**: Client-side geospatial operations (area, point-in-polygon, centroid)
- **Mulberry32 RNG**: Seedable random number generator for reproducibility
- **100% Client-side**: Zero data leaves your browser

## Scientific Foundation

This tool implements probability sampling designs as described in:

| Reference                       | Topic                                      |
|---------------------------------|--------------------------------------------|
| **Olofsson et al. (2014)**      | Accuracy assessment good practices         |
| **Cochran (1977)**              | Sampling techniques, stratified allocation |
| **Clark & Evans (1954)**        | Nearest Neighbor Index                     |
| **Stehman & Czaplewski (1998)** | Response design principles                 |
| **Congalton & Green (2019)**    | Accuracy assessment methods                |

### Key Principles

1. **Probability Sampling**: Every location has a known, non-zero inclusion probability
2. **Reproducibility**: Seeded RNG ensures repeatable samples  
3. **Design Weights**: All outputs include `inclusion_prob` and `weight` for proper inference
4. **Spatial Diagnostics**: NNI feedback helps detect clustering issues

## Development

```bash
# Lint check
npx htmlhint index.html

# Serve locally
npx serve .
```

See `dev/FULL_DEVELOPMENT_PLAN.md` for the complete roadmap.

## Color Scheme

| Element         | Light                           | Dark               |
|-----------------|---------------------------------|--------------------|
| Primary         | Teal `#0d9488`                  | Cyan `#14b8a6`     |
| Accent (Points) | Purple `#7c3aed`                | Lavender `#a78bfa` |
| Strata          | Categorical palette (10 colors) | Same               |

## File Structure

```
randomgeopoints/
├── index.html          # Main application (single file)
├── README.md           # This file
├── dev/                # Development documents
│   ├── AGENTS.md
│   ├── FULL_DEVELOPMENT_PLAN.md
│   ├── DEVELOPMENT_PLAN.md
│   └── ideas.md
└── references/         # Literature and documentation
```

## License

MIT

## Author

Dr. Sounny
