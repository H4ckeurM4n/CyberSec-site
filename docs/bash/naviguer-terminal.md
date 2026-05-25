---
tags:
  - terminal
  - débutant
---

# Naviguer dans le terminal

Avant tout le reste, il faut savoir se repérer et se déplacer dans
l'arborescence des fichiers. Trois commandes suffisent pour démarrer.

## Savoir où l'on est

La commande `pwd` (*print working directory*) affiche le dossier courant.

```bash
pwd
# /home/utilisateur/projets
```

## Lister le contenu d'un dossier

`ls` affiche les fichiers et dossiers. Quelques options utiles :

```bash
ls          # liste simple
ls -l       # format détaillé (permissions, taille, date)
ls -la      # idem + fichiers cachés (commençant par un point)
ls -lh      # tailles lisibles (Ko, Mo…)
```

!!! tip "Astuce"
    `ls -la` est probablement la commande que tu taperas le plus souvent. Les
    fichiers cachés comme `.bashrc` ou `.ssh` n'apparaissent qu'avec `-a`.

## Se déplacer

`cd` (*change directory*) change de dossier.

```bash
cd /etc           # chemin absolu
cd projets        # chemin relatif (depuis le dossier courant)
cd ..             # remonter d'un niveau
cd ~              # revenir à son dossier personnel
cd -              # revenir au dossier précédent
```

## Selon ton système

=== "Linux"

    Le terminal est natif. Ouvre-le avec ++ctrl+alt+t++ sur la plupart des
    distributions.

=== "macOS"

    Ouvre l'app **Terminal** (ou installe [iTerm2](https://iterm2.com)). Les
    commandes ci-dessus fonctionnent à l'identique.

=== "Windows"

    Installe **WSL** (Windows Subsystem for Linux) pour avoir un vrai
    environnement Linux, bien plus propre qu'une installation native.

## Récapitulatif

- [x] `pwd` — où suis-je ?
- [x] `ls` — qu'y a-t-il ici ?
- [x] `cd` — comment me déplacer ?

## Pour aller plus loin

- Prochaine fiche : gérer les fichiers et dossiers (à venir).
- Retour au pilier [Bash](index.md).
