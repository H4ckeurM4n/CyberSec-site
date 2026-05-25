---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Arguments et variables spéciales

Un **argument** est une information donnée à un script directement au moment de son lancement.

Contrairement à `read`, qui demande une saisie pendant l’exécution, les arguments sont transmis dès le départ.

```bash
./saluer.sh Alice
#            ↑ argument
```

Ici, `Alice` est un argument donné au script `saluer.sh`.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- comprendre ce qu’est un argument ;
- utiliser les paramètres positionnels `$0`, `$1`, `$2`, etc. ;
- connaître les variables spéciales utiles en Bash ;
- vérifier qu’un argument a bien été fourni ;
- afficher un message d’usage propre ;
- rediriger les erreurs vers la sortie d’erreur avec `>&2` ;
- utiliser `$#`, `$@`, `$*`, `$?` et `$$` ;
- comprendre la différence entre `"$@"` et `"$*"`;
- traiter les arguments un par un avec `shift`.

---

## À retenir en 30 secondes

Un argument est une valeur donnée au script au lancement :

```bash
./script.sh pomme banane cerise
```

Bash les range automatiquement :

```txt
$0 = ./script.sh
$1 = pomme
$2 = banane
$3 = cerise
```

Variables importantes :

```bash
echo "$0"    # nom du script
echo "$1"    # premier argument
echo "$#"    # nombre d'arguments
echo "$@"    # tous les arguments séparément
echo "$?"    # code retour de la dernière commande
echo "$$"    # PID du script
```

Vérifier qu’un argument existe :

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant" >&2
    exit 1
fi
```

---

## Argument vs saisie utilisateur

Il y a deux façons courantes de récupérer une information dans un script Bash.

Avec `read`, on demande l’information pendant l’exécution :

```bash
read -r -p "Ton prénom : " prenom
echo "Bonjour $prenom"
```

Avec un argument, on donne l’information au lancement :

```bash
./saluer.sh Alice
```

Dans le script :

```bash
#!/bin/bash

echo "Bonjour $1"
```

Résultat :

```txt
Bonjour Alice
```

!!! note "Différence simple"
    `read` demande une saisie pendant que le script tourne.  
    Les arguments sont donnés dès le lancement du script.

---

## Les paramètres positionnels

Chaque argument est stocké automatiquement dans une variable numérotée selon sa position.

```bash
./mon_script.sh pomme banane cerise
```

Correspondance :

```txt
$0 = ./mon_script.sh
$1 = pomme
$2 = banane
$3 = cerise
```

| Paramètre | Contenu |
|---|---|
| `$0` | nom du script |
| `$1` | premier argument |
| `$2` | deuxième argument |
| `$3` | troisième argument |
| `$n` | argument en position `n` |

---

## Exemple minimal : saluer une personne

Créer un fichier `saluer.sh` :

```bash
#!/bin/bash

echo "Bonjour, $1 !"
```

Rendre le script exécutable :

```bash
chmod +x saluer.sh
```

Exécuter :

```bash
./saluer.sh Alice
./saluer.sh Bob
```

Résultat :

```txt
Bonjour, Alice !
Bonjour, Bob !
```

---

## `$0` : le nom du script

La variable `$0` contient le nom utilisé pour lancer le script.

```bash
#!/bin/bash

echo "Nom du script : $0"
```

Exécution :

```bash
./infos.sh
```

Résultat :

```txt
Nom du script : ./infos.sh
```

!!! note
    `$0` est souvent utilisé dans les messages d’aide pour montrer comment utiliser le script.

Exemple :

```bash
echo "Usage : $0 <prenom>"
```

---

## `$1`, `$2`, `$3` : les arguments par position

Exemple :

```bash
#!/bin/bash

echo "Premier argument : $1"
echo "Deuxième argument : $2"
echo "Troisième argument : $3"
```

Exécution :

```bash
./script.sh pomme banane cerise
```

Résultat :

```txt
Premier argument : pomme
Deuxième argument : banane
Troisième argument : cerise
```

---

## Les variables spéciales utiles

Bash fournit plusieurs variables automatiques très utiles.

| Variable | Contenu | Description |
|---|---|---|
| `$0` | nom du script | contient le nom utilisé pour lancer le script |
| `$1`, `$2`, etc. | arguments par position | permet d’accéder aux arguments individuellement |
| `$#` | nombre d’arguments | indique combien d’arguments ont été passés |
| `$@` | tous les arguments | conserve les arguments séparément |
| `$*` | tous les arguments | fusionne les arguments en un seul bloc selon le contexte |
| `$?` | code de sortie | contient le code retour de la dernière commande |
| `$$` | PID du script | identifiant du processus en cours |
| `$n` | argument à la position `n` | récupère l’argument selon sa position |

