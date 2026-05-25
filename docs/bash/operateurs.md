---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Opérateurs et calculs

Bash sait faire des calculs, comparer des valeurs, tester des fichiers et combiner des conditions.

Cette fiche sert de référence pour comprendre les opérateurs les plus utiles en scripting Bash, notamment avant d’écrire des conditions plus avancées.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- faire des calculs entiers avec `$(( ... ))` ;
- utiliser les opérateurs arithmétiques ;
- stocker le résultat d’un calcul dans une variable ;
- comprendre le modulo `%` ;
- incrémenter et décrémenter un compteur ;
- comparer des nombres avec `-eq`, `-lt`, `-gt`, etc. ;
- comparer des chaînes de caractères ;
- tester si une chaîne est vide ou non ;
- tester des fichiers et dossiers avec `-e`, `-f`, `-d`, etc. ;
- combiner des conditions avec `&&`, `||` et `!` ;
- utiliser `bc` pour les calculs décimaux ;
- vérifier qu’un argument est bien un nombre entier ;
- lire un code de retour avec `$?`.

---

## À retenir en 30 secondes

Faire un calcul :

```bash
resultat=$((5 + 3))
echo "$resultat"
```

Comparer des nombres :

```bash
age=25
[[ "$age" -ge 18 ]] && echo "Majeur"
```

Comparer du texte :

```bash
nom="Alice"
[[ "$nom" == "Alice" ]] && echo "Bonjour Alice"
```

Tester un fichier :

```bash
[[ -f "notes.txt" ]] && echo "C'est un fichier"
```

Combiner des conditions :

```bash
[[ -e "$1" && -r "$1" ]] && echo "Le fichier existe et est lisible"
```

Vérifier qu’un argument est un entier :

```bash
[[ "$1" =~ ^-?[0-9]+$ ]]
```

---

## Faire un calcul avec `$(( ... ))`

En Bash, les calculs arithmétiques sur des nombres entiers se font avec :

```bash
$(( expression ))
```

Exemple :

```bash
a=10
b=3

echo "Addition       : $((a + b))"
echo "Soustraction   : $((a - b))"
echo "Multiplication : $((a * b))"
echo "Division       : $((a / b))"
echo "Modulo         : $((a % b))"
```

Résultat :

```txt
Addition       : 13
Soustraction   : 7
Multiplication : 30
Division       : 3
Modulo         : 1
```

!!! note
    Dans `$(( ... ))`, tu peux utiliser les variables sans mettre `$` devant leur nom :

    ```bash
    echo "$((a + b))"
    ```

---

## Attention : division entière

Bash ne gère que les entiers dans `$(( ... ))`.

```bash
echo "$((10 / 3))"
```

Résultat :

```txt
3
```

Pas :

```txt
3.33
```

!!! warning "Division entière"
    `$((10 / 3))` donne `3`, car Bash ignore la partie décimale dans les calculs entiers.

---

## Les opérateurs arithmétiques

| Opérateur | Signification | Exemple | Résultat |
|---|---|---|---|
| `+` | addition | `$((5 + 3))` | `8` |
| `-` | soustraction | `$((5 - 3))` | `2` |
| `*` | multiplication | `$((5 * 3))` | `15` |
| `/` | division entière | `$((5 / 3))` | `1` |
| `%` | modulo, reste de division | `$((5 % 3))` | `2` |
| `**` | puissance | `$((2 ** 3))` | `8` |
| `variable++` | incrémentation de 1 | `((x++))` | `x + 1` |
| `variable--` | décrémentation de 1 | `((x--))` | `x - 1` |

---

## Exemple complet des opérations

```bash
#!/bin/bash

increase=1
decrease=1

echo "Addition: 10 + 10 = $((10 + 10))"
echo "Subtraction: 10 - 10 = $((10 - 10))"
echo "Multiplication: 10 * 10 = $((10 * 10))"
echo "Division: 10 / 10 = $((10 / 10))"
echo "Modulus: 10 % 4 = $((10 % 4))"

((increase++))
echo "Increase Variable: $increase"

((decrease--))
echo "Decrease Variable: $decrease"
```

Résultat :

```txt
Addition: 10 + 10 = 20
Subtraction: 10 - 10 = 0
Multiplication: 10 * 10 = 100
Division: 10 / 10 = 1
Modulus: 10 % 4 = 2
Increase Variable: 2
Decrease Variable: 0
```

