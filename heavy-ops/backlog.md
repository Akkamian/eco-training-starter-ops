# Backlog heavy-ops

## Contexte du projet

Le projet represente un outil interne avec login, dashboard, table volumineuse, analytics et parametres.

## Contexte d'analyse

Diagnostic ecoconception realise avec Lighthouse, GreenIt-Analysis et EcoIndex le 11/06/2026.

**Resultats cles :**
- Page d'accueil : Note A (EcoIndex 93.65) — 13 Ko, 21 requetes, 21 elements DOM
- Vue dashboard : Note C (EcoIndex 69.73) — 2013 Ko, 22 requetes, 366 elements DOM
- Performance Lighthouse : 60/100 sur les deux pages (FCP ~5.8s, LCP ~10.1s)
- JS transfere : 1,66 MB dont 834 KB de code inutilise (50 %)
- Polling reseau : 4 appels API synchronises toutes les 5 secondes (48 req/min)

**Mauvaises pratiques identifiées :**
- Polling agressif sans lien avec l'interaction utilisateur
- Chargement integral des donnees pour toutes les pages
- Bundle React en mode development (react-dom.development.js ~900 KB)
- Absence totale de cache HTTP (Cache-Control: no-store sur toutes les routes backend)
- Pas de pagination sur les donnees volumineuses (366+ enregistrements dans le DOM)
- Logs console backend actifs en continu


## User story 1 — Remplacer le polling par un rafraichissement a la demande