!!! note "IFS"
    Les variables comme `$@` et `$*` sont liées à la façon dont Bash sépare les arguments.  
    Cette séparation dépend notamment de l’`IFS` (*Internal Field Separator*), c’est-à-dire le séparateur interne utilisé par le shell.

---

## Exemple complet avec variables spéciales

Créer un fichier `infos.sh` :

```bash
#!/bin/bash

echo "Nom du script : $0"
echo "Nombre d'arguments : $#"
echo "Tous les arguments : $@"
echo "Premier argument : $1"
echo "Deuxième argument : $2"
```

Exécution :

```bash
./infos.sh pomme banane cerise
```

Résultat :

```txt
Nom du script : ./infos.sh
Nombre d'arguments : 3
Tous les arguments : pomme banane cerise
Premier argument : pomme
Deuxième argument : banane
```

---

## `$#` : compter le nombre d’arguments

La variable `$#` contient le nombre total d’arguments donnés au script.

```bash
#!/bin/bash

echo "Nombre d'arguments : $#"
```

Exécution :

```bash
./script.sh un deux trois
```

Résultat :

```txt
Nombre d'arguments : 3
```

C’est très utile pour vérifier qu’un utilisateur a bien fourni les informations attendues.

---

## Vérifier qu’un argument a été fourni

Un script qui attend un argument devrait toujours vérifier que cet argument existe.

L’opérateur `-z` teste si une chaîne est vide.

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : donne un argument !" >&2
    echo -e "Usage :\n\t$0 <prenom>"
    exit 1
fi

echo "Bonjour $1 !"
```

Explication :

| Élément | Rôle |
|---|---|
| `[[ ... ]]` | test de condition |
| `-z "$1"` | teste si `$1` est vide |
| `>&2` | envoie le message vers la sortie d’erreur |
| `echo -e` | interprète `\n` et `\t` |
| `exit 1` | quitte le script avec une erreur |

---

## Pourquoi utiliser `>&2` ?

Par convention :

- la sortie normale va vers `stdout` ;
- les erreurs vont vers `stderr`.

Pour envoyer un message d’erreur vers `stderr`, on utilise :

```bash
echo "Erreur : argument manquant" >&2
```

Exemple :

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : veuillez renseigner votre prénom" >&2
    echo -e "Usage :\n\t$0 <prenom>"
    exit 1
fi
```

!!! tip "Bonne pratique"
    Les messages d’erreur devraient être envoyés vers `stderr` avec `>&2`.

---

## Afficher un message d’usage propre

Quand un argument manque, il faut expliquer comment utiliser le script.

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : veuillez renseigner votre prénom" >&2
    echo -e "Usage :\n\t$0 <prenom>"
    exit 1
fi

echo "Bonjour, $1 !"
```

Résultat si aucun argument n’est fourni :

```txt
Erreur : veuillez renseigner votre prénom
Usage :
    ./script.sh <prenom>
```

---

## `echo -e`, `\n` et `\t`

Avec `echo -e`, certaines séquences sont interprétées.

| Séquence | Effet |
|---|---|
| `\n` | retour à la ligne |
| `\t` | tabulation |

Exemple :

```bash
echo -e "Usage :\n\t$0 <fichier>"
```

Résultat :

```txt
Usage :
    ./script.sh <fichier>
```

!!! warning
    Sans `-e`, selon les environnements, `\n` et `\t` peuvent être affichés littéralement.

---

## Vérifier qu’un dossier existe

Si un script attend un dossier, il ne suffit pas de vérifier que `$1` existe.  
Il faut aussi vérifier que `$1` est bien un dossier.

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

echo "$(date) — $1 — $nbr_fichiers fichiers"
```

Explication :

| Test | Signification |
|---|---|
| `[[ -z "$1" ]]` | `$1` est vide |
| `[[ -d "$1" ]]` | `$1` est un dossier |
| `[[ ! -d "$1" ]]` | `$1` n’est pas un dossier |

