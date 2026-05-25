---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Manipuler les chaînes de caractères

Bash sait manipuler du texte directement, sans forcément utiliser des outils externes comme `cut`, `sed`, `awk` ou `tr`.

C’est très utile pour :

- vérifier la longueur d’une saisie ;
- construire un nom de fichier ;
- normaliser une réponse utilisateur ;
- nettoyer un numéro de téléphone ;
- remplacer des caractères ;
- extraire une extension ;
- découper une date ;
- préparer des valeurs pour un script.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- calculer la longueur d’une chaîne ;
- concaténer plusieurs variables ;
- extraire une sous-chaîne ;
- remplacer une occurrence ou toutes les occurrences ;
- supprimer des caractères ;
- transformer en majuscules ou minuscules ;
- tester si une chaîne est vide ou non ;
- extraire une extension de fichier ;
- retirer une extension ;
- nettoyer un nom de fichier ;
- normaliser une saisie utilisateur.

---

## À retenir en 30 secondes

Longueur :

```bash
mot="Bonjour"
echo "${#mot}"
```

Concaténation :

```bash
debut="Bon"
fin="jour"
mot="$debut$fin"
```

Sous-chaîne :

```bash
phrase="Bash est génial"
echo "${phrase:0:4}"
```

Remplacement :

```bash
tel="01-23-45-67-89"
echo "${tel//-/.}"
```

Suppression :

```bash
echo "${tel//-/}"
```

Majuscules / minuscules :

```bash
nom="alice"
echo "${nom^^}"
echo "${nom,,}"
echo "${nom^}"
```

Extension :

```bash
fichier="archive.tar.gz"
echo "${fichier##*.}"
echo "${fichier%.*}"
```

---

## Pourquoi manipuler des chaînes en Bash ?

Dans un script, beaucoup de données sont du texte :

- noms de fichiers ;
- chemins ;
- arguments ;
- réponses utilisateur ;
- noms d’hôtes ;
- adresses IP ;
- extensions ;
- résultats de commandes ;
- lignes de logs.

Savoir manipuler les chaînes permet d’éviter de tout faire manuellement.

Exemples d’usages :

```bash
rapport_2026-03-30.txt
```

```bash
rapport_final.txt
```

```bash
01.23.45.67.89
```

```bash
ALICE
```

---

## Longueur d’une chaîne : `${#var}`

Pour connaître la longueur d’une variable :

```bash
mot="Bonjour"

echo "${#mot}"
```

Résultat :

```txt
7
```

`${#mot}` renvoie le nombre de caractères contenus dans la variable `mot`.

---

## Exemple : vérifier un mot de passe trop court

```bash
mdp="abc123"

if [[ "${#mdp}" -lt 8 ]]; then
    echo "Mot de passe trop court"
fi
```

Résultat :

```txt
Mot de passe trop court
```

Ce type de test peut servir à :

- vérifier une saisie utilisateur ;
- imposer une longueur minimale ;
- refuser un identifiant trop court ;
- détecter une valeur anormalement longue.

!!! warning "Attention"
    Ce test ne suffit pas à faire une vraie politique de mot de passe robuste.  
    Il vérifie seulement la longueur.

---

## Exemple : vérifier la longueur d’un pseudo

```bash
read -r -p "Pseudo : " pseudo

if [[ "${#pseudo}" -lt 3 ]]; then
    echo "Pseudo trop court"
elif [[ "${#pseudo}" -gt 20 ]]; then
    echo "Pseudo trop long"
else
    echo "Pseudo accepté : $pseudo"
fi
```

---

## Concaténer des chaînes

Concaténer signifie coller plusieurs valeurs ensemble.

```bash
debut="Bon"
fin="jour"

mot="$debut$fin"

echo "$mot"
```

Résultat :

```txt
Bonjour
```

On peut aussi écrire directement :

```bash
echo "$debut$fin"
```

---

## Concaténer avec du texte autour

```bash
prenom="Alice"
message="Bonjour $prenom"

echo "$message"
```

