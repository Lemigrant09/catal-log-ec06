# 05 · Preuve recette simulée

**Projet CI/CD — Catal-Log · Validation de l'image en environnement de recette**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Workflow de validation recette

| Élément | Valeur |
|---|---|
| Workflow | `03-promote.yml` — job `validate-recette` |
| Environnement GitHub | `recette` |
| Déclenchement | Manuel (`workflow_dispatch`) |
| Tag source validé | `latest` |
| Digest observé | `sha256:42a4fda3833e9eb9f3b24374cfa2e1df338a94ef36a8dcbeb4f473d2ab4b64ec` |
| Lien du run | *(coller l'URL du run vert 03-promote)* |

## 2. Déroulement

Le job `validate-recette` s'exécute dans l'environnement GitHub `recette`. Il ne reconstruit pas l'image : il **télécharge l'image déjà publiée** dans GHCR (`docker pull`), la démarre dans un conteneur, puis vérifie par des requêtes HTTP que la page d'accueil et `version.json` répondent correctement.

Cette étape simule une **recette** : on valide un artefact existant dans un environnement dédié, avant toute promotion vers la production. Le digest est relevé pour tracer précisément quelle image a été validée.

## 3. Résultat

Le test HTTP réalisé sur l'image téléchargée s'est terminé avec succès : la page et le fichier de version sont servis correctement par le conteneur issu de l'image GHCR. La validation recette est donc **concluante**, ce qui autorise le passage à l'étape de promotion.

> 📷 **Insérer ici :** capture du résumé « Validation recette simulée » du run `03-promote`, montrant l'image source et le digest observé.

## 4. Point clé

La validation porte sur **l'image publiée telle quelle**, sans reconstruction. C'est ce qui garantit que l'artefact testé en recette est identique à celui qui sera promu en production simulée : même image, même digest, aucune divergence possible entre ce qui est validé et ce qui est déployé.
