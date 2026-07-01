# 01 · Cadrage du projet

**Projet CI/CD — Catal-Log · Publication et promotion d'une image Docker Nginx**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Formation | Administrateur Systèmes, Réseaux et Cybersécurité — RNCP39611 |
| Bloc | RNCP39611BC02 — Configurer et administrer l'infrastructure réseau et les solutions cloud |
| Évaluation | EC06 — Automatisation d'intégration et de déploiement continu |
| Date de réalisation | 01/07/2026 |

---

## 1. Objectif

Mettre en place une chaîne CI/CD permettant de **construire, tester, publier et promouvoir** une image Docker Nginx contenant un site web statique, dans le cadre du scénario Catal-Log. La chaîne doit être simple, lisible, traçable, et reposer intégralement sur GitHub Actions et GitHub Container Registry (GHCR), sans infrastructure à administrer.

## 2. Périmètre technique

| Composant | Rôle dans le projet |
|---|---|
| GitHub | Dépôt individuel, historique de commits, documentation, preuves |
| GitHub Actions | Contrôles, build Docker, tests, publication GHCR, promotion |
| Runners GitHub | Environnements d'exécution temporaires, sans serveur à maintenir |
| Dockerfile | Construction reproductible de l'image Nginx |
| GHCR | Publication de l'image, conservation des tags et digests |
| compose.yml | Orchestration légère et simulation de coordination de conteneurs |
| Environnements GitHub | `recette` et `production-simulee` (environnements simulés séparés) |

## 3. Contraintes

- Travail strictement individuel.
- Aucune infrastructure fournie ni administrée par le formateur.
- Pas de serveur distant, pas de SSH, pas de fournisseur cloud imposé.
- Les traitements principaux s'exécutent dans GitHub Actions sur des runners éphémères.
- Le test local Docker et l'usage d'une VM personnelle sont optionnels et doivent être justifiés s'ils ne sont pas réalisés.

## 4. Choix retenus

| Décision | Choix | Justification |
|---|---|---|
| Visibilité du dépôt | **Public** | Simplifie l'accès à l'image GHCR et rend les preuves directement consultables par le formateur. |
| Nommage du dépôt | `catal-log-ec06` | Identifie le client (Catal-Log) et l'évaluation (EC06). |
| Nom de l'image | `ghcr.io/lemigrant09/catal-log-ec06` | Dérivé automatiquement du dépôt (`${GITHUB_REPOSITORY}` en minuscules). |
| Stratégie de tags | `sha-<commit>`, `latest`, `production-simulee` | Un tag immuable par commit pour la traçabilité, un pointeur mobile sur le dernier build, un tag dédié à la promotion. |
| Environnement local Docker | **Non utilisé** | Voir justification §5. |
| VM personnelle | **Non utilisée** | Voir justification §5. |

## 5. Justification de l'exécution 100 % GitHub Actions

Le poste de travail personnel fonctionne sous Windows sans moteur Docker installé. Conformément au socle du sujet, qui rend le test local et la VM optionnels, l'intégralité de la chaîne (build, test HTTP, publication, promotion) est exécutée sur les **runners GitHub-hosted**, qui embarquent nativement Docker. Cette approche présente trois avantages cohérents avec l'objectif du module :

- **Reproductibilité** : chaque exécution part d'un environnement propre et identique, indépendant du poste.
- **Aucune administration** : les runners sont éphémères, il n'y a ni serveur ni VM à maintenir ou sécuriser.
- **Traçabilité** : chaque étape laisse un journal et un artefact vérifiables dans GitHub Actions et GHCR.

Le test local n'apporterait pas de garantie supplémentaire par rapport au test automatisé déjà réalisé dans `01-ci.yml`, qui construit l'image et vérifie la réponse HTTP du conteneur à chaque commit.