Résultat :

```txt
Bonjour Alice
```

Autre exemple :

```bash
dossier="/home/kali"
fichier="notes.txt"

chemin="$dossier/$fichier"

echo "$chemin"
```

Résultat :

```txt
/home/kali/notes.txt
```

---

## Créer un nom de fichier dynamique

Exemple : créer un nom de rapport avec une date.

```bash
nom="rapport"
date_jour="2026-03-30"

fichier="${nom}_${date_jour}.txt"

echo "$fichier"
```

Résultat :

```txt
rapport_2026-03-30.txt
```

!!! tip "Pourquoi utiliser les accolades ?"
    Les accolades permettent de bien séparer le nom de la variable du texte collé autour.

    ```bash
    fichier="${nom}_${date_jour}.txt"
    ```

---

## Extraire une sous-chaîne : `${var:position:longueur}`

Syntaxe :

```bash
${variable:position:longueur}
```

Les positions commencent à `0`.

```bash
phrase="Bash est génial"

echo "${phrase:0:4}"
echo "${phrase:5:3}"
echo "${phrase:9}"
```

Résultat :

```txt
Bash
est
génial
```

Explication :

| Syntaxe | Effet |
|---|---|
| `${phrase:0:4}` | 4 caractères depuis la position 0 |
| `${phrase:5:3}` | 3 caractères depuis la position 5 |
| `${phrase:9}` | tout depuis la position 9 |

---

## Exemple : extraire un préfixe

```bash
id="ABCDEF1234"

echo "${id:0:4}"
```

Résultat :

```txt
ABCD
```

Utile pour récupérer :

- un préfixe ;
- un code court ;
- le début d’un identifiant ;
- une partie fixe d’une chaîne.

---

## Exemple : découper une date

```bash
date_du_jour="2026-03-30"

annee="${date_du_jour:0:4}"
mois="${date_du_jour:5:2}"
jour="${date_du_jour:8:2}"

echo "$jour/$mois/$annee"
```

Résultat :

```txt
30/03/2026
```

Explication :

```txt
2026-03-30
0123456789
```

| Partie | Extraction |
|---|---|
| année | `${date_du_jour:0:4}` |
| mois | `${date_du_jour:5:2}` |
| jour | `${date_du_jour:8:2}` |

---

## Remplacer dans une chaîne

Syntaxe pour remplacer la première occurrence :

```bash
${variable/ancien/nouveau}
```

Exemple :

```bash
phrase="J'adore les pommes"

echo "${phrase/pommes/bananes}"
```

Résultat :

```txt
J'adore les bananes
```

---

## Remplacer toutes les occurrences

Pour remplacer toutes les occurrences, on utilise un double slash :

```bash
${variable//ancien/nouveau}
```

Exemple :

```bash
tel="01-23-45-67-89"

echo "${tel//-/.}"
```

Résultat :

```txt
01.23.45.67.89
```

Différence :

| Syntaxe | Effet |
|---|---|
| `${var/ancien/nouveau}` | remplace la première occurrence |
| `${var//ancien/nouveau}` | remplace toutes les occurrences |

---

## Exemple : transformer une extension

```bash
fichier="rapport.txt"

nouveau="${fichier/.txt/.pdf}"

echo "$nouveau"
```

Résultat :

```txt
rapport.pdf
```

---

## Exemple : adapter un chemin

```bash
chemin="/home/kali/Documents"

nouveau="${chemin/Documents/Desktop}"

echo "$nouveau"
```

Résultat :

```txt
/home/kali/Desktop
```

---

## Supprimer dans une chaîne

Pour supprimer une partie d’une chaîne, on remplace par rien.

```bash
${variable//ancien/}
```

Exemple :

```bash
tel="01-23-45-67-89"

echo "${tel//-/}"
```

Résultat :

```txt
0123456789
```

Ici, tous les tirets sont supprimés.

---

## Exemple : nettoyer un nom de fichier

```bash
nom_fichier="rapport final.txt"

echo "${nom_fichier// /_}"
```

