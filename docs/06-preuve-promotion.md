# 06 · Preuve promotion production-simulee

**Projet CI/CD — Catal-Log · Promotion d'un artefact validé, sans reconstruction**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Workflow de promotion

| Élément | Valeur |
|---|---|
| Workflow | `03-promote.yml` — job `promote-production-simulee` |
| Environnement GitHub | `production-simulee` |
| Dépendance | S'exécute uniquement après la validation recette (`needs: validate-recette`) |
| Tag source | `latest` |
| Tag cible | `production-simulee` |
| Lien du run | *(coller l'URL du run vert 03-promote)* |

## 2. Déroulement

Le job `promote-production-simulee` reprend l'image **déjà validée en recette**. Il applique le nouveau tag `production-simulee` sur ce même artefact, puis le republie dans GHCR :

```bash
docker pull  ghcr.io/lemigrant09/catal-log-ec06:latest
docker tag   ghcr.io/lemigrant09/catal-log-ec06:latest ghcr.io/lemigrant09/catal-log-ec06:production-simulee
docker push  ghcr.io/lemigrant09/catal-log-ec06:production-simulee
```

Aucune commande `docker build` n'intervient : l'image n'est pas reconstruite, seule une étiquette supplémentaire est posée sur un artefact existant.

## 3. Preuve d'absence de rebuild

Deux éléments prouvent que la promotion s'est faite **sans reconstruction** :

1. Le résumé du run affiche explicitement **« Mode : promotion sans rebuild »**, et les étapes du job ne contiennent aucune commande `docker build`.
2. Dans GHCR, le tag `production-simulee` apparaît sur **la même version d'image** que `latest` et `sha-...` : les trois tags partagent le même digest. Un tag posé sur une image existante ne crée pas de nouvelle image.

<img width="1304" height="688" alt="image" src="https://github.com/user-attachments/assets/52cad3af-3bfe-4070-bada-df9dee2493db" />


Preuve : [https://github.com/... (l'URL que tu copies)](https://github.com/Lemigrant09/catal-log-ec06/pkgs/container/catal-log-ec06)

## 4. Intérêt de cette approche

Promouvoir sans reconstruire est un principe fondamental de la livraison continue fiable : l'artefact déployé en production est **exactement** celui qui a été testé en recette. Une reconstruction, même à partir du même code source, pourrait introduire une différence (dépendance mise à jour, base modifiée). En réutilisant l'image identifiée par son digest, on élimine ce risque et on garantit la cohérence entre les environnements.
