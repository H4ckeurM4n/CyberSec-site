---
tags:
  - scripting
  - débutant
---

# Écrire son premier script Bash

Un script, c'est simplement un fichier texte qui contient une liste de commandes
que le shell exécute les unes après les autres. Avant de scripter, deux mots de
vocabulaire : le **terminal** est la fenêtre où tu tapes, le **shell** est le
programme à l'intérieur qui comprend et exécute tes commandes (Bash est un shell).

??? note "Glossaire des termes du scripting"
    | Terme | Définition simple |
    | --- | --- |
    | Terminal | La fenêtre où tu tapes des commandes texte |
    | Shell | Le programme qui lit et exécute tes commandes (Bash en est un) |
    | Script | Un fichier texte contenant une liste de commandes |
    | Variable | Un conteneur nommé qui stocke une valeur |
    | Argument | Une info donnée au script quand on le lance |
    | stdout | La sortie normale d'une commande (l'écran par défaut) |
    | stderr | La sortie d'erreur d'une commande |
    | Code de retour | Un nombre (0 = succès) renvoyé par chaque commande |

## Comment penser un script

Presque tous les scripts suivent le même schéma : ils reçoivent quelque chose,
le traitent, puis produisent un résultat.

!!! abstract "Le schéma de base"
    `ENTRÉE` (ce que le script reçoit) → `TRAITEMENT` (ce qu'il en fait) →
    `SORTIE` (ce qu'il produit). Concrètement, un script s'articule autour de cinq
    briques : recevoir des données, les stocker dans des variables, tester des
    conditions, répéter des actions, et enfin afficher ou enregistrer un résultat.

## Ouvrir un terminal

=== "Linux"

    Raccourci ++ctrl+alt+t++, ou cherche « Terminal » dans tes applications.

=== "macOS"

    Cherche « Terminal » dans Spotlight (ou installe iTerm2).

=== "Windows"

    Installe **WSL** (Windows Subsystem for Linux) pour un vrai environnement Linux.

Pour vérifier quel shell tu utilises :

```bash
echo $SHELL
# /bin/bash
```

## Créer et lancer un script, étape par étape

```bash
mkdir -p ~/mes_scripts        # 1. un dossier de travail
cd ~/mes_scripts
nano hello.sh                 # 2. créer le fichier
```

Dans l'éditeur, écris :

```bash
#!/bin/bash
# Mon tout premier script
echo "Hello, World !"
echo "Je suis un script Bash !"
```

Puis rends-le exécutable (une seule fois) et lance-le :

```bash
chmod +x hello.sh             # 3. rendre exécutable
./hello.sh                    # 4. lancer
```

## Le shebang : `#!/bin/bash`

La première ligne `#!/bin/bash` s'appelle le **shebang**. Elle indique au système
quel programme doit lire le fichier.

!!! note "À retenir"
    Mets toujours le shebang en première ligne de tes scripts. Sans lui, le système
    ne sait pas quel langage utiliser pour exécuter le fichier.

## Pourquoi `./` devant le script ?

Quand tu tapes `ls` ou `date`, le système trouve ces commandes grâce à la variable
`PATH`. Mais ton dossier personnel n'est pas dans le `PATH`, donc il faut préciser
« cherche dans le dossier actuel » avec `./`. Tu peux aussi contourner le `chmod`
en lançant directement `bash hello.sh`.

## Terminer un script avec `exit`

La commande `exit` termine le script en renvoyant un **code de retour**.

| Code | Signification |
| --- | --- |
| `0` | Succès — tout s'est bien passé |
| `1` | Erreur générale |
| `127` | Commande introuvable |

!!! tip "Le piège à connaître"
    En Bash, `0` = succès et **tout autre nombre = erreur**. C'est l'inverse de
    l'intuition ! D'autres codes existent (`2`, `126`, `130`…) mais tu n'as pas
    besoin de les mémoriser pour l'instant.

## Cheat sheet

| Commande / Syntaxe | Effet |
| --- | --- |
| `#!/bin/bash` | Shebang — toujours en 1ʳᵉ ligne |
| `echo $SHELL` | Vérifie quel shell est actif |
| `nano script.sh` | Crée ou ouvre un script |
| `chmod +x script.sh` | Rend le script exécutable |
| `./script.sh` | Lance le script depuis le dossier actuel |
| `bash script.sh` | Lance le script sans `chmod +x` |
| `exit 0` | Termine le script (succès) |
| `# commentaire` | Ligne ignorée par Bash |

!!! warning "Erreurs classiques"
    - Oublier le shebang → le script peut mal se comporter avec `./script.sh`.
    - Oublier `chmod +x` → message `Permission denied` au lancement.
    - Oublier le `./` → message `command not found`.

## À toi de jouer

!!! question "Exercice guidé"
    Crée un script `salut.sh` qui affiche deux lignes : « Bonjour ! » puis
    « Bienvenue dans le monde du scripting. »

!!! question "Exercice autonome"
    Crée un script `info.sh` qui affiche, sur des lignes séparées, ton nom
    d'utilisateur (`whoami`), la date (`date`) et le dossier actuel (`pwd`).

## Pour aller plus loin

- Fiche précédente : [Naviguer dans le terminal](naviguer-terminal.md).
- Fiche suivante : [Variables, affichage et saisie](variables.md).