Résultat :

```txt
rapport_final.txt
```

On remplace tous les espaces par des underscores `_`.

---

## Exemple : nettoyer un port

```bash
port=":8080"

echo "${port//:/}"
```

Résultat :

```txt
8080
```

Utile quand une valeur contient un séparateur qu’on ne veut pas garder.

---

## Majuscules et minuscules

Bash permet de changer la casse d’une chaîne.

```bash
nom="alice dupont"
NOM="ALICE DUPONT"

echo "${nom^^}"
echo "${NOM,,}"
echo "${nom^}"
```

Résultat :

```txt
ALICE DUPONT
alice dupont
Alice dupont
```

| Syntaxe | Effet |
|---|---|
| `${var^^}` | tout en majuscules |
| `${var,,}` | tout en minuscules |
| `${var^}` | première lettre en majuscule |
| `${var,}` | première lettre en minuscule |

---

## Exemple : uniformiser une réponse utilisateur

```bash
read -r -p "Continuer ? (oui/non) " reponse

if [[ "${reponse,,}" == "oui" ]]; then
    echo "C'est parti"
else
    echo "Arrêt"
fi
```

Ici :

- `OUI`
- `Oui`
- `oui`
- `oUi`

seront tous transformés en `oui`.

---

## Exemple : mettre un prénom en forme

```bash
prenom="camille"

echo "${prenom^}"
```

Résultat :

```txt
Camille
```

---

## Tester si une chaîne est vide

Pour tester si une chaîne est vide :

```bash
nom=""

if [[ -z "$nom" ]]; then
    echo "Nom est vide"
fi
```

Résultat :

```txt
Nom est vide
```

Pour tester si une chaîne n’est pas vide :

```bash
autre="Alice"

if [[ -n "$autre" ]]; then
    echo "Autre n'est pas vide"
fi
```

Résultat :

```txt
Autre n'est pas vide
```

| Test | Signification |
|---|---|
| `-z "$var"` | la chaîne est vide |
| `-n "$var"` | la chaîne n’est pas vide |

---

## Extraire une extension de fichier

Pour extraire ce qui se trouve après le dernier point :

```bash
fichier="archive.tar.gz"

echo "${fichier##*.}"
```

Résultat :

```txt
gz
```

Explication :

- `##` supprime le plus long motif depuis le début ;
- `*.` signifie “tout jusqu’au dernier point” ;
- il reste donc l’extension finale.

---

## Retirer une extension

Pour retirer ce qui se trouve après le dernier point :

```bash
fichier="archive.tar.gz"

echo "${fichier%.*}"
```

Résultat :

```txt
archive.tar
```

Explication :

- `%` supprime le plus petit motif depuis la fin ;
- `.*` signifie “point puis extension finale” ;
- il reste le nom sans la dernière extension.

---

## Différence entre `#`, `##`, `%` et `%%`

Ces opérateurs permettent de supprimer des motifs au début ou à la fin d’une chaîne.

| Syntaxe | Effet |
|---|---|
| `${var#motif}` | supprime le plus court motif depuis le début |
| `${var##motif}` | supprime le plus long motif depuis le début |
| `${var%motif}` | supprime le plus court motif depuis la fin |
| `${var%%motif}` | supprime le plus long motif depuis la fin |

Exemple :

```bash
fichier="archive.tar.gz"

echo "${fichier#*.}"
echo "${fichier##*.}"
echo "${fichier%.*}"
echo "${fichier%%.*}"
```

Résultat :

```txt
tar.gz
gz
archive.tar
archive
```

---

## Exemple : récupérer nom et extension

```bash
fichier="rapport.txt"

nom_sans_extension="${fichier%.*}"
extension="${fichier##*.}"

echo "Nom : $nom_sans_extension"
echo "Extension : $extension"
```

Résultat :

```txt
Nom : rapport
Extension : txt
```

---

## Exemple : normaliser un nom de rapport

```bash
nom="mon rapport final.txt"

nom_normalise="${nom// /_}"

echo "$nom_normalise"
```

