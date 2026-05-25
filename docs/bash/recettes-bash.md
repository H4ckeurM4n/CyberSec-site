---
tags:
  - bash
  - scripting
  - recettes
  - cas-pratique
---

# Recettes Bash utiles

Cette fiche regroupe des petits blocs de code Bash prêts à réutiliser.

Contrairement aux fiches précédentes, elle n’est pas pensée comme un cours linéaire.  
C’est une **boîte à outils** : tu viens chercher une recette quand tu as besoin de faire rapidement une action précise.

Tu y retrouveras des exemples pour :

- écrire proprement dans un fichier ;
- vérifier des arguments ;
- créer des menus ;
- manipuler les dates ;
- construire des noms d’archives ;
- faire une pause utilisateur ;
- encoder une valeur plusieurs fois ;
- tester le contenu d’une variable ;
- sauvegarder une sortie avec `tee` ;
- chercher un fichier ;
- comparer des fichiers ;
- lancer un petit scan autorisé ;
- traiter des arguments un par un.

!!! note
    Certaines recettes reprennent des notions déjà vues dans les fiches précédentes.  
    L’objectif ici est de les avoir sous forme de blocs courts et réutilisables.

---

## Écrire plusieurs lignes dans un fichier

Pour écrire plusieurs lignes dans un fichier sans répéter `>> fichier.txt` à chaque ligne, on peut rediriger un bloc entier.

```bash
{
    echo "Date : $(date)"
    echo "Dossier : $1"
    echo "Nombre de fichiers dans le dossier : $nbr_fichiers"
} > rapport.txt

cat rapport.txt
```

Le bloc entre accolades produit plusieurs lignes, puis la redirection `> rapport.txt` écrit tout dans le fichier.

!!! warning
    `>` écrase le fichier existant.  
    Pour ajouter à la fin, utilise `>>`.

---

## Générer un rapport simple sur un dossier

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : veuillez renseigner un dossier en argument" >&2
    echo -e "Usage :\n\t$0 <dossier>"
    exit 1
fi

if [[ ! -d "$1" ]]; then
    echo "Erreur : $1 n'est pas un dossier valide" >&2
    exit 1
fi

nbr_fichiers=$(ls "$1" | wc -l)

{
    echo "Date : $(date)"
    echo "Dossier : $1"
    echo "Nombre de fichiers dans le dossier : $nbr_fichiers"
} > rapport.txt

cat rapport.txt
```

Ce script vérifie :

- qu’un argument est fourni ;
- que cet argument est un dossier ;
- le nombre d’éléments dans le dossier ;
- l’écriture propre du rapport dans `rapport.txt`.

---

## Vérifier que des arguments sont des entiers

Pour vérifier qu’un argument est un nombre entier, on peut utiliser une expression régulière.

```bash
if [[ ! "$1" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : un nombre entier est attendu"
    exit 1
fi
```

Le motif :

```regex
^-?[0-9]+$
```

signifie :

| Élément | Signification |
|---|---|
| `^` | début de la chaîne |
| `-?` | signe moins optionnel |
| `[0-9]+` | un ou plusieurs chiffres |
| `$` | fin de la chaîne |

Valeurs acceptées :

```txt
2
45
-8
```

Valeurs refusées :

```txt
abc
4a
12.5
```

---

## Vérifier deux nombres puis calculer

```bash
#!/bin/bash

if [[ $# -ne 2 ]]; then
    echo "Erreur : il faut deux arguments"
    echo -e "Usage :\n\t$0 <nombre1> <nombre2>"
    exit 1
fi

if [[ ! "$1" =~ ^-?[0-9]+$ || ! "$2" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : les deux arguments doivent être des nombres entiers"
    exit 1
fi

echo "La somme de vos deux nombres est : $(($1 + $2))"
echo "La différence de vos deux nombres est : $(($1 - $2))"
echo "Le produit de vos deux nombres est : $(($1 * $2))"
```

Exemple :

```bash
./calcul.sh 10 3
```

Résultat :

```txt
La somme de vos deux nombres est : 13
La différence de vos deux nombres est : 7
Le produit de vos deux nombres est : 30
```

---

## Menu simple avec `case`

Ce modèle permet de créer un script qui réagit à une option passée en argument.

```bash
#!/bin/bash

afficher_aide() {
    echo "Utilisation : $0 [option]"
    echo "  -l    Lister les fichiers"
    echo "  -d    Afficher la date"
    echo "  -h    Afficher cette aide"
}

case "$1" in
    -l)
        ls -la
        ;;
    -d)
        date
        ;;
    -h|--help)
        afficher_aide
        ;;
    "")
        echo "Erreur : aucune option fournie." >&2
        afficher_aide
        exit 1
        ;;
    *)
        echo "Option '$1' inconnue." >&2
        afficher_aide
        exit 1
        ;;
esac
```

Exemples :

```bash
./outil.sh -l
./outil.sh -d
./outil.sh -h
```

---

## Menu avec options courtes

```bash
#!/bin/bash

afficher_aide() {
    echo "Options disponibles :"
    echo "  -h  aide"
    echo "  -d  date"
    echo "  -u  utilisateur"
}

case "$1" in
    -h)
        afficher_aide
        ;;
    -d)
        date
        ;;
    -u)
        whoami
        ;;
    *)
        echo "Option inconnue"
        afficher_aide
        ;;
