# MapLibre GL Teritorio Cluster

Render native MapLibre GL JS clusters as HTML marker.

Render:
- HTML cluster
- HTML group marked as small cluster or last zoom level
- Pin Marker on feature click

Allow visualization and interaction with all markers, even when superposed.
Can display and interact with small cluster without the need to zoom or uncluster.

See the [Demo page](https://teritorio.github.io/maplibre-gl-teritorio-cluster/index.html).

![alt text](image.png)

## Usage

Add to you project (or use CDN).
```bash
yarn add @teritorio/maplibre-gl-teritorio-cluster
```

> [!WARNING]
> Set your GeoJson source with `clusterMaxZoom: 22` in order to let the plugin handle cluster/individual marker rendering across all zoom level

```js
import { Map } from "maplibre-gl"
import { TeritorioCluster } from 'maplibre-gl-teritorio-cluster'

const map = new Map({
  container: "map",
  style: {
    version: 8,
    name: "Empty Style",
    metadata: { "maputnik:renderer": "mlgljs" },
    sources: {
      points: {
        type: "geojson",
        cluster: true,
        clusterRadius: 80,
        clusterMaxZoom: 22, // Required, set a value for clusterMaxZoom
        data: {
          type: "FeatureCollection",
          features: [
            {
              type: "Feature",
              properties: { id: 1 },
              geometry: { type: "Point", coordinates: [0, 0] }
            },
            {
              type: "Feature",
              properties: { id: 2 },
              geometry: { type: "Point", coordinates: [0, 1] }
            }
          ]
        }
      }
    },
    glyphs: "https://orangemug.github.io/font-glyphs/glyphs/{fontstack}/{range}.pbf",
    layers: [
      {
        id: "cluster",
        type: "circle",
        source: "points"
      }
    ],
    id: "muks8j3"
  }
});

map.on('load', () => {
  const TeritorioCluster = new TeritorioCluster(map, 'points', options)

  // Get feature click event
  TeritorioCluster.addEventListener('click', (e) => {
    console.log(e.detail.selectedFeature)
  })

  // Render feature on map data updates
  map.on('data', (e) => {
    if (e.sourceId !== 'points' || !e.isSourceLoaded)
      return;

    map.on('move', TeritorioCluster.render);
    map.on('moveend', TeritorioCluster.render);
    TeritorioCluster.render()
  });
})

// Create whatever HTML element you want as Cluster
const clusterRender = (element: HTMLDivElement, props: MapGeoJSONFeature['properties'], size?: number): void => {}

// Create whatever HTML element you want as individual Marker
const displayMarker = (element: HTMLDivElement, feature: MapGeoJSONFeature, size?: number): void => {}

// Create whatever HTML element you want as Pin Marker
const displayPinMarker = (coords: LngLatLike, offset: Point): Marker => {}
```

## API

- [TeritorioCluster](#teritoriocluster)
  - [Parameters](#parameters)
  - [Options](#options)
  - [addEventListener](#addeventlistener)

### TeritorioCluster

Create a new Maplibre GL JS plugin for feature (cluster / individual marker) rendering

#### Parameters
| Name     | Type                                                               | Description                                                                                      | Required | Default value |
|----------|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|----------|---------------|
| map      | [`Map`](https://maplibre.org/maplibre-gl-js/docs/API/classes/Map/) | The Map object represents the map on your page                                                   | ✅        | ❌             |
| sourceId | `string`                                                           | The ID of the source. Should be a vector source, preferably of type GeoJSON and not vector tiles | ✅        | ❌             |
| options  | `object`                                                           | Options to configure the plugin                                                                  | ❌        | `undefined`   |

#### Options
| Name                          | Type                                                                                                                                                                                                           | Description                                                        | Required | Default value                                       |
|-------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|----------|-----------------------------------------------------|
| clusterMaxZoom                | `number`                                                                                                                                                                                                       | Zoom level at which we force the rendering of the Unfolded Cluster | ❌        | `17`                                                |
| clusterRenderFn               | `(element: HTMLDivElement, props: MapGeoJSONFeature['properties']): void`                                                                                                                                      | Cluster render function                                            | ❌        | `src/utils/helpers.ts/clusterRenderDefault()`       |
| initialFeature                | [`MapGeoJSONFeature`](https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/MapGeoJSONFeature/)                                                                                                            | Feature to select on initial rendering                             | ❌        | `undefined`                                         |
| markerRenderFn                | `(element: HTMLDivElement, feature: MapGeoJSONFeature, markerSize: number): void`                                                                                                                              | Individual Marker render function                                  | ❌        | `src/utils/helpers.ts/markerRenderDefault()`        |
| unfoldedClusterRenderFn       | `(parent: HTMLDivElement, items: MapGeoJSONFeature[], markerSize: number, renderMarker: (feature: MapGeoJSONFeature) => HTMLDivElement, clickHandler: (e: Event, feature: MapGeoJSONFeature) => void) => void` | Unfolded Cluster render function                                   | ❌        | `src/utils/helpers.ts/unfoldedClusterRenderSmart()` |
| unfoldedClusterRenderSmart    | Mix between Circular and HexaShape shape Unfolded Cluster render function                                                                                                                                      | -                                                                  | -        | -                                                   |
| unfoldedClusterRenderGrid     | Grid shape Unfolded Cluster render function function                                                                                                                                                           | -                                                                  | -        | -                                                   |
| unfoldedClusterRenderCircle   | Circular shape Unfolded Cluster render function function                                                                                                                                                       | -                                                                  | -        | -                                                   |
| unfoldedClusterRenderHexaGrid | HexaGrid shape Unfolded Cluster render function function                                                                                                                                                       | -                                                                  | -        | -                                                   |
| unfoldedClusterMaxLeaves      | `number`                                                                                                                                                                                                       | Unfolded Cluster max leaves number                                 | ❌        | `5`                                                 |
| pinMarkerRenderFn             | `(coords: LngLatLike, offset: Point): Marker`                                                                                                                                                                  | Pin Marker render function                                         | ❌        | `src/utils/helpers.ts/pinMarkerRenderDefault()`     |


#### addEventListener

Listen to feature click and return a `maplibregl.Feature` for external control.

```js
TeritorioCluster.addEventListener('click', (e) => {
  console.log(e.detail.selectedFeature)
})

```

## Dev
Install dependencies
```bash
yarn install
```

Serve the demo page
```bash
yarn dev
```

## Requirements

Requires [maplibre-gl-js](https://github.com/maplibre/maplibre-gl-js) >= v4.0.0.

## Contribution

Please see the [contribution guide](CONTRIBUTING.md).

## Author

[Teritorio](https://teritorio.fr)
