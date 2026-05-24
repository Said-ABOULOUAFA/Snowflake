# 📚 Snowflake & Data Engineering — Documentation

Documentation personnelle sur Snowflake et le Data Engineering, générée avec [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## 🚀 Déploiement automatique

À chaque `git push` sur `main`, la doc est **automatiquement déployée** sur GitHub Pages via GitHub Actions.

## 🛠️ Lancer en local

```bash
# 1. Installer les dépendances
pip install -r requirements.txt

# 2. Lancer le serveur local (hot-reload)
mkdocs serve

# 3. Ouvrir dans le navigateur
# → http://127.0.0.1:8000
```

## 📁 Structure

```
docs/
├── index.md                        # Page d'accueil
├── getting-started/                # Connexion, Snowsight
├── architecture/                   # Concepts, stockage, calcul
├── data-engineering/               # Pipelines, Snowpipe, dbt
├── analytics/                      # SQL avancé, optimisation
├── ai-ml/                          # Cortex, ML
└── best-practices/                 # Sécurité, coûts, checklist
```

## ✏️ Ajouter du contenu

1. Créer un fichier `.md` dans le bon dossier `docs/`
2. L'ajouter dans `mkdocs.yml` sous `nav:`
3. `git add . && git commit -m "feat: nouvelle page" && git push`
4. La doc se déploie automatiquement ✅

## ⚙️ Configuration GitHub Pages

Après le premier push :

1. Aller dans **Settings → Pages** du repo
2. Source : **Deploy from a branch**
3. Branch : **gh-pages** / **root**
4. Sauvegarder → ta doc sera disponible sur `https://<username>.github.io/<repo>/`