---

## Exemple : générer un rapport sur un dossier

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : indiquez un dossier" >&2
    echo -e "Usage :\n\t$0 <dossier>"
    exit 1
fi

if [[ ! -d "$1" ]]; then
    echo "Erreur : $1 n'est pas un dossier valide" >&2
    exit 1
fi

nbr_fichiers=$(ls "$1" | wc -l)

echo "$(date) — $1 — $nbr_fichiers fichiers" > rapport.txt
cat rapport.txt
```

Exécution :

```bash
./rapport.sh /etc
```

Résultat possible :

```txt
Mon May 25 22:00:00 CEST 2026 — /etc — 210 fichiers
```

---

## Vérifier plusieurs arguments

Si ton script attend deux arguments, il faut vérifier les deux.

Exemple : comparer la taille de deux fichiers.

```bash
#!/bin/bash

if [[ -z "$1" || -z "$2" ]]; then
    echo "Erreur : le script attend <fichier1> et <fichier2>" >&2
    echo -e "Usage :\n\t$0 <fichier1> <fichier2>"
    exit 1
fi

echo "Comparaison de $1 et $2"

echo "Taille de $1 :"
wc -c < "$1"

echo "Taille de $2 :"
wc -c < "$2"
```

!!! warning "Guillemets obligatoires"
    Utilise `"$1"` et `"$2"` pour éviter les erreurs si les noms de fichiers contiennent des espaces.

---

## Vérifier le nombre d’arguments avec `$#`

Une autre façon propre consiste à vérifier directement le nombre d’arguments.

```bash
#!/bin/bash

if [[ "$#" -ne 2 ]]; then
    echo "Erreur : ce script attend exactement 2 arguments" >&2
    echo -e "Usage :\n\t$0 <fichier1> <fichier2>"
    exit 1
fi

echo "Fichier 1 : $1"
echo "Fichier 2 : $2"
```

| Test | Signification |
|---|---|
| `[[ "$#" -eq 0 ]]` | aucun argument |
| `[[ "$#" -lt 1 ]]` | moins d’un argument |
| `[[ "$#" -ne 2 ]]` | nombre d’arguments différent de 2 |
| `[[ "$#" -gt 3 ]]` | plus de 3 arguments |

---

## Exemple : script qui attend un domaine

Exemple classique dans un script de reconnaissance autorisée :

```bash
#!/bin/bash

if [[ "$#" -eq 0 ]]; then
    echo -e "You need to specify the target domain.\n" >&2
    echo -e "Usage:"
    echo -e "\t$0 <domain>"
    exit 1
else
    domain="$1"
fi

echo "Domaine choisi : $domain"
```

!!! warning "Cadre légal"
    Les scripts de reconnaissance doivent être utilisés uniquement sur tes domaines, un lab, un CTF ou un périmètre explicitement autorisé.

---

## Exemple : rechercher un fichier par son nom

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

Explication :

```bash
find / -iname "$1" 2>/dev/null
```

- `find /` cherche depuis la racine ;
- `-iname "$1"` cherche sans tenir compte de la casse ;
- `2>/dev/null` masque les erreurs de permission.

---

## `$?` : lire le code de retour de la dernière commande

Chaque commande renvoie un code de sortie.

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

Autre exemple :

```bash
ls /dossier_inexistant
echo "$?"
```

Résultat possible :

```txt
2
```

---

## Utiliser `$?` dans un script

```bash
#!/bin/bash

ls "$1"

if [[ "$?" -eq 0 ]]; then
    echo "Commande réussie."
else
    echo "Commande échouée."
fi
```

Cette forme fonctionne, mais il y a souvent plus propre :

```bash
#!/bin/bash

if ls "$1"; then
    echo "Commande réussie."
else
    echo "Commande échouée."
fi
```

!!! tip "Bonne pratique"
    Quand c’est possible, utilise directement la commande dans le `if`.  
    C’est plus lisible que de tester `$?` juste après.

---

## `$@` : tous les arguments séparément

`$@` représente tous les arguments du script.

La forme recommandée est :

```bash
"$@"
```

Elle conserve chaque argument séparément, même si certains contiennent des espaces.

Exemple :

