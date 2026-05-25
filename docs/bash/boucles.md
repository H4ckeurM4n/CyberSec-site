---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Les boucles

Une boucle permet de répéter des actions automatiquement.

En Bash, on utilise surtout :

- `for` pour parcourir une liste connue ;
- `while` pour répéter tant qu’une condition est vraie ;
- `until` pour répéter tant qu’une condition est fausse ;
- `while true` pour créer des menus interactifs ou des boucles contrôlées par `break`.

La bonne question à se poser avant d’écrire une boucle :

> Qu’est-ce qui change à chaque tour, et qu’est-ce qui arrête la boucle ?

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- utiliser une boucle `for` ;
- parcourir une liste de mots ;
- parcourir une plage de nombres ;
- parcourir des fichiers ;
- parcourir les arguments d’un script ;
- utiliser une boucle `while` ;
- lire un fichier ligne par ligne ;
- traiter des arguments progressivement avec `shift` ;
- créer un menu avec `while true` ;
- utiliser `sleep` et une fonction `pause` ;
- utiliser `until` ;
- utiliser `break` et `continue` ;
- écrire des boucles imbriquées ;
- déboguer une boucle.

---

## À retenir en 30 secondes

Boucle `for` :

```bash
for fruit in pomme banane cerise; do
    echo "$fruit"
done
```

Boucle `while` :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
    ((compteur++))
done
```

Lire un fichier ligne par ligne :

```bash
while read -r ligne; do
    echo "$ligne"
done < fichier.txt
```

Menu interactif :

```bash
while true; do
    read -r -p "Votre choix ? " choix

    case "$choix" in
        1) echo "Option 1" ;;
        2) echo "Option 2" ;;
        3) break ;;
        *) echo "Choix invalide" ;;
    esac
done
```

---

## Deux façons de penser les boucles

### `for` : parcourir une liste

Une boucle `for` parcourt une liste connue.

Exemples de listes :

- mots ;
- nombres ;
- fichiers ;
- arguments ;
- résultats d’une commande.

La boucle s’arrête quand la liste est terminée.

---

### `while` : répéter tant qu’une condition est vraie

Une boucle `while` continue tant qu’une condition reste vraie.

Elle s’arrête quand la condition devient fausse.

Exemples :

- répéter avec un compteur ;
- lire un fichier ligne par ligne ;
- traiter des arguments un par un ;
- attendre une condition.

---

## La boucle `for`

Forme générale :

```bash
for variable in liste; do
    commande
done
```

Exemple simple :

```bash
for fruit in pomme banane cerise; do
    echo "J'aime la $fruit"
done
```

Résultat :

```txt
J'aime la pomme
J'aime la banane
J'aime la cerise
```

À chaque tour :

- `fruit` prend une nouvelle valeur ;
- Bash exécute le bloc ;
- la boucle s’arrête quand la liste est terminée.

---

## Variante multi-lignes

Tu peux aussi écrire :

```bash
for variable in 1 2 3 4
do
    echo "$variable"
done
```

Résultat :

```txt
1
2
3
4
```

Les deux styles sont valides.

---

## Parcourir une liste de mots

```bash
for fruit in pomme banane cerise; do
    echo "Fruit : $fruit"
done
```

Autre exemple :

```bash
for user in alice bob charlie; do
    echo "Utilisateur : $user"
done
```

---

## Parcourir une plage de nombres

```bash
for i in {1..5}; do
    echo "Tour numéro $i"
done
```

Résultat :

```txt
Tour numéro 1
Tour numéro 2
Tour numéro 3
Tour numéro 4
Tour numéro 5
```

---

## Parcourir avec un pas

```bash
for i in {0..10..2}; do
    echo "$i"
done
```

Résultat :

```txt
0
2
4
6
8
10
```

Ici :

- départ : `0` ;
- fin : `10` ;
- pas : `2`.

---

## Parcourir des fichiers

Exemple : afficher tous les fichiers `.log` du dossier courant.

```bash
for fichier in *.log; do
    echo "$fichier"
