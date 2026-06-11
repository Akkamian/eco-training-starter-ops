# eco-training-starters

Ce dossier regroupe 4 repos pedagogiques autonomes destines a une formation certifiante en eco-conception de services numeriques.

## heavy-showcase

Objectif pedagogique: Travailler le poids de page, les medias, les scripts tiers et la structure d information.

Anti-patterns principaux:
- images lourdes non optimisees
- video autoplay
- polices et blocs visuels multiples
- scripts tiers simules

Commandes:
- `cd heavy-showcase`
- `npm install`
- `npm run dev`

## heavy-shop

Objectif pedagogique: Travailler la reduction de payloads, d appels reseau et l arbitrage fonctionnel.

Anti-patterns principaux:
- trop d images lourdes
- payload produit surcharge
- filtres qui declenchent trop de requetes
- polling de stock ou promo

Commandes:
- `cd heavy-shop`
- `npm install`
- `npm run dev`

## heavy-ops

Objectif pedagogique: Travailler le data fetching, la pagination, la charge CPU et la simplification d interface.

Anti-patterns principaux:
- polling agressif
- gros datasets charges d un coup
- graphiques inutiles
- re-renders frequents

Commandes:
- `cd heavy-ops`
- `npm install`
- `npm run dev`

## heavy-hub

Objectif pedagogique: Travailler la sobriete de navigation, le prechargement et les medias d espace membre.

Anti-patterns principaux:
- medias lourds
- contenus precharges inutilement
- avatars non optimises
- notifications bavardes

Commandes:
- `cd heavy-hub`
- `npm install`
- `npm run dev`

## Structure commune

- frontend/
- backend/
- data/
- assets/
- docs/
- README.md
- backlog.md

## Demarrage rapide

Chaque projet se lance independamment:

1. `cd <nom-du-projet>`
2. `npm install`
3. `npm run dev`
