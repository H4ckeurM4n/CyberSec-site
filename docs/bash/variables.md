---
tags:
  - bash
  - scripting
  - débutant
---

# Variables, affichage et saisie

Une variable est un conteneur avec une étiquette : un **nom** et une **valeur**.

En Bash, les variables sont indispensables pour stocker une information, la réutiliser plus tard, demander une saisie à l’utilisateur, récupérer le résultat d’une commande ou automatiser un script.

On peut se représenter une variable comme ceci :

```txt
prenom = Alice
```

- `prenom` est le nom de la variable ;
- `Alice` est la valeur stockée dedans.

!!! note "Idée simple"
    Une variable sert à éviter d’écrire une valeur en dur partout dans ton script.  
    Tu stockes une information une fois, puis tu la réutilises avec son nom.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- créer une variable ;
- afficher le contenu d’une variable ;
- comprendre pourquoi il ne faut pas mettre d’espace autour du `=` ;
- utiliser `$variable` et `${variable}` ;
- modifier une variable ;
- comprendre la différence entre guillemets simples et doubles ;
- demander une saisie avec `read` ;
- utiliser les options utiles de `read` ;
- stocker le résultat d’une commande avec `$(commande)` ;
- utiliser les variables d’environnement ;
- créer une variable en lecture seule avec `readonly`.

---

## À retenir en 30 secondes

Créer une variable :

```bash
prenom="Alice"
```

Afficher une variable :

```bash
echo "$prenom"
```

Demander une saisie :

```bash
read -p "Ton prénom : " prenom
```

Stocker le résultat d’une commande :

```bash
machine=$(hostname)
```

Afficher une variable d’environnement :

```bash
echo "$USER"
```

Règle critique :

```bash
prenom="Alice"     # correct
prenom = "Alice"   # incorrect
```

---

## Créer une variable

Pour créer une variable en Bash :

```bash
prenom="Alice"
```

Puis pour l’afficher :

```bash
echo "Bonjour, $prenom !"
```

Script complet :

```bash
#!/bin/bash

prenom="Alice"
echo "Bonjour, $prenom !"
```

Résultat :

```txt
Bonjour, Alice !
```

---

## La règle critique : pas d’espace autour du `=`

En Bash, il ne faut **jamais** mettre d’espace autour du `=` lors de l’affectation d’une variable.

Correct :

```bash
prenom="Alice"
```

Incorrect :

```bash
prenom = "Alice"
```

Pourquoi ?  
Parce que Bash interprète `prenom` comme une commande, puis `=` et `"Alice"` comme des arguments.

!!! danger "Règle à ne jamais oublier"
    `prenom="Alice"` est une affectation de variable.  
    `prenom = "Alice"` fait croire à Bash que `prenom` est une commande.

---

## Afficher le contenu d’une variable

Pour accéder au contenu d’une variable, on met un `$` devant son nom.

```bash
prenom="Alice"

echo "Bonjour, $prenom"
echo "Bonjour, prenom"
```

Résultat :

```txt
Bonjour, Alice
Bonjour, prenom
```

Dans le premier cas, Bash remplace `$prenom` par sa valeur.  
Dans le second cas, Bash affiche le mot `prenom` tel quel.

---

## Pourquoi mettre les variables entre guillemets ?

Même si ceci fonctionne souvent :

```bash
echo $prenom
```

il vaut mieux écrire :

```bash
echo "$prenom"
```

Pourquoi ?  
Parce que si la variable contient des espaces, Bash peut découper la valeur en plusieurs morceaux.

Exemple :

```bash
nom="Alice Dupont"

echo $nom
echo "$nom"
```

La bonne pratique est donc :

```bash
echo "$variable"
```

!!! tip "Bonne pratique"
    Entoure presque toujours tes variables avec des guillemets doubles : `"$var"`.  
    C’est plus robuste, surtout si la valeur contient des espaces.

---

## Modifier une variable

Une variable peut être modifiée après sa création.

```bash
#!/bin/bash

humeur="content"
echo "Je suis $humeur"

humeur="fatigué"
echo "Maintenant je suis $humeur"
```

Résultat :

```txt
Je suis content
Maintenant je suis fatigué
```

Ici, la variable `humeur` contient d’abord `content`, puis elle est remplacée par `fatigué`.

---

## Nommage des variables

En Bash, il est préférable d’utiliser :

- des lettres ;
- des chiffres ;
- des underscores `_` ;
- pas d’espace ;
- pas de tiret `-`.