esac
```

---

## Menu interactif avec `while true`

Ce modèle garde le menu actif tant que l’utilisateur ne choisit pas de quitter.

```bash
#!/bin/bash

pause() {
    read -r -p "Appuyez sur [Entrée] pour continuer..." key
}

liste_all_txt() {
    local list

    list=$(find . -type f -name "*.txt" 2>/dev/null)

    if [[ -z "$list" ]]; then
        echo "Il n'y a aucun fichier .txt dans le répertoire actuel ni dans les sous-dossiers"
    else
        echo -e "Voici la liste des fichiers .txt :\n$list"
    fi

    pause
}

compter_nbr_lignes() {
    local file
    local file_lignes

    read -r -p "Quel fichier voulez-vous analyser ? " file

    if [[ -z "$file" ]]; then
        echo "Veuillez renseigner un fichier"
    elif [[ ! -f "$file" ]]; then
        echo "Erreur : ce fichier n'existe pas"
    else
        file_lignes=$(wc -l < "$file")
        echo "Nombre de ligne(s) dans $file : $file_lignes"
    fi

    pause
}

while true; do
    clear

    echo "---------------------------------"
    echo "        M A I N - M E N U"
    echo "---------------------------------"
    echo "1. Lister tous les fichiers .txt dans le répertoire courant et en-dessous"
    echo "2. Compter le nombre de lignes dans un fichier donné"
    echo "3. Quitter"
    echo "---------------------------------"

    read -r -p "Quel est votre choix ? " choix

    case "$choix" in
        -l|lister|1)
            liste_all_txt
            ;;
        -n|number|2)
            compter_nbr_lignes
            ;;
        -q|quit|3)
            echo "Au revoir."
            break
            ;;
        *)
            echo "Sélectionnez un choix valide."
            pause
            ;;
    esac
done
```

---

## Fonction `pause`

Dans un menu interactif, `clear` efface l’écran à chaque tour.  
Pour laisser le temps de lire le résultat, utilise une fonction `pause`.

```bash
pause() {
    read -r -p "Appuyez sur [Entrée] pour continuer..." key
}
```

Utilisation :

```bash
echo "Résultat affiché"
pause
```

---

## Menu avec `select`

`select` génère automatiquement un menu numéroté.

```bash
#!/bin/bash

liste_all_txt() {
    local list

    list=$(find . -type f -name "*.txt" 2>/dev/null)

    if [[ -z "$list" ]]; then
        echo "Il n'y a aucun fichier .txt dans le répertoire actuel ni dans les sous-dossiers"
    else
        echo -e "Voici la liste des fichiers .txt :\n$list"
    fi
}

