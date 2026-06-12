# Audit initial du projet
Stratégie : récupérer les données de performance de:
- la page d'accueil 
- puis de la vue cockpit

## Lightouse

**Page d'accueil :**  
![alt text](./resources/Ai_1.png)

![alt text](./resources/Ai-2.png)

**Vue Cockpit**  

![alt text](./resources/Ai_3.png)

![alt text](./resources/AI_4.png)


Task manager de chrome indique au lancement de la page 
- Utilisation CPU : 5%

- Mémoire : 80 000 ko

au repos :
- Utilisation CPU : 0.7% à chaque appel réseau
- Mémoire : 54 500 ko

## GreenIt
**Page d'accueil**  

| Date | URL | Nombre de requêtes | Taille de la page (Ko) | Taille du DOM | GES (gCO2e) | Eau (cl) | EcoIndex | Note |
|------|-----|-------------------:|-----------------------:|--------------:|------------:|----------:|----------:|:----:|
| 11/06/2026 11:27:52 | http://localhost:5173/ | 21 | 13 | 21 | 1.13 | 1.69 | 93.65 | A |

**Vue Cockpit**  

| Date | URL | Nombre de requêtes | Taille de la page (Ko) | Taille du DOM | GES (gCO2e) | Eau (cl) | EcoIndex | Note |
|------|-----|-------------------:|-----------------------:|--------------:|------------:|----------:|----------:|:----:|
| 11/06/2026 11:34:25 | http://localhost:5173/ | 22 | 2013 | 366 | 1.61 | 2.41 | 69.73 | C |

## EcoIndex

**Page d'accueil**  
![alt text](./resources/EcoIndex_I_1.png)
![alt text](./resources/EcoIndex_I_2.png)
![alt text](./resources/EcoIndex_I_3.png)

**Vue Cockpit**  
![alt text](./resources/EcoIndex_I_4.png)
![alt text](./resources/EcoIndex_I_5.png)
![alt text](./resources/EcoIndex_I_6.png)

## Diagnostic des performances d'éco-conception et analyse

Les résultats des outils d'analyse sont plutôt encourageant mais laissent tout de même entrevoir des pistes d'amélioration.

Points forts du projet :
- mode sombre déjà implémenté

Points d'amélioration :

- Limitation des requetes réseau
4 appels au backend sont réalisés toutes les 5secondes  
Requêtes réseau inutiles (4 appels API × 12 fois/minute = 48 req/min)

```ts
 loadAll();
    const timer = window.setInterval(loadAll, 5000);
    return () => window.clearInterval(timer);
```
  -> remplacer cette approche par un bouton d'actualisation (fetch des données à la demande)

- Tous les contenus sont chargés au chargement de la page  
-> mise en place d'un lazy loading  
-> mise en place d'un chargement à la demande des barchart  
-> ajout d'une pagination 

- Parcours utilisateur complexe et redondant  
-> simplification du parcours  

- Cache désactivé en backend (index.ts)
sur la route assets : 
```ts
  res.setHeader('Cache-Control', 'no-store');
```  
-> mettre en cache les ressources statiques

- Images volumineuses et inutilisées dans /assets

- Le css est constuit en desktop first :  
-> implémenter le mobile first

- utilisation d'une polices inutiles dans un dashboard métier ->  suppression de 'IBM Plex Sans' qui de toute facon n'est pas utilisée

- nombreux console.log backend

- Le composant principal est re-render toutes les 5 secondes avec la mise à jour de states Dashboard, Records, Settings et Analystics.  
-> utiliser des useMemo pour déclencher le re-render seulement quand les données ont changé.