```bash
#!/bin/bash

for arg in "$@"; do
    echo "Argument : $arg"
done
```

Exécution :

```bash
./test.sh "Jean Pierre" Marie
```

Résultat :

```txt
Argument : Jean Pierre
Argument : Marie
```

Il y a bien deux arguments :

1. `Jean Pierre` ;
2. `Marie`.

---

## `$*` : tous les arguments en un seul bloc

`$*` représente aussi tous les arguments, mais son comportement est différent quand il est entre guillemets.

```bash
#!/bin/bash

for arg in "$*"; do
    echo "Argument : $arg"
done
```

Exécution :

```bash
./test.sh "Jean Pierre" Marie
```

Résultat :

```txt
Argument : Jean Pierre Marie
```

Ici, tout est traité comme un seul bloc.

---

## `"$@"` ou `"$*"` ?

Cas de test :

```bash
./test.sh "Jean Pierre" Marie
```

Avec `"$@"` :

```txt
Jean Pierre
Marie
```

Avec `"$*"` :

```txt
Jean Pierre Marie
```

| Forme | Comportement |
|---|---|
| `"$@"` | garde chaque argument séparé |
| `"$*"` | fusionne les arguments en un seul bloc |
| `$@` sans guillemets | dangereux si espaces |
| `$*` sans guillemets | dangereux si espaces |

!!! tip "Règle simple"
    Dans presque tous les cas, utilise `"$@"`.

---

## Démonstration complète `$@` vs `$*`

```bash
#!/bin/bash

echo 'Avec "$@" :'
for arg in "$@"; do
    echo "[$arg]"
done

echo

echo 'Avec "$*" :'
for arg in "$*"; do
    echo "[$arg]"
done
```

Exécution :

```bash
./demo.sh "Jean Pierre" Marie
```

Résultat :

```txt
Avec "$@" :
[Jean Pierre]
[Marie]

Avec "$*" :
[Jean Pierre Marie]
```

---

## `$$` : le PID du script

`$$` contient le PID, c’est-à-dire l’identifiant du processus du script en cours.

```bash
#!/bin/bash

echo "Mon PID est : $$"
```

Résultat possible :

```txt
Mon PID est : 1401
```

Si tu relances le script, le PID change :

```txt
Mon PID est : 1402
```

---

## Utiliser `$$` pour créer un fichier temporaire

Si un script crée un fichier temporaire avec un nom fixe, il peut y avoir un conflit.

Mauvais exemple :

```bash
fichier_temp="/tmp/mon_fichier.tmp"
```

Si le script est lancé deux fois en même temps :

- le premier lancement écrit dans `/tmp/mon_fichier.tmp` ;
- le second écrit dans le même fichier ;
- les données peuvent se mélanger ou être écrasées.

Avec `$$` :

```bash
fichier_temp="/tmp/mon_fichier_$$.tmp"
```

Exemples :

```txt
/tmp/mon_fichier_1401.tmp
/tmp/mon_fichier_1402.tmp
```

!!! note
    `$$` donne un identifiant pratique et simple.  
    Pour des scripts plus robustes en production, on préférera souvent `mktemp`.

---

## Exemple avec fichier temporaire

```bash
#!/bin/bash

fichier_temp="/tmp/rapport_$$.tmp"

echo "Début du rapport" > "$fichier_temp"
echo "Date : $(date)" >> "$fichier_temp"
echo "Utilisateur : $USER" >> "$fichier_temp"

cat "$fichier_temp"

rm -f "$fichier_temp"
```

Ce script :

1. crée un fichier temporaire unique ;
2. écrit quelques informations dedans ;
3. affiche son contenu ;
4. supprime le fichier temporaire.

---

## `shift` : traiter les arguments un par un

`shift` décale tous les arguments d’une position.

Après un `shift` :

```txt
$2 devient $1
$3 devient $2
$4 devient $3
```

L’ancien `$1` disparaît.

---

## Exemple visuel avec `shift`

Créer `shift-demo.sh` :

```bash
#!/bin/bash

echo "Avant shift :"
echo "\$1 = $1"
echo "\$2 = $2"
echo "\$3 = $3"

shift

echo "Après shift :"
echo "\$1 = $1"
echo "\$2 = $2"
echo "\$3 = $3"
```

Exécution :