compter_nbr_lignes() {
    local file
    local file_lignes

    read -r -p "Quel fichier voulez-vous analyser ? " file

    if [[ -z "$file" ]]; then
        echo "Veuillez renseigner un fichier"
    elif [[ ! -f "$file" ]]; then
        echo "Erreur : ce fichier n'existe pas"
    else
        file_lignes=$(wc -l < "$file")
        echo "Nombre de ligne(s) dans $file : $file_lignes"
    fi
}

echo "Que veux-tu faire ?"

select choix in "Lister les fichiers .txt" "Compter les lignes d'un fichier" "Quitter"; do
    case "$choix" in
        "Lister les fichiers .txt")
            liste_all_txt
            ;;
        "Compter les lignes d'un fichier")
            compter_nbr_lignes
            ;;
        "Quitter")
            echo "Au revoir."
            break
            ;;
        *)
            echo "Choix invalide"
            ;;
    esac
done
```

---

## Menu avec tableaux : nom → email

Ce modèle utilise deux tableaux liés par index.

```bash
#!/bin/bash

noms=("Alice" "Bob" "Charlie")
emails=("alice@mail.com" "bob@mail.com" "charlie@mail.com")

pause() {
    read -r -p "Appuyez sur [Entrée] pour continuer..." key
}

associer() {
    clear

    for i in "${!noms[@]}"; do
        echo "Nom : ${noms[$i]} -- Email : ${emails[$i]}"
    done

    pause
}

while true; do
    clear

    echo "---------------------------------"
    echo "        Find the mail"
    echo "---------------------------------"
    echo "1. Alice"
    echo "2. Bob"
    echo "3. Charlie"
    echo "4. Lister l'ensemble des agents"
    echo "5. Quitter script"
    echo "---------------------------------"

    read -r -p "L'adresse mail de quel agent souhaitez-vous ? " choix

    case "$choix" in
        1|2|3)
            index=$((choix - 1))
            echo "Nom : ${noms[$index]} -- Email : ${emails[$index]}"
            pause
            ;;
        4)
            associer
            ;;
        5)
            echo "Vous quittez le script"
            break
            ;;
        "")
            echo "Veuillez indiquer la personne souhaitée"
            pause
            ;;
        *)
            echo "Sélectionnez votre choix"
            pause
            ;;
    esac
done
```

---

## Modifier le format d’une date

Date compacte :

```bash
date +%Y%m%d
```

Résultat possible :

```txt
20260525
```

Date lisible avec heure :

```bash
date_heure=$(date '+%Y-%m-%d %H:%M:%S')
echo "$date_heure"
```

Résultat possible :

```txt
2026-05-25 21:30:00
```

Timestamp pour nom de fichier :

```bash
timestamp=$(date +%Y%m%d-%H%M%S)
echo "$timestamp"
```

Résultat possible :

```txt
20260525-213000
```

---

## Définir une destination de sortie

Tu peux centraliser un chemin de destination dans une variable.

```bash
destination="$HOME/nom_dossier"
```

Ou :

```bash
destination="/tmp/backups"
```

Créer le dossier si nécessaire :

```bash
mkdir -p "$destination"
```

---

## Construire un nom d’archive

Exemple simple :

```bash
dossier="$1"
destination="/tmp/backups"
date_du_jour=$(date +%Y%m%d)

mkdir -p "$destination"

archive="$destination/${date_du_jour}_${dossier}.tar.gz"

tar -czf "$archive" "$dossier"
```

Attention : si `dossier` contient des `/`, le nom d’archive peut être gênant.

---

## Construire un nom d’archive avec `basename`

Version plus propre :

```bash
dossier="$1"
destination="/tmp/backups"
date_du_jour=$(date +%Y%m%d)

mkdir -p "$destination"

nom_dossier=$(basename "$dossier")
archive="$destination/${date_du_jour}_${nom_dossier}.tar.gz"

tar -czf "$archive" "$dossier"

