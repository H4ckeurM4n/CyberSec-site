# CyberSec Notes

Blog de cours et tutoriels de cybersécurité (Bases, Bash, Python, Pentest, OSINT),
construit avec [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Prérequis

- Python 3.9 ou plus récent
- `pip`

## Lancer le site en local

```bash
# 1. (Optionnel mais recommandé) créer un environnement virtuel
python3 -m venv .venv
source .venv/bin/activate        # sous Windows : .venv\Scripts\activate

# 2. Installer les dépendances
pip install -r requirements.txt

# 3. Lancer le serveur de développement
mkdocs serve
```

Le site est alors visible sur <http://127.0.0.1:8000>. Il se recharge tout seul
à chaque modification d'un fichier.

> [!NOTE]
> **À propos de l'écosystème MkDocs (2026).** Material for MkDocs est passé en
> mode maintenance : il reçoit les correctifs de sécurité pendant au moins 12
> mois et reste un choix sûr aujourd'hui. Son successeur, **Zensical** (même
> équipe), lit les fichiers `mkdocs.yml` existants — ce projet pourra donc y
> migrer plus tard sans tout réécrire. Un avertissement concernant « MkDocs 2.0 »
> (un projet séparé) peut s'afficher au lancement : il est sans incidence. Pour
> le masquer, lance `NO_MKDOCS_2_WARNING=1 mkdocs serve`.

## Ajouter une fiche (un cours)

1. Crée un fichier `.md` dans le bon dossier, par ex. `docs/bash/les-pipes.md`.
2. Ajoute-le à la navigation dans `mkdocs.yml`, sous le bon pilier :

   ```yaml
   - Bash:
       - bash/index.md
       - Naviguer dans le terminal: bash/naviguer-terminal.md
       - Les pipes: bash/les-pipes.md   # <-- ta nouvelle fiche
   ```

3. Pense à relier la fiche aux autres (maillage interne) et à la lister dans la
   page `index.md` du pilier.

## Mettre en forme une fiche

Les fiches d'exemple (`docs/bases/modele-osi.md` et
`docs/bash/naviguer-terminal.md`) montrent les éléments disponibles :
encadrés (`!!! note`, `!!! tip`, `!!! warning`), onglets (`=== "Linux"`),
blocs de code avec bouton copier, tableaux, listes de tâches, et tags.

## Déployer sur GitHub Pages

1. Crée un dépôt GitHub et pousse ce dossier dessus :

   ```bash
   git init
   git add .
   git commit -m "Initialisation du blog"
   git branch -M main
   git remote add origin https://github.com/H4ckeurM4n/CyberSec-notes.git
   git push -u origin main
   ```

2. Le workflow `.github/workflows/deploy.yml` construit et publie le site
   automatiquement à chaque `push` sur `main`.
3. Dans les réglages du dépôt → **Pages**, choisis la branche `gh-pages`.
4. Pense à mettre à jour `site_url`, `repo_url` et `repo_name` dans `mkdocs.yml`.

## Structure du projet

```
CyberSec-notes/
├── mkdocs.yml          # Configuration (thème, nav, extensions)
├── requirements.txt    # Dépendances Python
├── docs/
│   ├── index.md        # Page d'accueil (hub)
│   ├── assets/         # CSS, favicon, images
│   ├── bases/          # Pilier Bases
│   ├── bash/           # Pilier Bash
│   ├── python/         # Pilier Python
│   ├── pentest/        # Pilier Pentest
│   └── osint/          # Pilier OSINT
└── .github/workflows/  # Déploiement automatique
```