```bash
./shift-demo.sh alpha beta gamma
```

Résultat :

```txt
Avant shift :
$1 = alpha
$2 = beta
$3 = gamma
Après shift :
$1 = beta
$2 = gamma
$3 =
```

---

## Parcourir tous les arguments avec `shift`

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

Explication :

| Élément | Rôle |
|---|---|
| `$#` | nombre d’arguments restants |
| `-gt 0` | supérieur à 0 |
| `$1` | argument courant |
| `shift` | passe à l’argument suivant |

---

## Variante avec `[[ -n "$1" ]]`

Tu peux aussi voir cette forme :

```bash
#!/bin/bash

while [[ -n "$1" ]]; do
    echo "Argument courant : $1"
    shift
done
```

Explication :

| Test | Sens |
|---|---|
| `-n "$1"` | `$1` n’est pas vide |
| `-z "$1"` | `$1` est vide |

Les deux approches existent, mais `[[ $# -gt 0 ]]` est souvent plus explicite quand on travaille sur les arguments.

---

## `shift 2` : consommer une option et sa valeur

Quand une option attend une valeur, on consomme souvent deux éléments.

Exemple :

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

Si on fait :

```bash
shift 2
```

Alors il reste :

```txt
$1 = -p
$2 = secret
```

On a consommé :

```txt
-u
alice
```

---

## Exemple : parser des options simples

```bash
#!/bin/bash

while [[ $# -gt 0 ]]; do
    case "$1" in
        -u)
            utilisateur="$2"
            shift 2
            ;;
        -p)
            motdepasse="$2"
            shift 2
            ;;
        -h|--help)
            echo "Usage : $0 -u <utilisateur> -p <motdepasse>"
            exit 0
            ;;
        *)
            echo "Option inconnue : $1" >&2
            exit 1
            ;;
    esac
done

echo "Utilisateur : $utilisateur"
echo "Mot de passe fourni : oui"
```

!!! warning "Attention"
    Cet exemple sert à comprendre `shift 2`.  
    Évite de passer de vrais mots de passe en argument : ils peuvent apparaître dans l’historique ou dans la liste des processus.

---

## Exemple plus propre : option avec valeur obligatoire

```bash
#!/bin/bash

while [[ $# -gt 0 ]]; do
    case "$1" in
        -f|--file)
            if [[ -z "$2" ]]; then
                echo "Erreur : l'option $1 attend un fichier" >&2
                exit 1
            fi

            fichier="$2"
            shift 2
            ;;
        -h|--help)
            echo "Usage : $0 -f <fichier>"
            exit 0
            ;;
        *)
            echo "Option inconnue : $1" >&2
            exit 1
            ;;
    esac
done

if [[ -z "$fichier" ]]; then
    echo "Erreur : aucun fichier fourni" >&2
    echo "Usage : $0 -f <fichier>"
    exit 1
fi

echo "Fichier choisi : $fichier"
```

---

## Mini-lab 1 : script `saluer.sh`

Objectif : créer un script qui attend un prénom en argument.

Contraintes :

- si aucun prénom n’est fourni, afficher une erreur ;
- afficher un usage clair ;
- quitter avec `exit 1` en cas d’erreur ;
- sinon afficher `Bonjour, <prenom> !`.

---

### Correction possible

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : veuillez renseigner votre prénom" >&2
    echo -e "Usage :\n\t$0 <prenom>"
    exit 1
fi

echo "Bonjour, $1 !"
```

Tests :

```bash
./saluer.sh
./saluer.sh Alice
```

---

## Mini-lab 2 : script `rapport_dossier.sh`

Objectif : créer un script qui prend un dossier en argument et génère un mini-rapport.

Contraintes :

- vérifier qu’un argument est fourni ;
- vérifier que l’argument est bien un dossier ;
- compter le nombre d’éléments dans le dossier ;
- afficher la date, le dossier et le nombre d’éléments.

---

### Correction possible

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : indiquez un dossier" >&2
    echo -e "Usage :\n\t$0 <dossier>"
    exit 1
fi

if [[ ! -d "$1" ]]; then
    echo "Erreur : $1 n'est pas un dossier valide" >&2
    exit 1
fi

nbr_elements=$(ls "$1" | wc -l)

echo "Date : $(date)"
echo "Dossier : $1"
echo "Nombre d'éléments : $nbr_elements"
```