echo "Archive créée : $archive"
```

Exemple :

```bash
./backup.sh /home/kali/Documents
```

Archive possible :

```txt
/tmp/backups/20260525_Documents.tar.gz
```

---

## Construire une archive avec timestamp

```bash
destination="/tmp/backups"
dossier="$1"

mkdir -p "$destination"

archive="$destination/$(date +%Y%m%d-%H%M%S).tar.gz"

tar -czf "$archive" "$dossier"

echo "Archive créée : $archive"
```

Ce format est utile si tu peux lancer plusieurs sauvegardes dans la même journée.

---

## Remplacer `/` par `_` dans un nom d’archive

Si tu veux transformer un chemin complet en nom de fichier :

```bash
dossier="/home/kali/Documents"

nom_archive=$(echo "$dossier" | tr '/' '_')
fichier="/tmp/backups/${nom_archive}-$(date +%Y-%m-%d).tar.gz"

echo "$fichier"
```

Résultat possible :

```txt
/tmp/backups/_home_kali_Documents-2026-05-25.tar.gz
```

---

## Sauvegarder la sortie avec `tee`

`tee` permet d’afficher une sortie à l’écran et de l’écrire dans un fichier.

```bash
commande | tee fichier.txt
```

Exemple :

```bash
ls -la | tee listing.txt
```

Pour ajouter sans écraser :

```bash
commande | tee -a fichier.txt
```

Exemple :

```bash
date | tee -a journal.log
```

!!! warning "Syntaxe"
    C’est `tee -a`, avec un espace.  
    Pas `tee-a`.

---

## Récupérer une sortie tout en la sauvegardant

```bash
hosts=$(host "$domain" | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)
```

Ici :

- la commande récupère les IP ;
- `tee` les écrit dans `discovered_hosts.txt` ;
- la variable `hosts` récupère aussi le résultat.

Autre exemple :

```bash
netrange=$(whois "$ip" | grep "NetRange\|CIDR" | tee -a CIDR.txt)
```

---

## Encoder une valeur plusieurs fois en base64

Pour encoder plusieurs fois une valeur, il faut réutiliser la même variable à chaque tour.

```bash
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}; do
    var=$(echo "$var" | base64)
done

echo "$var"
```

!!! note
    À chaque tour, `var` est remplacée par sa nouvelle version encodée.

---

## Retrouver la valeur à un tour précis

Exemple : récupérer la valeur au tour 35.

```bash
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}; do
    var=$(echo "$var" | base64)

    if [[ "$counter" -eq 35 ]]; then
        resultat_35="$var"
    fi
done

echo "$resultat_35"
```

---

## Retrouver la longueur à un tour précis

```bash
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}; do
    var=$(echo "$var" | base64)

    if [[ "$counter" -eq 35 ]]; then
        longueur=$(printf "%s" "$var" | wc -m)
    fi
done

echo "$longueur"
```

---

## Calculer la longueur d’une variable

Méthode Bash directe :

```bash
htb="HackTheBox"

echo "${#htb}"
```

Résultat :

```txt
10
```

Méthode avec `wc -m` :

```bash
var="Bonjour tout le monde"

longueur=$(printf "%s" "$var" | wc -m)

