# The Dons — Baldwin Hills 3D Map

Interactive 3D map of "The Dons", a neighborhood in Baldwin Hills, Los Angeles. Named after its 29 streets that all begin with the Spanish honorific "Don", this area sits on a hillside between La Brea and La Cienega with elevations ranging from 33m to 153m.

The map includes the Don streets plus a section of Hillcrest Drive from Don Milagro Drive down to 4150 Hillcrest Drive.

## Usage

Serve the project with any static HTTP server and open in a browser:

```sh
npx serve
# or
python3 -m http.server 8080
```

### Controls

- **Drag** — rotate the view
- **Shift + drag** (or right-drag) — pan. Shift can be pressed/released mid-drag to toggle between rotate and pan
- **Scroll** — zoom in/out
- **Hover** — tooltip shows the street address for houses or street name for roads
- **Escape** — toggle the side panel

### Visualization modes

- **Default** — all houses in purple, streets in white
- **Color by Street** — houses and streets colored by their street, with a legend
- **Color by Elevation** — blue (low) to red (high)
- **Density Heatmap** — colored by number of nearby buildings

### Layers

Toggle visibility of terrain, streets, major roads, houses, and street labels independently.

## Data sources

- **Street geometry** — [OpenStreetMap](https://www.openstreetmap.org/) via Overpass API. 29 Don streets plus Hillcrest Drive, queried by name prefix within the Baldwin Hills bounding box
- **Building footprints** — [OpenStreetMap](https://www.openstreetmap.org/) building polygons, matched to parcel addresses by spatial containment. Where no OSM building exists, an inset (60%) of the parcel polygon is used as a fallback footprint
- **Parcel addresses** — [LA County Assessor](https://public.gis.lacounty.gov/) via ArcGIS REST API. 1,147 parcels with situs addresses on Don streets and Hillcrest Drive (4150+)
- **Elevation** — [Open-Meteo Elevation API](https://open-meteo.com/). 425-point grid at 0.001-degree spacing (~100m), used for both terrain mesh and street elevation

## How it works

A single `index.html` file loads `map_data.json` and renders everything with [Three.js](https://threejs.org/) (r164):

- **Terrain** — `PlaneGeometry` with vertices displaced by elevation data, wrapped in a solid block with side walls and a bottom face. Vertex colors shade from dark green (low) to lighter green (high)
- **Streets** — linearly subdivided polylines (~5m spacing), elevated above terrain using bilinear interpolation with a 3m offset and triangle-kernel smoothing. Rendered as extruded slabs (1m thick) with `MeshBasicMaterial` so they stay bright regardless of lighting
- **Houses** — parcel or building footprints extruded to 7.5m with `ExtrudeGeometry`. Coordinates are projected from lat/lon to local meters, extruded along Y, then rotated upright
- **Labels** — `CSS2DRenderer` places street names as HTML elements that track 3D positions
- **Interaction** — `Raycaster` tests against house and street meshes on mousemove for tooltips. `OrbitControls` handles camera orbit/pan/zoom with a custom shift-key interceptor for mid-drag pan toggling

## License

[ISC](LICENSE.md)
