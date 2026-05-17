# deck.gl Layer Props Reference

Detailed props for the most commonly used layers. All layers also accept the base props listed in SKILL.md.

Colors are `[R, G, B]` or `[R, G, B, A]` where values are 0–255. Can also be a function `d => [r, g, b, a]`.

---

## ScatterplotLayer

**Import**: `import { ScatterplotLayer } from '@deck.gl/layers'`

```js
new ScatterplotLayer({
  id: 'scatter',
  data,

  // Accessors
  getPosition: d => [d.lng, d.lat],     // Required: [lng, lat] or [lng, lat, alt]
  getRadius: d => d.radius,             // Default: 1 (in meters or pixels)
  getFillColor: d => [255, 0, 0],       // Default: [0, 0, 0, 255]
  getLineColor: [0, 0, 0],              // Default: [0, 0, 0, 255]
  getLineWidth: 1,

  // Rendering options
  filled: true,
  stroked: false,
  radiusUnits: 'meters',                // 'meters' | 'pixels' | 'common'
  radiusScale: 1,                       // Multiplier on getRadius
  radiusMinPixels: 0,                   // Minimum screen size
  radiusMaxPixels: Infinity,
  lineWidthUnits: 'meters',
  lineWidthScale: 1,
  lineWidthMinPixels: 0,
  lineWidthMaxPixels: Infinity,
  billboard: false,                     // If true, circle always faces camera (3D mode)
  antialiasing: true,
})
```

---

## GeoJsonLayer

**Import**: `import { GeoJsonLayer } from '@deck.gl/layers'`

Accepts GeoJSON `FeatureCollection`, `Feature`, raw `Geometry`, or a URL. Automatically renders Points → ScatterplotLayer, LineStrings → PathLayer, Polygons → PolygonLayer.

```js
new GeoJsonLayer({
  id: 'geojson',
  data: 'https://example.com/data.geojson', // URL or GeoJSON object

  // Points
  pointType: 'circle',                  // 'circle' | 'icon' | 'text'
  getPointRadius: 4,                    // meters

  // Lines
  getLineColor: [0, 0, 0, 200],
  getLineWidth: 2,

  // Polygons
  filled: true,
  stroked: true,
  extruded: false,
  getFillColor: f => [f.properties.color_r, f.properties.color_g, 0, 180],
  getElevation: f => f.properties.height,  // Used when extruded: true

  wireframe: false,                     // Show edges of extruded polygons
  elevationScale: 1,

  pickable: true,
  autoHighlight: true,
})
```

---

## ArcLayer

**Import**: `import { ArcLayer } from '@deck.gl/layers'`

```js
new ArcLayer({
  id: 'arcs',
  data,
  getSourcePosition: d => [d.from.lng, d.from.lat],
  getTargetPosition: d => [d.to.lng, d.to.lat],
  getSourceColor: [255, 0, 128],
  getTargetColor: [0, 128, 255],
  getWidth: 3,
  widthUnits: 'pixels',
  greatCircle: false,                   // true for globe view
  numSegments: 50,                      // Arc smoothness
})
```

---

## PathLayer

**Import**: `import { PathLayer } from '@deck.gl/layers'`

```js
new PathLayer({
  id: 'paths',
  data,
  // Each datum should have a path: array of [lng, lat] or [lng, lat, alt]
  getPath: d => d.waypoints,
  getColor: d => d.color,
  getWidth: 3,
  widthUnits: 'pixels',
  widthMinPixels: 1,
  capRounded: true,
  jointRounded: true,
  billboard: false,
})
```

---

## PolygonLayer

**Import**: `import { PolygonLayer } from '@deck.gl/layers'`

```js
new PolygonLayer({
  id: 'polygons',
  data,
  // Each datum: polygon is array of [lng, lat] rings (first is outer, rest are holes)
  getPolygon: d => d.contour,
  getFillColor: d => [d.value * 5, 65, 123, 180],
  getLineColor: [255, 255, 255],
  getLineWidth: 1,
  getElevation: d => d.population / 1000,
  extruded: true,
  wireframe: false,
  filled: true,
  stroked: true,
  elevationScale: 1,
})
```

---

## HexagonLayer

**Import**: `import { HexagonLayer } from '@deck.gl/aggregation-layers'`

Aggregates point data into hexagonal bins. Does NOT require pre-binned data.

```js
new HexagonLayer({
  id: 'hexagon',
  data,
  getPosition: d => [d.lng, d.lat],
  radius: 1000,                         // Hexagon radius in meters
  elevationScale: 4,
  extruded: true,
  pickable: true,

  // Color scale
  colorRange: [
    [1, 152, 189],
    [73, 227, 206],
    [216, 254, 181],
    [254, 237, 177],
    [254, 173, 84],
    [209, 55, 78],
  ],
  colorDomain: null,                    // Auto-computed from data if null
  colorAggregation: 'SUM',             // 'SUM' | 'MEAN' | 'MIN' | 'MAX' | 'COUNT'

  // Elevation
  elevationRange: [0, 3000],
  elevationDomain: null,
  elevationAggregation: 'SUM',

  getColorWeight: d => d.value,         // Value used for color aggregation
  getElevationWeight: d => d.value,
})
```

---

## HeatmapLayer

**Import**: `import { HeatmapLayer } from '@deck.gl/aggregation-layers'`

GPU-accelerated density heatmap. Ideal for dense point clouds.

