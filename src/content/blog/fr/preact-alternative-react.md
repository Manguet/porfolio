---
title: "Preact : l'alternative légère à React pour des interfaces performantes"
description: "Pourquoi et comment utiliser Preact à la place de React : taille du bundle, API compatible, cas d'usage, et intégration dans un projet existant."
pubDate: 2026-01-18
tags: ["Preact", "React", "Frontend"]
lang: fr
slug: preact-alternative-react
---

## Introduction : le poids des bundles frontend

L'écosystème JavaScript moderne a un problème de poids. Une application React basique, même sans dépendances métier, embarque React + ReactDOM pour environ **45 kb minifiés et gzippés**. Sur mobile en 3G, chaque kilobyte compte : le Time to Interactive augmente, le score Lighthouse baisse, et les utilisateurs partent.

Pour des interfaces relativement simples — un widget interactif, un formulaire dynamique, quelques composants sur un site majoritairement statique — React est souvent surdimensionné. C'est là que Preact entre en scène.

## Qu'est-ce que Preact ?

Preact est une **alternative à React de 3 kb** (gzippé) avec une API quasi identique. Créé par Jason Miller, il implémente le même modèle de composants, les mêmes hooks et le même JSX. Pour la plupart des cas d'usage, passer de React à Preact ne nécessite que de changer les imports.

Le secret de cette légèreté : Preact n'implémente pas les fonctionnalités les plus rares de React (comme les `SyntheticEvents` complets ou certaines APIs expérimentales). Il vise la compatibilité Pareto : 80 % des besoins avec 20 % du poids.

`preact/compat` est un alias de compatibilité qui permet d'utiliser des librairies React tierces (React Router, React Query...) sans les modifier.

## Quand choisir Preact plutôt que React ?

**Choisir Preact quand :**
- Le projet est un site majoritairement statique avec quelques composants interactifs
- La performance (Core Web Vitals, LCP, TTI) est une contrainte forte
- L'application n'utilise pas les fonctionnalités avancées de React (Concurrent Mode, Suspense avancé)
- On intègre un widget dans un site existant sans contrôle sur le reste du bundle

**Garder React quand :**
- L'équipe est grande et la compatibilité avec l'écosystème React est essentielle
- Le projet utilise intensivement React Server Components ou des fonctionnalités Next.js avancées
- Des librairies tierces critiques ne fonctionnent pas bien avec `preact/compat`

## Exemple pratique : composant counter avec hooks

Un composant compteur classique — identique à ce qu'on écrirait en React :

```jsx
import { useState, useEffect } from 'preact/hooks';

export function Counter({ initialValue = 0, step = 1 }) {
  const [count, setCount] = useState(initialValue);

  useEffect(() => {
    document.title = `Compteur : ${count}`;
  }, [count]);

  return (
    <div class="counter">
      <button onClick={() => setCount(c => c - step)}>−</button>
      <span>{count}</span>
      <button onClick={() => setCount(c => c + step)}>+</button>
    </div>
  );
}
```

Notez `class` au lieu de `className` — Preact accepte les deux, mais préfère les attributs HTML natifs pour rester proche du DOM.

### Hooks disponibles

Preact supporte tous les hooks courants : `useState`, `useEffect`, `useRef`, `useContext`, `useMemo`, `useCallback`, `useReducer`. La syntaxe est identique à React, seul l'import change :

```jsx
// React
import { useState, useEffect } from 'react';

// Preact
import { useState, useEffect } from 'preact/hooks';
```

## Intégration dans un projet Symfony ou statique

### Dans un projet Symfony avec Webpack Encore

```javascript
// webpack.config.js
Encore
  .addAliases({
    'react': 'preact/compat',
    'react-dom': 'preact/compat',
  });
```

Cette simple configuration redirige tous les imports `react` vers Preact, y compris dans les dépendances tierces.

### En script autonome (sans build)

Pour un widget isolé sur un site existant, on peut utiliser Preact via CDN avec HTM (une alternative JSX sans compilation) :

```html
<script type="module">
  import { h, render } from 'https://esm.sh/preact';
  import { useState } from 'https://esm.sh/preact/hooks';
  import htm from 'https://esm.sh/htm';

  const html = htm.bind(h);

  function Counter() {
    const [count, setCount] = useState(0);
    return html`<button onClick=${() => setCount(c => c + 1)}>
      Clics : ${count}
    </button>`;
  }

  render(html`<${Counter} />`, document.getElementById('widget'));
</script>
```

Zéro build, zéro configuration, Preact livré en 3 kb.

## Limitations

Preact n'est pas un remplacement universel de React. Quelques limitations à connaître :

- **React Server Components** : non supportés nativement dans Preact (Fresh, le framework Preact, propose sa propre approche)
- **Compatibilité `preact/compat`** : la plupart des librairies React fonctionnent, mais certaines utilisent des API internes non documentées qui peuvent poser problème
- **DevTools** : les React DevTools ne fonctionnent pas directement (il existe une extension Preact DevTools séparée)
- **Communauté** : l'écosystème est plus petit — moins de tutoriels, moins de librairies testées spécifiquement

## Conclusion

Preact est un choix excellent pour les projets où la performance bundle est une contrainte réelle. Sa compatibilité avec l'API React le rend accessible sans courbe d'apprentissage, et son intégration dans Symfony, Astro ou tout projet avec un bundler est triviale.

Pour mon usage : Preact est mon choix par défaut pour les composants interactifs sur des sites essentiellement statiques, là où embarquer React entier serait disproportionné. Pour des SPA complexes, je reste sur React ou explore des alternatives comme SolidJS.
