---
tags:
  - bash
  - scripting
  - bonnes-pratiques
---

# Déboguer et écrire des scripts propres

Un script qui plante n’est pas une fatalité.

Bash fournit plusieurs outils pour comprendre ce qui se passe : vérification de syntaxe, affichage des variables, mode trace, mode verbeux, options strictes et bonnes pratiques d’écriture.

L’objectif n’est pas seulement de corriger les erreurs, mais aussi d’écrire des scripts plus lisibles, plus sûrs et plus faciles à maintenir.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- vérifier la syntaxe d’un script sans l’exécuter ;
- utiliser des `echo` de debug ;
- utiliser `bash -x` pour tracer l’exécution ;
- utiliser `bash -x -v` pour un diagnostic plus verbeux ;
- activer et désactiver la trace dans un script avec `set -x` et `set +x` ;
- comprendre `set -e`, `set -u` et `set -o pipefail` ;
- utiliser `set -euo pipefail` ;
- vérifier les arguments en début de script ;
- envoyer les erreurs vers `stderr` avec `>&2` ;
- reconnaître les erreurs Bash fréquentes ;
- structurer un script avec des fonctions ;
- utiliser un template de départ propre.

---

## À retenir en 30 secondes

Vérifier la syntaxe :

```bash
bash -n script.sh
```

Tracer l’exécution :

```bash
bash -x script.sh
```

Mode très verbeux :

```bash
bash -x -v script.sh
```

Activer la trace dans un script :

```bash
set -x
# commandes tracées
set +x
```

Mode strict :

```bash
set -euo pipefail
```

Afficher une erreur proprement :

```bash
echo "Erreur : argument manquant" >&2
exit 1
```

Toujours protéger les variables :

```bash
cat "$fichier"
```

---

## Vérifier la syntaxe sans exécuter : `bash -n`

La première commande à lancer quand un script ne fonctionne pas :

```bash
bash -n script.sh
```

Cette commande ne lance pas réellement le script.  
Elle vérifie seulement que Bash arrive à le lire correctement.

Elle permet de repérer rapidement :

- un `fi` manquant ;
- un `then` manquant ;
- un `do` manquant ;
- un `done` manquant ;
- une erreur de structure ;
- une mauvaise syntaxe.

Exemple :

```bash
bash -n script_bug.sh
```

Si tout va bien, la commande ne renvoie rien.

!!! tip "Réflexe"
    Lance `bash -n` avant de chercher un bug logique.  
    Si le script n’est même pas lisible par Bash, inutile d’aller plus loin.

---

## Exemple d’erreur détectée par `bash -n`

Script incorrect :

```bash
#!/bin/bash

if [[ -f "$1" ]]; then
    echo "Fichier trouvé"
```

Il manque `fi`.

Commande :

```bash
bash -n script.sh
```

Résultat possible :

```txt
script.sh: line 5: syntax error: unexpected end of file
```

Correction :

```bash
#!/bin/bash

if [[ -f "$1" ]]; then
    echo "Fichier trouvé"
fi
```

---

## Les `echo` de debug

La méthode la plus simple pour comprendre un script consiste à afficher les valeurs importantes en cours d’exécution.

Exemple :

```bash
fichier="$1"

echo "[DEBUG] fichier = '$fichier'"
```

Le préfixe `[DEBUG]` permet de retrouver facilement les messages temporaires.

---

## Exemple avec plusieurs `echo` de debug

```bash
#!/bin/bash

fichier="$1"

echo "[DEBUG] fichier = '$fichier'"

if [[ -f "$fichier" ]]; then
    echo "[DEBUG] Le fichier existe"

    nb_lignes=$(wc -l < "$fichier")
    echo "[DEBUG] nb_lignes = '$nb_lignes'"
fi
```

Ce type de debug permet de vérifier :

- ce que contient une variable ;
- si le script entre dans un `if` ;
- si une commande produit bien le résultat attendu ;
- où le script s’arrête.

!!! note
    Une fois le bug corrigé, supprime les `echo "[DEBUG] ..."` ou transforme-les en vraie fonction de log désactivable.

---

## Le mode trace : `bash -x`