Exemples corrects :

```bash
prenom="Alice"
nom_utilisateur="alice"
ip_cible="192.168.1.10"
nb_fichiers=12
```

Exemples à éviter :

```bash
nom utilisateur="Alice"   # espace interdit
nom-utilisateur="Alice"   # tiret problématique
1nom="Alice"              # commence par un chiffre
```

!!! tip "Convention simple"
    Pour tes propres variables, utilise souvent le style `snake_case` :

    ```bash
    nom_utilisateur="alice"
    fichier_sortie="resultats.txt"
    ip_cible="192.168.1.10"
    ```

---

## La forme `${variable}` pour éviter les ambiguïtés

Parfois, Bash ne sait pas où s’arrête le nom d’une variable.

Exemple :

```bash
animal="chat"

echo "J'ai 3 ${animal}s"
echo "J'ai 3 $animals"
```

Résultat :

```txt
J'ai 3 chats
J'ai 3
```

Dans la deuxième ligne, Bash cherche une variable nommée `animals`, qui n’existe pas.

La bonne forme est donc :

```bash
echo "J'ai 3 ${animal}s"
```

!!! note "À retenir"
    `${variable}` permet de délimiter clairement le nom de la variable.

---

## Guillemets simples ou doubles ?

Les guillemets doubles permettent à Bash d’interpréter les variables.

```bash
prenom="Alice"

echo "Bonjour, $prenom"
```

Résultat :

```txt
Bonjour, Alice
```

Les guillemets simples affichent tout littéralement.

```bash
prenom="Alice"

echo 'Bonjour, $prenom'
```

Résultat :

```txt
Bonjour, $prenom
```

---

## Comparaison simple

```bash
prenom="Alice"

echo "Bonjour, $prenom"
echo 'Bonjour, $prenom'
```

Résultat :

```txt
Bonjour, Alice
Bonjour, $prenom
```

| Syntaxe | Effet |
|---|---|
| `"Bonjour, $prenom"` | la variable est interprétée |
| `'Bonjour, $prenom'` | tout est affiché tel quel |

!!! note "À retenir"
    Guillemets doubles `" "` → Bash interprète les variables.  
    Guillemets simples `' '` → tout est littéral.

---

## Demander une saisie avec `read`

La commande `read` demande à l’utilisateur de taper quelque chose, puis stocke la réponse dans une variable.

```bash
#!/bin/bash

read -p "Ton prénom : " prenom
read -p "Ton âge : " age

echo "Tu es $prenom et tu as $age ans."
```

Exemple d’exécution :

```txt
Ton prénom : Alice
Ton âge : 25
Tu es Alice et tu as 25 ans.
```

L’option `-p` permet d’afficher un message sur la même ligne que la saisie.

---

## Exemple simple avec `read`

```bash
#!/bin/bash

read -p "Comment vous appelez-vous ? " name
echo "Bonjour $name"
```

Résultat :

```txt
Comment vous appelez-vous ? Cam
Bonjour Cam
```

---

## Saisie invisible avec `read -s`

Pour demander une information sensible, comme un mot de passe, on peut utiliser `-s`.

```bash
read -s -p "Mot de passe : " motdepasse
echo
echo "Mot de passe saisi."
```

L’option `-s` masque la saisie à l’écran.

!!! warning "Attention"
    Ne stocke pas de vrais mots de passe en clair dans tes scripts.  
    Cet exemple sert seulement à comprendre le fonctionnement de `read -s`.

---

## Timeout avec `read -t`

L’option `-t` permet de limiter le temps de saisie.

```bash
read -t 5 -p "Choix rapide : " choix
echo "Choix : $choix"
```

Ici, l’utilisateur a 5 secondes pour répondre.

Exemple plus propre :

```bash
if read -t 5 -p "Choix rapide : " choix; then
    echo "Tu as choisi : $choix"
else
    echo
    echo "Temps écoulé."
fi
```

---

## Options utiles de `read`

| Option | Rôle |
|---|---|
| `-p` | affiche un message avant la saisie |
| `-s` | masque la saisie |
| `-t` | définit un timeout |
| `-r` | évite l’interprétation des backslashes |

Exemple recommandé quand tu lis une saisie texte simple :

```bash
read -r -p "Ton texte : " texte
```