---

## Stocker le résultat d’un calcul

Tu peux stocker le résultat d’un calcul dans une variable.

```bash
prix=50
reduction=15

prix_final=$((prix - reduction))

echo "Le prix final est $prix_final euros"
```

Résultat :

```txt
Le prix final est 35 euros
```

---

## Le modulo `%`

Le modulo donne le reste d’une division.

```bash
echo "$((10 % 4))"
```

Résultat :

```txt
2
```

Pourquoi ?  
Parce que `10 / 4 = 2`, et il reste `2`.

Le modulo est très utile pour :

- tester si un nombre est pair ou impair ;
- gérer des cycles ;
- déclencher une action toutes les X itérations ;
- travailler avec des compteurs.

---

## Tester si un nombre est pair ou impair

Un nombre est pair si le reste de sa division par 2 vaut `0`.

```bash
#!/bin/bash

result=$(( $1 % 2 ))

if (( result == 0 )); then
    echo "$1 est pair"
else
    echo "$1 est impair"
fi
```

Exécution :

```bash
./pair.sh 8
./pair.sh 7
```

Résultat :

```txt
8 est pair
7 est impair
```

!!! tip "Règle simple"
    `nombre % 2 == 0` → pair.  
    `nombre % 2 != 0` → impair.

---

## Incrémenter et décrémenter un compteur

Un compteur est une variable qu’on augmente ou diminue progressivement.

```bash
compteur=0

((compteur++))
((compteur++))
((compteur += 5))

echo "$compteur"
```

Résultat :

```txt
7
```

Explication :

1. `compteur=0`
2. `compteur++` → `1`
3. `compteur++` → `2`
4. `compteur += 5` → `7`

---

## Opérateurs d’incrémentation

| Syntaxe | Effet |
|---|---|
| `((var++))` | ajoute 1 |
| `((var--))` | retire 1 |
| `((var += 5))` | ajoute 5 |
| `((var -= 5))` | retire 5 |
| `((var *= 2))` | multiplie par 2 |
| `((var /= 2))` | divise par 2 |

Exemple :

```bash
fichiers=0

((fichiers++))
((fichiers++))

echo "Nombre de fichiers traités : $fichiers"
```

Résultat :

```txt
Nombre de fichiers traités : 2
```

---

## Exemple : compteur dans une boucle

```bash
#!/bin/bash

tab_name=("Camille" "Maya" "Manco" "Billal" "Alex")
compteur=0

for name in "${tab_name[@]}"; do
    echo "$compteur $name"
    ((compteur++))
done
```

Résultat :

```txt
0 Camille
1 Maya
2 Manco
3 Billal
4 Alex
```

---

## Exemple cyber : compter des hôtes testés

Dans un contexte de lab ou d’audit autorisé, un compteur peut servir à suivre le nombre d’hôtes testés ou actifs.

```bash
hosts_up=0
hosts_total=0

for host in $cidr_ips; do
    ping -c 2 "$host" > /dev/null 2>&1

    if [[ "$?" -eq 0 ]]; then
        echo "$host is up."
        ((hosts_up++))
        ((hosts_total++))
    else
        echo "$host is down."
        ((hosts_total++))
    fi
done

echo "Hôtes actifs : $hosts_up"
echo "Hôtes testés : $hosts_total"
```

!!! warning "Cadre légal"
    À utiliser uniquement dans un lab, un CTF, sur tes propres machines ou sur un périmètre explicitement autorisé.

---

## Les décimales avec `bc`

Pour les calculs décimaux, on utilise souvent `bc`.

```bash
echo "scale=2; 10 / 3" | bc
```

Résultat :

```txt
3.33
```

Stocker le résultat :

```bash
resultat=$(echo "scale=2; 10 / 3" | bc)
echo "Le résultat est $resultat"
```

Explication :

- `scale=2` indique deux chiffres après la virgule ;
- `10 / 3` est l’opération ;
- `bc` fait le calcul.

!!! note
    `bc` peut ne pas être installé par défaut sur tous les systèmes minimalistes.

---

## Comparer des nombres

Pour comparer des nombres dans `[[ ... ]]`, on utilise des opérateurs spécifiques.

