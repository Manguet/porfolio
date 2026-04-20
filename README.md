# Portfolio — Benjamin Manguet

Portfolio personnel de **Benjamin Manguet**, Développeur Full-Stack & Auditeur Sécurité Web freelance basé en Charente-Maritime.

🌐 [benjaminmanguet.fr](https://benjaminmanguet.fr) *(à venir)*

---

## Stack

- **[Astro](https://astro.build/)** — génération de site statique
- **CSS custom properties** — design system dark/purple
- **Vanilla JS** — i18n FR/EN, animations
- **Netlify** — hébergement & formulaire de contact
- **Netlify Forms** — traitement des messages sans backend

## Fonctionnalités

- Bilingue FR / EN (switcher en temps réel, `localStorage`)
- Sections : Hero, Services, Audit Flash, Projets, Stack, Expérience, Contact
- Formulaire de contact via Netlify Forms (spam honeypot inclus)
- Réservation Calendly intégrée
- Composant `<Image>` Astro (optimisation WebP automatique)
- Responsive mobile-first

## Structure

```
src/
├── assets/         # Logo (optimisé via <Image>)
├── components/     # Composants Astro (Nav, Hero, Services, ...)
├── i18n/           # Traductions FR/EN (translations.ts)
├── layouts/        # Layout principal
└── pages/          # index.astro
public/             # Favicons, manifest
```

## Commandes

| Commande          | Action                                      |
| :---------------- | :------------------------------------------ |
| `npm install`     | Installe les dépendances                    |
| `npm run dev`     | Serveur de développement sur `localhost:4321` |
| `npm run build`   | Build de production dans `./dist/`          |
| `npm run preview` | Prévisualisation du build en local          |

## Contact

- **Email** : benjamin.manguet@gmail.com
- **LinkedIn** : [linkedin.com/in/benjaminmanguet](https://www.linkedin.com/in/benjaminmanguet)
- **Malt** : [malt.fr/profile/benjaminmanguet](https://www.malt.fr/profile/benjaminmanguet)

## Licence

MIT — voir [LICENSE](LICENSE)
