---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Conditions et tests

Les conditions permettent à un script de prendre des décisions.

Un script peut ainsi exécuter une action si une condition est vraie, une autre si elle est fausse, ou arrêter son exécution si une erreur est détectée.

Exemples simples :

```bash
if [[ "$age" -ge 18 ]]; then
    echo "Majeur"
fi
```

```bash
if [[ -f "$1" ]]; then
    echo "C'est un fichier"
else
    echo "Ce n'est pas un fichier"
fi
```

Pour la liste complète des opérateurs de comparaison, garde sous la main la fiche [Opérateurs et calculs](operateurs.md).

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- écrire une condition simple avec `if` ;
- utiliser `else` et `elif` ;
- respecter la syntaxe Bash des conditions ;
- comparer des nombres ;
- comparer des chaînes de caractères ;
- tester si une variable est vide ou non ;
- tester des fichiers et dossiers ;
- combiner des conditions avec `&&`, `||` et `!` ;
- écrire des conditions imbriquées ;
- comprendre la différence entre `[ ]` et `[[ ]]` ;
- éviter les erreurs classiques de syntaxe.

---

## À retenir en 30 secondes

Structure minimale :

```bash
if [[ condition ]]; then
    commandes
fi
```

Avec alternative :

```bash
if [[ condition ]]; then
    commandes_si_vrai
else
    commandes_si_faux
fi
```

Avec plusieurs cas :

```bash
if [[ condition_1 ]]; then
    commandes
elif [[ condition_2 ]]; then
    autres_commandes
else
    commandes_par_defaut
fi
```

Tests fréquents :

```bash
[[ "$age" -ge 18 ]]     # nombre supérieur ou égal à 18
[[ "$nom" == "Alice" ]] # texte égal à Alice
[[ -z "$nom" ]]         # variable vide
[[ -f "$1" ]]           # fichier
[[ -d "$1" ]]           # dossier
[[ ! -e "$1" ]]         # n'existe pas
```

---

## La structure `if`

La structure de base est :

```bash
if [[ condition ]]; then
    # code si la condition est vraie
fi
```

Exemple :

```bash
nombre=15

if [[ "$nombre" -gt 10 ]]; then
    echo "Le nombre est supérieur à 10"
fi
```

Résultat :

```txt
Le nombre est supérieur à 10
```

Ici :

- `if` démarre la condition ;
- `[[ "$nombre" -gt 10 ]]` est le test ;
- `then` introduit le bloc exécuté si le test est vrai ;
- `fi` ferme la condition.

!!! note "Astuce"
    `fi`, c’est `if` écrit à l’envers.  
    Il sert à fermer le bloc `if`.

---

## Règles de syntaxe importantes

En Bash, les espaces sont très importants.

Correct :

```bash
if [[ "$nombre" -gt 10 ]]; then
    echo "OK"
fi
```

Incorrect :

```bash
if [[$nombre -gt 10]]; then
    echo "OK"
fi
```

Règles à retenir :

- espace après `[[` ;
- espace avant `]]` ;
- espace autour de l’opérateur ;
- `then` sur la même ligne avec `;`, ou sur la ligne suivante ;
- `fi` obligatoire à la fin.

Deux syntaxes valides :

```bash
if [[ "$nombre" -gt 10 ]]; then
    echo "OK"
fi
```

ou :

```bash
if [[ "$nombre" -gt 10 ]]
then
    echo "OK"
fi
```

!!! tip "Recommandation"
    Pour garder un style lisible, utilise souvent :

    ```bash
    if [[ condition ]]; then
        commandes
    fi
    ```

---

## `if ... else`

`else` permet d’exécuter un bloc si la condition est fausse.

```bash
nombre=5

if [[ "$nombre" -gt 10 ]]; then
    echo "Supérieur à 10"
else
    echo "Inférieur ou égal à 10"
fi
```

Résultat :

```txt
Inférieur ou égal à 10
```

Structure générale :

```bash
if [[ condition ]]; then
    commandes_si_vrai
else
    commandes_si_faux
fi
```

---

## `if ... elif ... else`

`elif` signifie “sinon si”.

Il permet de tester plusieurs conditions les unes après les autres.

```bash
age="$1"

if [[ "$age" -lt 13 ]]; then
    echo "Tu es un enfant."
elif [[ "$age" -lt 18 ]]; then
    echo "Tu es un adolescent."
elif [[ "$age" -lt 65 ]]; then
    echo "Tu es un adulte."
else
    echo "Tu es un senior."
fi
```

Exemples :

```bash
./age.sh 10
./age.sh 16
./age.sh 25
./age.sh 70
```