| Opérateur | Signification | Mnémotechnique |
|---|---|---|
| `-eq` | égal | **eq**ual |
| `-ne` | différent | **n**ot **e**qual |
| `-lt` | inférieur | **l**ess **t**han |
| `-le` | inférieur ou égal | **l**ess or **e**qual |
| `-gt` | supérieur | **g**reater **t**han |
| `-ge` | supérieur ou égal | **g**reater or **e**qual |

Exemple :

```bash
age=25

[[ "$age" -ge 18 ]] && echo "Majeur"
[[ "$age" -eq 30 ]] && echo "A 30 ans"
```

Résultat :

```txt
Majeur
```

---

## Exemples de comparaisons numériques

```bash
age=25

if [[ "$age" -ge 18 ]]; then
    echo "Majeur"
fi
```

```bash
tentatives=5

if [[ "$tentatives" -gt 3 ]]; then
    echo "Trop de tentatives"
fi
```

```bash
if [[ "$#" -eq 0 ]]; then
    echo "Aucun argument fourni"
fi
```

```bash
if [[ "$#" -lt 1 ]]; then
    echo "Nombre d'arguments inférieur à 1"
elif [[ "$#" -gt 1 ]]; then
    echo "Nombre d'arguments supérieur à 1"
else
    echo "Nombre d'arguments égal à 1"
fi
```

---

## Comparer avec `(( ... ))`

Pour les comparaisons mathématiques, on peut aussi utiliser `(( ... ))`.

Dans ce cas, on utilise les symboles habituels :

| Opérateur | Signification |
|---|---|
| `==` | égal |
| `!=` | différent |
| `<` | inférieur |
| `<=` | inférieur ou égal |
| `>` | supérieur |
| `>=` | supérieur ou égal |

Exemple :

```bash
a=10

if (( a > 5 )); then
    echo "a est supérieur à 5"
fi
```

Autre exemple :

```bash
tentatives=4

if (( tentatives >= 3 )); then
    echo "Compte bloqué"
fi
```

!!! tip "À retenir"
    Dans `(( ... ))`, tu n’as pas besoin de mettre `$` devant les variables.

---

## `[[ ... ]]` ou `(( ... ))` pour les nombres ?

Les deux peuvent être utilisés.

Avec `[[ ... ]]` :

```bash
if [[ "$age" -ge 18 ]]; then
    echo "Majeur"
fi
```

Avec `(( ... ))` :

```bash
if (( age >= 18 )); then
    echo "Majeur"
fi
```

La syntaxe `(( ... ))` est souvent plus naturelle pour les calculs et comparaisons numériques.

---

## Comparer du texte

Pour comparer des chaînes de caractères, on utilise d’autres opérateurs.

| Opérateur | Signification |
|---|---|
| `=` ou `==` | chaînes identiques |
| `!=` | chaînes différentes |
| `-z` | chaîne vide |
| `-n` | chaîne non vide |
| `<` | plus petit selon l’ordre ASCII |
| `>` | plus grand selon l’ordre ASCII |

Exemple :

```bash
nom="Alice"

[[ "$nom" == "Alice" ]] && echo "Bonjour Alice"
[[ "$nom" != "Bob" ]] && echo "Tu n'es pas Bob"
[[ -z "$nom" ]] && echo "Le nom est vide"
[[ -n "$nom" ]] && echo "Le nom n'est pas vide"
```

Résultat :

```txt
Bonjour Alice
Tu n'es pas Bob
Le nom n'est pas vide
```

---

## Tester si une variable est vide

```bash
nom=""

if [[ -z "$nom" ]]; then
    echo "Le nom est vide"
fi
```

Tester si une variable n’est pas vide :

```bash
nom="Camille"

if [[ -n "$nom" ]]; then
    echo "Le nom n'est pas vide"
fi
```

---

## Tester si une chaîne contient un motif

Avec `[[ ... ]]`, on peut utiliser des motifs avec `*`.

```bash
var="8dm7KsjU28B7v621Jls"
value="Ksj"

if [[ "$var" == *"$value"* ]]; then
    echo "La variable contient la valeur recherchée"
fi
```

Autre exemple :

```bash
message="Erreur critique détectée"

if [[ "$message" == *"critique"* ]]; then
    echo "Message critique"
fi
```

!!! tip "Important"
    Les `*` doivent être en dehors des guillemets autour de la variable recherchée :

    ```bash
    [[ "$var" == *"$value"* ]]
    ```