Résultat :

```txt
mon_rapport_final.txt
```

On peut aussi mettre en minuscules :

```bash
nom="Mon Rapport Final.txt"

nom_normalise="${nom,,}"
nom_normalise="${nom_normalise// /_}"

echo "$nom_normalise"
```

Résultat :

```txt
mon_rapport_final.txt
```

---

## Exemple complet : générer un nom de fichier propre

```bash
#!/bin/bash

read -r -p "Nom du rapport : " nom

if [[ -z "$nom" ]]; then
    echo "Erreur : nom vide"
    exit 1
fi

date_jour=$(date +%F)

nom="${nom,,}"
nom="${nom// /_}"

fichier="${nom}_${date_jour}.txt"

echo "Fichier généré : $fichier"
```

Exemple :

```txt
Nom du rapport : Rapport Final
Fichier généré : rapport_final_2026-03-30.txt
```

---

## Exemple complet : vérifier une saisie utilisateur

```bash
#!/bin/bash

read -r -p "Pseudo : " pseudo

if [[ -z "$pseudo" ]]; then
    echo "Erreur : pseudo vide"
    exit 1
fi

if [[ "${#pseudo}" -lt 3 ]]; then
    echo "Erreur : pseudo trop court"
    exit 1
fi

pseudo="${pseudo,,}"
pseudo="${pseudo// /_}"

echo "Pseudo normalisé : $pseudo"
```

Exemples :

```txt
Pseudo : Cam
Pseudo normalisé : cam
```

```txt
Pseudo : Cam H
Pseudo normalisé : cam_h
```

---

## Mini-lab 1 : longueur d’une chaîne

Objectif : créer un script `check_mdp.sh`.

Contraintes :

1. demander un mot de passe avec `read -s` ;
2. vérifier qu’il contient au moins 8 caractères ;
3. afficher `Mot de passe accepté` ou `Mot de passe trop court`.

---

### Correction possible

```bash
#!/bin/bash

read -r -s -p "Mot de passe : " mdp
echo

if [[ "${#mdp}" -lt 8 ]]; then
    echo "Mot de passe trop court"
else
    echo "Mot de passe accepté"
fi
```

---

## Mini-lab 2 : nettoyer un numéro de téléphone

Objectif : nettoyer un numéro contenant des tirets.

Entrée :

```txt
01-23-45-67-89
```

Sortie attendue :

```txt
0123456789
```

---

### Correction possible

```bash
#!/bin/bash

tel="01-23-45-67-89"

tel_clean="${tel//-/}"

echo "$tel_clean"
```

---

## Mini-lab 3 : transformer un nom de fichier

Objectif : transformer un nom de fichier avec espaces en nom compatible script.

Entrée :

```txt
rapport final.txt
```

Sortie attendue :

```txt
rapport_final.txt
```

---

### Correction possible

```bash
#!/bin/bash

nom_fichier="rapport final.txt"

nom_clean="${nom_fichier// /_}"

echo "$nom_clean"
```

---

## Mini-lab 4 : extraire une date

Objectif : découper une date au format `YYYY-MM-DD`.

Entrée :

```txt
2026-03-30
```

Sortie attendue :

```txt
30/03/2026
```

---

### Correction possible

```bash
#!/bin/bash

date_du_jour="2026-03-30"

annee="${date_du_jour:0:4}"
mois="${date_du_jour:5:2}"
jour="${date_du_jour:8:2}"

echo "$jour/$mois/$annee"
```

---

## Mini-lab 5 : extraire extension et nom

Objectif : depuis le fichier suivant :

```txt
archive.tar.gz
```

afficher :

```txt
Extension : gz
Nom sans dernière extension : archive.tar
Nom principal : archive
```

---

### Correction possible

```bash
#!/bin/bash

fichier="archive.tar.gz"

extension="${fichier##*.}"
nom_sans_derniere_extension="${fichier%.*}"
nom_principal="${fichier%%.*}"

echo "Extension : $extension"
echo "Nom sans dernière extension : $nom_sans_derniere_extension"
echo "Nom principal : $nom_principal"
```

