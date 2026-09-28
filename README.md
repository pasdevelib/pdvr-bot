# pdvr-bot — PasDeVélib, villes en région

Ce dépôt héberge les workflows GitHub Actions pour les **8 villes hors
Paris** (bordeaux, lille, lyon, montpellier, nantes, rennes, strasbourg,
toulouse). Paris reste géré par
[`pasdevelib/pdv-bot`](https://github.com/pasdevelib/pdv-bot).

## Pourquoi un dépôt séparé (2026-09-28)

GitHub Actions retarde et laisse tomber la plupart des déclenchements
`schedule` quand un même dépôt en cumule trop, surtout à haute fréquence
(`*/5 * * * *`). Ça a provoqué une régression de 3 mois sur le scraping
Paris (voir l'historique de `pdv-bot`), découverte fin septembre 2026.
Une première consolidation (passage de 8 workflows par ville à un seul
workflow en matrice) a réduit le nombre de déclarations `schedule` dans
`pdv-bot`, sans totalement l'éliminer.

Ce dépôt va plus loin : en isolant les villes en région dans leur propre
dépôt, **Paris dispose de son propre quota de déclenchements programmés**,
complètement indépendant de celui des 7 autres villes. Un pic d'activité
ou un futur bug de scheduling sur les villes en région ne peut plus
affecter Paris, et inversement.

## Pas de code dupliqué

Ce dépôt ne contient aucun code Python — seulement des workflows. Chaque
job installe le package partagé directement depuis `pdv-bot` :

```
pip install "git+https://github.com/pasdevelib/pdv-bot.git"
```

Toute la logique métier (scraping, consolidation, prévision, stockage)
vit dans un seul endroit (`pdv-bot`), donc aucun risque de divergence
entre deux copies du même code.

## Stockage

Les données (releases GitHub `cities-live`, `cities-history`, etc.)
restent hébergées sur **`pasdevelib/pdv-bot`**, comme avant — rien ne
change côté webapp ni côté format de données. Chaque workflow fixe
explicitement `GITHUB_REPOSITORY: pasdevelib/pdv-bot` pour que
`storage.py` continue de lire/écrire au bon endroit, avec un token
dédié (`PDV_BOT_TOKEN`, secret de ce dépôt) qui a les droits d'écriture
sur les releases de `pdv-bot`.

## Secret requis

`PDV_BOT_TOKEN` — fine-grained PAT scopé sur `pasdevelib/pdv-bot`,
permission `Contents: Read and write`. À créer dans
Settings → Secrets and variables → Actions de **ce** dépôt.