---

## Comparaison ASCII avec `<` et `>`

En comparaison de chaînes, `<` et `>` ne veulent pas dire “plus petit nombre” ou “plus grand nombre”.

Ils comparent selon l’ordre lexicographique / ASCII.

Exemple :

```bash
[[ "A" < "B" ]] && echo "A est avant B"
```

Résultat :

```txt
A est avant B
```

Quelques repères ASCII :

| Décimal | Hexadécimal | Caractère | Description |
|---:|---|---|---|
| `65` | `41` | `A` | lettre A majuscule |
| `66` | `42` | `B` | lettre B majuscule |
| `67` | `43` | `C` | lettre C majuscule |
| `97` | `61` | `a` | lettre a minuscule |
| `98` | `62` | `b` | lettre b minuscule |
| `127` | `7F` | `DEL` | suppression |

!!! warning
    Pour comparer des nombres, n’utilise pas `<` et `>` dans `[[ ... ]]`.  
    Utilise `-lt`, `-gt`, etc., ou bien `(( ... ))`.

---

## Exemple : vérifier un argument texte

```bash
#!/bin/bash

if [[ "$1" != "HackTheBox" ]]; then
    echo -e "You need to give 'HackTheBox' as argument."
    exit 1
elif [[ "$#" -gt 1 ]]; then
    echo -e "Too many arguments given."
    exit 1
else
    domain="$1"
    echo -e "Success!"
fi
```

---

## Tester des fichiers et dossiers

Bash permet de vérifier l’existence, le type ou les permissions d’un chemin.

| Opérateur | Signification |
|---|---|
| `-e fichier` | le chemin existe |
| `-f fichier` | c’est un fichier normal |
| `-d fichier` | c’est un dossier |
| `-s fichier` | le fichier n’est pas vide |
| `-r fichier` | le fichier est lisible |
| `-w fichier` | le fichier est modifiable |
| `-x fichier` | le fichier est exécutable |

---

## Exemples de tests sur fichiers

Tester si un chemin existe :

```bash
if [[ -e "notes.txt" ]]; then
    echo "Le fichier existe"
fi
```

Tester si c’est un fichier normal :

```bash
if [[ -f "notes.txt" ]]; then
    echo "C'est un fichier"
fi
```

Tester si c’est un dossier :

```bash
if [[ -d "/home/kali" ]]; then
    echo "C'est un dossier"
fi
```

Tester si un fichier n’est pas vide :

```bash
if [[ -s "rapport.txt" ]]; then
    echo "Le fichier contient quelque chose"
fi
```

Tester si un fichier est lisible :

```bash
if [[ -r "/etc/passwd" ]]; then
    echo "Le fichier est lisible"
fi
```

Tester si un fichier est modifiable :

```bash
if [[ -w "rapport.txt" ]]; then
    echo "Je peux écrire dedans"
fi
```

Tester si un fichier est exécutable :

```bash
if [[ -x "script.sh" ]]; then
    echo "Le script est exécutable"
fi
```

---

## Exemple complet : analyser un chemin

```bash
#!/bin/bash

chemin="$1"

if [[ -z "$chemin" ]]; then
    echo "Erreur : vous devez indiquer un chemin"
    exit 1

elif [[ ! -e "$chemin" ]]; then
    echo "Erreur : $chemin n'existe pas"
    exit 1

elif [[ -f "$chemin" ]]; then
    echo "$chemin est un fichier"

    [[ -r "$chemin" ]] && echo "Le fichier est lisible" || echo "Le fichier n'est pas lisible"
    [[ -w "$chemin" ]] && echo "Le fichier est modifiable" || echo "Le fichier n'est pas modifiable"
    [[ -x "$chemin" ]] && echo "Le fichier est exécutable" || echo "Le fichier n'est pas exécutable"

elif [[ -d "$chemin" ]]; then
    echo "$chemin est un dossier"
    nbr_elements=$(ls "$chemin" | wc -l)
    echo "Il y a $nbr_elements éléments dans le dossier"

else
    echo "$chemin existe, mais ce n'est ni un fichier ni un dossier"
fi
```

Exécution :

```bash
./analyse_chemin.sh /etc/passwd
./analyse_chemin.sh /home
./analyse_chemin.sh /inexistant
```

---

## Opérateurs logiques : `&&`, `||`, `!`