done
```

Autre exemple :

```bash
for fichier in *.txt; do
    echo "Fichier texte : $fichier"
done
```

!!! warning "Attention"
    Si aucun fichier ne correspond au motif `*.txt`, Bash peut laisser le motif tel quel selon la configuration.  
    Ajoute un `echo "[DEBUG] fichier = '$fichier'"` si tu veux comprendre ce que la boucle traite réellement.

---

## Parcourir les arguments d’un script

La bonne forme est :

```bash
for arg in "$@"; do
    echo "Argument : $arg"
done
```

Exécution :

```bash
./script.sh "Jean Pierre" Marie
```

Résultat :

```txt
Argument : Jean Pierre
Argument : Marie
```

`"$@"` conserve correctement les arguments séparés, même s’ils contiennent des espaces.

!!! tip "Rappel"
    Pour parcourir les arguments, utilise presque toujours `"$@"`.

---

## Exemple : table de multiplication

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Usage : $0 <nombre>"
    exit 1
fi

for i in {1..10}; do
    echo "$1 x $i = $(($1 * i))"
done
```

Exécution :

```bash
./multiplication.sh 7
```

Résultat :

```txt
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
...
7 x 10 = 70
```

---

## Exemple cyber : pinguer une liste d’IP

Dans un lab ou sur un périmètre autorisé :

```bash
for ip in 10.10.10.170 10.10.10.174 10.10.10.175; do
    ping -c 1 "$ip"
done
```

Version plus silencieuse :

```bash
for ip in 10.10.10.170 10.10.10.174 10.10.10.175; do
    if ping -c 1 "$ip" > /dev/null 2>&1; then
        echo "$ip est joignable"
    else
        echo "$ip ne répond pas"
    fi
done
```

!!! warning "Cadre légal"
    Utilise ce type de boucle uniquement dans un lab, un CTF, sur tes machines ou sur un périmètre explicitement autorisé.

---

## Exemple cyber : boucle dans une fonction de reconnaissance

Exemple pédagogique pour comprendre la structure :

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

Ce qu’il faut retenir ici :

- `for ip in $ipaddr` parcourt une liste d’IP ;
- chaque IP est traitée une par une ;
- les résultats sont affichés et sauvegardés avec `tee`.

---

## Le `for` style C

La syntaxe `{1..5}` est pratique, mais elle ne fonctionne pas toujours avec des variables.

Par exemple, ceci ne marche pas comme on l’imagine :

```bash
limite=5

for i in {1..$limite}; do
    echo "$i"
done
```

Pour une limite dynamique, utilise le style C :

```bash
limite="$1"

for (( i=1; i<=limite; i++ )); do
    echo "Tour $i"
done
```

Exécution :

```bash
./script.sh 5
```

Résultat :

```txt
Tour 1
Tour 2
Tour 3
Tour 4
Tour 5
```

---

## La boucle `while`

Forme générale :

```bash
while condition; do
    commande
done
```

Exemple avec compteur :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
    ((compteur++))
done
```

Résultat :

```txt
1
2
3
4
5
```

À chaque tour :

- la condition est testée ;
- si elle est vraie, le bloc s’exécute ;
- le compteur est incrémenté ;
- quand la condition devient fausse, la boucle s’arrête.

---

## Attention aux boucles infinies

Erreur classique :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
done
```

Ici, `compteur` ne change jamais.  
La condition reste vraie en permanence.

Correction :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
    ((compteur++))
done
```

!!! danger "Boucle infinie"
    Dans une boucle `while`, vérifie toujours ce qui fait évoluer la condition.

---

## Lire un fichier ligne par ligne

```bash
while read -r ligne; do
    echo "Lu : $ligne"
