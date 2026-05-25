---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Redirections, erreurs et pipes

Chaque commande Bash travaille avec des **flux de données**.

Comprendre ces flux permet de :

- rediriger ce qui s’affiche à l’écran ;
- enregistrer une sortie dans un fichier ;
- capturer ou masquer les erreurs ;
- séparer les résultats normaux des messages d’erreur ;
- enchaîner plusieurs commandes entre elles avec des pipes ;
- sauvegarder un résultat tout en l’affichant avec `tee`.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- comprendre les trois flux standards : `stdin`, `stdout`, `stderr` ;
- rediriger une sortie normale vers un fichier ;
- distinguer `>` et `>>` ;
- rediriger les erreurs avec `2>` ;
- masquer les erreurs avec `2>/dev/null` ;
- tout rediriger avec `2>&1` ;
- envoyer volontairement un message vers `stderr` avec `>&2` ;
- utiliser les pipes `|` ;
- utiliser `tee` pour afficher et sauvegarder en même temps ;
- rediriger l’entrée avec `<`.

---

## À retenir en 30 secondes

Chaque commande a trois flux principaux :

| Numéro | Flux | Rôle |
|---:|---|---|
| `0` | `stdin` | entrée standard |
| `1` | `stdout` | sortie normale |
| `2` | `stderr` | sortie d’erreur |

Redirections essentielles :

```bash
commande > fichier          # stdout vers fichier, écrase
commande >> fichier         # stdout vers fichier, ajoute
commande 2> erreurs.txt     # stderr vers fichier
commande > log.txt 2>&1     # stdout + stderr vers fichier
commande 2>/dev/null        # masque les erreurs
cmd1 | cmd2                 # stdout de cmd1 vers stdin de cmd2
commande | tee fichier      # affiche et sauvegarde
```

---

## Les trois flux standards

Une commande travaille avec trois flux principaux.

```txt
                      ┌──────────────┐
Entrée standard ────▶ │   Commande   │ ──▶ 1 = stdout
stdin = 0             │              │ ──▶ 2 = stderr
                      └──────────────┘
```

| Numéro | Nom | Rôle | Par défaut |
|---:|---|---|---|
| `0` | `stdin` | entrée standard | clavier |
| `1` | `stdout` | sortie normale | écran |
| `2` | `stderr` | sortie d’erreur | écran |

Par défaut :

- une commande lit depuis le clavier ou depuis une entrée ;
- affiche son résultat normal à l’écran ;
- affiche ses erreurs à l’écran aussi.

Les redirections permettent de changer la destination de ces flux.

---

## `stdout` : la sortie normale

La sortie normale correspond au résultat attendu d’une commande.

Exemple :

```bash
ls /etc
```

Ici, la liste des fichiers et dossiers de `/etc` est envoyée sur `stdout`.

On peut rediriger cette sortie normale vers un fichier.

```bash
ls /etc > liste.txt
```

---

## `stderr` : la sortie d’erreur

La sortie d’erreur correspond aux messages d’erreur.

Exemple :

```bash
ls /dossier_inexistant
```

Résultat possible :

```txt
ls: cannot access '/dossier_inexistant': No such file or directory
```

Ce message part sur `stderr`, pas sur `stdout`.

C’est important, car cela permet de séparer les vrais résultats des erreurs.

---

## Rediriger la sortie vers un fichier avec `>`

Le symbole `>` redirige la sortie normale vers un fichier.

```bash
echo "Bonjour" > message.txt
```

Si le fichier n’existe pas, il est créé.  
S’il existe déjà, il est écrasé.

Exemples :

```bash
echo "Bonjour" > message.txt
ls /etc > liste.txt
date > date.txt
```

!!! danger "Piège classique"
    `>` écrase le contenu existant sans demander confirmation.

---

## Exemple : écrasement avec `>`

```bash
echo "Ligne 1" > journal.txt
echo "Ligne 2" > journal.txt
cat journal.txt
```

Résultat :

```txt
Ligne 2
```

La deuxième commande a écrasé la première ligne.

---

## Ajouter à la suite avec `>>`