Les opérateurs logiques permettent de combiner ou inverser des conditions.

| Opérateur | Signification |
|---|---|
| `&&` | ET : les deux conditions doivent être vraies |
| `\|\|` | OU : au moins une condition doit être vraie |
| `!` | NON : inverse la condition |

---

## Exemples avec `&&`

```bash
age=25

if [[ "$age" -ge 18 && "$age" -le 65 ]]; then
    echo "Âge dans la tranche"
fi
```

Ici, il faut que :

- `age >= 18`
- et `age <= 65`

soient vrais.

---

## Exemples avec `||`

```bash
age=70

if [[ "$age" -lt 18 || "$age" -gt 65 ]]; then
    echo "Tarif spécial"
fi
```

Ici, il faut que :

- `age < 18`
- ou `age > 65`

soit vrai.

---

## Exemples avec `!`

```bash
if [[ ! -d "$1" ]]; then
    echo "Ce n'est pas un dossier"
fi
```

Autre exemple :

```bash
age=25

if [[ ! "$age" -lt 18 ]]; then
    echo "La personne n'est pas mineure"
fi
```

---

## Combiner existence et permissions

```bash
if [[ -e "$1" && -r "$1" ]]; then
    echo "Le fichier existe et il est lisible"
fi
```

Version plus détaillée :

```bash
#!/bin/bash

if [[ -e "$1" && -r "$1" ]]; then
    echo -e "We can read the file that has been specified."
    exit 0

elif [[ ! -e "$1" ]]; then
    echo -e "The specified file does not exist."
    exit 2

elif [[ -e "$1" && ! -r "$1" ]]; then
    echo -e "We don't have read permission for this file."
    exit 1

else
    echo -e "Error occurred."
    exit 5
fi
```

---

## Raccourcis entre commandes avec `&&` et `||`

`&&` et `||` ne servent pas seulement dans `[[ ... ]]`.  
Ils peuvent aussi chaîner des commandes.

Avec `&&` :

```bash
mkdir mon_dossier && echo "Dossier créé !"
```

La deuxième commande ne s’exécute que si la première réussit.

Avec `||` :

```bash
cd /inexistant || echo "Le dossier n'existe pas"
```

La deuxième commande ne s’exécute que si la première échoue.

---

## Exemples de raccourcis utiles

Créer un dossier puis entrer dedans :

```bash
mkdir projet && cd projet
```

Tester l’existence d’un fichier :

```bash
[[ -f "notes.txt" ]] && echo "Le fichier existe"
```

Afficher une erreur si un dossier n’existe pas :

```bash
[[ -d "$1" ]] || echo "Ce dossier n'existe pas"
```

!!! warning "Lisibilité"
    Les raccourcis `&&` et `||` sont pratiques, mais pour une logique complexe, un vrai `if` est souvent plus lisible.

---

## Calculer la longueur d’une variable

Bash permet de connaître la longueur d’une variable avec :

```bash
${#variable}
```

Exemple :

```bash
htb="HackTheBox"

echo "${#htb}"
```

Résultat :

```txt
10
```

Stocker cette longueur :

```bash
longueur=${#htb}
echo "$longueur"
```

---

## Calculer une longueur avec `wc -m`

On peut aussi utiliser `wc -m`.

```bash
var="Bonjour tout le monde"

longueur=$(echo "$var" | wc -m)

echo "$longueur"
```

!!! note
    `wc -m` compte les caractères, mais `echo` ajoute souvent un retour à la ligne.  
    Pour être plus précis, on peut utiliser `printf`.

Version plus propre :

```bash
var="Bonjour tout le monde"

longueur=$(printf "%s" "$var" | wc -m)

echo "$longueur"
```

---

## Exemple : retrouver une valeur à un compteur précis

Exemple pédagogique : encoder une variable en base64 plusieurs fois, puis récupérer l’état au tour 35.

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

Même logique pour mesurer la longueur au tour 35 :

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

## Vérifier qu’un argument est un nombre entier

Pour vérifier qu’une valeur est un entier, on peut utiliser une expression régulière.

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

se lit ainsi :

| Élément | Signification |
|---|---|
| `^` | début de la chaîne |
| `-?` | signe moins optionnel |
| `[0-9]+` | un ou plusieurs chiffres |
| `$` | fin de la chaîne |

Ce motif accepte :

