# 07 · Sécurité minimale et analyse « production réelle »

**Projet CI/CD — Catal-Log · Sécurité DevOps, secrets, rollback, sauvegarde**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Permissions des workflows (principe du moindre privilège)

Chaque workflow ne déclare que les permissions strictement nécessaires à sa tâche. Le jeton d'authentification (`GITHUB_TOKEN`) est ainsi limité au minimum requis.

| Workflow | Permissions | Justification |
|---|---|---|
| `01-ci.yml` | `contents: read` | N'a besoin que de lire le code pour construire et tester l'image. Aucune écriture. |
| `02-publish-ghcr.yml` | `contents: read`, `packages: write` | Lit le code et publie l'image dans GHCR (écriture sur les packages). |
| `03-promote.yml` | `contents: read`, `packages: write` | Lit le code et republie l'image promue dans GHCR. |
| `04-scaling.yml` | `contents: read` | Ne fait que démarrer des conteneurs localement sur le runner. |

**Pourquoi limiter ?** Si le jeton était compromis pendant un run, un périmètre restreint réduit fortement les dégâts possibles : un token en lecture seule ne peut ni publier d'image malveillante ni modifier le dépôt. C'est l'application directe du principe du moindre privilège.

## 2. Gestion des secrets

### 2.1 Pourquoi aucun secret ne doit être écrit dans le code

Un secret inscrit dans le code (mot de passe, clé API, token) est **exposé** de plusieurs façons : il reste dans l'historique Git même après suppression, il est visible par toute personne ayant accès au dépôt (ici public), et il peut fuiter via une capture ou un fork. Un secret « en clair » dans un dépôt doit être considéré comme **compromis définitivement**. La bonne pratique est de ne jamais stocker de secret dans le code, mais de l'injecter à l'exécution depuis un magasin dédié.

### 2.2 Usage du GITHUB_TOKEN dans ce projet

Ce projet n'a besoin d'**aucun secret créé manuellement**. La connexion à GHCR utilise le `GITHUB_TOKEN`, un jeton **généré automatiquement par GitHub à chaque exécution** de workflow. Ses caractéristiques :

- il est **éphémère** : créé au début du run, révoqué à la fin ;
- il est **à portée limitée** : ses droits sont ceux déclarés dans le bloc `permissions` du workflow ;
- il n'est **jamais écrit dans le code** : on y accède via `${{ secrets.GITHUB_TOKEN }}`, injecté à l'exécution.

C'est la raison pour laquelle la publication et la promotion fonctionnent sans qu'aucun mot de passe n'apparaisse dans le dépôt.

### 2.3 Ce qu'il faudrait sécuriser en production réelle

Dans un contexte de production, des secrets supplémentaires seraient nécessaires. Ils devraient être placés dans **GitHub Secrets** (au niveau dépôt ou environnement) ou, pour une organisation plus mature, dans un **coffre de secrets dédié** (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault). Exemples :

- identifiants d'un registre d'images privé ;
- clés d'accès au fournisseur cloud (déploiement de l'infrastructure) ;
- clés SSH ou tokens de déploiement vers les serveurs cibles ;
- identifiants de base de données, clés d'API tierces ;
- certificats TLS et clés privées.

Ces secrets seraient injectés à l'exécution, chiffrés au repos, à accès tracé, et **rotés régulièrement**.

## 3. Rollback (retour à une version précédente)

Le rollback s'appuie sur le fait que chaque image publiée est **conservée et identifiée** dans GHCR par un tag immuable (`sha-<commit>`) et un digest (`sha256:...`).

**Procédure de retour arrière, sans reconstruction :**

1. Identifier la dernière version stable connue via son tag `sha-...` ou son digest dans GHCR.
2. Re-promouvoir cet artefact antérieur en réappliquant le tag `production-simulee` dessus (même mécanisme que la promotion normale : `pull` → `tag` → `push`), ou en déployant directement l'image **par son digest**.
3. Aucun `docker build` n'est nécessaire : on redéploie une image **déjà construite et déjà testée**.

L'intérêt est double : le retour arrière est **rapide** (pas de reconstruction) et **fiable** (on restaure exactement la version voulue, garantie par le digest, sans risque de divergence).

## 4. Sauvegarde et restauration

| Élément à sauvegarder | Où / comment | Rôle en restauration |
|---|---|---|
| Code, `Dockerfile`, `compose.yml`, workflows | Versionnés dans Git ; miroir/clone régulier hors GitHub | Reconstituer entièrement le projet |
| Documentation (`docs/*.md`) | Versionnée dans Git | Retrouver cadrage, preuves, analyses |
| Images publiées | Conservées dans GHCR ; export ou miroir vers un second registre | Redéployer sans reconstruire |
| Configuration hors dépôt (environnements, secrets, règles de protection) | **Documentée** (elle n'est pas dans le code) | Recréer les environnements `recette` / `production-simulee` et réinjecter les secrets |
| Preuves (captures, logs de runs) | Archivées (dossier de preuves, export des logs Actions) | Justifier et tracer les exécutions passées |

**Restauration type :** recloner le dépôt (récupère code + workflows + docs), recréer les environnements et secrets à partir de la documentation, republier ou re-tirer les images depuis les digests conservés. Le caractère versionné du dépôt et l'immuabilité des images rendent la reprise simple et déterministe.

## 5. Éléments complémentaires pour une production réelle

Au-delà des trois points obligatoires, plusieurs briques seraient à ajouter. Quatre sont particulièrement pertinentes pour ce projet :

**a) Validation manuelle avant production** *(déjà en place)*
La promotion est déclenchée manuellement (`workflow_dispatch`) et passe par l'environnement `production-simulee`. En production réelle, on ajouterait des **required reviewers** sur cet environnement, imposant une approbation humaine avant tout déploiement.

**b) Séparation stricte des environnements** *(déjà en place)*
Les environnements `recette` et `production-simulee` sont distincts. En production, ils auraient des secrets, des accès et éventuellement des infrastructures séparés, empêchant toute fuite d'un environnement vers l'autre.

**c) Contrôle des vulnérabilités** *(à ajouter)*
Un scan d'image automatisé (par exemple Trivy) intégré au pipeline permettrait de détecter les CVE dans l'image Nginx avant publication, et de bloquer la chaîne en cas de vulnérabilité critique.

**d) Gestion des accès** *(à ajouter)*
En production, l'accès au dépôt, au registre et aux environnements serait restreint par rôles (principe du moindre privilège appliqué aux personnes), avec authentification forte et journalisation des accès.

## 6. Comparaison avec une orchestration de production (Kubernetes)

Ce projet démontre une chaîne CI/CD complète mais repose sur Docker Compose, adapté à la démonstration et non à la production. Une production réelle s'appuierait plutôt sur un orchestrateur type **Kubernetes**, qui apporte ce que Compose ne couvre pas : haute disponibilité multi-nœuds, répartition de charge, auto-scaling selon la charge, redémarrage automatique des conteneurs défaillants, déploiements progressifs (rolling update) et rollback intégré. Le pipeline construit ici resterait valable : il produirait toujours une image identifiée par son digest, que Kubernetes déploierait à la place de Compose.