Le mode trace affiche chaque commande avant de l’exécuter, avec les variables remplacées par leurs valeurs.

```bash
bash -x script.sh
```

Exemple :

```bash
bash -x mon_script.sh
```

C’est très utile pour voir précisément :

- quelles commandes sont exécutées ;
- avec quelles valeurs ;
- dans quelle branche `if/else` le script entre ;
- où le script échoue.

---

## Exemple de trace

Script :

```bash
#!/bin/bash

nom="Alice"
echo "Bonjour $nom"
nombre=$((5 + 3))
echo "$nombre"
```

Commande :

```bash
bash -x script.sh
```

Sortie possible :

```txt
+ nom=Alice
+ echo 'Bonjour Alice'
Bonjour Alice
+ nombre=8
+ echo 8
8
```

Les lignes qui commencent par `+` sont les commandes tracées par Bash.

---

## Activer et désactiver la trace dans le script

Tu peux activer la trace seulement sur une partie du script.

```bash
set -x
echo "Ceci sera tracé"
nombre=$((5 + 3))
set +x

echo "Ceci ne sera plus tracé"
```

Sortie possible :

```txt
+ echo 'Ceci sera tracé'
Ceci sera tracé
+ nombre=8
+ set +x
Ceci ne sera plus tracé
```

!!! tip "Cas utile"
    Utilise `set -x` autour d’un bloc précis plutôt que sur tout le script si la sortie devient trop longue.

---

## Mode verbeux : `bash -x -v`

On peut combiner :

- `-x` : affiche les commandes exécutées avec leurs valeurs réelles ;
- `-v` : affiche le code lu par Bash.

Commande :

```bash
bash -x -v script.sh
```

C’est plus bavard, mais utile quand tu veux voir à la fois :

- le code source lu ;
- les commandes réellement exécutées ;
- les substitutions de variables.

---

## Exemple de sortie avec `bash -x -v`

Commande :

```bash
bash -x -v CIDR.sh
```

Sortie typique :

```txt
#!/bin/bash

# Check for given argument
if [ $# -eq 0 ]
then
    echo -e "You need to specify the target domain.\n"
    echo -e "Usage:"
    echo -e "\t$0 <domain>"
    exit 1
else
    domain=$1
fi
+ '[' 0 -eq 0 ']'
+ echo -e 'You need to specify the target domain.\n'
You need to specify the target domain.

+ echo -e Usage:
Usage:
+ echo -e '\tCIDR.sh <domain>'
    CIDR.sh <domain>
+ exit 1
```

Ici, on voit :

- le code lu ;
- le test réellement exécuté ;
- les commandes `echo` ;
- la sortie affichée ;
- le `exit 1`.

---

## Vérifier les arguments en début de script

Un script propre vérifie ses arguments au début.

Exemple :

```bash
if [[ $# -lt 1 ]]; then
    echo "Erreur : argument manquant" >&2
    echo "Utilisation : $0 <fichier>" >&2
    exit 1
fi
```

Pourquoi ?

- le script échoue tôt ;
- l’utilisateur comprend ce qu’il doit fournir ;
- tu évites des erreurs plus loin dans le script ;
- ton code est plus lisible.

---

## Pourquoi utiliser `>&2` ?

`>&2` envoie un message vers `stderr`, la sortie d’erreur.

```bash
echo "Erreur : argument manquant" >&2
```

C’est une bonne pratique pour les messages d’erreur.

Exemple :

```bash
if [[ $# -lt 1 ]]; then
    echo "Erreur : argument manquant" >&2
    echo "Utilisation : $0 <fichier>" >&2
    exit 1
fi
```

Si l’utilisateur redirige la sortie normale :

```bash
./script.sh > resultat.txt
```

Les erreurs restent visibles à l’écran et ne polluent pas `resultat.txt`.

---

## Le mode strict : `set -euo pipefail`

Le mode strict permet de faire échouer le script plus tôt et plus clairement.

```bash
set -euo pipefail
```

Il combine trois protections :

| Option | Effet |
|---|---|
| `set -e` | arrête le script si une commande échoue |
| `set -u` | provoque une erreur si une variable non définie est utilisée |
| `set -o pipefail` | fait échouer un pipeline si une commande du pipeline échoue |