Le symbole `>>` ajoute la sortie à la fin du fichier sans supprimer ce qui existe déjà.

```bash
echo "Ligne 1" > journal.txt
echo "Ligne 2" >> journal.txt
echo "Ligne 3" >> journal.txt

cat journal.txt
```

Résultat :

```txt
Ligne 1
Ligne 2
Ligne 3
```

| Syntaxe | Effet |
|---|---|
| `>` | crée ou écrase |
| `>>` | ajoute à la fin |

!!! tip "Réflexe"
    Si tu veux conserver l’ancien contenu, utilise `>>`.

---

## Rediriger les erreurs avec `2>`

Le flux d’erreur est le flux numéro `2`.

Pour rediriger les erreurs vers un fichier :

```bash
ls /dossier_inexistant 2> erreurs.txt
```

Ici :

- la sortie normale reste affichée à l’écran ;
- les erreurs vont dans `erreurs.txt`.

Exemple :

```bash
find / -name "secret" 2> erreurs.txt
```

Les erreurs de permission seront sauvegardées dans `erreurs.txt`.

---

## Masquer les erreurs avec `2>/dev/null`

`/dev/null` est une sorte de “poubelle” du système.  
Tout ce qu’on y envoie disparaît.

Exemple :

```bash
find / -name "mon_fichier" 2>/dev/null
```

Ici :

- les résultats normaux s’affichent ;
- les erreurs sont ignorées.

Cas typique :

```bash
find / -name "*.log" 2>/dev/null
```

Sans `2>/dev/null`, tu risques d’avoir beaucoup de messages `Permission denied`.

!!! note "À retenir"
    `/dev/null` sert à supprimer volontairement une sortie ou des erreurs.

---

## Rediriger stdout et stderr séparément

Tu peux envoyer la sortie normale et les erreurs dans deux fichiers différents.

```bash
commande > sortie.txt 2> erreurs.txt
```

Exemple :

```bash
find / -name "*.log" > resultats.txt 2> erreurs.txt
```

Ici :

- les fichiers trouvés vont dans `resultats.txt` ;
- les erreurs vont dans `erreurs.txt`.

---

## Tout rediriger avec `2>&1`

La syntaxe `2>&1` signifie :

> envoie le flux `2` au même endroit que le flux `1`.

Exemple :

```bash
./script.sh > log.txt 2>&1
```

Lecture de gauche à droite :

1. `> log.txt` redirige `stdout` vers `log.txt` ;
2. `2>&1` redirige `stderr` vers le même endroit que `stdout` ;
3. donc tout finit dans `log.txt`.

---

## Comparaison des redirections

```bash
./script.sh > log.txt
```

- `stdout` va dans `log.txt` ;
- `stderr` reste à l’écran.

```bash
./script.sh 2> log.txt
```

- `stderr` va dans `log.txt` ;
- `stdout` reste à l’écran.

```bash
./script.sh > log.txt 2>&1
```

- `stdout` et `stderr` vont dans `log.txt`.

```bash
./script.sh > /dev/null 2>&1
```

- `stdout` et `stderr` sont supprimés.

---

## Silence total

Pour rendre une commande silencieuse :

```bash
commande > /dev/null 2>&1
```

Exemple :

```bash
ping -c 1 8.8.8.8 > /dev/null 2>&1
```

Ce type de redirection est utile quand tu veux seulement savoir si une commande réussit, sans afficher sa sortie.

Exemple :

```bash
if ping -c 1 8.8.8.8 > /dev/null 2>&1; then
    echo "Hôte joignable"
else
    echo "Hôte injoignable"
fi
```

---

## Attention à l’ordre des redirections

L’ordre est important.

Correct :

```bash
commande > log.txt 2>&1
```

Ici, `stdout` va dans `log.txt`, puis `stderr` suit `stdout`.

Piégeux :

```bash
commande 2>&1 > log.txt
```

Ici :

1. `stderr` est envoyé vers l’endroit actuel de `stdout`, donc l’écran ;
2. ensuite seulement `stdout` est redirigé vers `log.txt`.