```txt
2
45
-8
```

Il refuse :

```txt
abc
4a
12.5
```

---

## Exemple : script de calcul avec deux nombres

```bash
#!/bin/bash

if [[ "$#" -ne 2 ]]; then
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

Exécution :

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

## Lire le code de retour avec `$?`

Chaque commande renvoie un code de retour.

```bash
ma_commande
echo "Code de retour : $?"
```

Convention :

| Code | Sens |
|---:|---|
| `0` | succès |
| autre que `0` | erreur |

Exemple :

```bash
ls /tmp
echo "$?"
```

Résultat possible :

```txt
0
```

Exemple avec erreur :

```bash
ls /inexistant
echo "$?"
```

Résultat possible :

```txt
2
```

!!! tip
    Pour tester directement une commande, tu peux souvent faire :

    ```bash
    if ls /tmp; then
        echo "Commande réussie"
    else
        echo "Commande échouée"
    fi
    ```

---

## Mini-lab 1 : pair ou impair

Objectif : créer un script `pair.sh`.

Contraintes :

1. le script attend exactement un argument ;
2. l’argument doit être un nombre entier ;
3. le script affiche si le nombre est pair ou impair.

---

### Correction possible

```bash
#!/bin/bash

if [[ "$#" -ne 1 ]]; then
    echo "Usage : $0 <nombre>"
    exit 1
fi

