# Projet CI/CD — Catal-Log

Chaîne d'intégration et de déploiement continu pour la publication d'une image Docker Nginx servant un site web statique, via **GitHub Actions** et **GitHub Container Registry (GHCR)**.

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Formation | Administrateur Systèmes, Réseaux et Cybersécurité — RNCP39611 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |
| Registre d'image | `ghcr.io/lemigrant09/catal-log-ec06` |

---

## Objectif

Construire, tester, publier et promouvoir une image Docker de façon automatisée et traçable : d'un simple commit jusqu'à la promotion d'un artefact identifié vers une production simulée, sans reconstruction et sans serveur à administrer.

## Chaîne en bref

```
Commit  ->  01-ci (build + test HTTP)  ->  02-publish (GHCR : tag + digest)
        ->  03-promote (recette -> production-simulee, sans rebuild)
```

## Arborescence

```
catal-log-ec06/
├── site/
│   ├── index.html          Site statique servi par Nginx
│   └── version.json        Version, auteur, contexte EC06
├── Dockerfile              Image Nginx reproductible
├── compose.yml             Orchestration légère (web + tester)
├── .github/workflows/
│   ├── 01-ci.yml           Build de l'image + test HTTP automatisé
│   ├── 02-publish-ghcr.yml Publication dans GHCR (tag + digest)
│   ├── 03-promote.yml      Promotion recette -> production-simulee
│   └── 04-scaling.yml      Simulation de scaling (docker compose --scale)
└── docs/                   Documentation et preuves (voir ci-dessous)
```

## Documentation

| Document | Contenu |
|---|---|
| [01 · Cadrage](docs/01-cadrage-projet.md) | Objectif, contraintes, choix techniques |
| [02 · Schéma de la chaîne](docs/02-schema-chaine-cicd.md) | Diagramme et rôle de chaque étape |
| [03 · Fiche tests](docs/03-fiche-tests.md) | Test automatisé, scaling, limites |
| [04 · Preuve image GHCR](docs/04-preuve-image.md) | Image, tags, digest, traçabilité |
| [05 · Preuve recette](docs/05-preuve-recette.md) | Validation en environnement recette |
| [06 · Preuve promotion](docs/06-preuve-promotion.md) | Promotion sans reconstruction |
| [07 · Sécurité et production réelle](docs/07-securite-minimale.md) | Secrets, rollback, sauvegarde, moindre privilège |
| [08 · Compte rendu final](docs/08-compte-rendu-final.md) | Synthèse, compétences, apprentissages |

## Workflows

| Workflow | Déclenchement | Rôle |
|---|---|---|
| `01-ci.yml` | Push / PR | Construit l'image et teste la réponse HTTP du conteneur |
| `02-publish-ghcr.yml` | Push sur `main` | Publie l'image dans GHCR avec tag et digest |
| `03-promote.yml` | Manuel (`workflow_dispatch`) | Valide en recette puis promeut en production simulée, sans rebuild |
| `04-scaling.yml` | Manuel (`workflow_dispatch`) | Démarre deux instances du service web (mise à l'échelle) |

## Principes appliqués

- **Traçabilité** : chaque image est identifiée par un tag immuable (`sha-<commit>`) et un digest `sha256`.
- **Promotion sans rebuild** : l'artefact validé en recette est exactement celui promu en production.
- **Moindre privilège** : chaque workflow ne déclare que les permissions strictement nécessaires.
- **Aucun secret dans le code** : authentification GHCR via le `GITHUB_TOKEN` éphémère.
