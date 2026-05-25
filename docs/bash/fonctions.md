---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Fonctions et case

Une fonction est un bloc de code réutilisable auquel on donne un nom.  
`case`, lui, permet de choisir quoi faire selon une valeur.

Ensemble, ils servent à structurer des scripts plus propres : `case` aiguille vers une action, et les fonctions exécutent les blocs de traitement.

## Objectifs

À la fin de cette fiche, tu dois savoir :

- définir et appeler une fonction ;
- passer des arguments à une fonction ;
- comprendre la différence entre les arguments du script et ceux d’une fonction ;
- récupérer le résultat d’une fonction ;
- utiliser `local` pour éviter d’écraser des variables globales ;
- utiliser `return` et les codes de retour ;
- structurer un script avec `case` ;
- créer un menu simple avec `select` ;
- traiter des options avec `shift`.

## À retenir en 30 secondes

Une fonction Bash sert à regrouper un bloc de commandes pour pouvoir le réutiliser.

```bash
saluer() {
    echo "Bonjour, $1 !"
}

saluer "Alice"
```

`case` sert à choisir une action selon une valeur :

```bash
case "$1" in
    -h) afficher_aide ;;
    -d) date ;;
    *) echo "Option inconnue" ;;
esac
```

Le duo classique dans un script propre :

```bash
aide() {
    echo "Utilisation : $0 [option]"
}

case "$1" in
    -h) aide ;;
    *) echo "Option inconnue" ;;
esac
```

---

## Pourquoi utiliser des fonctions ?

Quand un script grandit, il devient vite difficile à lire si toutes les commandes sont écrites les unes après les autres.

Les fonctions permettent de :

- éviter de répéter le même code ;
- donner un nom clair à une action ;
- découper un script en blocs logiques ;
- faciliter le débogage ;
- rendre le script plus maintenable.

Exemple simple :

```bash
afficher_aide() {
    echo "Usage : $0 <option>"
}
```

Même si cette fonction n’est appelée qu’une seule fois, son nom explique déjà ce que fait le bloc.

!!! tip "Bonne pratique"
    Une fonction doit idéalement faire une seule chose clairement identifiable.

---

## Définir et appeler une fonction

La syntaxe la plus courante est :

```bash
nom_de_la_fonction() {
    commandes
}
```

Exemple :

```bash
saluer() {
    echo "Salut, bienvenue !"
}

saluer
```

Résultat :

```txt
Salut, bienvenue !
```

On appelle une fonction par son nom, **sans parenthèses**.

```bash
saluer
```

et non :

```bash
saluer()
```

!!! warning "Erreur fréquente"
    Les parenthèses servent à définir la fonction, pas à l’appeler.

---

## Règle d’ordre

En Bash, la fonction doit être définie **avant** son appel.

Correct :

```bash
saluer() {
    echo "Salut !"
}

saluer
```

Incorrect :

```bash
saluer

saluer() {
    echo "Salut !"
}
```

Dans le second cas, Bash ne connaît pas encore la fonction au moment où tu l’appelles.

---

## Variante avec le mot-clé `function`

Il existe aussi une autre syntaxe :

```bash
function print_pars {
    echo "$1" "$2" "$3"
}

one="First parameter"
two="Second parameter"
three="Third parameter"

print_pars "$one" "$two" "$three"
```

Résultat :

```txt
First parameter Second parameter Third parameter
```

Les deux syntaxes fonctionnent :

```bash
ma_fonction() {
    echo "Hello"
}
```

ou :

```bash
function ma_fonction {
    echo "Hello"
}
```

!!! tip "Recommandation"
    Pour rester simple et lisible, utilise plutôt la syntaxe :

    ```bash
    ma_fonction() {
        commandes
    }
    ```

---

## Passer des arguments à une fonction

À l’intérieur d’une fonction, `$1`, `$2`, `$3` représentent les arguments donnés à la fonction.

```bash
saluer() {
    echo "Bonjour, $1 ! Tu as $2 ans."
}

saluer "Alice" 25
saluer "Bob" 30
```

Résultat :

```txt
Bonjour, Alice ! Tu as 25 ans.
Bonjour, Bob ! Tu as 30 ans.
```

Ici :

| Élément | Valeur lors du premier appel |
|---|---|
| `$1` | `Alice` |
| `$2` | `25` |

---

## Pourquoi passer des arguments ?

L’intérêt est d’éviter les fonctions figées avec des valeurs écrites en dur.

Moins flexible :