- Contexte: En tant qu'**utilisateur dashboard**, je veux **cliquer sur un bouton pour rafraichir les donnees** plutot que de subir un polling automatique toutes les 5 secondes, afin de **reduire le trafic reseau et la consommation energetique cote serveur et client**.
- Objectif: Reduire le nombre de requetes API d'au moins 90 % (de ~48 req/min a moins de 5 req/min)
- Bonne pratique d eco-conception ciblee: **Limiter les requetes reseau** — déclenchement manuel (bouton de rafraichissement)
- KPI associe: Nombre de requetes API par minute mesure avec l'onglet Reseau des DevTools (cible : < 5 req/min)
- Repo ou ecran concerne: Frontend `OpsApp.tsx` (fonction `loadAll` et `setInterval` ligne 653) + Backend (toutes les routes /api/*)
- Critere de reussite: Aucun appel API automatique periodique ; un bouton "Rafraichir" visible dans l'interface declenche `loadAll()` a la demande ; le `setInterval(loadAll, 5000)` est supprime
- Niveau de priorite: **Haute**

## User story 2 — Charger les donnees par page (lazy loading)

- Contexte: En tant qu'**utilisateur dashboard**, je veux **que seules les donnees necessaires a la page active soient chargees** (et non les 4 endpoints simultanement au montage), afin de **reduire le volume de donnees transferees et travailler sur une application réactive**.
- Objectif: Reduire le volume de donnees transfere au chargement initial de 70 % (passer de ~2 013 Ko a < 600 Ko sur la vue dashboard)
- Bonne pratique d eco-conception ciblee: **Chargement a la demande de type lazy loading** — ne charger que les donnees utiles a la page active
- KPI associe: Taille de la page (Ko) mesuree par GreenIt-Analysis (cible : < 600 Ko sur la vue dashboard)
- Repo ou ecran concerne: Frontend `OpsApp.tsx` (useEffect centralise ligne 633-655, tous les composants de page)
- Critere de reussite: Le composant `DashboardPage` ne charge que `/api/dashboard`, `TablePage` ne charge que `/api/records`, `AnalyticsPage` ne charge que `/api/analytics` ; le chargement est declenche par la navigation et non par le montage initial de `OpsApp`
- Niveau de priorite: **Haute**

## User story 3 — Paginer les donnees cote backend

- Contexte: En tant qu'**operateur dashboard**, je veux **que la file active soit paginee** (20 enregistrements par page plutot que les 366+), afin de **reduire la taille du DOM, la memoire consommee et le temps de rendu**.
- Objectif: Reduire la taille du DOM sur la vue dashboard mesuree par GreenIt de 366 a ≤ 50 elements
- Bonne pratique d eco-conception ciblee: **Pagination des donnees volumineuses** — ne charger et afficher que les donnees visibles
- KPI associe: Taille du DOM mesuree par GreenIt-Analysis (cible : < 100 elements)
- Repo ou ecran concerne: Backend `src/index.ts` (route `/api/records`) + Frontend `OpsApp.tsx` (composant `TablePage`)
- Critere de reussite: L'API `/api/records` accepte les parametres `?page=1&limit=20` ; le composant `TablePage` n'affiche que 20 lignes maximum avec une navigation paginee (precedent/suivant) ; le nombre total d'elements DOM sur la page Table ne depasse pas 100
- Niveau de priorite: **Haute**

## User story 4 — Simplifier le CSS et passer en mobile first

- Contexte: En tant qu'**equipe front-end**, je veux **refondre le CSS en adoptant une approche mobile first et en supprimant les declarations inutilisees**, afin de **reduire le poids des feuilles de style et la consommation memoire du rendu navigateur sur tous les terminaux**.
- Objectif: Reduire la taille du fichier CSS d'au moins 30 % (de ~12 Ko a < 8 Ko) et supprimer le bloc `@media (max-width: 1180px)` au profit de breakpoints progressifs (mobile → tablette → desktop)
- Bonne pratique d eco-conception ciblee: **CSS optimise et adaptatif** — conception mobile first, suppression des declarations orphelines, polices inutiles retirees
- KPI associe: Taille du fichier CSS (Ko) mesuree dans le panneau Reseau des DevTools (cible : < 8 Ko) et nombre de declarations CSS inutilisees mesure par l'audit Lighthouse "Unused CSS Rules" (cible : 0)
- Repo ou ecran concerne: Frontend `ops.css` (integralite du fichier — toutes les pages)
- Critere de reussite:
  - La police `'IBM Plex Sans'` est supprimee de la declaration `font-family` (police inutile dans un dashboard metier) ; seule `'Trebuchet MS', sans-serif` est conservee
  - Le point de rupture unique `@media (max-width: 1180px)` est remplace par des breakpoints mobile first : `min-width: 480px` (petits ecrans), `min-width: 768px` (tablette), `min-width: 1024px` (desktop)
  - Les grilles complexes (`grid-template-columns: repeat(N, 1fr)`) passent en `1fr` par defaut (mobile) et s'etendent progressivement via les media queries ascendantes
  - Aucune regle CSS inutilisee ou redondante ne subsiste ; les selecteurs orphelins sont retires
  - Aucune regression visuelle sur les 4 ecrans du dashboard (accueil, table, analytics, parametres)
  - Le theme sombre (`--ops-bg`, `--ops-surface`, etc.) reste intact
- Niveau de priorite: **Haute**

## User story 5 — Minifier et optimiser le bundle JS pour la production

- Contexte: En tant qu'**equipe front-end**, je veux **configurer le build pour utiliser les versions de production des dependances React** (react-dom.production.min.js au lieu de react-dom.development.js), afin de **reduire la taille du bundle JS de ~900 KB a ~130 KB et ameliorer le temps de chargement**.
- Objectif: Reduire le poids du bundle JS transfere de 1,66 MB a < 500 KB (cible Lighthouse : score "Unused JavaScript" passe a 1)
- Bonne pratique d eco-conception ciblee: **Minification et tree-shaking du code JavaScript** — supprimer le code de debug et les modules inutilises
- KPI associe: Poids total des ressources JS transferees mesure par Lighthouse (cible : < 500 KB)
- Repo ou ecran concerne: Configuration `vite.config.ts`, variables d'environnement `NODE_ENV=production`, import des dependances npm
- Critere de reussite: L'audit Lighthouse "Unminified JavaScript" est reussi (score 1) ; l'audit "Unused JavaScript" ne signale que < 50 KB de code inutilise ; le bundle ne contient plus les fichiers `*.development.js` de React
- Niveau de priorite: **Moyenne**

## User story 6 — Mettre en place un cache applicatif et supprimer les logs de debug

- Contexte: En tant qu'**equipe backend**, je veux **activer un cache HTTP approprie sur les donnees quasi-statiques et supprimer les logs console en environnement de production**, afin de **reduire la charge serveur inutile et le traitement cote client**.
- Objectif: Atteindre le score maximum de 100/100 pour la categorie Best Practices Lighthouse et reduire le cout serveur
- Bonne pratique d eco-conception ciblee: **Mise en cache des ressources** + **nettoyage des logs de debug**
- KPI associe: Score "Best Practices" de Lighthouse (cible : 100/100)
- Repo ou ecran concerne: Backend `src/index.ts` (middleware Cache-Control ligne 23, log console ligne 18)
- Critere de reussite: Les donnees quasi-statiques (records, settings) sont servies avec `Cache-Control: public, max-age=60` ; les assets statiques avec `Cache-Control: public, max-age=31536000, immutable` ; le middleware de log de requete est desactive lorsque `NODE_ENV=production`
- Niveau de priorite: **Moyenne**

## Notes

- Vous pouvez ajouter d autres user stories si necessaire.
- Le niveau de detail attendu doit permettre une priorisation exploitable.
- Les priorites sont definies selon l'impact environnemental potentiel (gain energétique x facilite de mise en oeuvre).
- US 1, 2 et 3 sont independantes et peuvent etre developpees en parallele.
- US 4 et 5 dependent d'une configuration d'environnement (NODE_ENV) et peuvent etre traitees ensemble.