Résultat :

- `stdout` va dans `log.txt` ;
- `stderr` reste à l’écran.

!!! warning "Point important"
    `commande > log.txt 2>&1` et `commande 2>&1 > log.txt` ne font pas la même chose.

---

## Raccourci moderne : `&>`

En Bash, on peut aussi voir :

```bash
commande &> log.txt
```

Cela redirige `stdout` et `stderr` vers le même fichier.

Équivalent proche :

```bash
commande > log.txt 2>&1
```

Pour ajouter au lieu d’écraser :

```bash
commande &>> log.txt
```

!!! note
    `&>` est pratique, mais `> fichier 2>&1` reste très courant et plus explicite pour apprendre.

---

## Ne pas polluer la sortie : `>&2`

Quand un script affiche un message d’erreur, il devrait l’envoyer sur `stderr`.

```bash
echo "Erreur : argument manquant" >&2
```

Pourquoi ?

Imagine ce script :

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant"
    exit 1
fi

echo "Dossier : $1"
```

Si l’utilisateur fait :

```bash
./script.sh > resultat.txt
```

Sans `>&2`, le message d’erreur peut finir dans `resultat.txt`.

Ce fichier contiendrait alors :

```txt
Erreur : argument manquant
```

Alors que ce n’est pas un résultat normal.

---

## Version propre avec `>&2`

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : argument manquant" >&2
    exit 1
fi

echo "Dossier : $1"
```

Si l’utilisateur lance :

```bash
./script.sh > resultat.txt
```

Alors :

- `resultat.txt` contient uniquement la sortie normale ;
- l’erreur reste affichée à l’écran.

!!! tip "Bonne pratique"
    Dans tes scripts, envoie les erreurs vers `stderr` avec `>&2`.

---

## Rediriger l’entrée avec `<`

Le symbole `<` permet de donner un fichier comme entrée à une commande.

Exemple :

```bash
wc -l < mon_fichier.txt
```

Cela compte les lignes du fichier.

Autre exemple :

```bash
sort < liste_noms.txt
```

Cela trie le contenu du fichier.

---

## Différence entre `cat fichier | commande` et `commande < fichier`

Ces deux commandes peuvent produire un résultat similaire :

```bash
cat fichier.txt | wc -l
```

```bash
wc -l < fichier.txt
```

La deuxième est souvent plus propre, car elle évite d’utiliser `cat` inutilement.

!!! note "À retenir"
    Quand une commande sait lire depuis `stdin`, tu peux souvent utiliser `< fichier`.

---

## Les pipes `|`

Un pipe envoie la sortie normale d’une commande comme entrée de la commande suivante.

```bash
cmd1 | cmd2
```

Cela signifie :

> la sortie de `cmd1` devient l’entrée de `cmd2`.

Exemples :

```bash
ls | wc -l
ps aux | grep firefox
cat prenoms.txt | sort | uniq
```

---

## Exemple : compter les fichiers

```bash
ls | wc -l
```

Ce qui se passe :

1. `ls` liste les fichiers ;
2. sa sortie est envoyée à `wc -l` ;
3. `wc -l` compte les lignes.

---

## Exemple : filtrer un processus

```bash
ps aux | grep firefox
```

Ce qui se passe :

1. `ps aux` liste les processus ;
2. `grep firefox` filtre les lignes contenant `firefox`.

---

## Exemple : trier et dédoublonner

```bash
cat prenoms.txt | sort | uniq
```

Ce qui se passe :

1. `cat prenoms.txt` affiche le fichier ;
2. `sort` trie les lignes ;
3. `uniq` supprime les doublons consécutifs.

Version souvent plus propre :

```bash
sort prenoms.txt | uniq
```

ou :

```bash
sort -u prenoms.txt
```

---

## Enchaîner plusieurs pipes

Les pipes permettent de construire une chaîne de traitement.

Exemple :

```bash
ls /etc | grep "\.conf" | wc -l
```

Ce pipeline :

1. liste le contenu de `/etc` ;
2. garde les lignes contenant `.conf` ;
3. compte le nombre de lignes restantes.

