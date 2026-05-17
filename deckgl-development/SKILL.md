---
name: deckgl
description: Build high-performance geospatial and data visualizations using deck.gl. Use this skill whenever the user mentions deck.gl, wants to visualize geographic data on a map (points, paths, polygons, heatmaps, hexbins, arcs, tiles), needs WebGL-powered large-scale data rendering, or asks for map overlays with interactivity. Trigger for terms like "scatter map", "geospatial viz", "hexagon layer", "GeoJSON map", "trip animation", "map tiles", or "data on a map". Also trigger when the user wants to upgrade from Leaflet or Mapbox GL JS to something more performant.
---

# deck.gl Skill

deck.gl is a WebGL2-powered framework for visual exploration of large-scale geospatial and non-geospatial datasets. It renders hundreds of millions of data points at 60fps via GPU instancing.

## Key Concepts

- **Layers**: Core abstraction. Each layer takes an array (or URL) of data objects and renders them via GPU.
- **DeckGL component**: The React wrapper around the `Deck` JS class. Always use `DeckGL` from `@deck.gl/react`, never the raw `Deck` class in React apps.
- **ViewState**: Represents the camera (longitude, latitude, zoom, pitch, bearing for map views).
- **Accessors**: Functions (or constants) on each layer that map a datum to a visual property (position, color, radius, etc.). Changing accessor *identity* does not trigger re-render — deck.gl intentionally ignores function reference changes for performance.
- **Picking**: Built-in hover/click hit-testing. Use `pickable: true` and `onHover`/`onClick` callbacks.

---

## Installation

```bash
# Full bundle (recommended for most apps)
npm install deck.gl

# Modular install (for tree-shaking)
npm install @deck.gl/core @deck.gl/layers @deck.gl/react

# With a base map (MapLibre — free, no token required)
npm install react-map-gl maplibre-gl
```

For Mapbox base maps, use `react-map-gl/mapbox` and provide a `MAPBOX_TOKEN`.

---

## Basic React Setup

```jsx
import { DeckGL } from '@deck.gl/react';
import { Map } from 'react-map-gl/maplibre';
import { ScatterplotLayer } from '@deck.gl/layers';
import 'maplibre-gl/dist/maplibre-gl.css';

const INITIAL_VIEW_STATE = {
  longitude: -122.45,
  latitude: 37.78,
  zoom: 12,
  pitch: 0,
  bearing: 0,
};

const FREE_MAP_STYLE = 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json';

export default function App({ data }) {
  const layers = [
    new ScatterplotLayer({
      id: 'scatter',
      data,
      getPosition: d => d.coordinates,
      getRadius: d => Math.sqrt(d.exits),
      getFillColor: [255, 228, 0],
      radiusScale: 6,
      pickable: true,
      onHover: ({ object }) => console.log('Hovered:', object),
    }),
  ];

  return (
    <DeckGL
      initialViewState={INITIAL_VIEW_STATE}
      controller={true}
      layers={layers}
    >
      <Map mapStyle={FREE_MAP_STYLE} />
    </DeckGL>
  );
}
```

> **CRITICAL**: The `DeckGL` component must fill its container. Set `width: 100%; height: 100%` on a parent div, or set the container to `position: relative` with explicit dimensions. Without this the canvas won't appear.

---

## Layer Catalog

See `references/layers.md` for full props reference. Summary:

### Core / @deck.gl/layers
| Layer | Use case |
|-------|----------|
| `ScatterplotLayer` | Points/circles at lat/lng |
| `LineLayer` | Lines between source and target positions |
| `ArcLayer` | Curved arcs between two points |
| `PathLayer` | Polylines / routes |
| `PolygonLayer` | Filled + stroked polygons |
| `SolidPolygonLayer` | Filled polygons only (no stroke, faster) |
| `GeoJsonLayer` | Auto-routes to sub-layers by geometry type |
| `IconLayer` | Custom icons/markers at points |
| `TextLayer` | Text labels at positions |
| `ColumnLayer` | 3D extruded columns at positions |

### Aggregation / @deck.gl/aggregation-layers
| Layer | Use case |
|-------|----------|
| `HexagonLayer` | Hexbin aggregation with elevation |
| `GridLayer` | Square grid aggregation |
| `HeatmapLayer` | Smooth density heatmap (GPU) |
| `ScreenGridLayer` | Screen-space grid aggregation |
| `ContourLayer` | Contour/isoline density |

### Geo / @deck.gl/geo-layers
| Layer | Use case |
|-------|----------|
| `TileLayer` | Slippy map tiles (raster or vector) |
| `MVTLayer` | Mapbox Vector Tile protocol |
| `H3HexagonLayer` | Uber's H3 spatial index |
| `TripsLayer` | Animated trajectories over time |
| `TerrainLayer` | 3D terrain from elevation tiles |
| `GreatCircleLayer` | Great-circle arcs on globe |

### Mesh / @deck.gl/mesh-layers
| Layer | Use case |
|-------|----------|
| `SimpleMeshLayer` | Custom 3D mesh per data point |
| `ScenegraphLayer` | glTF models per data point |

---

## Common Layer Props

Every layer accepts these base props:

```js
{
  id: 'unique-string',           // Required — used to reconcile layers
  data: arrayOrUrl,              // Array of objects, URL string, or Promise
  visible: true,                 // Toggle without unmounting
  opacity: 1,                    // 0–1
  pickable: false,               // Enable hover/click picking
  autoHighlight: false,          // Highlight hovered item automatically
  highlightColor: [0,0,128,128], // Color of auto-highlight
  onHover: ({object, x, y}) => {},
  onClick: ({object, x, y}) => {},
  updateTriggers: {},            // Force accessor re-evaluation (see below)
  transitions: {},               // Animate prop changes
  extensions: [],                // BrushingExtension, DataFilterExtension, etc.
}
```