!!! tip "Bonne pratique"
    Utilise souvent `read -r` pour éviter que Bash interprète certains caractères spéciaux comme `\`.

---

## Exemple cyber : faciliter un scan Nmap

Dans un lab ou un environnement autorisé, tu peux demander une IP à scanner.

```bash
#!/bin/bash

read -r -p "L'adresse à scanner : " ip_scan

echo "Scan de $ip_scan en cours..."
nmap -F -sV "$ip_scan"
```

Ce script :

1. demande une adresse IP ;
2. la stocke dans `ip_scan` ;
3. lance un scan rapide avec Nmap.

!!! warning "Cadre légal"
    N’utilise ce type de script que sur tes propres machines, un lab, un CTF ou un périmètre explicitement autorisé.

---

## Stocker le résultat d’une commande : `$(commande)`

La substitution de commande permet d’exécuter une commande et de récupérer sa sortie.

Syntaxe :

```bash
variable=$(commande)
```

Exemple :

```bash
aujourdhui=$(date)
echo "Nous sommes le : $aujourdhui"
```

Résultat possible :

```txt
Nous sommes le : Mon May 25 21:30:00 CEST 2026
```

---

## Utiliser `$(commande)` directement dans un texte

On peut aussi utiliser `$(commande)` directement dans un `echo`.

```bash
echo "Bienvenue sur cette machine : $(hostname) !"
echo "La date du jour : $(date)"
```

Résultat possible :

```txt
Bienvenue sur cette machine : kali !
La date du jour : Mon May 25 21:30:00 CEST 2026
```

---

## Stocker plusieurs résultats de commandes

```bash
#!/bin/bash

aujourdhui=$(date)
utilisateur=$(whoami)
machine=$(hostname)
nb_fichiers=$(ls | wc -l)

echo "Machine : $machine"
echo "Utilisateur : $utilisateur"
echo "Nombre d'éléments dans le dossier : $nb_fichiers"
echo "Date : $aujourdhui"
```

---

## Exemple : compter les fichiers dans un dossier

```bash
nb_fichiers=$(ls | wc -l)
echo "Il y a $nb_fichiers éléments dans ce dossier"
```

Ce qui se passe :

1. `ls` liste les éléments du dossier ;
2. `wc -l` compte le nombre de lignes ;
3. `$(...)` récupère le résultat ;
4. la variable `nb_fichiers` stocke ce résultat.

---

## Exemple : récupérer les 20 derniers caractères d’une variable

```bash
var="Ceci est une longue chaîne de caractères utilisée pour un test."

last_20=$(echo "$var" | tail -c 20)

echo "$last_20"
```

Ici :

- `echo "$var"` affiche le contenu de la variable ;
- `tail -c 20` garde les 20 derniers caractères ;
- `last_20=$(...)` stocke le résultat.

---

## Exemple : mesurer la longueur d’une variable

```bash
var="Bonjour tout le monde"
longueur=$(echo "$var" | wc -m)

echo "Longueur : $longueur"
```

Tu peux l’utiliser dans une condition :

```bash
counter=35
var="Bonjour tout le monde"

if [[ "$counter" -eq 35 ]]; then
    longueur=$(echo "$var" | wc -m)
    echo "Longueur de la variable : $longueur"
fi
```

!!! note
    `wc -m` compte les caractères.  
    `wc -c` compte les octets.

---

## Ancienne syntaxe avec backticks

Tu peux parfois voir cette ancienne forme :

```bash
date_du_jour=`date`
```

Elle fonctionne, mais elle est moins lisible.

Préférer :

```bash
date_du_jour=$(date)
```

!!! tip "Bonne pratique"
    Utilise `$(commande)` plutôt que les backticks.  
    C’est plus clair, plus moderne et plus facile à imbriquer.

---

## Variables d’environnement

Le système définit déjà des variables utiles.

Pour les afficher toutes :

```bash
env
```

Quelques variables courantes :

```bash
echo "Utilisateur : $USER"
echo "Dossier perso : $HOME"
echo "Shell actuel : $SHELL"
echo "Dossier actuel : $PWD"
```

Résultat possible :

```txt
Utilisateur : cam
Dossier perso : /home/cam
Shell actuel : /bin/bash
Dossier actuel : /home/cam/scripts
```

---

## Variables d’environnement utiles

| Variable | Signification |
|---|---|
| `$USER` | utilisateur courant |
| `$HOME` | dossier personnel |
| `$SHELL` | shell utilisé |
| `$PWD` | dossier courant |
| `$PATH` | chemins où Bash cherche les commandes |
| `$LANG` | langue/locale du système |

Exemple :

```bash
echo "$PATH"
```

Résultat possible :

```txt
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