```js
new HeatmapLayer({
  id: 'heatmap',
  data,
  getPosition: d => [d.lng, d.lat],
  getWeight: d => d.weight,            // Default: 1
  radiusPixels: 60,                    // Blur radius in screen pixels
  intensity: 1,
  threshold: 0.03,                     // Min relative density to show (0–1)
  colorRange: [
    [255, 255, 178],
    [254, 204, 92],
    [253, 141, 60],
    [240, 59, 32],
    [189, 0, 38],
  ],
  aggregation: 'SUM',                  // 'SUM' | 'MEAN'
})
```

---

## TileLayer

**Import**: `import { TileLayer } from '@deck.gl/geo-layers'`

Renders slippy map tiles. Use for custom tile servers, raster imagery, or when you want deck.gl layers without a base map library.

```js
import { BitmapLayer } from '@deck.gl/layers';

new TileLayer({
  id: 'tiles',
  data: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
  minZoom: 0,
  maxZoom: 19,
  tileSize: 256,

  renderSubLayers: props => {
    const { bbox: { west, south, east, north } } = props.tile;
    return new BitmapLayer(props, {
      data: null,
      image: props.data,
      bounds: [west, south, east, north],
    });
  },
})
```

---

## TripsLayer

**Import**: `import { TripsLayer } from '@deck.gl/geo-layers'`

Animates trajectories over time with a trailing tail effect.

```js
// Data format: each object has a `waypoints` array of {coordinates, timestamp}
new TripsLayer({
  id: 'trips',
  data,
  getPath: d => d.waypoints.map(p => p.coordinates),
  getTimestamps: d => d.waypoints.map(p => p.timestamp),
  getColor: d => d.vendor === 0 ? [253, 128, 93] : [23, 184, 190],
  currentTime: animationTime,          // Drives which part of the trip is visible
  trailLength: 180,                    // How many time units the trail persists
  widthMinPixels: 2,
  capRounded: true,
})
```

Animate with `requestAnimationFrame`:
```js
const [time, setTime] = useState(0);
useEffect(() => {
  let raf;
  const animate = () => {
    setTime(t => (t + 1) % LOOP_LENGTH);
    raf = requestAnimationFrame(animate);
  };
  raf = requestAnimationFrame(animate);
  return () => cancelAnimationFrame(raf);
}, []);
```

---

## H3HexagonLayer

**Import**: `import { H3HexagonLayer } from '@deck.gl/geo-layers'`

Renders pre-aggregated data using Uber's H3 spatial indexing system.

```js
new H3HexagonLayer({
  id: 'h3-hexagon',
  data,
  getHexagon: d => d.h3index,          // H3 cell ID string
  getFillColor: d => [255, (1 - d.value / maxValue) * 255, 0],
  getElevation: d => d.value,
  extruded: true,
  elevationScale: 20,
  filled: true,
  stroked: false,
})
```

---

## IconLayer

**Import**: `import { IconLayer } from '@deck.gl/layers'`

```js
// Using an icon atlas (spritesheet)
new IconLayer({
  id: 'icons',
  data,
  iconAtlas: '/path/to/atlas.png',
  iconMapping: {
    marker: { x: 0, y: 0, width: 128, height: 128, mask: true },
  },
  getIcon: d => 'marker',
  getPosition: d => [d.lng, d.lat],
  getSize: 40,
  getColor: [255, 0, 0],
  sizeUnits: 'pixels',
  sizeScale: 1,
  sizeMinPixels: 10,
  sizeMaxPixels: 60,
  billboard: true,
})
```

---

## TextLayer

**Import**: `import { TextLayer } from '@deck.gl/layers'`

```js
new TextLayer({
  id: 'labels',
  data,
  getPosition: d => [d.lng, d.lat],
  getText: d => d.label,
  getSize: 14,
  getColor: [0, 0, 0, 200],
  getAngle: 0,
  getTextAnchor: 'middle',             // 'start' | 'middle' | 'end'
  getAlignmentBaseline: 'center',      // 'top' | 'center' | 'bottom'
  fontFamily: 'Monaco, monospace',
  fontWeight: 'normal',
  billboard: true,
  sizeUnits: 'pixels',
  sizeScale: 1,
  background: false,
  getBackgroundColor: [255, 255, 255, 200],
  getBorderColor: [0, 0, 0, 200],
  getBorderWidth: 0,
  backgroundPadding: [4, 2, 4, 2],
})
```

---

## Coordinate Systems

```js
import { COORDINATE_SYSTEM } from '@deck.gl/core';

// Geospatial (default): [lng, lat] or [lng, lat, alt_meters]
coordinateSystem: COORDINATE_SYSTEM.LNGLAT

// Meter offsets from an origin point
coordinateSystem: COORDINATE_SYSTEM.METER_OFFSETS
coordinateOrigin: [lng, lat]

// Cartesian (non-geo, e.g. 3D scatter)
coordinateSystem: COORDINATE_SYSTEM.CARTESIAN
```

---

## Transitions

Smoothly animate between layer states:

```js
new ScatterplotLayer({
  data,
  getRadius: d => d.value,
  transitions: {
    getRadius: { duration: 500, easing: d3.easeCubicInOut },
    getFillColor: 300,  // shorthand for { duration: 300 }
  },
})
```

Supported: `getPosition`, `getRadius`, `getFillColor`, `getLineColor`, `getLineWidth`, `getElevation`, and more depending on layer.