echo "$longueur"
```

!!! tip
    Pour une variable simple, préfère `${#var}`.  
    Pour une sortie de commande ou un flux, `wc -m` peut être utile.

---

## Vérifier si une variable dépasse une longueur donnée

```bash
if [[ "${#var}" -gt 113450 ]]; then
    echo "La variable dépasse 113450 caractères"
fi
```

---

## Stocker une longueur dans une autre variable

```bash
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..28}; do
    var=$(echo "$var" | base64)

    if [[ "$counter" -eq 28 ]]; then
        longueur=$(printf "%s" "$var" | wc -m)
    fi
done

salt="$longueur"

echo "$salt"
```

---

## Tester si une chaîne contient une autre chaîne

```bash
var="8dm7KsjU28B7v621Jls"
value="Ksj"

if [[ "$var" == *"$value"* ]]; then
    echo "La variable var contient la valeur de value"
fi
```

Le motif :

```bash
*"$value"*
```

signifie : “n’importe quoi avant, puis la valeur, puis n’importe quoi après”.

---

## Récupérer les 20 derniers caractères d’une variable

```bash
var="Ceci est une chaîne assez longue pour faire un test."

last_20=$(printf "%s" "$var" | tail -c 20)

echo "$last_20"
```

---

## Traiter les arguments un par un avec `shift`

```bash
#!/bin/bash

while [[ $# -gt 0 ]]; do
    echo "Je traite : $1"
    shift
done
```

Exécution :

```bash
./script.sh Alice Bob Charlie
```

Résultat :

```txt
Je traite : Alice
Je traite : Bob
Je traite : Charlie
```

Variante :

```bash
#!/bin/bash

while [[ -n "${1:-}" ]]; do
    echo "Argument courant : $1"
    shift
done
```

!!! note
    Avec `set -u`, préfère `${1:-}` pour éviter une erreur si `$1` n’existe pas.

---

## Rechercher un fichier par son nom

Version simple :

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Veuillez renseigner le fichier que vous cherchez"
    echo "Usage : $0 <fichier>"
    exit 1
fi

echo "Le fichier $1 est à l'emplacement suivant :"
find / -iname "$1" 2>/dev/null
```

---

## Rechercher un fichier avec gestion d’erreur

Version plus propre :

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : veuillez indiquer le nom d'un fichier" >&2
    echo -e "Usage :\n\t$0 <fichier>"
    exit 1
fi

emplacement=$(find / -iname "$1" 2>/dev/null)

if [[ -z "$emplacement" ]]; then
    echo "Aucun fichier trouvé"
    echo "Le fichier $1 ne semble pas présent sur le système"
    exit 1
fi

echo "L'emplacement de $1 :"
echo "$emplacement"
```

---

## Comparer deux fichiers

Ce script affiche la taille de deux fichiers en octets.

```bash
#!/bin/bash

if [[ "$#" -ne 2 ]]; then
    echo "Erreur : le script attend <fichier1> et <fichier2>" >&2
    echo -e "Usage :\n\t$0 <fichier1> <fichier2>"
    exit 1
fi

if [[ ! -f "$1" ]]; then
    echo "Erreur : $1 n'est pas un fichier valide" >&2
    exit 1
fi

if [[ ! -f "$2" ]]; then
    echo "Erreur : $2 n'est pas un fichier valide" >&2
    exit 1
fi

echo "Comparaison de $1 et $2"

echo "Taille de $1 :"
wc -c < "$1"

echo "Taille de $2 :"
wc -c < "$2"
```

---

## Compter les lignes d’un fichier et sauvegarder le résultat

```bash
#!/bin/bash

lignes_pass=$(wc -l < /etc/passwd)

echo "Il y a $lignes_pass lignes dans le fichier passwd" | tee ex_pass.txt
```

---

## Demander un fichier puis compter ses lignes

```bash
#!/bin/bash

read -r -p "Quel fichier souhaitez-vous analyser ? " fichier

if [[ -z "$fichier" ]]; then
    echo "Erreur : aucun fichier fourni" >&2
    exit 1
fi

if [[ ! -f "$fichier" ]]; then
    echo "Erreur : $fichier n'est pas un fichier valide" >&2
    exit 1
fi

lignes=$(wc -l < "$fichier")

echo "Il y a $lignes lignes dans $fichier" | tee nbr_lignes.txt
```

---

## Expliquer comment utiliser un script en cas d’erreur

Modèle classique :

```bash
#!/bin/bash

if [[ $# -eq 0 ]]; then
    echo -e "You need to specify the target domain.\n" >&2
    echo -e "Usage:" >&2
    echo -e "\t$0 <domain>" >&2
    exit 1
else
    domain="$1"
fi
```

Sortie possible :

```txt
You need to specify the target domain.

Usage:
    ./cidr.sh <domain>
```

---

## Mini-script Nmap autorisé

Version simple :

```bash
#!/bin/bash

read -r -p "L'adresse à scanner : " ip_scan

echo "Scan de $ip_scan en cours..."
nmap -F -sV "$ip_scan"
```

Version un peu plus propre :

```bash
#!/bin/bash

read -r -p "Adresse IP ou hôte à scanner : " target

if [[ -z "$target" ]]; then
    echo "Erreur : aucune cible saisie." >&2
    exit 1
fi

echo "Scan de $target en cours..."
nmap -F -sV "$target"
```

!!! warning "Cadre légal"
    À utiliser uniquement sur tes machines, un lab, un CTF ou un périmètre explicitement autorisé.

---

## Évolutions possibles pour le script Nmap

Idées d’amélioration :

- vérifier que la cible n’est pas vide ;
- proposer plusieurs types de scans ;
- sauvegarder le résultat dans un fichier ;
- horodater le fichier de sortie ;
- ajouter un mode dry-run ;
- afficher une aide avec `-h`.

Exemple de sauvegarde datée :

```bash
timestamp=$(date +%Y%m%d-%H%M%S)
sortie="scan_${timestamp}.txt"

nmap -F -sV "$target" | tee "$sortie"
```

---

## Tableau de domaines

```bash
domains=(www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com www2.inlanefreight.com)

echo "${domains[0]}"
```

Parcourir tous les domaines :

```bash
for domain in "${domains[@]}"; do
    echo "Domaine : $domain"
done
```

---

## Fonction pour identifier une plage réseau

Exemple pédagogique :

```bash
network_range() {
    for ip in $ipaddr; do
        netrange=$(whois "$ip" | grep "NetRange\|CIDR" | tee -a CIDR.txt)
        cidr=$(whois "$ip" | grep "CIDR" | awk '{print $2}')
        cidr_ips=$(prips "$cidr")

        echo -e "\nNetRange for $ip:"
        echo -e "$netrange"
    done
}
```

!!! warning "Cadre légal"
    Ce type de fonction est lié à de la reconnaissance réseau.  
    À utiliser uniquement sur un périmètre autorisé.

---

## Renommer une extension

Exemple : transformer `.jpeg` en `.jpg`.

```bash
fichier="image.jpeg"

nouveau="${fichier%.jpeg}.jpg"

echo "$nouveau"
```

Résultat :

```txt
image.jpg
```

---

## Renommer tous les fichiers `.jpeg` d’un dossier

```bash
#!/bin/bash

dossier="${1:-.}"
compteur=0

if [[ ! -d "$dossier" ]]; then
    echo "Erreur : $dossier n'est pas un dossier valide" >&2
    exit 1
fi

for fichier in "$dossier"/*.jpeg; do
    [[ -f "$fichier" ]] || continue

    nouveau="${fichier%.jpeg}.jpg"
    mv "$fichier" "$nouveau"

    echo "Renommé : $fichier → $nouveau"
    ((compteur++))
done

echo "Total : $compteur fichier(s) renommé(s)."
```

---

## Créer un fichier de rapport daté

```bash
#!/bin/bash

destination="$HOME/rapports"
mkdir -p "$destination"

date_jour=$(date +%Y%m%d)
rapport="$destination/rapport_${date_jour}.txt"

{
    echo "Rapport du $(date '+%Y-%m-%d %H:%M:%S')"
    echo "Utilisateur : $USER"
    echo "Machine : $(hostname)"
    echo "Dossier courant : $PWD"
} > "$rapport"

echo "Rapport créé : $rapport"
```

---

## Créer un log horodaté

```bash
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

log "Début du script"
log "Traitement en cours"
log "Fin du script"
```

Sortie possible :

```txt
[2026-05-25 21:30:00] Début du script
[2026-05-25 21:30:01] Traitement en cours
[2026-05-25 21:30:02] Fin du script
```

---

## Fonction d’erreur réutilisable

```bash
erreur() {
    echo "Erreur : $1" >&2
}
```

Utilisation :

```bash
if [[ -z "$1" ]]; then
    erreur "argument manquant"
    exit 1
fi
```

---

## Vérifier qu’une commande existe

Avant d’utiliser un outil externe, tu peux vérifier s’il est installé.

```bash
if ! command -v nmap >/dev/null 2>&1; then
    echo "Erreur : nmap n'est pas installé" >&2
    exit 1
fi
```

Autre exemple :

```bash
for outil in host whois prips; do
    if ! command -v "$outil" >/dev/null 2>&1; then
        echo "Erreur : outil manquant : $outil" >&2
        exit 1
    fi
done
```

---

## Créer un dossier temporaire

```bash
tmp_dir=$(mktemp -d)

echo "Dossier temporaire : $tmp_dir"

# traitements...

rm -rf "$tmp_dir"
```

!!! warning
    Fais attention avec `rm -rf`.  
    Vérifie toujours que la variable contient bien ce que tu attends.

---

## Créer un fichier temporaire

```bash
tmp_file=$(mktemp)

echo "Fichier temporaire : $tmp_file"

echo "test" > "$tmp_file"
cat "$tmp_file"

rm -f "$tmp_file"
```

---

## Faire un dry-run

Un mode dry-run affiche ce qui serait fait, sans l’exécuter.

```bash
dry_run=true
fichier="test.txt"
destination="/tmp/test.txt"

if [[ "$dry_run" == true ]]; then
    echo "[DRY-RUN] mv \"$fichier\" \"$destination\""
else
    mv "$fichier" "$destination"
fi
```

---

## Mini-template de script avec options

```bash
#!/bin/bash

dry_run=false

afficher_aide() {
    echo "Utilisation : $0 [options]"
    echo "  -n, --dry-run   Affiche les actions sans les exécuter"
    echo "  -h, --help      Affiche cette aide"
}

while [[ $# -gt 0 ]]; do
    case "$1" in
        -n|--dry-run)
            dry_run=true
            shift
            ;;
        -h|--help)
            afficher_aide
            exit 0
            ;;
        *)
            echo "Option inconnue : $1" >&2
            afficher_aide
            exit 1
            ;;
    esac
done

if [[ "$dry_run" == true ]]; then
    echo "Mode dry-run activé"
fi
```

---

## Recette complète : petit outil de fichiers

Ce script regroupe plusieurs recettes :

- menu ;
- fonctions ;
- pause ;
- recherche de fichiers `.txt` ;
- comptage de lignes ;
- vérification de fichier.

```bash
#!/bin/bash

pause() {
    read -r -p "Appuyez sur [Entrée] pour continuer..." key
}

liste_all_txt() {
    local list

    list=$(find . -type f -name "*.txt" 2>/dev/null)

    if [[ -z "$list" ]]; then
        echo "Il n'y a aucun fichier .txt dans le répertoire actuel ni dans les sous-dossiers"
    else
        echo -e "Voici la liste des fichiers .txt :\n$list"
    fi

    pause
}

compter_nbr_lignes() {
    local file
    local file_lignes

    read -r -p "Quel fichier voulez-vous analyser ? " file

    if [[ -z "$file" ]]; then
        echo "Veuillez renseigner un fichier"

    elif [[ ! -f "$file" ]]; then
        echo "Erreur : ce fichier n'existe pas"

    else
        file_lignes=$(wc -l < "$file")
        echo "Nombre de ligne(s) dans $file : $file_lignes"
    fi

    pause
}

while true; do
    clear

    echo "---------------------------------"
    echo "        M A I N - M E N U"
    echo "---------------------------------"
    echo "1. Lister tous les fichiers .txt dans le répertoire courant et en-dessous"
    echo "2. Compter le nombre de lignes dans un fichier donné"
    echo "3. Quitter"
    echo "---------------------------------"

    read -r -p "Quel est votre choix ? " choix

    case "$choix" in
        -l|lister|1)
            liste_all_txt
            ;;
        -n|number|2)
            compter_nbr_lignes
            ;;
        -q|quit|3)
            echo "Au revoir."
            break
            ;;
        *)
            echo "Sélectionnez votre choix"
            pause
            ;;
    esac
done
```

---

## Erreurs classiques dans ces recettes

### Oublier les guillemets

À éviter :

```bash
wc -l $fichier
```

Préférer :

```bash
wc -l "$fichier"
```

---

### Utiliser `cat` inutilement

À éviter :

```bash
cat /etc/passwd | wc -l
```

Préférer :

```bash
wc -l < /etc/passwd
```

---

### Oublier `2>/dev/null` avec `find /`

Commande bruyante :

```bash
find / -iname "$1"
```

Version plus propre :

```bash
find / -iname "$1" 2>/dev/null
```

---

### Oublier `read -r`

À éviter :

```bash
read -p "Fichier : " fichier
```

Préférer :

```bash
read -r -p "Fichier : " fichier
```

---

### Oublier de vérifier une cible

À éviter :

```bash
nmap -F -sV "$target"
```

sans vérifier que `$target` n’est pas vide.

Préférer :

```bash
if [[ -z "$target" ]]; then
    echo "Erreur : cible vide" >&2
    exit 1
fi
```

---

## Cheat sheet

| Recette | Syntaxe |
|---|---|
| Écrire plusieurs lignes | `{ echo ...; } > fichier.txt` |
| Vérifier un entier | `[[ "$1" =~ ^-?[0-9]+$ ]]` |
| Menu avec argument | `case "$1" in ... esac` |
| Menu interactif | `while true; do ... done` |
| Menu automatique | `select choix in ...; do ... done` |
| Pause utilisateur | `read -r -p "Appuyez sur Entrée..." key` |
| Date compacte | `date +%Y%m%d` |
| Date horodatée | `date '+%Y-%m-%d %H:%M:%S'` |
| Archive datée | `archive="$destination/$(date +%Y%m%d-%H%M%S).tar.gz"` |
| Créer destination | `mkdir -p "$destination"` |
| Voir et sauvegarder | `commande \| tee fichier.txt` |
| Ajouter avec `tee` | `commande \| tee -a fichier.txt` |
| Longueur variable | `${#var}` |
| Contient une chaîne | `[[ "$var" == *"$value"* ]]` |
| Derniers caractères | `printf "%s" "$var" \| tail -c 20` |
| Parcourir arguments | `while [[ $# -gt 0 ]]; do ... shift; done` |
| Chercher fichier | `find / -iname "$1" 2>/dev/null` |
| Vérifier commande | `command -v outil >/dev/null 2>&1` |
| Fichier temporaire | `mktemp` |
| Dossier temporaire | `mktemp -d` |

---

## Checklist d’utilisation

Avant de réutiliser une recette, vérifie :

- [ ] les variables sont entre guillemets ;
- [ ] les arguments sont vérifiés ;
- [ ] les erreurs importantes vont vers `stderr` ;
- [ ] les fichiers/dossiers existent avant d’être utilisés ;
- [ ] les commandes sensibles sont testées en environnement sûr ;
- [ ] les chemins importants sont absolus si le script est automatisé ;
- [ ] les fichiers de sortie ne sont pas écrasés sans raison ;
- [ ] les outils externes nécessaires sont installés ;
- [ ] les commandes réseau sont utilisées uniquement dans un cadre autorisé.

---

## Résumé

Cette fiche est une boîte à outils.

Quelques réflexes à garder :

```bash
"$variable"
```

pour éviter les problèmes d’espaces.

```bash
[[ -z "$1" ]]
```

pour vérifier un argument vide.

```bash
>&2
```

pour envoyer une erreur vers `stderr`.

```bash
>> log.txt 2>&1
```

pour journaliser sortie et erreurs.

```bash
while true; do ... done
```

pour construire un menu interactif.

```bash
command -v outil
```

pour vérifier qu’un outil existe.

Le principe général :

> Garde les recettes courtes, testables et réutilisables.  
> Quand une recette devient trop longue, transforme-la en fiche dédiée.

---

## Pour aller plus loin

- Fiche précédente : [Automatisation : cron et cas pratiques](automatisation.md).
- Retour au pilier [Bash](index.md).