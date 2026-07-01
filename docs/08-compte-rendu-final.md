# 08 · Compte rendu final

**Projet CI/CD — Catal-Log · Synthèse personnelle**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Synthèse

J'ai mis en place une chaîne CI/CD complète pour le scénario Catal-Log : à partir d'un simple commit, une image Docker Nginx contenant un site statique est automatiquement construite, testée, publiée dans GitHub Container Registry (GHCR), validée en recette simulée, puis promue vers une production simulée sans reconstruction. L'ensemble s'exécute sur des runners GitHub, sans serveur ni infrastructure à administrer, et chaque étape laisse une preuve vérifiable.

## 2. Fonctionnement technique (le chemin complet)

1. **Commit / push** sur le dépôt GitHub.
2. **`01-ci.yml`** : vérification des fichiers, `docker build`, démarrage du conteneur et test HTTP (`curl`) de la page et de `version.json`.
3. **`02-publish-ghcr.yml`** : construction et publication de l'image dans GHCR avec un tag `sha-<commit>`, le tag `latest` et un digest immuable.
4. **`03-promote.yml`, job recette** : l'image publiée est téléchargée (sans rebuild) et validée par un test HTTP dans l'environnement `recette`.
5. **`03-promote.yml`, job production** : le tag `production-simulee` est appliqué sur le même artefact, qui est republié — sans reconstruction.

## 3. Conteneurisation (compétence C12)

Le `Dockerfile` part de l'image officielle `nginx:1.27-alpine`, copie le site statique dans le répertoire servi par Nginx, expose le port 80 et déclare un `HEALTHCHECK`. L'image est construite automatiquement dans GitHub Actions et exécutée dans un conteneur testé en HTTP. La preuve de conteneurisation est le run `01-ci` en succès (build + test), détaillée dans `03-fiche-tests.md`.

## 4. Orchestration et scaling (compétence C13)

Le `compose.yml` décrit deux services coordonnés : `web` (Nginx) et `tester` (vérification HTTP via curl). Docker Compose joue le rôle d'orchestration légère (déclaration des services, dépendances, réseau commun). La mise à l'échelle a été démontrée réellement dans `04-scaling.yml` avec `docker compose up --scale web=2`, qui a lancé deux instances `web-1` et `web-2` en parallèle. Les limites de cette approche (pas de load balancer, pas de haute disponibilité, pas d'auto-scaling) sont analysées dans `03-fiche-tests.md` et `07-securite-minimale.md`.

## 5. Automatisation et sécurité (compétence C14)

L'automatisation repose sur quatre workflows GitHub Actions. La sécurité s'appuie sur trois principes :

- **Moindre privilège** : chaque workflow ne déclare que les permissions nécessaires (`contents: read`, et `packages: write` uniquement pour la publication et la promotion).
- **Aucun secret dans le code** : l'authentification à GHCR utilise le `GITHUB_TOKEN`, généré automatiquement et éphémère, jamais écrit dans le dépôt.
- **Traçabilité et rollback** : chaque image est identifiée par un tag immuable et un digest, ce qui permet un retour arrière fiable sans reconstruction.

Le détail figure dans `07-securite-minimale.md`.

## 6. Passage vers une production réelle

Les trois points obligatoires sont traités en détail dans `07-securite-minimale.md` :

- **Gestion des secrets** : usage du `GITHUB_TOKEN`, recours à GitHub Secrets ou à un coffre (Vault, KMS cloud) pour les secrets de production.
- **Rollback** : redéploiement d'une image antérieure par son digest ou son tag immuable, sans rebuild.
- **Sauvegarde / restauration** : versionnement Git du code et des workflows, conservation des images dans GHCR, documentation de la configuration hors dépôt.

Éléments complémentaires retenus : validation manuelle avant production et séparation stricte des environnements (déjà en place), ainsi que contrôle des vulnérabilités et gestion des accès (à ajouter en production).

## 7. Preuves

| Preuve | Emplacement |
|---|---|
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Run build + test HTTP | *(lien run 01-ci)* — voir `03-fiche-tests.md` |
| Publication GHCR + digest | *(lien run 02-publish)* — voir `04-preuve-image.md` |
| Image GHCR (tags + digest) | *(lien page GHCR)* — voir `04-preuve-image.md` |
| Validation recette | *(lien run 03-promote)* — voir `05-preuve-recette.md` |
| Promotion sans rebuild | *(lien run 03-promote)* — voir `06-preuve-promotion.md` |
| Simulation de scaling | *(lien run 04-scaling)* — voir `03-fiche-tests.md` |
| Digest de l'image | `sha256:4fe660ae6e6eccd0caf23af9ade895cbc2550568628b524b30889768d7b455b4` |

## 8. Difficultés rencontrées et apprentissages

**Difficultés :**

- Au départ, j'ai cru qu'il fallait installer Docker et monter une VM sur mon poste. J'ai compris que, dans ce projet, Docker s'exécute sur les runners GitHub : il n'y a rien à installer localement, ce que le sujet prévoit explicitement.
- Lors du premier envoi des fichiers, le dossier `.github` (qui commence par un point) n'a pas été pris par le glisser-déposer sous Windows ; j'ai dû recréer les workflows à la main.
- J'ai fait une erreur de chemin en créant un workflow (dossiers dupliqués), corrigée en retapant le chemin proprement.
- La promotion nécessitait de créer au préalable les environnements `recette` et `production-simulee`, sinon le workflow échouait.

**Apprentissages :**

- La différence entre un **tag** (mobile) et un **digest** (immuable), et pourquoi le digest est la référence fiable pour la traçabilité et le rollback.
- Le principe de **promotion sans rebuild** : ce qui est validé en recette est exactement ce qui part en production, car c'est le même artefact.
- L'intérêt du **moindre privilège** et du `GITHUB_TOKEN` pour éviter tout secret en clair.
- Le fonctionnement de la **mise à l'échelle horizontale** avec Docker Compose et pourquoi l'absence de port fixe la rend possible.

Ce projet m'a permis de comprendre concrètement ce qu'est une chaîne CI/CD industrialisée : automatiser pour fiabiliser, tracer chaque étape, et manipuler des artefacts identifiés plutôt que de refaire à chaque fois.