```bash
afficher_fichier() {
    cat notes.txt
}
```

Plus flexible :

```bash
afficher_fichier() {
    cat "$1"
}

afficher_fichier notes.txt
afficher_fichier rapport.txt
```

La même fonction peut maintenant afficher plusieurs fichiers différents.

!!! tip "Réflexe"
    Quand une valeur peut changer, passe-la en argument à la fonction.

---

## Arguments du script vs arguments de la fonction

Un script peut recevoir ses propres arguments.  
Une fonction peut aussi recevoir ses propres arguments.

Il ne faut pas les confondre.

```bash
#!/bin/bash

echo "Argument du script : $1"

ma_fonction() {
    echo "Argument de la fonction : $1"
}

ma_fonction "test"
```

Si tu lances :

```bash
./script.sh bonjour
```

Résultat :

```txt
Argument du script : bonjour
Argument de la fonction : test
```

Dans le corps principal du script :

```bash
$1 = bonjour
```

Dans la fonction :

```bash
$1 = test
```

!!! warning "Point important"
    `$1` ne désigne pas toujours la même chose.  
    Dans le script, c’est le premier argument du script.  
    Dans une fonction, c’est le premier argument donné à cette fonction.

---

## Compter les arguments d’une fonction

Comme dans un script, `$#` indique le nombre d’arguments reçus par la fonction.

```bash
compter_arguments() {
    echo "Nombre d'arguments reçus : $#"
}

compter_arguments
compter_arguments "un" "deux"
```

Résultat :

```txt
Nombre d'arguments reçus : 0
Nombre d'arguments reçus : 2
```

Tu peux t’en servir pour vérifier qu’une fonction reçoit bien ce dont elle a besoin.

```bash
afficher_fichier() {
    if [[ $# -lt 1 ]]; then
        echo "Erreur : aucun fichier fourni."
        return 1
    fi

    cat "$1"
}
```

---

## Récupérer le résultat d’une fonction

En Bash, une fonction ne “renvoie” pas directement une chaîne de caractères comme dans Python.

Le plus souvent, elle affiche un résultat avec `echo`, puis on récupère cette sortie avec une substitution de commande :

```bash
resultat=$(ma_fonction)
```

Exemple :

```bash
addition() {
    echo $(( $1 + $2 ))
}

resultat=$(addition 15 27)
echo "La somme est : $resultat"
```

Résultat :

```txt
La somme est : 42
```

Ce qui se passe :

1. la fonction calcule `15 + 27` ;
2. elle affiche `42` avec `echo` ;
3. `$(...)` récupère cette sortie ;
4. la variable `resultat` reçoit `42`.

---

## Exemple avec des arguments du script

```bash
#!/bin/bash

addition() {
    echo $(( $1 + $2 ))
}

resultat=$(addition "$1" "$2")
echo "La somme est : $resultat"
```

Exécution :

```bash
./addition.sh 3 4
```

Résultat :

```txt
La somme est : 7
```

!!! warning "Attention"
    Si tu veux récupérer une vraie donnée, utilise souvent `echo` + `$(...)`.  
    `return` sert plutôt à renvoyer un code de succès ou d’erreur.

---

## Variables dans les fonctions : globales vs locales

Par défaut, une variable créée ou modifiée dans une fonction peut impacter le reste du script.

Exemple :

```bash
#!/bin/bash

nom="Global"

modifier() {
    nom="Local"
    echo "Dans la fonction : $nom"
}

echo "Avant : $nom"
modifier
echo "Après : $nom"
```

Résultat :

```txt
Avant : Global
Dans la fonction : Local
Après : Local
```

La fonction a modifié la variable globale `nom`.

---

## Utiliser `local`

Pour éviter ce problème, on utilise `local`.

```bash
#!/bin/bash

nom="Global"

modifier() {
    local nom="Local"
    echo "Dans la fonction : $nom"
}

echo "Avant : $nom"
modifier
echo "Après : $nom"
```

Résultat :

```txt
Avant : Global
Dans la fonction : Local
Après : Global
```

Cette fois, la variable `nom` dans la fonction reste confinée à la fonction.

!!! tip "Bonne pratique"
    Déclare en `local` toutes les variables internes à tes fonctions.

---

## Exemple complet : local vs global

```bash
#!/bin/bash

nom="Global"

modifier() {
    local nom="Local"
    echo "Dans la fonction : $nom"
}

echo "Avant : $nom"
modifier
echo "Après : $nom"

remodifier() {
    nom="Modif"
    echo "Dans la fonction test : $nom"
}

echo "Avant : $nom"
remodifier
echo "Après : $nom"
```

