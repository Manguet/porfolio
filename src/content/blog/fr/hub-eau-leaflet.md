---
title: "Construire un visualiseur de données hydrologiques avec Hub'eau et Leaflet — retour technique"
description: "Comment j'ai construit Hydro Explorer : intégration de l'API Hub'eau avec Leaflet 1.9 pour visualiser 3 000 stations hydrométriques en temps réel, clustering, graphiques Plotly et PWA."
pubDate: 2026-05-12
tags: ["Cartographie", "Leaflet", "Hub'eau"]
lang: fr
slug: hub-eau-leaflet
---

## Introduction

Hydro Explorer est un visualiseur de données hydrologiques open source que j'ai développé pour explorer l'API Hub'eau et approfondir mes compétences en cartographie interactive. Le résultat est disponible en ligne : les stations hydrométriques françaises sur une carte Leaflet, avec graphiques en temps réel et couches optionnelles (piézométrie, Vigicrues).

Ce retour technique couvre les choix d'architecture, les difficultés rencontrées, et les solutions adoptées.

## L'API Hub'eau Hydrométrie v2

Hub'eau est le portail de données sur l'eau du service public français. L'API Hydrométrie v2 expose les stations de mesure (hauteur et débit) sur l'ensemble des cours d'eau français.

Les endpoints clés utilisés :

- `/referentiel/stations` — liste des 3 000+ stations actives avec coordonnées géographiques
- `/observations_tr` — observations en temps réel (hauteur H et débit Q) sur une période donnée

La pagination est gérée via les paramètres `size` et `cursor`. Les coordonnées sont en WGS84, compatibles directement avec Leaflet.

```javascript
const response = await fetch(
  'https://hubeau.eaufrance.fr/api/v2/hydrometrie/referentiel/stations'
  + '?size=5000&fields=code_station,libelle_station,longitude_station,latitude_station'
  + '&en_service=true&format=json'
);
const { data } = await response.json();
```

## Intégration Leaflet 1.9

Leaflet reste la référence pour la cartographie web légère. Pas de framework, pas de build step — juste un fichier JS et une balise `<script>`.

Les tuiles IGN Géoplateforme remplacent OpenStreetMap pour un rendu plus adapté au contexte hydrologique français :

```javascript
L.tileLayer(
  'https://data.geopf.fr/wmts?SERVICE=WMTS&REQUEST=GetTile'
  + '&layer=GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2'
  + '&style=normal&tilematrixset=PM&TileMatrix={z}&TileCol={x}&TileRow={y}'
  + '&format=image/png',
  { attribution: '© IGN Géoplateforme', maxZoom: 18 }
).addTo(map);
```

## Clustering des 3 000 markers

Afficher 3 000 markers sans clustering dégrade sévèrement les performances. `Leaflet.markercluster` regroupe les markers selon le niveau de zoom :

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

Les couleurs des markers indiquent la fraîcheur des données : vert (données récentes), jaune (vigilance), gris (inactif).

## Graphiques Plotly avec double axe

Plotly gère les séries temporelles avec un double axe H/Q (hauteur et débit sur la même période) :

```javascript
Plotly.react('chart', [
  { x: dates, y: hauteurs, name: 'Hauteur (m)', yaxis: 'y' },
  { x: dates, y: debits, name: 'Débit (m³/s)', yaxis: 'y2', type: 'scatter' },
], {
  yaxis: { title: 'Hauteur (m)' },
  yaxis2: { title: 'Débit (m³/s)', overlaying: 'y', side: 'right' },
});
```

## PWA et cache IndexedDB

Le Service Worker met en cache le shell (HTML, CSS, JS) pour un fonctionnement hors-ligne. Les données d'observation sont cachées dans IndexedDB avec un TTL de 30 minutes pour éviter de solliciter l'API à chaque clic.

```javascript
// Cache composite key : station + période
const cacheKey = `${codeStation}_${periodDays}`;
const cached = debitCache.get(cacheKey);
if (cached) return cached;
```

## Conclusion

Hydro Explorer est disponible sur [GitHub](https://github.com/Manguet/hydro-explorer) et en démo live sur [GitHub Pages](https://manguet.github.io/hydro-explorer/).

Le projet m'a permis de maîtriser Leaflet avancé, l'API Hub'eau, le clustering de markers, et les Progressive Web Apps — compétences directement utiles pour des projets de visualisation de données environnementales.