Résultats possibles :

```txt
Tu es un enfant.
Tu es un adolescent.
Tu es un adulte.
Tu es un senior.
```

!!! note
    Tu peux mettre autant de `elif` que nécessaire, mais seulement un `else`, placé à la fin.

---

## Structure complète

```bash
if [[ condition_1 ]]; then
    commandes_1
elif [[ condition_2 ]]; then
    commandes_2
elif [[ condition_3 ]]; then
    commandes_3
else
    commandes_par_defaut
fi
```

Lecture :

- si `condition_1` est vraie, Bash exécute `commandes_1` ;
- sinon, il teste `condition_2` ;
- sinon, il teste `condition_3` ;
- si aucune condition n’est vraie, il exécute `else`.

---

## Exemple : valider le nombre d’arguments

Exemple classique : un script attend exactement un argument.

```bash
#!/bin/bash

if [[ "$#" -eq 0 ]]; then
    echo -e "You need to specify the target domain.\n"
    echo -e "Usage:\n\t$0 <domain>"
    exit 1

elif [[ "$#" -eq 1 ]]; then
    domain="$1"
    echo "Domaine choisi : $domain"

else
    echo -e "Too many arguments given."
    exit 1
fi
```

Ce script :

1. vérifie si aucun argument n’a été donné ;
2. accepte exactement un argument ;
3. refuse s’il y en a trop.

!!! warning "Cadre légal"
    Si ce type de script sert ensuite à faire de la reconnaissance, utilise-le uniquement sur tes domaines, un lab, un CTF ou un périmètre autorisé.

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

Exemples :

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

---

## Exemples de comparaisons numériques

Tester si aucun argument n’a été donné :

```bash
if [[ "$#" -eq 0 ]]; then
    echo "Aucun argument fourni"
fi
```

Tester si le nombre d’arguments est inférieur à 1 :

```bash
if [[ "$#" -lt 1 ]]; then
    echo "Number of given arguments is less than 1"
    exit 1
fi
```

Tester si le nombre d’arguments est supérieur à 1 :

```bash
if [[ "$#" -gt 1 ]]; then
    echo "Number of given arguments is greater than 1"
    exit 1
fi
```

Tester un âge :

```bash
age=25

if [[ "$age" -ge 18 ]]; then
    echo "Majeur"
fi
```

---

## Comparer des chaînes de caractères

Pour comparer du texte, on utilise d’autres opérateurs.

| Opérateur | Signification |
|---|---|
| `=` ou `==` | chaînes identiques |
| `!=` | chaînes différentes |
| `-z` | chaîne vide |
| `-n` | chaîne non vide |

Exemples :

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

Avec `-z` :

```bash
nom=""

if [[ -z "$nom" ]]; then
    echo "Le nom est vide"
fi
```

Avec `-n` :

```bash
nom="Camille"

if [[ -n "$nom" ]]; then
    echo "Le nom n'est pas vide"
fi
```

Cas très fréquent : vérifier qu’un argument a été fourni.

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant" >&2
    exit 1
fi
```

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

Ce script vérifie :

- que le premier argument est bien `HackTheBox` ;
- qu’il n’y a pas trop d’arguments ;
- sinon il valide l’entrée.

---

## Tester des fichiers et dossiers

Les conditions servent très souvent à vérifier des fichiers, dossiers et permissions.

| Opérateur | Vrai si… |
|---|---|
| `-e fichier` | le chemin existe |
| `-f fichier` | c’est un fichier normal |
| `-d fichier` | c’est un dossier |
| `-s fichier` | le fichier n’est pas vide |
| `-r fichier` | le fichier est lisible |
| `-w fichier` | le fichier est modifiable |
| `-x fichier` | le fichier est exécutable |

---

## Exemples de tests fichiers

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

## Tests utiles à connaître

| Test | Signification | Exemple |
|---|---|---|
| `-f` | le fichier existe et est un fichier normal | `[[ -f "fichier.txt" ]]` |
| `-d` | le dossier existe | `[[ -d "/home" ]]` |
| `-e` | le chemin existe | `[[ -e "$1" ]]` |
| `-z` | la chaîne est vide | `[[ -z "$nom" ]]` |
| `-n` | la chaîne n’est pas vide | `[[ -n "$nom" ]]` |
| `-gt` | le nombre est supérieur | `[[ "$a" -gt "$b" ]]` |

---

## Exemple concret : vérifier un chemin

```bash
#!/bin/bash

fichier="$1"

if [[ -z "$fichier" ]]; then
    echo "Erreur : donne un chemin en argument." >&2
    exit 1