Résultat possible :

```txt
Avant : Global
Dans la fonction : Local
Après : Global
Avant : Global
Dans la fonction test : Modif
Après : Modif
```

La première fonction utilise `local`, donc elle ne modifie pas la variable globale.  
La seconde n’utilise pas `local`, donc elle écrase la variable globale.

---

## Valeur par défaut pour un argument

La syntaxe suivante permet d’utiliser une valeur par défaut si aucun argument n’est fourni :

```bash
${1:-"valeur"}
```

Exemple :

```bash
saluer() {
    local nom=${1:-"Inconnu"}
    echo "Bonjour, $nom !"
}

saluer "Alice"
saluer
```

Résultat :

```txt
Bonjour, Alice !
Bonjour, Inconnu !
```

Ici :

- si `$1` existe, on l’utilise ;
- sinon, on utilise `"Inconnu"`.

!!! tip "Cas utile"
    Les valeurs par défaut sont très pratiques pour rendre tes scripts plus tolérants.

---

## Code de retour d’une fonction : `return`

Une fonction peut renvoyer un code de retour avec `return`.

Convention classique :

| Code | Sens |
|---:|---|
| `0` | Succès |
| autre que `0` | Erreur |

Exemple :

```bash
fichier_existe() {
    if [[ -f "$1" ]]; then
        return 0
    else
        return 1
    fi
}

if fichier_existe "/etc/passwd"; then
    echo "Le fichier existe."
else
    echo "Le fichier n'existe pas."
fi
```

Ici, la fonction ne renvoie pas du texte.  
Elle renvoie un statut : succès ou échec.

---

## Lire le code de retour avec `$?`

La variable `$?` contient le code de retour de la dernière commande exécutée.

```bash
given_args() {
    if [[ $# -lt 1 ]]; then
        echo "Number of arguments: $#"
        return 1
    else
        echo "Number of arguments: $#"
        return 0
    fi
}

given_args
echo "Function status code: $?"

given_args "argument"
echo "Function status code: $?"
```

Résultat :

```txt
Number of arguments: 0
Function status code: 1
Number of arguments: 1
Function status code: 0
```

---

## Capturer la sortie d’une fonction dans une variable

On peut aussi capturer ce qu’une fonction affiche :

```bash
given_args() {
    if [[ $# -lt 1 ]]; then
        echo "Number of arguments: $#"
        return 1
    else
        echo "Number of arguments: $#"
        return 0
    fi
}

content=$(given_args "argument")

echo -e "Content of the variable:\n\t$content"
```

Résultat :

```txt
Content of the variable:
    Number of arguments: 1
```

!!! warning "Différence importante"
    `return` transmet un code de retour.  
    `echo` transmet du texte récupérable avec `$(...)`.

---

## Codes de retour courants

Quelques codes utiles à connaître :

| Code | Description |
|---:|---|
| `0` | Succès |
| `1` | Erreur générale |
| `2` | Mauvaise utilisation d’une commande shell |
| `126` | Commande trouvée mais impossible à exécuter |
| `127` | Commande introuvable |
| `128` | Argument invalide passé à `exit` |
| `128+n` | Erreur fatale liée au signal `n` |
| `130` | Script interrompu avec `Ctrl+C` |
| `255` | Code de sortie hors limites |

!!! note
    Dans un script simple, retiens surtout :
    
    - `0` = succès ;
    - `1` = erreur générale ;
    - `127` = commande introuvable ;
    - `130` = interruption avec `Ctrl+C`.

---

## L’aiguillage `case`

`case` compare une valeur à plusieurs motifs.

Il est souvent plus lisible qu’une longue chaîne de `if / elif / else` quand on teste une même variable.

Syntaxe générale :

```bash
case <expression> in
    motif_1) commandes ;;
    motif_2) commandes ;;
    motif_3) commandes ;;
    *) commandes par défaut ;;
esac
```

Exemple simple :

```bash
case "$1" in
    oui)
        echo "Tu as répondu oui."
        ;;
    non)
        echo "Tu as répondu non."
        ;;
    *)
        echo "Réponse inconnue."
        ;;
esac
```

!!! note "Le motif `*`"
    Le motif `*` correspond au cas par défaut.  
    Il est exécuté si aucun autre motif ne correspond.

---

## Exemple simple avec des options