Test :

```bash
./rapport_dossier.sh /etc
```

---

## Mini-lab 3 : script `compare.sh`

Objectif : comparer la taille de deux fichiers.

Contraintes :

- le script attend exactement deux arguments ;
- chaque argument doit être un fichier ;
- afficher la taille de chaque fichier en octets.

---

### Correction possible

```bash
#!/bin/bash

if [[ "$#" -ne 2 ]]; then
    echo "Erreur : ce script attend exactement deux fichiers" >&2
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

echo "Taille de $1 :"
wc -c < "$1"

echo "Taille de $2 :"
wc -c < "$2"
```

---

## Erreurs classiques

### Ne pas vérifier si l’argument existe

```bash
echo "Bonjour, $1"
```

Si le script est lancé sans argument :

```txt
Bonjour,
```

Le script ne plante pas, mais le comportement est incomplet et silencieux.

---

### Ne pas mettre de guillemets

Incorrect :

```bash
cat $1
```

Si le fichier s’appelle `mon document.txt`, Bash peut le découper en deux morceaux.

Correct :

```bash
cat "$1"
```

---

### Confondre `$@` et `$*`

À préférer :

```bash
for arg in "$@"; do
    echo "$arg"
done
```

À éviter dans la majorité des cas :

```bash
for arg in "$*"; do
    echo "$arg"
done
```

---

### Oublier `exit 1` après une erreur

Incorrect :

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant"
fi

echo "Traitement de $1"
```

Le script continue même si l’argument est absent.

Correct :

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant" >&2
    exit 1
fi

echo "Traitement de $1"
```

---

## Cheat sheet

| Variable / syntaxe | Contenu / effet |
|---|---|
| `$0` | nom du script |
| `$1`, `$2`, `$3` | arguments passés au script dans l’ordre |
| `$#` | nombre d’arguments |
| `"$@"` | tous les arguments séparément, à préférer |
| `"$*"` | tous les arguments fusionnés en un seul bloc |
| `$?` | code de sortie de la dernière commande |
| `$$` | PID du script |
| `shift` | décale les arguments, consomme `$1` |
| `shift 2` | consomme une option et sa valeur |
| `[[ -z "$1" ]]` | teste si `$1` est vide |
| `[[ -n "$1" ]]` | teste si `$1` n’est pas vide |
| `[[ "$#" -eq 0 ]]` | teste s’il n’y a aucun argument |
| `[[ "$#" -ne 2 ]]` | teste si le nombre d’arguments est différent de 2 |
| `>&2` | redirige un message vers la sortie d’erreur |
| `echo -e` | interprète `\n` et `\t` |

---

## Checklist script propre

Avant de considérer ton script comme correct, vérifie :

- [ ] les arguments obligatoires sont vérifiés ;
- [ ] les messages d’erreur sont clairs ;
- [ ] les erreurs utilisent `>&2` ;
- [ ] le script affiche un `Usage` compréhensible ;
- [ ] le script quitte avec `exit 1` en cas d’erreur ;
- [ ] les variables comme `$1`, `$2`, `$@` sont entre guillemets ;
- [ ] les fichiers ou dossiers sont testés avec `-f` ou `-d` ;
- [ ] `"$@"` est préféré à `"$*"` pour parcourir les arguments ;
- [ ] `shift` est utilisé proprement si les arguments sont traités un par un.

---

## Résumé

Un argument est une information donnée au lancement du script :

```bash
./script.sh Alice
```

Les paramètres positionnels permettent de les récupérer :

```bash
$0 = nom du script
$1 = premier argument
$2 = deuxième argument
```

Un script robuste vérifie toujours ses arguments :

```bash
if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant" >&2
    echo -e "Usage :\n\t$0 <argument>"
    exit 1
fi
```

Pour parcourir plusieurs arguments :

```bash
for arg in "$@"; do
    echo "$arg"
done
```

ou :

```bash
while [[ $# -gt 0 ]]; do
    echo "$1"
    shift
done
```

La règle à retenir :

```bash
"$@" > "$*"
```

dans la majorité des cas.

---

## Pour aller plus loin

- Fiche précédente : [Variables, affichage et saisie](variables.md).
- Fiche suivante : [Redirections, erreurs et pipes](redirections.md).