---

## Erreurs classiques

### Mauvaise affectation de variable

Incorrect :

```bash
mot:="Bonjour"
```

Correct :

```bash
mot="Bonjour"
```

---

### Oublier les guillemets

À éviter :

```bash
echo ${nom_fichier// /_}
```

Préférer :

```bash
echo "${nom_fichier// /_}"
```

---

### Oublier les espaces dans les conditions

Incorrect :

```bash
if [[${#mdp}-lt8 ]];then
    echo "Trop court"
fi
```

Correct :

```bash
if [[ "${#mdp}" -lt 8 ]]; then
    echo "Trop court"
fi
```

---

### Coller `echo` et la chaîne

Incorrect :

```bash
echo"$nouveau"
```

Correct :

```bash
echo "$nouveau"
```

---

### Confondre première occurrence et toutes les occurrences

```bash
texte="a-b-c-d"

echo "${texte/-/_}"
echo "${texte//-/_}"
```

Résultat :

```txt
a_b-c-d
a_b_c_d
```

Le premier remplace seulement le premier `-`.  
Le second remplace tous les `-`.

---

## Cheat sheet

| Syntaxe | Effet |
|---|---|
| `${#var}` | longueur de la chaîne |
| `${var:pos:len}` | extrait une sous-chaîne |
| `${var:pos}` | extrait depuis une position jusqu’à la fin |
| `${var/ancien/nouveau}` | remplace la première occurrence |
| `${var//ancien/nouveau}` | remplace toutes les occurrences |
| `${var//ancien/}` | supprime toutes les occurrences |
| `${var^^}` | met tout en majuscules |
| `${var,,}` | met tout en minuscules |
| `${var^}` | met la première lettre en majuscule |
| `${var,}` | met la première lettre en minuscule |
| `${var#motif}` | supprime le plus court motif depuis le début |
| `${var##motif}` | supprime le plus long motif depuis le début |
| `${var%motif}` | supprime le plus court motif depuis la fin |
| `${var%%motif}` | supprime le plus long motif depuis la fin |
| `${fichier##*.}` | récupère l’extension finale |
| `${fichier%.*}` | retire la dernière extension |
| `[[ -z "$var" ]]` | teste si la chaîne est vide |
| `[[ -n "$var" ]]` | teste si la chaîne n’est pas vide |

---

## Checklist script propre

Avant de considérer tes manipulations de chaînes comme propres, vérifie :

- [ ] les variables sont entourées de guillemets ;
- [ ] les variables collées à du texte utilisent des accolades si besoin ;
- [ ] les conditions ont bien des espaces autour des opérateurs ;
- [ ] tu sais si tu veux remplacer une seule occurrence ou toutes ;
- [ ] les noms de fichiers générés ne contiennent pas d’espaces non voulus ;
- [ ] les entrées utilisateur sont normalisées si nécessaire ;
- [ ] les chaînes vides sont gérées ;
- [ ] les extensions sont extraites avec la bonne syntaxe.

---

## Résumé

Pour connaître la longueur :

```bash
echo "${#var}"
```

Pour extraire une partie :

```bash
echo "${var:0:4}"
```

Pour remplacer :

```bash
echo "${var/ancien/nouveau}"
```

Pour remplacer partout :

```bash
echo "${var//ancien/nouveau}"
```

Pour supprimer :

```bash
echo "${var//ancien/}"
```

Pour changer la casse :

```bash
echo "${var^^}"
echo "${var,,}"
```

Pour extraire une extension :

```bash
echo "${fichier##*.}"
```

Le réflexe important :

> Bash peut déjà manipuler beaucoup de texte sans outil externe.  
> Pour des traitements simples, commence par les expansions `${...}`.

---

## Pour aller plus loin

- Fiche précédente : [Fonctions et case](fonctions.md).
- Fiche suivante : [Les tableaux](tableaux.md).