fi

if [[ -f "$fichier" ]]; then
    echo "$fichier est un fichier."
elif [[ -d "$fichier" ]]; then
    echo "$fichier est un dossier."
else
    echo "$fichier n'existe pas."
fi
```

Exécutions possibles :

```bash
./check.sh /etc/passwd
./check.sh /home
./check.sh /inexistant
```

---

## Exemple complet : inspecter un fichier ou dossier

```bash
#!/bin/bash

chemin="$1"

if [[ -z "$chemin" ]]; then
    echo "Erreur : vous devez indiquer un chemin" >&2
    exit 1

elif [[ ! -e "$chemin" ]]; then
    echo "Erreur : $chemin n'existe pas" >&2
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

Ce script vérifie :

1. qu’un argument est fourni ;
2. que le chemin existe ;
3. s’il s’agit d’un fichier ;
4. s’il s’agit d’un dossier ;
5. les permissions du fichier si besoin.

---

## Combiner conditions et logique

On combine plusieurs tests avec :

| Opérateur | Signification |
|---|---|
| `&&` | ET : les deux conditions doivent être vraies |
| `\|\|` | OU : au moins une condition doit être vraie |
| `!` | NON : inverse la condition |

---

## Combiner avec `&&`

Exemple :

```bash
age=25
nom="Alice"

if [[ "$age" -ge 18 && "$nom" == "Alice" ]]; then
    echo "Alice est majeure."
fi
```

Ici, il faut que les deux conditions soient vraies :

- `age >= 18` ;
- `nom == Alice`.

---

## Combiner avec `||`

Exemple :

```bash
age=9

if [[ "$age" -lt 10 || "$age" -gt 80 ]]; then
    echo "Âge extrême."
fi
```

Ici, il suffit qu’une des deux conditions soit vraie :

- âge inférieur à 10 ;
- ou âge supérieur à 80.

---

## Inverser avec `!`

Exemple :

```bash
if [[ ! -d "$1" ]]; then
    echo "Ce n'est pas un dossier"
fi
```

Autre exemple :

```bash
if [[ ! -e "$1" ]]; then
    echo "Le chemin n'existe pas"
fi
```

---

## Raccourci : condition puis action

Bash permet d’utiliser des raccourcis :

```bash
test && action_si_vrai
```

Exemple :

```bash
[[ -f "$1" ]] && echo "C'est un fichier"
```

Et :

```bash
test || action_si_faux
```

Exemple :

```bash
[[ -d "$1" ]] || echo "Ce n'est pas un dossier"
```

On peut combiner :

```bash
[[ -r "$chemin" ]] && echo "lisible" || echo "non lisible"
```

!!! warning "Lisibilité"
    Ce raccourci est pratique pour des tests simples.  
    Pour une logique plus complexe, préfère un vrai bloc `if`.

---

## Conditions imbriquées

Une condition imbriquée est un `if` placé dans un autre `if`.

```bash
temperature="$1"

if [[ "$temperature" -gt 0 ]]; then
    if [[ "$temperature" -lt 15 ]]; then
        echo "Il fait frais."
    elif [[ "$temperature" -lt 25 ]]; then
        echo "Il fait bon."
    else
        echo "Il fait chaud."
    fi
else
    echo "Il gèle !"
fi
```

Exemples :

```bash
./temperature.sh -5
./temperature.sh 10
./temperature.sh 20
./temperature.sh 30
```

Résultats possibles :

```txt
Il gèle !
Il fait frais.
Il fait bon.
Il fait chaud.
```

!!! tip "Bonne pratique"
    Au-delà de 2 ou 3 niveaux d’imbrication, essaye de simplifier la logique ou de découper le script en fonctions.

---

## Exemple : vérifier plusieurs chemins

Ce script accepte plusieurs chemins en arguments et indique s’ils sont des fichiers, dossiers ou inexistants.

```bash
#!/bin/bash

if [[ "$#" -eq 0 ]]; then
    echo "Utilisation : $0 <chemin1> [chemin2] ..." >&2
    exit 1
fi

for chemin in "$@"; do
    if [[ -f "$chemin" ]]; then
        taille=$(wc -c < "$chemin")
        echo "✓ $chemin — fichier ($taille octets)"

    elif [[ -d "$chemin" ]]; then
        nb=$(ls "$chemin" | wc -l)
        echo "✓ $chemin — dossier ($nb éléments)"

    else
        echo "✗ $chemin — n'existe pas"
    fi
done
```

Exécution :

```bash
./check_multi.sh /etc/passwd /home /inexistant
```

---