---

## Accessor Pattern & updateTriggers

Accessors are functions evaluated per-datum. deck.gl intentionally ignores function reference changes. To signal that accessor results have changed (e.g. color scale changed), use `updateTriggers`:

```js
new ScatterplotLayer({
  id: 'scatter',
  data,
  getFillColor: d => colorScale(d.value),  // function changes every render
  updateTriggers: {
    getFillColor: [colorScaleParam],  // re-run accessor when this changes
  },
})
```

---

## Tooltips & Picking

```jsx
<DeckGL
  layers={layers}
  getTooltip={({ object }) =>
    object && { html: `<b>${object.name}</b><br/>${object.value}` }
  }
/>
```

Or manage state manually:

```jsx
const [hovered, setHovered] = useState(null);

const layer = new ScatterplotLayer({
  pickable: true,
  onHover: ({ object, x, y }) => setHovered(object ? { object, x, y } : null),
});
```

---

## Controlled vs Uncontrolled View State

**Uncontrolled** (recommended for simple apps):
```jsx
<DeckGL initialViewState={INITIAL_VIEW_STATE} controller />
```

**Controlled** (needed when you want to programmatically fly to a location):
```jsx
const [viewState, setViewState] = useState(INITIAL_VIEW_STATE);

<DeckGL
  viewState={viewState}
  onViewStateChange={({ viewState }) => setViewState(viewState)}
  controller
/>
```

To fly to a new location, update `viewState` with `transitionDuration` and a `transitionInterpolator`:
```js
import { FlyToInterpolator } from '@deck.gl/core';

setViewState({
  longitude: -87.65,
  latitude: 41.85,
  zoom: 11,
  transitionDuration: 1000,
  transitionInterpolator: new FlyToInterpolator(),
});
```

---

## Map Base Layer Options

| Option | Package | Notes |
|--------|---------|-------|
| MapLibre (free) | `react-map-gl/maplibre` + `maplibre-gl` | Use CartoBasemaps, no token needed |
| Mapbox | `react-map-gl/mapbox` | Requires `MAPBOX_TOKEN` |
| Google Maps | `@vis.gl/react-google-maps` + `@deck.gl/google-maps` | GoogleMapsOverlay or DeckGL wrapping |
| No base map | — | Works for non-geo visualizations |

Free CartoBasemap styles:
- `https://basemaps.cartocdn.com/gl/positron-gl-style/style.json` (light)
- `https://basemaps.cartocdn.com/gl/dark-matter-gl-style/style.json` (dark)
- `https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json` (color)

---

## Performance Tips

1. **Use `data` as a URL string** when possible — deck.gl fetches and caches automatically.
2. **Binary data**: For millions of points, pass typed arrays (`Float32Array`) via `data.attributes`. Avoid JSON for huge datasets.
3. **Layer ID stability**: Use stable IDs (`'scatter-layer'`, not `'layer-' + Math.random()`). Mismatched IDs cause full GPU re-initialization.
4. **Avoid recreating data arrays** in render. Memoize with `useMemo`.
5. **`visible: false`** is cheaper than conditional rendering for toggling layers.
6. **`radiusMinPixels` / `lineWidthMinPixels`**: Prevent features from becoming invisible at high zoom.

---

## Extensions

Extensions add capabilities to existing layers:

```js
import { DataFilterExtension } from '@deck.gl/extensions';

new ScatterplotLayer({
  data,
  getFilterValue: d => d.timestamp,
  filterRange: [startTime, endTime],
  extensions: [new DataFilterExtension({ filterSize: 1 })],
})
```

Common extensions:
- `DataFilterExtension` — GPU-side data filtering by numeric range
- `BrushingExtension` — Highlight points near the cursor
- `MaskExtension` — Clip layers to a geofence polygon
- `PathStyleExtension` — Dashed/offset path lines

---

## Views

deck.gl supports multiple views and non-map views:

```js
import { MapView, OrbitView, OrthographicView, GlobeView } from '@deck.gl/core';

// Globe view
<DeckGL
  views={new GlobeView()}
  initialViewState={{ longitude: 0, latitude: 0, zoom: 1 }}
  layers={layers}
/>

// Non-geospatial orbit view (3D scatter, network graphs)
<DeckGL
  views={new OrbitView()}
  initialViewState={{ target: [0, 0, 0], rotationX: 30, zoom: 5 }}
  layers={layers}
/>
```

---

## Vanilla JS (non-React)

```js
import { Deck } from '@deck.gl/core';
import { ScatterplotLayer } from '@deck.gl/layers';

const deck = new Deck({
  canvas: 'deck-canvas',
  initialViewState: { longitude: -122.45, latitude: 37.78, zoom: 12 },
  controller: true,
  layers: [new ScatterplotLayer({ data, getPosition: d => d.coords })],
});
```

---

## References

- `references/layers.md` — Full props for common layers (ScatterplotLayer, GeoJsonLayer, HexagonLayer, TileLayer, TripsLayer)
- `references/performance.md` — Performance should be a priority.
- Official docs: https://deck.gl/docs
- Layer catalog: https://deck.gl/docs/api-reference/layers
- Examples gallery: https://deck.gl/examples