---

## `set -e` : arrêter à la première erreur

Par défaut, Bash peut continuer même si une commande échoue.

Avec `set -e` :

```bash
set -e

echo "Étape 1"
cd /dossier_inexistant
echo "Étape 2"
```

Résultat :

```txt
Étape 1
cd: /dossier_inexistant: No such file or directory
```

`Étape 2` n’est jamais exécutée.

!!! warning
    `set -e` est utile, mais peut surprendre dans certains cas avancés.  
    Pour débuter, retiens surtout qu’il force le script à ne pas continuer après une erreur non gérée.

---

## `set -u` : interdire les variables non définies

Avec `set -u`, Bash échoue si tu utilises une variable qui n’existe pas.

```bash
set -u

echo "Mon nom est $nom"
```

Si `nom` n’est pas défini, tu obtiens une erreur.

Résultat possible :

```txt
nom: unbound variable
```

Cela évite les bugs silencieux causés par des variables mal orthographiées.

---

## `set -o pipefail` : mieux gérer les pipes

Sans `pipefail`, Bash considère souvent le code de retour de la dernière commande du pipeline.

Exemple :

```bash
commande_inexistante | wc -l
```

Le `wc -l` peut réussir même si la première commande a échoué.

Avec :

```bash
set -o pipefail
```

le pipeline échoue si n’importe quelle commande du pipeline échoue.

Exemple de mode strict :

```bash
set -euo pipefail
```

!!! tip "Bonne pratique"
    Pour les scripts sérieux, place souvent ceci en haut :

    ```bash
    set -euo pipefail
    ```

---

## Attention avec `set -u` et les arguments optionnels

Avec `set -u`, utiliser une variable non définie provoque une erreur.

Si tu écris :

```bash
set -u

echo "$1"
```

et que le script est lancé sans argument, Bash peut échouer.

Préférer :

```bash
set -u

if [[ $# -lt 1 ]]; then
    echo "Usage : $0 <argument>" >&2
    exit 1
fi

echo "$1"
```

Ou utiliser une valeur par défaut :

```bash
nom="${1:-Inconnu}"
echo "$nom"
```

---

## Les erreurs les plus fréquentes

| Erreur | Message typique | Solution |
|---|---|---|
| espaces autour de `=` | `command not found` | `var="valeur"` sans espace |
| `then` ou `fi` oublié | `syntax error` | vérifier que chaque `if` a son `then` et son `fi` |
| `do` ou `done` oublié | `syntax error` | vérifier que chaque boucle a son `do` et son `done` |
| guillemets oubliés | `unary operator expected` ou comportement étrange | utiliser `"$var"` |
| `-eq` confondu avec `==` | résultat inattendu | `-eq` pour les nombres, `==` pour le texte |
| variable mal orthographiée | valeur vide ou `unbound variable` | activer `set -u` |
| pipe mal testé | erreur ignorée | utiliser `set -o pipefail` |

---

## Exemples d’erreurs fréquentes

### Espaces autour du `=`

Incorrect :

```bash
nom = "Alice"
```

Résultat possible :

```txt
nom: command not found
```

Correct :

```bash
nom="Alice"
```

---

### `then` oublié

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

### `do` oublié

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

### Guillemets oubliés

À éviter :

```bash
cat $fichier
```

Correct :

```bash
cat "$fichier"
```

Si `fichier="mon document.txt"`, la première version peut casser.

---

## La checklist de débogage

Quand un script ne marche pas, teste dans cet ordre :

```bash
bash -n script.sh
bash script.sh
bash script.sh inexistant.txt
echo "bonjour" > test.txt
bash script.sh test.txt
bash -x script.sh test.txt
```

Lecture :

1. vérifier si Bash arrive à lire le script ;
2. tester le script sans argument ;
3. tester un cas d’erreur évident ;
4. créer un fichier de test ;
5. tester le cas normal ;
6. lancer une trace fine avec `bash -x`.

---

## Questions à se poser quand on débogue

Quand tu débogues un script Bash, demande-toi :