## `[ ]` ou `[[ ]]` ?

Tu peux rencontrer deux syntaxes :

```bash
[ condition ]
```

et :

```bash
[[ condition ]]
```

La syntaxe `[ ]` est plus ancienne et plus fragile.

Exemple risqué :

```bash
[ $nom = "Alice" ]
```

Si `$nom` est vide, Bash peut produire une erreur.

Version plus robuste :

```bash
[[ "$nom" == "Alice" ]]
```

!!! tip "Règle simple"
    Utilise `[[ ]]` dans tes scripts Bash.  
    Si tu vois `[ ]` dans des scripts existants, sache que c’est une ancienne syntaxe encore très courante.

---

## Comparaison `[ ]` vs `[[ ]]`

Ancienne syntaxe :

```bash
if [ "$nom" = "Alice" ]; then
    echo "Bonjour Alice"
fi
```

Syntaxe moderne Bash :

```bash
if [[ "$nom" == "Alice" ]]; then
    echo "Bonjour Alice"
fi
```

`[[ ]]` est généralement préférable car :

- elle est plus robuste ;
- elle gère mieux les chaînes vides ;
- elle permet des motifs comme `[[ "$var" == *"motif"* ]]` ;
- elle évite certaines erreurs de parsing.

---

## Attention : comparer nombres et texte

Dans `[[ ... ]]`, ceci compare comme du texte :

```bash
if [[ "$a" > "$b" ]]; then
    echo "a est supérieur à b"
fi
```

Pour comparer des nombres, utilise :

```bash
if [[ "$a" -gt "$b" ]]; then
    echo "a est supérieur à b"
fi
```

Ou la syntaxe mathématique :

```bash
if (( a > b )); then
    echo "a est supérieur à b"
fi
```

!!! warning "Point important"
    `>` dans `[[ ... ]]` compare des chaînes.  
    `-gt` compare des nombres.  
    `>` dans `(( ... ))` compare des nombres.

---

## Mini-lab 1 : vérifier un âge

Objectif : créer un script `age.sh`.

Contraintes :

1. le script attend un âge en argument ;
2. s’il n’y a pas d’argument, afficher un usage ;
3. si l’âge est inférieur à 13, afficher `enfant` ;
4. inférieur à 18, afficher `adolescent` ;
5. inférieur à 65, afficher `adulte` ;
6. sinon afficher `senior`.

---

### Correction possible

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Usage : $0 <age>" >&2
    exit 1
fi

age="$1"

if [[ "$age" -lt 13 ]]; then
    echo "Enfant"
elif [[ "$age" -lt 18 ]]; then
    echo "Adolescent"
elif [[ "$age" -lt 65 ]]; then
    echo "Adulte"
else
    echo "Senior"
fi
```

---

## Mini-lab 2 : vérifier un chemin

Objectif : créer un script `check_path.sh`.

Contraintes :

1. le script attend un chemin ;
2. s’il n’y a pas d’argument, afficher une erreur ;
3. si le chemin n’existe pas, afficher une erreur ;
4. si c’est un fichier, afficher ses permissions ;
5. si c’est un dossier, afficher le nombre d’éléments.

---

### Correction possible

```bash
#!/bin/bash

chemin="$1"

if [[ -z "$chemin" ]]; then
    echo "Erreur : indiquez un chemin" >&2
    exit 1

elif [[ ! -e "$chemin" ]]; then
    echo "Erreur : $chemin n'existe pas" >&2
    exit 1

elif [[ -f "$chemin" ]]; then
    echo "$chemin est un fichier"

    [[ -r "$chemin" ]] && echo "lisible" || echo "non lisible"
    [[ -w "$chemin" ]] && echo "modifiable" || echo "non modifiable"
    [[ -x "$chemin" ]] && echo "exécutable" || echo "non exécutable"

elif [[ -d "$chemin" ]]; then
    echo "$chemin est un dossier"
    echo "Il contient $(ls "$chemin" | wc -l) éléments"
fi
```

---

## Mini-lab 3 : valider un argument de domaine

Objectif : créer un script qui attend exactement un domaine.

Contraintes :

1. aucun argument → erreur ;
2. un argument → stocker dans `domain` ;
3. plus d’un argument → erreur ;
4. afficher le domaine choisi.

---

### Correction possible

```bash
#!/bin/bash

if [[ "$#" -eq 0 ]]; then
    echo -e "You need to specify the target domain.\n" >&2
    echo -e "Usage:\n\t$0 <domain>"
    exit 1

elif [[ "$#" -eq 1 ]]; then
    domain="$1"
    echo "Domaine choisi : $domain"