Autre exemple :

```bash
ls -lS | head -5
```

Ce pipeline affiche les 5 plus gros éléments selon le tri de `ls -lS`.

---

## Limite importante des pipes

Par défaut, un pipe transporte surtout `stdout`, pas `stderr`.

Exemple :

```bash
find / -name "*.log" | grep syslog
```

Les erreurs de permission peuvent encore apparaître à l’écran, car elles sont sur `stderr`.

Pour les masquer :

```bash
find / -name "*.log" 2>/dev/null | grep syslog
```

---

## `tee` : afficher et sauvegarder en même temps

La commande `tee` lit depuis l’entrée standard, affiche à l’écran et écrit dans un fichier.

```bash
commande | tee fichier.txt
```

Exemple :

```bash
ls -la | tee log.txt
```

Résultat :

- la sortie s’affiche à l’écran ;
- la sortie est aussi écrite dans `log.txt`.

---

## Ajouter avec `tee -a`

Par défaut, `tee` écrase le fichier.

```bash
date | tee journal.log
```

Pour ajouter à la fin :

```bash
date | tee -a journal.log
```

| Syntaxe | Effet |
|---|---|
| `tee fichier` | écrit dans le fichier en écrasant |
| `tee -a fichier` | ajoute à la fin du fichier |

---

## Exemple cyber : garder une trace de hosts découverts

Dans un contexte autorisé, on peut utiliser `tee` pour afficher des résultats et les sauvegarder.

```bash
hosts=$(host "$domain" | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)
```

Ce pipeline :

1. résout un domaine avec `host` ;
2. filtre les lignes contenant `has address` ;
3. extrait le quatrième champ ;
4. affiche et sauvegarde dans `discovered_hosts.txt` ;
5. stocke aussi le résultat dans la variable `hosts`.

Autre exemple :

```bash
netrange=$(whois "$ip" | grep "NetRange\|CIDR" | tee -a CIDR.txt)
```

Ici, les informations sont ajoutées dans `CIDR.txt`.

!!! warning "Cadre légal"
    Utilise ce type de commande uniquement sur tes domaines, un lab, un CTF ou un périmètre explicitement autorisé.

---

## Sauvegarder les erreurs avec `tee`

Si tu veux afficher et sauvegarder les erreurs, tu peux rediriger `stderr` vers `stdout`, puis utiliser `tee`.

```bash
commande 2>&1 | tee log.txt
```

Exemple :

```bash
find / -name "*.log" 2>&1 | tee recherche.log
```

Ici :

- résultats et erreurs sont affichés ;
- résultats et erreurs sont sauvegardés dans `recherche.log`.

---

## Mini-lab 1 : créer un journal

Objectif : créer un fichier `journal.txt`.

Contraintes :

1. écrire une première ligne avec `>` ;
2. ajouter deux lignes avec `>>` ;
3. afficher le résultat avec `cat`.

---

### Correction possible

```bash
echo "Début du journal" > journal.txt
echo "Date : $(date)" >> journal.txt
echo "Utilisateur : $USER" >> journal.txt

cat journal.txt
```

---

## Mini-lab 2 : séparer résultats et erreurs

Objectif : chercher des fichiers `.log` depuis `/`.

Contraintes :

1. mettre les résultats dans `resultats.txt` ;
2. mettre les erreurs dans `erreurs.txt`.

---

### Correction possible

```bash
find / -name "*.log" > resultats.txt 2> erreurs.txt
```

Afficher le nombre de résultats :

```bash
wc -l < resultats.txt
```

Afficher le nombre d’erreurs :

```bash
wc -l < erreurs.txt
```

---

## Mini-lab 3 : pipeline simple

Objectif : compter le nombre de fichiers `.conf` dans `/etc`.

---

### Correction possible

```bash
ls /etc | grep "\.conf" | wc -l
```

Version alternative avec `find` :

```bash
find /etc -maxdepth 1 -name "*.conf" 2>/dev/null | wc -l
```

---

## Mini-lab 4 : afficher et sauvegarder

