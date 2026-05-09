---
title: "Building a hydrological data visualizer with Hub'eau and Leaflet — technical retrospective"
description: "How I built Hydro Explorer: integrating the Hub'eau API with Leaflet 1.9 to visualise 3,000 hydrometric stations in real time, with clustering, Plotly charts and PWA."
pubDate: 2026-05-12
tags: ["Mapping", "Leaflet", "Hub'eau"]
lang: en
slug: hub-eau-leaflet
---

## Introduction

Hydro Explorer is an open source hydrological data visualizer I built to explore the Hub'eau API and deepen my skills in interactive mapping. The result is publicly available: French hydrometric stations on a Leaflet map, with real-time charts and optional layers (piezometry, Vigicrues).

This technical retrospective covers the architecture choices, challenges encountered, and solutions adopted.

## The Hub'eau Hydrometry v2 API

Hub'eau is the French public water data portal. The Hydrometry v2 API exposes measurement stations (water level and flow rate) across all French waterways.

Key endpoints used:

- `/referentiel/stations` — list of 3,000+ active stations with geographic coordinates
- `/observations_tr` — real-time observations (level H and flow Q) over a given period

Pagination is handled via `size` and `cursor` parameters. Coordinates are in WGS84, directly compatible with Leaflet.

```javascript
const response = await fetch(
  'https://hubeau.eaufrance.fr/api/v2/hydrometrie/referentiel/stations'
  + '?size=5000&fields=code_station,libelle_station,longitude_station,latitude_station'
  + '&en_service=true&format=json'
);
const { data } = await response.json();
```

## Leaflet 1.9 Integration

Leaflet remains the reference for lightweight web mapping. No framework, no build step — just a JS file and a `<script>` tag.

IGN Géoplateforme tiles replace OpenStreetMap for a rendering better suited to the French hydrological context:

```javascript
L.tileLayer(
  'https://data.geopf.fr/wmts?SERVICE=WMTS&REQUEST=GetTile'
  + '&layer=GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2'
  + '&style=normal&tilematrixset=PM&TileMatrix={z}&TileCol={x}&TileRow={y}'
  + '&format=image/png',
  { attribution: '© IGN Géoplateforme', maxZoom: 18 }
).addTo(map);
```

## Clustering 3,000 Markers

Displaying 3,000 markers without clustering severely degrades performance. `Leaflet.markercluster` groups markers according to zoom level:

```javascript
const clusterGroup = L.markerClusterGroup({
  chunkedLoading: true,
  maxClusterRadius: 60,
});
stations.forEach(station => {
  const marker = L.marker([station.latitude, station.longitude]);
  clusterGroup.addLayer(marker);
});
map.addLayer(clusterGroup);
```

Marker colours indicate data freshness: green (recent data), yellow (warning), grey (inactive).

## Plotly Charts with Dual Axis

Plotly handles time series with a dual H/Q axis (water level and flow rate over the same period):

```javascript
Plotly.react('chart', [
  { x: dates, y: levels, name: 'Level (m)', yaxis: 'y' },
  { x: dates, y: flows, name: 'Flow (m³/s)', yaxis: 'y2', type: 'scatter' },
], {
  yaxis: { title: 'Level (m)' },
  yaxis2: { title: 'Flow (m³/s)', overlaying: 'y', side: 'right' },
});
```

## PWA and IndexedDB Cache

The Service Worker caches the shell (HTML, CSS, JS) for offline use. Observation data is cached in IndexedDB with a 30-minute TTL to avoid hitting the API on every click.

```javascript
// Composite cache key: station + period
const cacheKey = `${stationCode}_${periodDays}`;
const cached = flowCache.get(cacheKey);
if (cached) return cached;
```

## Conclusion

Hydro Explorer is available on [GitHub](https://github.com/Manguet/hydro-explorer) and as a live demo on [GitHub Pages](https://manguet.github.io/hydro-explorer/).

The project gave me hands-on mastery of advanced Leaflet, the Hub'eau API, marker clustering, and Progressive Web Apps — skills directly applicable to environmental data visualisation projects.