done < mon_fichier.txt
```

Explication :

- `read -r ligne` lit une ligne ;
- la ligne est stockée dans `ligne` ;
- `< mon_fichier.txt` donne le fichier comme entrée à la boucle.

!!! tip "Bonne pratique"
    Utilise `read -r` pour éviter que Bash interprète les caractères spéciaux comme `\`.

---

## Exemple cyber : lire un fichier de cibles

```bash
while read -r cible; do
    echo "Scan de $cible"
done < cibles.txt
```

Version avec une commande fictive ou à utiliser en lab :

```bash
while read -r cible; do
    echo "Scan de $cible"
    # nmap -F -sV "$cible"
done < cibles.txt
```

!!! warning "Cadre légal"
    Ne lance des scans que sur des cibles autorisées.

---

## Traiter les arguments avec `while` et `shift`

```bash
while [[ $# -gt 0 ]]; do
    echo "Argument courant : $1"
    shift
done
```

Exécution :

```bash
./script.sh un deux trois
```

Résultat :

```txt
Argument courant : un
Argument courant : deux
Argument courant : trois
```

Ici :

- `$#` = nombre d’arguments restants ;
- `$1` = argument courant ;
- `shift` supprime `$1` et décale les autres arguments.

---

## La boucle `until`

`until` est l’inverse de `while`.

Elle répète tant que la condition est fausse.

```bash
compteur=1

until [[ "$compteur" -gt 5 ]]; do
    echo "Compteur : $compteur"
    ((compteur++))
done
```

Résultat :

```txt
Compteur : 1
Compteur : 2
Compteur : 3
Compteur : 4
Compteur : 5
```

Ici, la boucle continue jusqu’à ce que `compteur` devienne supérieur à 5.

!!! note
    En pratique, `while` est beaucoup plus courant.  
    `until` existe surtout comme alternative quand la logique est plus lisible “tant que ce n’est pas vrai”.

---

## `break` : sortir d’une boucle

`break` arrête immédiatement la boucle.

```bash
for i in {1..10}; do
    if [[ "$i" -eq 6 ]]; then
        echo "Stop à $i"
        break
    fi

    echo "Numéro $i"
done
```

Résultat :

```txt
Numéro 1
Numéro 2
Numéro 3
Numéro 4
Numéro 5
Stop à 6
```

---

## `continue` : sauter au tour suivant

`continue` saute le reste du tour courant et passe au suivant.

```bash
for i in {1..5}; do
    if [[ "$i" -eq 3 ]]; then
        continue
    fi

    echo "Numéro $i"
done
```

Résultat :

```txt
Numéro 1
Numéro 2
Numéro 4
Numéro 5
```

Le `3` est ignoré.

---

## Exemple avec `break` et `continue`

```bash
for counter in {1..5}; do
    if [[ "$counter" -eq 2 ]]; then
        continue
    elif [[ "$counter" -eq 4 ]]; then
        break
    fi

    echo "Compteur : $counter"
done
```

Résultat :

```txt
Compteur : 1
Compteur : 3
```

Explication :

- `2` est sauté ;
- à `4`, la boucle s’arrête.

---

## `while true` : boucle infinie contrôlée

`while true` crée une boucle infinie volontaire.

Elle doit être contrôlée avec `break`, `exit`, ou une condition interne.

```bash
while true; do
    echo "Menu"
    read -r -p "Choix : " choix

    if [[ "$choix" == "q" ]]; then
        break
    fi
done
```

Ce modèle est très pratique pour faire des menus.

---

## Menu interactif avec `while true`, `clear` et `case`

```bash
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
            echo "Choix invalide."
            ;;
    esac
done
```

Ici :

- `while true` garde le menu actif ;
- `clear` nettoie l’écran ;
- `read` récupère le choix ;
- `case` aiguille vers la bonne action ;
- `break` quitte le menu.

---

## `sleep` : ralentir une boucle

`sleep` met le script en pause pendant une durée donnée.

```bash
sleep .5   # attend 0,5 seconde
sleep 5    # attend 5 secondes
sleep 5s   # attend 5 secondes
sleep 5m   # attend 5 minutes
sleep 5h   # attend 5 heures
sleep 5d   # attend 5 jours
```

C’est utile pour :

- laisser le temps de lire un résultat ;
- éviter qu’un menu se recharge trop vite ;
- ralentir un traitement ;
- attendre entre deux tentatives.

---

## Exemple avec `sleep`

```bash
liste_all_txt() {
    list=$(find . -type f -name "*.txt" 2>/dev/null)

    if [[ -z "$list" ]]; then
        echo "Il n'y a aucun fichier .txt dans le répertoire actuel ni dans les sous-dossiers"
    else
        echo -e "Voici la liste des fichiers .txt :\n$list"
    fi

    sleep 5s
}
```

Ici, le script attend 5 secondes avant de revenir au menu.

---

## Fonction `pause`

Dans un menu, `sleep` force une attente fixe.  
Une fonction `pause` est souvent plus agréable : l’utilisateur appuie sur Entrée quand il a fini de lire.

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

## Exemple complet : menu avec `pause`

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

## Boucles imbriquées

Une boucle imbriquée est une boucle dans une autre boucle.

```bash
for i in {1..3}; do
    for j in {1..3}; do
        echo "i=$i, j=$j"
    done
done
```

Résultat :

```txt
i=1, j=1
i=1, j=2
i=1, j=3
i=2, j=1
i=2, j=2
i=2, j=3
i=3, j=1
i=3, j=2
i=3, j=3
```

À chaque tour de la boucle externe, la boucle interne s’exécute entièrement.

---

## Déboguer une boucle

Pour comprendre ce qui se passe dans une boucle, ajoute des `echo` temporaires.

```bash
for fichier in *.txt; do
    echo "[DEBUG] fichier = '$fichier'"
done
```

Si rien ne s’affiche, la boucle ne s’exécute peut-être pas, ou la liste est vide.

Autre exemple :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "[DEBUG] compteur = $compteur"
    ((compteur++))
done
```

!!! tip "Méthode simple"
    Quand une boucle ne fait pas ce que tu veux, affiche la variable qui change à chaque tour.

---

## Mini-lab 1 : parcourir une liste

Objectif : créer un script qui affiche une liste de prénoms.

Contraintes :

1. utiliser une boucle `for` ;
2. afficher chaque prénom avec son numéro ;
3. utiliser un compteur.

---

### Correction possible

```bash
#!/bin/bash

prenoms=("Alice" "Bob" "Charlie")
compteur=1

for prenom in "${prenoms[@]}"; do
    echo "$compteur. $prenom"
    ((compteur++))
done
```

---

## Mini-lab 2 : lire un fichier ligne par ligne

Objectif : créer un fichier `cibles.txt`, puis le lire avec une boucle.

Créer `cibles.txt` :

```txt
192.168.1.1
192.168.1.10
example.local
```

Script :

```bash
#!/bin/bash

while read -r cible; do
    echo "Cible : $cible"
done < cibles.txt
```

---

## Mini-lab 3 : menu interactif

Objectif : créer un menu avec trois actions.

Contraintes :

1. option 1 : afficher la date ;
2. option 2 : afficher l’utilisateur courant ;
3. option 3 : quitter ;
4. choix invalide : afficher une erreur ;
5. utiliser `while true`, `read` et `case`.

---

### Correction possible

```bash
#!/bin/bash

pause() {
    read -r -p "Appuyez sur [Entrée] pour continuer..." key
}

while true; do
    clear

    echo "1. Afficher la date"
    echo "2. Afficher l'utilisateur courant"
    echo "3. Quitter"

    read -r -p "Votre choix ? " choix

    case "$choix" in
        1)
            date
            pause
            ;;
        2)
            whoami
            pause
            ;;
        3)
            echo "Au revoir."
            break
            ;;
        *)
            echo "Choix invalide."
            pause
            ;;
    esac