- est-ce que le script **se lit** correctement ? (`bash -n`) ;
- est-ce qu’il **reçoit bien** ses arguments ? ;
- est-ce qu’il **entre dans la bonne branche** `if/else` ? ;
- est-ce que les **variables contiennent bien ce que j’attends** ? ;
- est-ce que la **commande produit bien ce que je crois** ? ;
- est-ce que les erreurs sont correctement gérées ? ;
- est-ce qu’un fichier ou dossier attendu existe vraiment ? ;
- est-ce qu’une variable est vide sans que je m’en rende compte ?

---

## Structurer avec des fonctions

Un script long de 200 lignes devient vite difficile à déboguer.

À éviter :

```bash
# Script monolithique de 200 lignes
# Tout est écrit à la suite
```

Préférer :

```bash
verifier_arguments() {
    # ...
}

traiter_fichier() {
    # ...
}

generer_rapport() {
    # ...
}

verifier_arguments "$@"
traiter_fichier "$1"
generer_rapport
```

Avantages :

- chaque fonction a un rôle clair ;
- le script est plus lisible ;
- le debug est plus simple ;
- le code est plus facile à modifier ;
- tu peux tester une partie du script plus facilement.

---

## Toujours mettre les variables entre guillemets

Mauvais réflexe :

```bash
cat $fichier
```

Bon réflexe :

```bash
cat "$fichier"
```

Pourquoi ?

Si la variable contient un espace :

```bash
fichier="mon document.txt"
```

Alors :

```bash
cat $fichier
```

peut être interprété comme :

```bash
cat mon document.txt
```

au lieu de :

```bash
cat "mon document.txt"
```

!!! tip "Règle simple"
    Dans le doute, mets tes variables entre guillemets :

    ```bash
    "$variable"
    ```

---

## Écrire une fonction de log simple

Pour éviter d’avoir des `echo` partout, tu peux créer une fonction de log.

```bash
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}
```

Utilisation :

```bash
log "Début du script"
log "Traitement du fichier $fichier"
log "Fin du script"
```

Résultat possible :

```txt
[2026-05-25 22:10:00] Début du script
[2026-05-25 22:10:01] Traitement du fichier notes.txt
[2026-05-25 22:10:02] Fin du script
```

---

## Écrire une fonction d’erreur

Tu peux aussi créer une fonction dédiée aux erreurs.

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

Avantage : tous les messages d’erreur ont le même format et partent vers `stderr`.

---

## Template de départ propre

Tu peux utiliser ce modèle pour démarrer un script Bash sérieux.

```bash
#!/bin/bash
# =============================================================
# Nom         : mon_script.sh
# Description : [ce que fait le script]
# Utilisation : ./mon_script.sh [options] <arguments>
# =============================================================

set -euo pipefail

afficher_aide() {
    echo "Utilisation : $0 [options] <argument>"
    echo "  -h, --help   Afficher cette aide"
}

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

erreur() {
    echo "Erreur : $1" >&2
}

if [[ $# -lt 1 ]]; then
    afficher_aide
    exit 1
fi

case "$1" in
    -h|--help)
        afficher_aide
        exit 0
        ;;
esac

log "Début du script"

# ...
# Ton code ici
# ...

log "Fin du script"
```

---

## Template avec fonction de validation

```bash
#!/bin/bash
set -euo pipefail

afficher_aide() {
    echo "Utilisation : $0 <fichier>"
}

erreur() {
    echo "Erreur : $1" >&2
}

verifier_arguments() {
    if [[ $# -lt 1 ]]; then
        erreur "argument manquant"
        afficher_aide
        exit 1
    fi

    if [[ ! -f "$1" ]]; then
        erreur "$1 n'est pas un fichier valide"
        exit 1
    fi
}

traiter_fichier() {
    local fichier="$1"
    local nb_lignes

    nb_lignes=$(wc -l < "$fichier")
    echo "Nombre de lignes : $nb_lignes"
}

verifier_arguments "$@"
traiter_fichier "$1"
```

---

## Mini-lab 1 : déboguer un script

Voici un script volontairement incorrect :

```bash
#!/bin/bash

nom = "Alice"

if [[ "$nom" == "Alice" ]]
    echo "Bonjour Alice"
fi
```

Objectif :