---

## Créer une variable d’environnement avec `export`

Une variable classique n’est disponible que dans le shell courant.

```bash
nom="Alice"
```

Pour transmettre une variable aux commandes ou scripts enfants, on utilise `export`.

```bash
export NOM="Alice"
```

Exemple :

```bash
export LAB_NAME="cyber-lab"
echo "$LAB_NAME"
```

!!! note
    `export` sera approfondi plus tard avec les notions de shell, environnement et scripts enfants.

---

## Variables en lecture seule : `readonly`

Pour qu’une variable ne puisse plus être modifiée, on utilise `readonly`.

```bash
readonly TOKEN="valeur_fixe"
```

Si tu essaies de la modifier :

```bash
TOKEN="autre"
```

Bash renvoie une erreur.

Exemple :

```bash
#!/bin/bash

readonly TEST="Texte qu'on ne pourra pas modifier"

echo "$TEST"

TEST="Essayer de modifier la variable"
```

Résultat possible :

```txt
Texte qu'on ne pourra pas modifier
./script.sh: line 7: TEST: readonly variable
```

!!! warning "Attention"
    `readonly` est utile pour éviter une modification accidentelle, mais ce n’est pas un mécanisme de sécurité fort.

---

## Les “types” en Bash

En Bash, une variable est principalement du texte.

Exemple :

```bash
nombre="42"
echo "$nombre"
```

Même si `42` ressemble à un nombre, Bash le traite d’abord comme une chaîne de caractères.

Pour faire un calcul, il faut utiliser une syntaxe adaptée :

```bash
a=2
b=3

resultat=$(( a + b ))

echo "$resultat"
```

Résultat :

```txt
5
```

!!! note "À retenir"
    En Bash, tout est traité comme du texte par défaut, sauf quand tu utilises une syntaxe de calcul comme `$(( ... ))`.

---

## Supprimer une variable avec `unset`

Pour supprimer une variable :

```bash
prenom="Alice"
echo "$prenom"

unset prenom
echo "$prenom"
```

Après `unset`, la variable n’existe plus.

!!! tip "Cas utile"
    `unset` peut servir à nettoyer une variable temporaire ou éviter de réutiliser une ancienne valeur par erreur.

---

## Exemple complet : script de bienvenue

Objectif :

1. afficher le nom de la machine ;
2. afficher la date ;
3. demander le prénom ;
4. afficher l’utilisateur courant.

```bash
#!/bin/bash

echo "Bienvenue sur cette machine : $(hostname) !"
echo "La date du jour : $(date)"

read -r -p "Comment vous appelez-vous ? " name

echo "Bonjour $name, connecté en tant que $(whoami)"
```

Exécution :

```bash
chmod +x bienvenue.sh
./bienvenue.sh
```

Résultat possible :

```txt
Bienvenue sur cette machine : kali !
La date du jour : Mon May 25 21:30:00 CEST 2026
Comment vous appelez-vous ? Cam
Bonjour Cam, connecté en tant que kali
```

---

## Exemple initial vs version améliorée

Version qui fonctionne, mais moins propre :

```bash
#!/bin/bash

machine=$(hostname)
echo "Bienvenue sur cette machine : "$machine" !"

aujourdhui=$(date)
echo "La date du jour : "$aujourdhui""

read -p "Comment vous appelez-vous ? " name
echo "Bonjour "$name", connecté en tant que $(whoami)"
```

Version plus propre :

```bash
#!/bin/bash

echo "Bienvenue sur cette machine : $(hostname) !"
echo "La date du jour : $(date)"

read -r -p "Comment vous appelez-vous ? " name
echo "Bonjour $name, connecté en tant que $(whoami)"
```

Pourquoi c’est mieux ?

- moins de variables inutiles ;
- `$(hostname)` et `$(date)` sont utilisés directement ;
- `read -r` est plus robuste ;
- le code est plus lisible.

---

## Erreurs classiques

### Mettre des espaces autour du `=`

Incorrect :

```bash
prenom = "Alice"
```

Correct :

```bash
prenom="Alice"
```

---

### Oublier le `$`

```bash
prenom="Alice"

echo "Bonjour, prenom"
```

Résultat :

```txt
Bonjour, prenom
```

Correct :

```bash
echo "Bonjour, $prenom"
```

---

### Oublier les guillemets

À éviter :