```bash
#!/bin/bash

case "$1" in
    -l)
        ls -la
        ;;
    -d)
        date
        ;;
    -u)
        whoami
        ;;
    *)
        echo "Option inconnue."
        ;;
esac
```

Exécutions possibles :

```bash
./outil.sh -l
./outil.sh -d
./outil.sh -u
./outil.sh -z
```

---

## `case` avec plusieurs motifs

Tu peux accepter plusieurs valeurs pour le même bloc avec `|`.

```bash
case "$1" in
    -h|--help)
        echo "Affichage de l'aide."
        ;;
    -v|--version)
        echo "Version 1.0"
        ;;
    *)
        echo "Option inconnue."
        ;;
esac
```

Ici, `-h` et `--help` déclenchent la même action.

---

## Le duo gagnant : `case` + fonctions

Dans un script propre :

- `case` choisit l’action ;
- les fonctions contiennent les actions.

Exemple :

```bash
#!/bin/bash

afficher_aide() {
    echo "Utilisation : $0 [option]"
    echo "  -l    Lister les fichiers"
    echo "  -d    Afficher la date"
    echo "  -u    Afficher l'utilisateur courant"
    echo "  -h    Afficher cette aide"
}

lister_fichiers() {
    ls -la
}

afficher_date() {
    date
}

afficher_utilisateur() {
    whoami
}

case "$1" in
    -l)
        lister_fichiers
        ;;
    -d)
        afficher_date
        ;;
    -u)
        afficher_utilisateur
        ;;
    -h|--help)
        afficher_aide
        ;;
    "")
        echo "Erreur : aucune option fournie."
        afficher_aide
        exit 1
        ;;
    *)
        echo "Option '$1' inconnue."
        afficher_aide
        exit 1
        ;;
esac
```

Exemples d’utilisation :

```bash
./outil.sh -l
./outil.sh -d
./outil.sh -u
./outil.sh -h
./outil.sh -z
```

---

## Exemple plus court : aide, date, utilisateur

```bash
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

## Exemple cyber encadré : menu après une reconnaissance autorisée

Dans un contexte de lab, de CTF ou d’audit autorisé, tu peux utiliser `case` pour proposer plusieurs actions.

```bash
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case "$opt" in
    "1")
        network_range
        ;;
    "2")
        ping_host
        ;;
    "3")
        network_range && ping_host
        ;;
    "*")
        exit 0
        ;;
esac
```

!!! warning "Cadre légal"
    Ce type de script doit être utilisé uniquement sur un périmètre autorisé : lab personnel, CTF, machine de test ou audit validé.

---

## Exemple de fonction pour une plage réseau

Exemple pédagogique inspiré d’un workflow de reconnaissance :

```bash
network_range() {
    for ip in $ipaddr; do
        netrange=$(whois "$ip" | grep "NetRange\|CIDR" | tee -a CIDR.txt)
        cidr=$(whois "$ip" | grep "CIDR" | awk '{print $2}')
        echo -e "\nNetRange for $ip:"
        echo -e "$netrange"
        echo -e "CIDR: $cidr"
    done
}
```

Cette fonction :

1. parcourt une liste d’adresses IP ;
2. interroge `whois` ;
3. extrait les champs `NetRange` ou `CIDR` ;
4. sauvegarde certains résultats dans `CIDR.txt`.

!!! note
    L’objectif ici est surtout de comprendre la structure :
    
    - une fonction contient une action ;
    - une boucle traite plusieurs éléments ;
    - `case` choisit quelle fonction lancer.

---

## Menu interactif avec `select`

`select` génère automatiquement un menu numéroté.

Syntaxe simple :

```bash
echo "Que veux-tu faire ?"

select choix in "Date" "Utilisateur" "Quitter"; do
    echo "Tu as choisi : $choix"
done
```

Résultat :

```txt
1) Date
2) Utilisateur
3) Quitter
#?
```

---

## Exemple complet avec `select` et `case`

```bash
#!/bin/bash

echo "Que veux-tu faire ?"

select choix in "Lister les fichiers" "Afficher la date" "Quitter"; do
    case "$choix" in
        "Lister les fichiers")
            ls
            ;;
        "Afficher la date")
            date
            ;;
        "Quitter")
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

- `select` affiche le menu ;
- l’utilisateur choisit un numéro ;
- `case` exécute l’action correspondante ;
- `break` permet de sortir de la boucle.

---

## Traiter des options avec `shift`

`shift` sert à supprimer le premier argument, puis à décaler tous les autres.

L’idée :