else
    echo -e "Too many arguments given." >&2
    exit 1
fi
```

---

## Erreurs classiques

### Oublier `then`

Incorrect :

```bash
if [[ "$a" -gt 5 ]]
    echo "Grand"
fi
```

Correct :

```bash
if [[ "$a" -gt 5 ]]; then
    echo "Grand"
fi
```

---

### Oublier les espaces dans `[[ ]]`

Incorrect :

```bash
if [[$a -gt 5]]; then
    echo "Grand"
fi
```

Correct :

```bash
if [[ "$a" -gt 5 ]]; then
    echo "Grand"
fi
```

---

### Utiliser `>` pour comparer des nombres

Incorrect pour des nombres :

```bash
if [[ "$a" > "$b" ]]; then
    echo "a est supérieur à b"
fi
```

Correct :

```bash
if [[ "$a" -gt "$b" ]]; then
    echo "a est supérieur à b"
fi
```

Ou :

```bash
if (( a > b )); then
    echo "a est supérieur à b"
fi
```

---

### Oublier `fi`

Incorrect :

```bash
if [[ "$a" -gt 5 ]]; then
    echo "Grand"
```

Correct :

```bash
if [[ "$a" -gt 5 ]]; then
    echo "Grand"
fi
```

---

### Ne pas protéger les variables

À éviter :

```bash
if [[ -f $chemin ]]; then
    echo "Fichier"
fi
```

Préférer :

```bash
if [[ -f "$chemin" ]]; then
    echo "Fichier"
fi
```

---

## Cheat sheet

| Test | Signification |
|---|---|
| `[[ -f "f.txt" ]]` | le fichier existe et est un fichier normal |
| `[[ -d "/home" ]]` | le dossier existe |
| `[[ -e "$1" ]]` | le chemin existe |
| `[[ ! -e "$1" ]]` | le chemin n’existe pas |
| `[[ -z "$nom" ]]` | la chaîne est vide |
| `[[ -n "$nom" ]]` | la chaîne n’est pas vide |
| `[[ "$nom" == "Alice" ]]` | le texte est égal à `Alice` |
| `[[ "$nom" != "Bob" ]]` | le texte est différent de `Bob` |
| `[[ "$a" -gt "$b" ]]` | `a` est supérieur à `b` |
| `[[ "$a" -lt "$b" ]]` | `a` est inférieur à `b` |
| `[[ "$a" -eq "$b" ]]` | `a` est égal à `b` |
| `[[ "$a" -ge "$b" ]]` | `a` est supérieur ou égal à `b` |
| `[[ "$a" -le "$b" ]]` | `a` est inférieur ou égal à `b` |
| `[[ condition1 && condition2 ]]` | les deux conditions sont vraies |
| `[[ condition1 \|\| condition2 ]]` | au moins une condition est vraie |
| `[[ ! condition ]]` | inverse la condition |

---

## Checklist script propre

Avant de considérer tes conditions comme propres, vérifie :

- [ ] chaque `if` est fermé par un `fi` ;
- [ ] les espaces autour de `[[` et `]]` sont présents ;
- [ ] `then` est bien présent ;
- [ ] les variables sont entre guillemets ;
- [ ] les nombres sont comparés avec `-eq`, `-gt`, `-lt`, etc. ou `(( ... ))` ;
- [ ] les chaînes sont comparées avec `==`, `!=`, `-z`, `-n` ;
- [ ] les fichiers et dossiers sont testés avec `-e`, `-f`, `-d`, etc. ;
- [ ] les erreurs utilisent `>&2` si nécessaire ;
- [ ] les conditions trop imbriquées sont simplifiées ou découpées en fonctions.

---

## Résumé

Les conditions permettent de contrôler le comportement d’un script.

Structure simple :

```bash
if [[ condition ]]; then
    commandes
fi
```

Structure complète :

```bash
if [[ condition_1 ]]; then
    commandes_1
elif [[ condition_2 ]]; then
    commandes_2
else
    commandes_par_defaut
fi
```

Tests fréquents :

```bash
[[ -z "$1" ]]          # argument vide
[[ -f "$chemin" ]]     # fichier
[[ -d "$chemin" ]]     # dossier
[[ "$age" -ge 18 ]]    # nombre
[[ "$nom" == "Alice" ]] # texte
```

Règle importante :

```bash
[[ ... ]]   # tests généraux
(( ... ))   # tests/calculs numériques
```

---

## Pour aller plus loin

- Fiche précédente : [Opérateurs et calculs](operateurs.md).
- Fiche suivante : [Les boucles](boucles.md).