# 03 · Fiche tests

**Projet CI/CD — Catal-Log · Tests automatisés et simulation de scaling**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Test automatisé (GitHub Actions)

| Élément | Détail |
|---|---|
| Workflow | `01-ci.yml` — job `build-and-test` |
| Déclenchement | Automatique à chaque `push` et `pull_request` |
| Lien du run | *(coller l'URL du run vert 01-ci)* |

**Ce qui est testé :**

1. Présence des fichiers attendus (`Dockerfile`, `compose.yml`, `site/index.html`, `site/version.json`, `docs/08-compte-rendu-final.md`).
2. Validité de la configuration Compose (`docker compose config`).
3. Construction de l'image Docker (`docker build`).
4. Démarrage du conteneur et réponse HTTP de la page d'accueil et de `version.json` (`curl`).
5. Présence du contenu attendu dans la page (recherche de la chaîne « Projet CICD »).

**Résultat observé :** run en succès, résumé « Test HTTP : OK ». Le conteneur répond correctement en HTTP, le test valide donc à la fois la construction de l'image et le bon fonctionnement du site servi par Nginx.

<img width="1065" height="664" alt="image" src="https://github.com/user-attachments/assets/eec3dd92-920c-4e2b-9d00-0093e4695f5f" />

## 2. Test local Docker (justification de non-réalisation)

Le test local avec Docker ou Docker Compose n'a pas été réalisé sur le poste de travail : celui-ci fonctionne sous Windows sans moteur Docker installé, situation explicitement prévue par le sujet (« ou justification documentée si l'environnement personnel ne le permet pas »).

Ce choix n'entraîne aucune perte de couverture de test : le même enchaînement (`docker build`, démarrage du conteneur, vérification HTTP) est exécuté automatiquement dans `01-ci.yml` sur un runner GitHub, à chaque commit. Le test automatisé offre une garantie supérieure au test manuel, car il s'exécute dans un environnement propre et reproductible, indépendant du poste.

## 3. Simulation de scaling (compétence C13)

La simulation de mise à l'échelle a été exécutée réellement, dans un workflow dédié (`04-scaling.yml`), sur un runner GitHub disposant de Docker Compose.

| Élément | Détail |
|---|---|
| Workflow | `04-scaling.yml` |
| Commande | `docker compose up -d --build --scale web=2` |
| Contrôle | Le workflow échoue si le nombre d'instances `web` n'est pas exactement 2 |
| Lien du run | *(coller l'URL du run vert 04-scaling)* |

**Résultat observé — deux instances du service `web` en parallèle :**

```
NAME                   IMAGE                     SERVICE   STATUS
catal-log-ec06-web-1   projet-cicd-nginx:local   web       running
catal-log-ec06-web-2   projet-cicd-nginx:local   web       running
```

> 📷 **Insérer ici :** capture du résumé « Simulation de scaling » du run `04-scaling`, montrant `catal-log-ec06-web-1` et `catal-log-ec06-web-2`.

Le service `web` ne publie pas de port fixe sur l'hôte (il utilise `expose` et non `ports`). C'est cette absence de port mappé qui permet de démarrer plusieurs instances sans conflit — principe de base de la mise à l'échelle horizontale.

## 4. Limites de la simulation

Cette simulation démontre la coordination de plusieurs conteneurs, mais ne constitue pas une orchestration de production. Ses limites principales :

- **Pas de répartiteur de charge** : aucune brique ne distribue automatiquement le trafic entre `web-1` et `web-2`. En production, un load balancer (ou un Ingress) serait nécessaire.
- **Pas de haute disponibilité** : les deux instances tournent sur une seule machine (le runner). Si elle tombe, tout tombe. Une vraie HA suppose plusieurs nœuds.
- **Pas d'auto-scaling** : le nombre d'instances est fixé manuellement, sans adaptation à la charge.
- **Pas de supervision ni de reprise automatique** : aucun redémarrage automatisé en cas de défaillance d'une instance.
- **Environnement éphémère** : les conteneurs sont détruits à la fin du run (`docker compose down`).

Ces limites justifient le recours à un orchestrateur (type Kubernetes) pour une production réelle, comme analysé dans le compte rendu final.