1. lancer `bash -n script.sh` ;
2. repérer les erreurs ;
3. corriger le script ;
4. relancer le script.

---

### Correction possible

```bash
#!/bin/bash

nom="Alice"

if [[ "$nom" == "Alice" ]]; then
    echo "Bonjour Alice"
fi
```

---

## Mini-lab 2 : ajouter des traces

Objectif : créer un script qui prend un fichier en argument, puis ajouter des traces de debug.

Script :

```bash
#!/bin/bash

fichier="$1"

echo "[DEBUG] fichier = '$fichier'"

if [[ -f "$fichier" ]]; then
    echo "[DEBUG] Le fichier existe"
    nb_lignes=$(wc -l < "$fichier")
    echo "[DEBUG] nb_lignes = '$nb_lignes'"
else
    echo "Erreur : fichier introuvable" >&2
    exit 1
fi
```

Tests :

```bash
bash -n script.sh
bash -x script.sh notes.txt
```

---

## Mini-lab 3 : écrire un script propre

Objectif : écrire un script `count_lines.sh`.

Contraintes :

1. activer `set -euo pipefail` ;
2. vérifier qu’un argument est fourni ;
3. vérifier que l’argument est un fichier ;
4. compter le nombre de lignes ;
5. envoyer les erreurs vers `stderr`.

---

### Correction possible

```bash
#!/bin/bash
set -euo pipefail

erreur() {
    echo "Erreur : $1" >&2
}

if [[ $# -lt 1 ]]; then
    erreur "argument manquant"
    echo "Utilisation : $0 <fichier>" >&2
    exit 1
fi

fichier="$1"

if [[ ! -f "$fichier" ]]; then
    erreur "$fichier n'est pas un fichier valide"
    exit 1
fi

nb_lignes=$(wc -l < "$fichier")

echo "Nombre de lignes : $nb_lignes"
```

---

## Cheat sheet

| Commande / syntaxe | Effet |
|---|---|
| `bash -n script.sh` | vérifie la syntaxe sans exécuter |
| `bash script.sh` | exécute le script normalement |
| `bash -x script.sh` | trace chaque commande exécutée |
| `bash -x -v script.sh` | affiche le code lu et les commandes exécutées |
| `set -x` | active la trace dans le script |
| `set +x` | désactive la trace |
| `set -e` | arrête le script à la première erreur |
| `set -u` | erreur si variable non définie |
| `set -o pipefail` | un pipeline échoue si une commande du pipe échoue |
| `set -euo pipefail` | mode strict courant |
| `echo "Erreur" >&2` | envoie un message vers `stderr` |
| `"$var"` | protège une variable |
| `bash -n && bash -x` | combo classique de debug |

---

## Checklist script propre

Avant de considérer ton script comme propre, vérifie :

- [ ] `bash -n script.sh` ne renvoie pas d’erreur ;
- [ ] les arguments sont vérifiés au début ;
- [ ] les erreurs sont envoyées vers `stderr` ;
- [ ] les variables sont entre guillemets ;
- [ ] les variables internes aux fonctions utilisent `local` ;
- [ ] les commandes critiques sont testées ;
- [ ] les fichiers/dossiers sont vérifiés avant usage ;
- [ ] le script a un message d’aide ou d’usage ;
- [ ] les fonctions ont des noms clairs ;
- [ ] les `echo "[DEBUG] ..."` temporaires ont été retirés ;
- [ ] le script a été testé avec un cas normal et un cas d’erreur.

---

## Résumé

Pour déboguer un script Bash, commence par :

```bash
bash -n script.sh
```

Puis teste l’exécution normale :

```bash
bash script.sh
```

Puis trace si besoin :

```bash
bash -x script.sh
```

Pour des scripts plus robustes :

```bash
set -euo pipefail
```

Pour les erreurs :

```bash
echo "Erreur : message" >&2
exit 1
```

Le réflexe essentiel :

> Vérifie d’abord la syntaxe, puis les arguments, puis les branches `if/else`, puis les variables, puis les commandes.

---

## Pour aller plus loin

- Fiche précédente : [Les tableaux](tableaux.md).
- Fiche suivante : [Automatisation : cron et cas pratiques](automatisation.md).