if [[ ! "$1" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : un nombre entier est attendu"
    exit 1
fi

if (( $1 % 2 == 0 )); then
    echo "$1 est pair"
else
    echo "$1 est impair"
fi
```

---

## Mini-lab 2 : calculatrice simple

Objectif : créer un script `calcul.sh`.

Contraintes :

1. le script attend deux nombres ;
2. il vérifie que les deux arguments sont des entiers ;
3. il affiche addition, soustraction, multiplication, division entière et modulo.

---

### Correction possible

```bash
#!/bin/bash

if [[ "$#" -ne 2 ]]; then
    echo -e "Usage :\n\t$0 <nombre1> <nombre2>"
    exit 1
fi

if [[ ! "$1" =~ ^-?[0-9]+$ || ! "$2" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : les deux arguments doivent être des nombres entiers"
    exit 1
fi

if [[ "$2" -eq 0 ]]; then
    echo "Erreur : division par zéro impossible"
    exit 1
fi

echo "Addition       : $(($1 + $2))"
echo "Soustraction   : $(($1 - $2))"
echo "Multiplication : $(($1 * $2))"
echo "Division       : $(($1 / $2))"
echo "Modulo         : $(($1 % $2))"
```

---

## Mini-lab 3 : inspecteur de chemin

Objectif : créer un script `inspect.sh`.

Contraintes :

1. le script attend un chemin ;
2. il vérifie que le chemin existe ;
3. il indique si c’est un fichier ou un dossier ;
4. s’il s’agit d’un fichier, il indique s’il est lisible, modifiable et exécutable.

---

### Correction possible

```bash
#!/bin/bash

chemin="$1"

if [[ -z "$chemin" ]]; then
    echo "Usage : $0 <chemin>"
    exit 1
fi

if [[ ! -e "$chemin" ]]; then
    echo "Erreur : $chemin n'existe pas"
    exit 1
fi

if [[ -f "$chemin" ]]; then
    echo "$chemin est un fichier"

    [[ -r "$chemin" ]] && echo "Lisible : oui" || echo "Lisible : non"
    [[ -w "$chemin" ]] && echo "Modifiable : oui" || echo "Modifiable : non"
    [[ -x "$chemin" ]] && echo "Exécutable : oui" || echo "Exécutable : non"

elif [[ -d "$chemin" ]]; then
    echo "$chemin est un dossier"
    echo "Nombre d'éléments : $(ls "$chemin" | wc -l)"
else
    echo "$chemin existe, mais ce n'est ni un fichier ni un dossier"
fi
```

---

## Erreurs classiques

### Confondre comparaison texte et comparaison numérique

À éviter pour comparer un nombre :

```bash
if [[ "$age" = 18 ]]; then
    echo "Age égal à 18"
fi
```

Préférer :

```bash
if [[ "$age" -eq 18 ]]; then
    echo "Age égal à 18"
fi
```

Ou :

```bash
if (( age == 18 )); then
    echo "Age égal à 18"
fi
```

---

### Oublier `$(( ... ))` pour calculer

Incorrect :

```bash
resultat=5+3
echo "$resultat"
```

Résultat :

```txt
5+3
```

Correct :

```bash
resultat=$((5 + 3))
echo "$resultat"
```

Résultat :

```txt
8
```

---

### Oublier que Bash fait une division entière

```bash
echo "$((10 / 3))"
```

Résultat :

```txt
3
```

Pour obtenir une décimale :

```bash
echo "scale=2; 10 / 3" | bc
```

---

### Ne pas vérifier que l’argument est un nombre

Risque :

```bash
resultat=$(($1 + 2))
```

Si l’utilisateur donne `abc`, le script peut se comporter de manière inattendue.

Préférer :

```bash
if [[ ! "$1" =~ ^-?[0-9]+$ ]]; then
    echo "Erreur : un nombre entier est attendu"
    exit 1
fi
```

---

### Confondre `&&` raccourci et logique complexe

Court et lisible :

```bash
[[ -f "$1" ]] && echo "Fichier trouvé"
```

Moins lisible si ça devient long :

```bash
[[ -f "$1" ]] && [[ -r "$1" ]] && [[ -s "$1" ]] && echo "OK" || echo "KO"
```

Dans ce cas, préfère un vrai `if`.

---

## Cheat sheet

| Élément | Usage concret |
|---|---|
| `$((a + b))` | faire un calcul entier |
| `prix_final=$((prix - reduction))` | stocker un résultat |
| `%` | obtenir le reste d’une division |
| `((compteur++))` | incrémenter un compteur |
| `((compteur--))` | décrémenter un compteur |
| `((compteur += 5))` | ajouter 5 à un compteur |
| `-eq`, `-ne` | égal, différent pour les nombres |
| `-lt`, `-le` | inférieur, inférieur ou égal |
| `-gt`, `-ge` | supérieur, supérieur ou égal |
| `==`, `!=` | comparer du texte |
| `-z`, `-n` | chaîne vide, chaîne non vide |
| `-e`, `-f`, `-d` | existe, fichier, dossier |
| `-r`, `-w`, `-x` | lisible, modifiable, exécutable |
| `&&` | ET logique ou exécuter si succès |
| `\|\|` | OU logique ou exécuter si échec |
| `!` | inverse une condition |
| `bc` | calculs décimaux |
| `(( a > 5 ))` | comparaison numérique lisible |
| `${#variable}` | longueur d’une variable |
| `[[ "$1" =~ regex ]]` | tester avec une expression régulière |
| `$?` | code retour de la dernière commande |

---

## Checklist script propre

Avant de considérer ton script comme propre, vérifie :

- [ ] les calculs utilisent `$(( ... ))` ;
- [ ] les comparaisons numériques utilisent `-eq`, `-gt`, etc. ou `(( ... ))` ;
- [ ] les comparaisons texte utilisent `==`, `!=`, `-z`, `-n` ;
- [ ] les variables sont entourées de guillemets dans `[[ ... ]]` ;
- [ ] les arguments numériques sont validés avec une regex ;
- [ ] les divisions par zéro sont évitées ;
- [ ] les fichiers sont testés avec `-e`, `-f`, `-d`, `-r`, etc. ;
- [ ] les conditions longues sont écrites avec un vrai `if` plutôt qu’un raccourci illisible ;
- [ ] les erreurs donnent un message clair.

---

## Résumé

Pour faire un calcul entier :

```bash
resultat=$((a + b))
```

Pour comparer des nombres :

```bash
[[ "$age" -ge 18 ]]
```

ou :

```bash
(( age >= 18 ))
```

Pour comparer du texte :

```bash
[[ "$nom" == "Alice" ]]
```

Pour tester un fichier :

```bash
[[ -f "$chemin" ]]
```

Pour combiner des conditions :

```bash
[[ -e "$chemin" && -r "$chemin" ]]
```

Pour vérifier un entier :

```bash
[[ "$1" =~ ^-?[0-9]+$ ]]
```

La règle importante :

```bash
$(( ... ))    # calcul
[[ ... ]]     # test général
(( ... ))     # test/calcul numérique
```

---

## Pour aller plus loin

- Fiche précédente : [Redirections, erreurs et pipes](redirections.md).
- Fiche suivante : [Conditions et tests](conditions.md).