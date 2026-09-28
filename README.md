# pdvr-bot

Workflows GitHub Actions pour les **7 villes en région** de [pasdevelib.app](https://pasdevelib.app) : Bordeaux, Lille, Lyon, Montpellier, Nantes, Rennes, Strasbourg, Toulouse. Paris reste géré par [`pasdevelib/pdv-bot`](https://github.com/pasdevelib/pdv-bot).

Ce dépôt ne contient **aucun code Python** — seulement des workflows. Chaque job installe le paquet partagé directement depuis `pdv-bot` :

```bash
pip install "git+https://github.com/pasdevelib/pdv-bot.git"
```

Toute la logique métier (scraping, consolidation, prévision, stockage) vit dans un seul endroit, donc aucun risque de divergence entre deux copies du même code.

## ⚠️ Ce dépôt doit être PUBLIC

Comme `pdv-bot`, ce dépôt doit rester public : `blog.pasdevelib.app` et le workflow `stats-cities.yml` de `pdv-bot` lisent ses releases de façon anonyme, sans token. Un dépôt privé casserait ces deux chemins de lecture.

## Pourquoi un dépôt séparé (2026-09-28)

GitHub Actions retarde et laisse tomber la plupart des déclenchements `schedule` quand un même dépôt en cumule trop, surtout à haute fréquence (`*/5 * * * *`) — root cause d'une régression de 3 mois sur le scraping Paris, découverte fin septembre 2026. Une première consolidation (8 workflows par ville → 1 workflow en matrice) a réduit le nombre de déclarations `schedule` dans `pdv-bot`, sans l'éliminer. Isoler les villes en région dans leur propre dépôt donne à **Paris son propre quota de déclenchements programmés**, complètement indépendant de celui des 7 autres villes.

## Workflows

| Fichier                    | Fréquence     | Sortie (release de CE dépôt)                          |
|-----------------------------|---------------|---------------------------------------------------------|
| `scrape-cities.yml`         | */5 min       | `cities-live` (snapshots temps réel)                    |
| `consolidate-cities.yml`    | quotidien 3h30| `cities-history` (`hourly_history_<ville>.parquet`)      |
| `forecast-cities.yml`       | quotidien 5h45| `cities-aggregates` (`forecast_7d_<ville>.parquet`)      |
| `geocode-cities.yml`        | hebdo (lundi) | `cities-live` (enrichit `stations_cities.json`)          |

Tous en matrice (`fail-fast: false`) : un plantage sur une ville n'affecte jamais les autres, et un seul cron est enregistré côté GitHub pour les 8 villes de chaque job.

`stats-cities.yml` (classements/analyses, y compris Paris) et `daily-digest.yml` (bilan quotidien IA) restent dans `pdv-bot` : ils ont besoin de lire les données des deux dépôts dans la même exécution — voir le README de `pdv-bot`.

## Stockage

Chaque workflow écrit avec le token par défaut de GitHub Actions (`secrets.GITHUB_TOKEN`, `contents: write` sur CE dépôt) — **aucun secret à créer manuellement**.

Exception : `forecast-cities.yml` lit deux fichiers partagés avec Paris (`calendar.parquet`, `weather.parquet`, release `aggregates` de `pdv-bot`) en lecture anonyme, `pdv-bot` étant public.

## Pages consommant ces données

`blog.pasdevelib.app/bordeaux`, `/lyon`, `/lille`, `/rennes`, `/strasbourg`, `/toulouse` (pages vitrine par ville), `/donnees?ville=<id>&periode=week` (dashboard), `/blog/articles/dailymonitoring/<ville>` (bilan quotidien) — toutes via [`pasdevelib-webapp`](https://github.com/pasdevelib/pasdevelib-webapp), en lecture directe sur les releases de ce dépôt.