1. je regarde `$1` ;
2. je décide quoi en faire ;
3. je fais `shift` ;
4. l’argument suivant devient `$1`.

Exemple simple :

```bash
while [[ $# -gt 0 ]]; do
    echo "Argument courant : $1"
    shift
done
```

Si tu lances :

```bash
./script.sh un deux trois
```

Le script affiche :

```txt
Argument courant : un
Argument courant : deux
Argument courant : trois
```

---

## Comprendre `shift`

Au départ :

```txt
$1 = un
$2 = deux
$3 = trois
```

Après un premier `shift` :

```txt
$1 = deux
$2 = trois
```

Après un deuxième `shift` :

```txt
$1 = trois
```

Après un troisième `shift`, il ne reste plus rien.

---

## Utiliser `shift 2`

Quand une option prend une valeur, il faut consommer deux éléments :

```bash
-n 42
```

Ici :

```txt
$1 = -n
$2 = 42
```

Si on veut consommer le flag et sa valeur, on utilise :

```bash
shift 2
```

---

## Exemple : parser des options avec `shift 2`

```bash
#!/bin/bash

while [[ $# -gt 0 ]]; do
    case "$1" in
        -n)
            nombre="$2"
            shift 2
            ;;
        -s)
            texte="$2"
            shift 2
            ;;
        -h|--help)
            echo "Aide : $0 -n <nombre> -s <texte>"
            exit 0
            ;;
        *)
            echo "Option inconnue : $1"
            exit 1
            ;;
    esac
done

echo "Nombre : $nombre"
echo "Texte : $texte"
```

Exécution :

```bash
./script.sh -n 42 -s "bonjour"
```

Résultat :

```txt
Nombre : 42
Texte : bonjour
```

---

## Ce qui se passe concrètement avec `shift 2`

Commande lancée :

```bash
./script.sh -u alice -p secret
```

Au départ :

```txt
$1 = -u
$2 = alice
$3 = -p
$4 = secret
```

Si le script fait :

```bash
shift 2
```

Alors il reste :

```txt
$1 = -p
$2 = secret
```

Tu as donc consommé :

```txt
-u
alice
```

!!! tip "Règle simple"
    Utilise `shift` quand tu consommes un argument seul.  
    Utilise `shift 2` quand tu consommes une option et sa valeur.

---

## Mini-lab : créer un petit outil Bash

Objectif : créer un script `outil.sh` capable de gérer plusieurs options.

Fonctionnalités attendues :

| Option | Action |
|---|---|
| `-h` ou `--help` | affiche l’aide |
| `-l` | liste les fichiers |
| `-d` | affiche la date |
| `-u` | affiche l’utilisateur |
| `-f <fichier>` | affiche le nombre de lignes d’un fichier |

---

### Correction possible

```bash
#!/bin/bash

afficher_aide() {
    echo "Utilisation : $0 [options]"
    echo
    echo "Options :"
    echo "  -h, --help       Afficher l'aide"
    echo "  -l               Lister les fichiers"
    echo "  -d               Afficher la date"
    echo "  -u               Afficher l'utilisateur courant"
    echo "  -f <fichier>     Compter les lignes d'un fichier"
}

lister_fichiers() {
    ls -la
}

afficher_date() {
    date
}

afficher_utilisateur() {
    whoami
}

compter_lignes() {
    local fichier="$1"

    if [[ ! -f "$fichier" ]]; then
        echo "Erreur : fichier introuvable : $fichier"
        return 1
    fi

    wc -l "$fichier"
}

if [[ $# -eq 0 ]]; then
    echo "Erreur : aucune option fournie."
    afficher_aide
    exit 1
fi

while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help)
            afficher_aide
            exit 0
            ;;
        -l)
            lister_fichiers
            shift
            ;;
        -d)
            afficher_date
            shift
            ;;
        -u)
            afficher_utilisateur
            shift
            ;;
        -f)
            if [[ -z "$2" ]]; then
                echo "Erreur : l'option -f attend un fichier."
                exit 1
            fi

            compter_lignes "$2"
            shift 2
            ;;
        *)
            echo "Option inconnue : $1"
            afficher_aide
            exit 1
            ;;
    esac
done
```

Rendre le script exécutable :

```bash
chmod +x outil.sh
```

Tests :

```bash
./outil.sh -h
./outil.sh -l
./outil.sh -d
./outil.sh -u
./outil.sh -f /etc/passwd
```

---

## Erreurs fréquentes

### Appeler une fonction avant sa définition

```bash
saluer

saluer() {
    echo "Salut"
}
```

