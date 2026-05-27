# CyberSec

Blog de cours et tutoriels de cybersécurité : Bases, Bash, Python, Pentest, OSINT.

Construit avec [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Prérequis

- Python 3.9 ou plus récent
- `pip`
- Git

## Lancer le site en local

```bash
# 1. Créer un environnement virtuel
python3 -m venv .venv

# 2. Activer l'environnement virtuel
source .venv/bin/activate

# Sous Windows :
# .venv\Scripts\activate

# 3. Installer les dépendances
pip install -r requirements.txt

# 4. Lancer le serveur de développement
mkdocs serve
```

Le site est alors visible sur :

```txt
http://127.0.0.1:8000
```

Il se recharge automatiquement à chaque modification d’un fichier.

> [!NOTE]
> **À propos de l’écosystème MkDocs.**  
> Le site utilise MkDocs Material. Si un avertissement concernant MkDocs 2.0 apparaît au lancement, il peut être masqué avec :
>
> ```bash
> NO_MKDOCS_2_WARNING=1 mkdocs serve
> ```

## Workflow courant

Pendant la rédaction :

```bash
source .venv/bin/activate
mkdocs serve
```

Avant de publier :

```bash
mkdocs build
```

Publier les modifications sur GitHub :

```bash
git status
git add .
git commit -m "Met à jour le site"
git push
```

## Ajouter une fiche

1. Créer un fichier `.md` dans le bon dossier.

Exemple :

```txt
docs/bash/les-pipes.md
```

2. Ajouter la fiche dans la navigation du fichier `mkdocs.yml`.

Exemple :

```yaml
- Bash:
    - bash/index.md
    - Scripting Bash:
        - Les pipes: bash/les-pipes.md
```

3. Ajouter éventuellement un lien dans la page `index.md` du pilier concerné.

4. Tester le site :

```bash
mkdocs build
```

5. Publier :

```bash
git add .
git commit -m "Ajoute une fiche"
git push
```

## Mettre en forme une fiche

Les fiches peuvent utiliser les éléments MkDocs Material suivants :

- encadrés : `!!! note`, `!!! tip`, `!!! warning`, `!!! danger`
- blocs repliables : `??? example`
- blocs de code avec bouton copier
- tableaux Markdown
- listes de tâches
- tags
- liens internes entre fiches

Exemple :

```md
!!! tip "Bonne pratique"
    Entoure tes variables avec des guillemets : `"$variable"`.
```

## Déployer sur GitHub Pages

1. Crée un dépôt GitHub et pousse ce dossier dessus :

   ```bash
   git init
   git add .
   git commit -m "Initialisation du blog"
   git branch -M main
   git remote add origin https://github.com/TONPSEUDO/blog-cyber.git
   git push -u origin main
   ```

2. Le workflow `.github/workflows/deploy.yml` construit et publie le site
   automatiquement à chaque `push` sur `main`.
3. Dans les réglages du dépôt → **Pages**, choisis la branche `gh-pages`.
4. Pense à mettre à jour `site_url`, `repo_url` et `repo_name` dans `mkdocs.yml`.
5. Le site sera disponible ici :

```txt
https://h4ckeurm4n.github.io/CyberSec-site/
```

## Structure du projet

```txt
blog-cyber/
├── mkdocs.yml              # Configuration du site : thème, navigation, extensions
├── requirements.txt        # Dépendances Python
├── README.md               # Documentation du projet
├── .gitignore              # Fichiers ignorés par Git
├── docs/
│   ├── index.md            # Page d'accueil
│   ├── assets/             # CSS, favicon, images
│   │   └── extra.css       # Style personnalisé du site
│   ├── bases/              # Pilier Bases
│   ├── bash/               # Pilier Bash / Terminal
│   ├── python/             # Pilier Python
│   ├── pentest/            # Pilier Pentest
│   └── osint/              # Pilier OSINT
└── .github/
    └── workflows/
        └── deploy.yml      # Déploiement automatique GitHub Pages
```