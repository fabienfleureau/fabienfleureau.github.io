# Réparer l'heure Carminat TomTom

Outil web pour patcher le fichier `PNDNavigator` sur la carte SD du système
Carminat TomTom (Clio II / Mégane II / Laguna II).

## Déploiement GitHub Pages

1. Créer un dépôt GitHub (ex. `renault-horloge`)
2. Pousser `main` : `git push -u origin main`
3. Settings → Pages → Source : `main` / `/ (root)`
4. Partager l'URL : `https://<username>.github.io/renault-horloge/`

## Fonctionnement

- 100 % côté client — le fichier ne quitte jamais l'ordinateur de l'utilisateur
- Patch : offset `0x00402928`, remplace `0xD6` par `0xB9`