Mauvais ordre : Bash ne connaît pas encore la fonction.

---

### Oublier les guillemets

À éviter :

```bash
cat $1
```

Préférer :

```bash
cat "$1"
```

Pourquoi ?  
Si le chemin contient un espace, la première version peut casser.

---

### Confondre `echo` et `return`

Mauvais réflexe :

```bash
addition() {
    return $(( $1 + $2 ))
}
```

À éviter pour renvoyer une vraie donnée.

Meilleur réflexe :

```bash
addition() {
    echo $(( $1 + $2 ))
}

resultat=$(addition 3 4)
```

---

### Oublier `local`

```bash
nom="Global"

changer_nom() {
    nom="Local"
}
```

Cette fonction modifie la variable globale.  
Préférer :

```bash
changer_nom() {
    local nom="Local"
}
```

---

### Oublier le `;;` dans `case`

Incorrect :

```bash
case "$1" in
    -h) echo "Aide"
    -d) date ;;
esac
```

Correct :

```bash
case "$1" in
    -h) echo "Aide" ;;
    -d) date ;;
esac
```

---

## Mini-modèles à retenir

### Modèle 1 — fonction simple

```bash
saluer() {
    echo "Bonjour !"
}

saluer
```

---

### Modèle 2 — fonction avec argument

```bash
saluer() {
    echo "Bonjour, $1 !"
}

saluer "Alice"
```

---

### Modèle 3 — fonction avec plusieurs arguments

```bash
print_pars() {
    echo "$1" "$2" "$3"
}

print_pars "un" "deux" "trois"
```

---

### Modèle 4 — fonction qui “renvoie” un résultat

```bash
addition() {
    echo $(( $1 + $2 ))
}

resultat=$(addition 3 4)
echo "$resultat"
```

---

### Modèle 5 — fonction avec variable locale

```bash
saluer() {
    local nom=${1:-"Inconnu"}
    echo "Bonjour, $nom !"
}
```

---

### Modèle 6 — fonction avec code de retour

```bash
fichier_existe() {
    [[ -f "$1" ]]
}

if fichier_existe "/etc/passwd"; then
    echo "OK"
else
    echo "KO"
fi
```

---

### Modèle 7 — `case` simple

```bash
case "$1" in
    oui)
        echo "Oui"
        ;;
    non)
        echo "Non"
        ;;
    *)
        echo "Inconnu"
        ;;
esac
```

---

### Modèle 8 — `case` + fonction

```bash
aide() {
    echo "Utilisation : $0 [option]"
}

case "$1" in
    -h)
        aide
        ;;
    *)
        echo "Option inconnue"
        ;;
esac
```

---

### Modèle 9 — boucle `while` + `shift`

```bash
while [[ $# -gt 0 ]]; do
    echo "Argument courant : $1"
    shift
done
```

---

### Modèle 10 — option avec valeur

```bash
while [[ $# -gt 0 ]]; do
    case "$1" in
        -u)
            utilisateur="$2"
            shift 2
            ;;
        *)
            echo "Option inconnue : $1"
            exit 1
            ;;
    esac
done
```

---

## Checklist script propre

Avant de considérer ton script comme correct, vérifie :

- [ ] les fonctions sont définies avant leur appel ;
- [ ] les variables internes aux fonctions utilisent `local` ;
- [ ] les variables sont entourées de guillemets ;
- [ ] les erreurs sont gérées avec un message clair ;
- [ ] les options inconnues sont traitées ;
- [ ] l’aide est accessible avec `-h` ou `--help`;
- [ ] les fonctions renvoient un code de succès ou d’erreur cohérent ;
- [ ] les résultats textuels sont renvoyés avec `echo` si besoin ;
- [ ] le script a été testé avec et sans arguments.

---

## Résumé

Les fonctions permettent de structurer ton script.  
`case` permet de choisir une action.  
`select` permet de créer un menu interactif.  
`shift` permet de traiter les arguments un par un.

Le modèle le plus propre à retenir :

```bash
#!/bin/bash

aide() {
    echo "Utilisation : $0 [option]"
}

action() {
    echo "Action exécutée"
}

case "$1" in
    -h|--help)
        aide
        ;;
    -a)
        action
        ;;
    *)
        echo "Option inconnue"
        aide
        exit 1
        ;;
esac
```

---

## Pour aller plus loin

- Fiche précédente : [Les boucles](boucles.md).
- Fiche suivante : [Manipuler les chaînes](chaines.md).