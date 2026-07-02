# 04 · Preuve image GHCR

**Projet CI/CD — Catal-Log · Image publiée, tag et digest**

| Référence | Valeur |
|---|---|
| Auteur | Candidat n° 04 |
| Dépôt GitHub | https://github.com/Lemigrant09/catal-log-ec06 |
| Bloc / Évaluation | RNCP39611BC02 — EC06 |

---

## 1. Image publiée

| Élément | Valeur |
|---|---|
| Nom de l'image | `ghcr.io/lemigrant09/catal-log-ec06` |
| Tags | `latest`, `sha-2707f30`, `production-simulee` |
| Digest | `sha256:4fe660ae6e6eccd0caf23af9ade895cbc2550568628b524b30889768d7b455b4` |
| Workflow de publication | `02-publish-ghcr.yml` |
| Visibilité | Public |
| Lien de la page GHCR | https://github.com/Lemigrant09/catal-log-ec06/pkgs/container/catal-log-ec06 |
| Lien du run de publication | https://github.com/Lemigrant09/catal-log-ec06/actions/runs/28494204901 |

<img width="1324" height="873" alt="image" src="https://github.com/user-attachments/assets/09538187-08be-4854-8ef9-e0567295001c" />

<img width="1007" height="701" alt="image" src="https://github.com/user-attachments/assets/6c0226ef-b72a-40cf-b388-2364b27a92b7" />

## 2. Rôle du tag et du digest

L'image est identifiée à deux niveaux complémentaires : le tag et le digest.

**Le tag** est une étiquette lisible, mais **mobile**. `latest` désigne le dernier build publié sur `main` : il peut demain pointer vers une image différente. Le tag `sha-2707f30` est plus stable car lié à un commit précis, et le tag `production-simulee` marque l'image promue.

**Le digest** (`sha256:...`) est une empreinte cryptographique **immuable** du contenu de l'image. Un même digest désigne toujours exactement la même image, octet pour octet. Si le contenu change, le digest change.

## 3. Pourquoi c'est utile pour la traçabilité et le rollback

- **Traçabilité** : à partir du digest, on sait précisément quelle image tourne, quel build l'a produite et quel commit du dépôt elle contient. Il n'y a aucune ambiguïté possible, contrairement à un tag qui peut être réattribué.
- **Rollback fiable** : pour revenir à une version antérieure, on redéploie une image **par son digest** (ou un tag immuable de type `sha-...`), et non par `latest`. On a ainsi la certitude de restaurer exactement la version voulue, déjà construite et déjà testée, sans reconstruction.
- **Promotion sûre** : la promotion vers `production-simulee` s'appuie sur une image déjà publiée et identifiée. Ce qui est validé en recette est, par construction, exactement ce qui est promu.
