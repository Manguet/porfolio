# Blog bilingue — Design Spec

**Projet :** benjamin-manguet.fr (portfolio Astro)
**Date :** 9 mai 2026
**Auteur :** Benjamin Manguet

---

## Objectif

Ajouter un blog bilingue (FR/EN) au portfolio Astro existant. Le blog accueillera des articles techniques (cartographie, Symfony, sécurité). Le premier article publié sera le retour technique sur Hydro Explorer / Hub'eau + Leaflet, destiné notamment à être référencé dans le cadre de la démarche EPMP.

---

## Architecture

### Approche retenue : Content Collections avec dossiers `fr/` et `en/`

```
src/content/
  config.ts                        ← schéma Zod de la collection "blog"
  blog/
    fr/
      hub-eau-leaflet.md
    en/
      hub-eau-leaflet.md

src/pages/
  blog/
    [lang]/
      index.astro                  ← liste /blog/fr et /blog/en
      [...slug].astro              ← article individuel

src/components/
  Articles.astro                   ← section homepage (3 derniers articles)
  BlogCard.astro                   ← card réutilisable
  TableOfContents.astro            ← sidebar sticky avec headings

src/i18n/
  translations.ts                  ← clés UI blog ajoutées (FR + EN)
```

**Fichiers modifiés :**
- `src/pages/index.astro` — ajout du composant `<Articles />`
- `src/i18n/translations.ts` — nouvelles clés pour le blog

---

## Schéma des articles (frontmatter)

```yaml
---
title: string           # obligatoire
description: string     # obligatoire — utilisé en meta description et dans les cards
pubDate: Date           # obligatoire — format YYYY-MM-DD
tags: string[]          # obligatoire — au moins 1 tag
lang: "fr" | "en"       # obligatoire — doit correspondre au dossier parent
slug: string            # obligatoire — identique en FR et EN (lie les deux versions)
---
```

Le champ `slug` identique entre les deux versions permet au switcher de langue de naviguer vers la traduction de l'article courant.

---

## Routes

| URL | Contenu |
|-----|---------|
| `/blog/fr` | Liste de tous les articles en français |
| `/blog/en` | Liste de tous les articles en anglais |
| `/blog/fr/hub-eau-leaflet` | Article en français |
| `/blog/en/hub-eau-leaflet` | Article en anglais |

Le paramètre `[lang]` accepte uniquement `fr` et `en`. Toute autre valeur renvoie une 404.

---

## Composants

### `BlogCard.astro`

**Props :** `title`, `description`, `pubDate`, `tags` (tableau, seul `tags[0]` affiché), `slug`, `lang`

**Rendu :**
- Tag (JetBrains Mono, violet `#a78bfa`, fond `rgba(124,58,237,0.15)`)
- Titre (Space Grotesk 700)
- Description (texte muted `#888`)
- Date formatée (JetBrains Mono, `#555`)
- Toute la card est un lien vers `/blog/[lang]/[slug]`
- Hover : border `#7c3aed`, légère élévation

### `Articles.astro`

Reçoit un prop `lang: "fr" | "en"` depuis `index.astro`. Charge les 3 articles les plus récents filtrés par ce `lang`.

Affiche :
- Label section (`// ARTICLES` en JetBrains Mono, style cohérent avec les autres sections)
- Titre de section ("Derniers articles" / "Latest articles")
- Grille 3 colonnes de `BlogCard`
- Lien "Voir tous les articles →" / "See all articles →" vers `/blog/[lang]`

Sur mobile : grille passe en 1 colonne.

### `TableOfContents.astro`

**Props :** `headings` (tableau `{ depth, slug, text }[]` fourni par Astro via `entry.render()`)

**Rendu :**
- Sidebar sticky à droite (220px)
- Label "Table des matières" / "Table of contents" en JetBrains Mono `#a78bfa`
- Liste des headings H2 et H3 (H3 indentés)
- Highlight de la section active via `IntersectionObserver` (vanilla JS inline)
- Couleur active : `#a78bfa`, couleur inactive : `#555`

### `[lang]/index.astro` — page liste

- Récupère toutes les entrées de la collection `blog` filtrées par `lang`
- Trie par `pubDate` décroissant
- Affiche en grille 3 colonnes de `BlogCard`
- Breadcrumb : `← Accueil` / `← Home` en `#a78bfa`
- Meta : `<title>Blog — Benjamin Manguet</title>`

### `[lang]/[...slug].astro` — page article

- Récupère l'entrée par `slug` + `lang`
- 404 si introuvable
- Layout : colonne principale (contenu markdown) + sidebar droite (`TableOfContents`)
- Breadcrumb : `← Blog` en `#a78bfa` (violet, pas de bleu)
- En-tête article : tag, titre H1, auteur + date + temps de lecture estimé
- Séparateur horizontal avant le contenu
- Switcher de langue : on query la collection `blog` avec le même `slug` et la langue opposée — le lien vers `/blog/[autre-lang]/[slug]` n'est affiché que si l'entrée existe
- Meta : `title` et `description` depuis le frontmatter

---

## i18n

Nouvelles clés à ajouter dans `translations.ts` :

| Clé | FR | EN |
|-----|----|----|
| `blog.title` | `Blog` | `Blog` |
| `blog.latest` | `Derniers articles` | `Latest articles` |
| `blog.seeAll` | `Voir tous les articles →` | `See all articles →` |
| `blog.readMore` | `Lire l'article` | `Read article` |
| `blog.backToBlog` | `← Blog` | `← Blog` |
| `blog.backToHome` | `← Accueil` | `← Home` |
| `blog.toc` | `Table des matières` | `Table of contents` |
| `blog.readingTime` | `min de lecture` | `min read` |

---

## Styles

Tous les composants suivent les variables CSS existantes :

```css
--purple:       #7c3aed
--purple-light: #a78bfa
--bg:           #0a0a0a
--bg-2:         #111111
--bg-3:         #1a1a1a
--border:       #2a2a2a
--text:         #f0f0f0
--text-muted:   #888888
```

Les liens de navigation (breadcrumb, retour) utilisent `#a78bfa` — jamais de bleu.

Le contenu markdown de l'article est stylisé via CSS global (titres, code blocks, listes) dans la page article, pas via un plugin externe.

---

## Hors périmètre

- Commentaires
- Système de tags cliquables / filtrage par tag
- Pagination (inutile avec peu d'articles)
- Images de couverture par article
- Flux RSS (peut être ajouté plus tard)
- Recherche full-text

---

## Contenu du premier article

**Slug :** `hub-eau-leaflet`
**Tags :** `["Cartographie", "Leaflet", "Hub'eau"]`
**Titre FR :** *Construire un visualiseur de données hydrologiques avec Hub'eau et Leaflet — retour technique*
**Titre EN :** *Building a hydrological data visualizer with Hub'eau and Leaflet — technical retrospective*

Le contenu de l'article sera rédigé séparément après l'implémentation de la structure.