done
```

---

## Erreurs classiques

### Oublier `do`

Incorrect :

```bash
for i in {1..5}
    echo "$i"
done
```

Correct :

```bash
for i in {1..5}; do
    echo "$i"
done
```

---

### Oublier `done`

Incorrect :

```bash
for i in {1..5}; do
    echo "$i"
```

Correct :

```bash
for i in {1..5}; do
    echo "$i"
done
```

---

### Créer une boucle infinie sans le vouloir

Incorrect :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
done
```

Correct :

```bash
compteur=1

while [[ "$compteur" -le 5 ]]; do
    echo "$compteur"
    ((compteur++))
done
```

---

### Oublier ce que contient la variable de boucle

```bash
for fichier in *.txt; do
    echo "$fichier"
done
```

Ici, `fichier` ne contient qu’un seul nom de fichier à la fois, pas toute la liste.

---

### Mauvaise liste d’IP

À éviter :

```bash
for ip in "10.10.10.170 10.10.10.174 10.10.10.175"; do
    ping -c 1 "$ip"
done
```

Ici, la liste entière est traitée comme un seul élément.

Préférer :

```bash
for ip in 10.10.10.170 10.10.10.174 10.10.10.175; do
    ping -c 1 "$ip"
done
```

---

## Cheat sheet