```bash
echo $nom
```

Préférer :

```bash
echo "$nom"
```

---

### Confondre guillemets simples et doubles

```bash
prenom="Alice"

echo 'Bonjour, $prenom'
```

Résultat :

```txt
Bonjour, $prenom
```

Si tu veux afficher la valeur :

```bash
echo "Bonjour, $prenom"
```

---

### Coller une variable à du texte sans accolades

Incorrect :

```bash
animal="chat"
echo "J'ai 3 $animals"
```

Correct :

```bash
animal="chat"
echo "J'ai 3 ${animal}s"
```

---

## Cheat sheet

| Syntaxe | Effet |
|---|---|
| `nom="Alice"` | crée une variable |
| `echo "$nom"` | affiche la valeur de la variable |
| `${nom}` | délimite clairement le nom de la variable |
| `"texte $var"` | guillemets doubles : variable interprétée |
| `'texte $var'` | guillemets simples : tout est littéral |
| `read -r -p "Texte : " var` | demande une saisie utilisateur |
| `read -s -p "MDP : " var` | saisie invisible |
| `read -t 5 -p "Choix : " var` | saisie avec timeout |
| `resultat=$(commande)` | stocke le résultat d’une commande |
| `echo "$(commande)"` | exécute une commande directement dans un texte |
| `env` | affiche les variables d’environnement |
| `$USER` | utilisateur courant |
| `$HOME` | dossier personnel |
| `$SHELL` | shell courant |
| `$PWD` | dossier courant |
| `export VAR="x"` | rend une variable disponible pour les processus enfants |
| `readonly VAR="x"` | rend une variable non modifiable |
| `unset VAR` | supprime une variable |
| `$(( a + b ))` | effectue un calcul |

---

## Mini-lab : script `bienvenue.sh`

Crée un script `bienvenue.sh` qui :

1. affiche `Bienvenue sur cette machine : [hostname]` ;
2. affiche la date du jour ;
3. demande le prénom de l’utilisateur ;
4. affiche `Bonjour [prénom], connecté en tant que [whoami]` ;
5. affiche le dossier courant avec `$PWD`.

---

### Correction possible

```bash
#!/bin/bash

echo "Bienvenue sur cette machine : $(hostname) !"
echo "La date du jour : $(date)"
echo "Dossier courant : $PWD"

read -r -p "Comment vous appelez-vous ? " name

echo "Bonjour $name, connecté en tant que $(whoami)"
```

Rendre le script exécutable :

```bash
chmod +x bienvenue.sh
```

Lancer le script :

```bash
./bienvenue.sh
```

---

## Mini-lab bonus : script de scan autorisé

Objectif : créer un petit script qui demande une adresse IP et prépare une commande Nmap.

```bash
#!/bin/bash

read -r -p "Adresse IP à scanner : " ip_scan

echo "Cible choisie : $ip_scan"
echo "Commande exécutée : nmap -F -sV \"$ip_scan\""

nmap -F -sV "$ip_scan"
```

!!! warning "Cadre légal"
    À utiliser uniquement sur un lab, une VM, un CTF ou une cible explicitement autorisée.

---

## Checklist à retenir

Avant de considérer ton script comme propre, vérifie :

- [ ] les variables sont créées sans espace autour du `=` ;
- [ ] les variables sont appelées avec `$` ;
- [ ] les variables sont entourées de guillemets doubles ;
- [ ] les cas ambigus utilisent `${variable}` ;
- [ ] les saisies utilisateur utilisent `read -r` ;
- [ ] les commandes sont récupérées avec `$(commande)` ;
- [ ] les variables sensibles ne sont pas écrites en dur inutilement ;
- [ ] les valeurs importantes sont nommées clairement.

---

## Résumé

Une variable Bash sert à stocker une information.

```bash
nom="Alice"
echo "$nom"
```

`read` permet de demander une saisie utilisateur :

```bash
read -r -p "Ton nom : " nom
```

`$(commande)` permet de récupérer le résultat d’une commande :

```bash
date_du_jour=$(date)
```

Les variables d’environnement sont déjà définies par le système :

```bash
echo "$USER"
echo "$HOME"
```

Et la règle la plus importante reste :

```bash
variable="valeur"    # correct
variable = "valeur"  # incorrect
```

---

## Pour aller plus loin

- Fiche précédente : [Écrire son premier script](premier-script.md).
- Fiche suivante : [Arguments et variables spéciales](arguments.md).