Objectif : afficher la liste détaillée du dossier courant et la sauvegarder dans `log.txt`.

---

### Correction possible

```bash
ls -la | tee log.txt
```

Pour ajouter au lieu d’écraser :

```bash
date | tee -a log.txt
```

---

## Erreurs classiques

### Confondre `>` et `>>`

Incorrect si tu veux conserver l’ancien contenu :

```bash
echo "nouveau" > config.txt
```

Cela écrase tout.

Correct pour ajouter :

```bash
echo "nouveau" >> config.txt
```

---

### Oublier `2>`

Commande bruyante :

```bash
find / -name "*.log"
```

Version plus propre :

```bash
find / -name "*.log" 2>/dev/null
```

---

### Croire que `|` transporte aussi les erreurs

```bash
find / -name "*.log" | grep syslog
```

Les erreurs peuvent continuer à s’afficher.

Version propre :

```bash
find / -name "*.log" 2>/dev/null | grep syslog
```

---

### Mal placer `2>&1`

Correct :

```bash
commande > log.txt 2>&1
```

Piégeux :

```bash
commande 2>&1 > log.txt
```

L’ordre change le résultat.

---

### Écraser un log avec `tee`

```bash
date | tee journal.log
```

Écrase le fichier.

Pour ajouter :

```bash
date | tee -a journal.log
```

---

## Cheat sheet

| Syntaxe | Effet |
|---|---|
| `commande > fichier` | redirige `stdout` vers un fichier en écrasant |
| `commande >> fichier` | redirige `stdout` vers un fichier en ajoutant |
| `commande 2> fichier` | redirige `stderr` vers un fichier |
| `commande > fichier 2>&1` | redirige `stdout` et `stderr` vers un fichier |
| `commande > /dev/null 2>&1` | silence total |
| `commande 2>/dev/null` | masque uniquement les erreurs |
| `commande > sortie.txt 2> erreurs.txt` | sépare sortie normale et erreurs |
| `commande < fichier` | utilise un fichier comme entrée |
| `cmd1 \| cmd2` | envoie `stdout` de `cmd1` vers `stdin` de `cmd2` |
| `cmd1 2>/dev/null \| cmd2` | pipe sans erreurs bruyantes |
| `commande \| tee fichier` | affiche et sauvegarde |
| `commande \| tee -a fichier` | affiche et ajoute au fichier |
| `echo "Erreur" >&2` | envoie un message vers `stderr` |
| `commande 2>&1 \| tee log.txt` | affiche et sauvegarde sortie + erreurs |

---

## Checklist script propre

Avant de considérer ton script comme propre, vérifie :

- [ ] tu sais si tu veux écraser (`>`) ou ajouter (`>>`) ;
- [ ] les erreurs sont redirigées avec `2>` si nécessaire ;
- [ ] les messages d’erreur du script utilisent `>&2` ;
- [ ] tu ne masques pas les erreurs importantes sans raison ;
- [ ] les fichiers de log ne sont pas écrasés par erreur ;
- [ ] les pipelines sont lisibles ;
- [ ] `tee -a` est utilisé si tu veux conserver l’historique ;
- [ ] les commandes bruyantes comme `find /` gèrent les erreurs de permission.

---

## Résumé

Les redirections permettent de contrôler les flux d’une commande.

```bash
commande > fichier
```

écrit la sortie normale dans un fichier.

```bash
commande 2> erreurs.txt
```

écrit les erreurs dans un fichier.

```bash
commande > log.txt 2>&1
```

écrit sortie normale et erreurs dans le même fichier.

Les pipes permettent d’enchaîner les commandes :

```bash
cmd1 | cmd2 | cmd3
```

Et `tee` permet d’afficher et sauvegarder en même temps :

```bash
commande | tee log.txt
```

Le réflexe le plus important :

```bash
> écrase
>> ajoute
2> redirige les erreurs
| enchaîne les commandes
tee affiche et sauvegarde
```

---

## Pour aller plus loin

- Fiche précédente : [Arguments et variables spéciales](arguments.md).
- Fiche suivante : [Opérateurs et calculs](operateurs.md).