| Syntaxe | Effet |
|---|---|
| `for x in a b c; do ... done` | parcourir une liste |
| `for i in {1..5}; do ... done` | parcourir une plage |
| `for i in {0..10..2}; do ... done` | parcourir avec un pas |
| `for (( i=1; i<=n; i++ )); do ... done` | plage dynamique |
| `for arg in "$@"; do ... done` | parcourir les arguments |
| `while [[ condition ]]; do ... done` | répéter tant que la condition est vraie |
| `while read -r ligne; do ... done < fichier` | lire un fichier ligne par ligne |
| `while [[ $# -gt 0 ]]; do ... shift; done` | traiter les arguments un par un |
| `until [[ condition ]]; do ... done` | répéter tant que la condition est fausse |
| `while true; do ... done` | boucle infinie contrôlée |
| `break` | sortir de la boucle |
| `continue` | passer au tour suivant |
| `sleep 5` | attendre 5 secondes |
| `pause() { read -r -p "..."; }` | créer une pause interactive |

---

## Checklist script propre

Avant de considérer ta boucle comme correcte, vérifie :

- [ ] la boucle a une condition d’arrêt claire ;
- [ ] la variable qui change à chaque tour évolue bien ;
- [ ] les variables sont entre guillemets ;
- [ ] `"$@"` est utilisé pour parcourir les arguments ;
- [ ] `read -r` est utilisé pour lire des lignes ;
- [ ] les menus `while true` ont un `break` ou une sortie prévue ;
- [ ] les boucles trop complexes sont découpées en fonctions ;
- [ ] tu as ajouté des messages de debug si le comportement est étrange.

---

## Résumé

`for` sert à parcourir une liste :

```bash
for item in liste; do
    echo "$item"
done
```

`while` répète tant qu’une condition est vraie :

```bash
while [[ condition ]]; do
    commandes
done
```

`until` répète tant qu’une condition est fausse :

```bash
until [[ condition ]]; do
    commandes
done
```

`while true` sert souvent à créer un menu :

```bash
while true; do
    read -r -p "Choix : " choix
    [[ "$choix" == "q" ]] && break
done
```

Le réflexe essentiel :

> Dans une boucle, identifie toujours ce qui change à chaque tour et ce qui arrête la boucle.

---

## Pour aller plus loin

- Fiche précédente : [Conditions et tests](conditions.md).
- Fiche suivante : [Fonctions et case](fonctions.md).