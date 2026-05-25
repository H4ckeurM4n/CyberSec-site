---
tags:
  - bash
  - scripting
  - intermédiaire
---

# Les tableaux

Un tableau permet de stocker **plusieurs valeurs dans une seule variable**.

Chaque valeur est accessible grâce à un **index**, qui commence à `0`.

Les tableaux sont très utiles pour gérer :

- une liste de noms ;
- une liste d’IP ;
- une liste de domaines ;
- une liste de fichiers ;
- les arguments d’un script ;
- des couples nom/email ;
- des menus interactifs.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- créer un tableau ;
- comprendre les index ;
- accéder à un élément précis ;
- afficher tous les éléments ;
- compter le nombre d’éléments ;
- afficher tous les index ;
- ajouter, modifier et supprimer un élément ;
- récupérer les arguments d’un script dans un tableau ;
- parcourir un tableau avec une boucle ;
- parcourir un tableau avec ses index ;
- associer deux tableaux par leur index ;
- trier un tableau ;
- comprendre les tableaux multi-types ;
- utiliser un tableau associatif avec `declare -A`.

---

## À retenir en 30 secondes

Créer un tableau :

```bash
fruits=("pomme" "banane" "cerise")
```

Accéder à un élément :

```bash
echo "${fruits[0]}"
```

Afficher tous les éléments :

```bash
echo "${fruits[@]}"
```

Compter les éléments :

```bash
echo "${#fruits[@]}"
```

Parcourir un tableau :

```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

Parcourir avec les index :

```bash
for i in "${!fruits[@]}"; do
    echo "Index $i : ${fruits[$i]}"
done
```

Créer un tableau associatif :

```bash
declare -A mails
mails[Alice]="alice@mail.com"
```

---

## Pourquoi utiliser des tableaux ?

Sans tableau, tu risques de créer plusieurs variables séparées :

```bash
fruit1="pomme"
fruit2="banane"
fruit3="cerise"
```

Ce n’est pas pratique si tu veux parcourir les valeurs automatiquement.

Avec un tableau :

```bash
fruits=("pomme" "banane" "cerise")
```

Tu peux ensuite faire :

```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

Résultat :

```txt
pomme
banane
cerise
```

---

## Créer un tableau

Syntaxe :

```bash
tableau=("valeur1" "valeur2" "valeur3")
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")
```

Représentation :

```txt
Index :     0         1         2
         ┌─────┐  ┌──────┐  ┌──────┐
         │pomme│  │banane│  │cerise│
         └─────┘  └──────┘  └──────┘
```

!!! note "Important"
    En Bash, les index commencent à `0`, pas à `1`.

---

## Exemple avec des domaines

```bash
domains=(www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com www2.inlanefreight.com)
```

Ce tableau contient 4 éléments :

```txt
Index 0 : www.inlanefreight.com
Index 1 : ftp.inlanefreight.com
Index 2 : vpn.inlanefreight.com
Index 3 : www2.inlanefreight.com
```

---

## Attention aux guillemets

Les guillemets peuvent changer le nombre d’éléments dans le tableau.

Trois éléments distincts :

```bash
domains=(www.site.com ftp.site.com vpn.site.com)
```

Un seul élément contenant des espaces :

```bash
domains=("www.site.com ftp.site.com vpn.site.com")
```

Deux éléments :

```bash
domains=("www.site.com ftp.site.com" vpn.site.com)
```

!!! warning "Attention"
    Tout ce qui est dans les mêmes guillemets compte comme **une seule valeur**.

---

## Accéder à un élément

Pour accéder à un élément précis :

```bash
${tableau[index]}
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

echo "${fruits[0]}"
echo "${fruits[1]}"
echo "${fruits[2]}"
```

Résultat :

```txt
pomme
banane
cerise
```

---

## Afficher tous les éléments

Pour afficher tous les éléments :

```bash
echo "${fruits[@]}"
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

echo "${fruits[@]}"
```

Résultat :

```txt
pomme banane cerise
```

!!! tip "Bonne pratique"
    Pour parcourir un tableau, utilise presque toujours `"${tableau[@]}"` avec les guillemets.

---

## Compter les éléments d’un tableau

Pour connaître le nombre d’éléments :

```bash
echo "${#fruits[@]}"
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

echo "${#fruits[@]}"
```

Résultat :

```txt
3
```

---

## Afficher tous les index

Pour afficher tous les index existants :

```bash
echo "${!fruits[@]}"
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

echo "${!fruits[@]}"
```

Résultat :

```txt
0 1 2
```

C’est très utile quand tu veux parcourir un tableau avec les index.

---

## Récapitulatif des accès

```bash
fruits=("pomme" "banane" "cerise")

echo "${fruits[0]}"      # premier élément
echo "${fruits[@]}"      # tous les éléments
echo "${#fruits[@]}"     # nombre d'éléments
echo "${!fruits[@]}"     # tous les index
```

Résultat :

```txt
pomme
pomme banane cerise
3
0 1 2
```

---

## Ajouter un élément

Pour ajouter un élément à la fin du tableau :

```bash
fruits+=("kiwi")
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

fruits+=("kiwi")

echo "${fruits[@]}"
```

Résultat :

```txt
pomme banane cerise kiwi
```

---

## Modifier un élément

Pour modifier un élément précis :

```bash
fruits[1]="fraise"
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise" "kiwi")

fruits[1]="fraise"

echo "${fruits[@]}"
```

Résultat :

```txt
pomme fraise cerise kiwi
```

L’élément d’index `1`, donc `banane`, a été remplacé par `fraise`.

---

## Supprimer un élément

Pour supprimer un élément :

```bash
unset fruits[1]
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise" "kiwi")

unset fruits[1]

echo "${fruits[@]}"
```

Résultat :

```txt
pomme cerise kiwi
```

Attention : l’index supprimé peut créer un “trou” dans le tableau.

```bash
echo "${!fruits[@]}"
```

Résultat possible :

```txt
0 2 3
```

!!! note
    Après `unset`, Bash ne réindexe pas forcément automatiquement le tableau.

---

## Réindexer un tableau

Si tu veux supprimer les trous dans les index, tu peux recréer le tableau à partir de ses valeurs.

```bash
fruits=("${fruits[@]}")
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise" "kiwi")

unset fruits[1]

echo "Index avant : ${!fruits[@]}"

fruits=("${fruits[@]}")

echo "Index après : ${!fruits[@]}"
```

Résultat possible :

```txt
Index avant : 0 2 3
Index après : 0 1 2
```

---

## Récupérer les arguments d’un script dans un tableau

Pour stocker tous les arguments d’un script dans un tableau :

```bash
tab_args=("$@")
```

Exemple :

```bash
#!/bin/bash

tab_names=("$@")

for name in "${tab_names[@]}"; do
    echo "$name"
done
```

Exécution :

```bash
./script.sh Alice Bob Charlie
```

Résultat :

```txt
Alice
Bob
Charlie
```

!!! tip "Pourquoi `("$@")` ?"
    `"$@"` conserve correctement chaque argument séparé, même si un argument contient des espaces.

---

## Exemple : numéroter les arguments

```bash
#!/bin/bash

tab_names=("$@")
compteur=1

for name in "${tab_names[@]}"; do
    echo "$compteur : $name"
    ((compteur++))
done
```

Exécution :

```bash
./script.sh Alice Bob Charlie
```

Résultat :

```txt
1 : Alice
2 : Bob
3 : Charlie
```

---

## Parcourir un tableau par valeurs

La forme la plus courante :

```bash
for fruit in "${fruits[@]}"; do
    echo "Fruit : $fruit"
done
```

Exemple complet :

```bash
fruits=("pomme" "banane" "cerise" "kiwi")

for fruit in "${fruits[@]}"; do
    echo "Fruit : $fruit"
done
```

Résultat :

```txt
Fruit : pomme
Fruit : banane
Fruit : cerise
Fruit : kiwi
```

---

## Parcourir un tableau avec un compteur

```bash
tab_name=("Camille" "Maya" "Manco" "Billal" "Alex")
compteur=1

for name in "${tab_name[@]}"; do
    echo "$compteur : $name"
    ((compteur++))
done
```

Résultat :

```txt
1 : Camille
2 : Maya
3 : Manco
4 : Billal
5 : Alex
```

---

## Parcourir un tableau par index

Pour parcourir un tableau avec ses index :

```bash
for i in "${!fruits[@]}"; do
    echo "Index $i : ${fruits[$i]}"
done
```

Exemple :

```bash
fruits=("pomme" "banane" "cerise")

for i in "${!fruits[@]}"; do
    echo "Index $i : ${fruits[$i]}"
done
```

Résultat :

```txt
Index 0 : pomme
Index 1 : banane
Index 2 : cerise
```

!!! warning "Attention"
    Pour parcourir les index, il faut utiliser :

    ```bash
    "${!tableau[@]}"
    ```

    et non :

    ```bash
    "${tableau[@]}"
    ```

---

## Associer deux tableaux par leur index

Tu peux utiliser deux tableaux en parallèle.

Exemple :

```bash
noms=("Alice" "Bob" "Charlie")
emails=("alice@mail.com" "bob@mail.com" "charlie@mail.com")

for i in "${!noms[@]}"; do
    echo "${noms[$i]} → ${emails[$i]}"
done
```

Résultat :

```txt
Alice → alice@mail.com
Bob → bob@mail.com
Charlie → charlie@mail.com
```

Ici :

- `noms[0]` correspond à `emails[0]` ;
- `noms[1]` correspond à `emails[1]` ;
- `noms[2]` correspond à `emails[2]`.

---

## Exemple avec affichage plus détaillé

```bash
noms=("Alice" "Bob" "Charlie")
emails=("alice@mail.com" "bob@mail.com" "charlie@mail.com")

for i in "${!noms[@]}"; do
    echo "Nom : ${noms[$i]} -- Email : ${emails[$i]}"
done
```

Résultat :

```txt
Nom : Alice -- Email : alice@mail.com
Nom : Bob -- Email : bob@mail.com
Nom : Charlie -- Email : charlie@mail.com
```

---

## Exemple : menu basé sur deux tableaux

On peut utiliser un choix utilisateur pour retrouver une valeur dans deux tableaux.

```bash
#!/bin/bash

noms=("Alice" "Bob" "Charlie")
emails=("alice@mail.com" "bob@mail.com" "charlie@mail.com")

pause() {
    read -r -p "Appuyez sur la touche [Entrée] pour continuer..." key
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

Ce script montre une utilisation concrète :

- deux tableaux liés par index ;
- un menu utilisateur ;
- un calcul d’index avec `choix - 1` ;
- une fonction pour afficher tous les agents.

---

## Trier un tableau

Pour trier un tableau, on peut envoyer ses valeurs à `sort`.

```bash
tab_names=("$@")

printf "%s\n" "${tab_names[@]}" | sort
```

Exécution :

```bash
./tri.sh Charlie Alice Bob
```

Résultat :

```txt
Alice
Bob
Charlie
```

---

## Trier avec une boucle

Autre façon d’écrire :

```bash
tab_names=("$@")

for name in "${tab_names[@]}"; do
    echo "$name"
done | sort
```

Les deux versions fonctionnent.

!!! tip "Bonne pratique"
    `printf "%s\n" "${tab[@]}"` est souvent plus propre que `echo "${tab[@]}"` pour traiter un élément par ligne.

---

## Tableaux multi-types

Bash ne force pas un type strict pour les valeurs.  
Un tableau peut donc contenir du texte, des nombres ou des rôles.

```bash
infos=("Alice" 25 "Paris" "admin")

echo "Nom  : ${infos[0]}"
echo "Âge  : ${infos[1]}"
echo "Ville: ${infos[2]}"
echo "Rôle : ${infos[3]}"
```

Résultat :

```txt
Nom  : Alice
Âge  : 25
Ville: Paris
Rôle : admin
```

!!! note
    Même si `25` ressemble à un nombre, Bash le traite surtout comme du texte, sauf dans un calcul.

---

## Les tableaux associatifs

Un tableau associatif ressemble à un dictionnaire.

Au lieu d’utiliser des index numériques (`0`, `1`, `2`), il utilise des **clés nommées**.

Il faut le déclarer avec :

```bash
declare -A nom_du_tableau
```

Exemple :

```bash
declare -A capitales

capitales[France]="Paris"
capitales[Allemagne]="Berlin"
capitales[Espagne]="Madrid"
```

---

## Accéder à un tableau associatif

```bash
echo "${capitales[France]}"
```

Résultat :

```txt
Paris
```

Afficher toutes les clés :

```bash
echo "${!capitales[@]}"
```

Résultat possible :

```txt
France Allemagne Espagne
```

Afficher toutes les valeurs :

```bash
echo "${capitales[@]}"
```

Résultat possible :

```txt
Paris Berlin Madrid
```

---

## Parcourir un tableau associatif

```bash
declare -A capitales

capitales[France]="Paris"
capitales[Allemagne]="Berlin"
capitales[Espagne]="Madrid"

for pays in "${!capitales[@]}"; do
    echo "La capitale de $pays est ${capitales[$pays]}"
done
```

Résultat possible :

```txt
La capitale de France est Paris
La capitale de Allemagne est Berlin
La capitale de Espagne est Madrid
```

!!! warning "Ordre d’affichage"
    Les tableaux associatifs ne garantissent pas forcément l’ordre d’affichage des clés.

---

## Exemple : emails avec tableau associatif

```bash
declare -A mails

mails[Alice]="alice@mail.com"
mails[Bob]="bob@mail.com"
mails[Charlie]="charlie@mail.com"

for name in "${!mails[@]}"; do
    echo "Nom : $name — Email : ${mails[$name]}"
done
```

Résultat possible :

```txt
Nom : Alice — Email : alice@mail.com
Nom : Bob — Email : bob@mail.com
Nom : Charlie — Email : charlie@mail.com
```

---

## Tableau classique ou tableau associatif ?

| Besoin | Solution |
|---|---|
| liste simple d’éléments | tableau classique |
| accéder par numéro | tableau classique |
| relier une clé à une valeur | tableau associatif |
| stocker `nom → email` | tableau associatif |
| garder un ordre précis | tableau classique préférable |

Exemple tableau classique :

```bash
fruits=("pomme" "banane" "cerise")
```

Exemple tableau associatif :

```bash
declare -A mails
mails[Alice]="alice@mail.com"
```

---

## Mini-lab 1 : afficher une liste numérotée

Objectif : créer un script qui affiche une liste de prénoms avec un numéro.

Contraintes :

1. créer un tableau `prenoms` ;
2. parcourir le tableau ;
3. afficher chaque prénom avec un compteur.

---

### Correction possible

```bash
#!/bin/bash

prenoms=("Alice" "Bob" "Charlie")
compteur=1

for prenom in "${prenoms[@]}"; do
    echo "$compteur : $prenom"
    ((compteur++))
done
```

---

## Mini-lab 2 : afficher les index

Objectif : afficher les valeurs d’un tableau avec leur index réel.

Contraintes :

1. créer un tableau `fruits` ;
2. parcourir les index avec `"${!fruits[@]}"` ;
3. afficher `Index X : valeur`.

---

### Correction possible

```bash
#!/bin/bash

fruits=("pomme" "banane" "cerise")

for i in "${!fruits[@]}"; do
    echo "Index $i : ${fruits[$i]}"
done
```

---

## Mini-lab 3 : récupérer les arguments dans un tableau

Objectif : créer un script `args_tableau.sh`.

Contraintes :

1. récupérer tous les arguments dans un tableau ;
2. afficher le nombre d’arguments ;
3. afficher chaque argument sur une ligne.

---

### Correction possible

```bash
#!/bin/bash

tab_args=("$@")

echo "Nombre d'arguments : ${#tab_args[@]}"

for arg in "${tab_args[@]}"; do
    echo "Argument : $arg"
done
```

Test :

```bash
./args_tableau.sh "Jean Pierre" Marie Alice
```

---

## Mini-lab 4 : associer noms et emails

Objectif : créer deux tableaux, puis afficher les associations.

Contraintes :

1. créer un tableau `noms` ;
2. créer un tableau `emails` ;
3. parcourir les index ;
4. afficher `Nom → Email`.

---

### Correction possible

```bash
#!/bin/bash

noms=("Alice" "Bob" "Charlie")
emails=("alice@mail.com" "bob@mail.com" "charlie@mail.com")

for i in "${!noms[@]}"; do
    echo "${noms[$i]} → ${emails[$i]}"
done
```

---

## Mini-lab 5 : utiliser un tableau associatif

Objectif : créer un dictionnaire nom → email.

Contraintes :

1. déclarer un tableau associatif ;
2. ajouter trois entrées ;
3. afficher l’email de `Alice` ;
4. parcourir toutes les clés.

---

### Correction possible

```bash
#!/bin/bash

declare -A mails

mails[Alice]="alice@mail.com"
mails[Bob]="bob@mail.com"
mails[Charlie]="charlie@mail.com"

echo "Email d'Alice : ${mails[Alice]}"

for name in "${!mails[@]}"; do
    echo "$name → ${mails[$name]}"
done
```

---

## Erreurs classiques

### Oublier les parenthèses à la création

Incorrect :

```bash
fruits="pomme" "banane" "cerise"
```

Correct :

```bash
fruits=("pomme" "banane" "cerise")
```

---

### Croire que l’index commence à 1

```bash
fruits=("pomme" "banane" "cerise")

echo "${fruits[1]}"
```

Résultat :

```txt
banane
```

L’index `1` correspond au deuxième élément.

---

### Mettre plusieurs valeurs dans les mêmes guillemets

```bash
domains=("www.site.com ftp.site.com vpn.site.com")
```

Ici, il n’y a qu’un seul élément.

Préférer :

```bash
domains=("www.site.com" "ftp.site.com" "vpn.site.com")
```

---

### Oublier les guillemets autour de `"${tab[@]}"`

À éviter :

```bash
for item in ${tab[@]}; do
    echo "$item"
done
```

Préférer :

```bash
for item in "${tab[@]}"; do
    echo "$item"
done
```

Sinon, les valeurs contenant des espaces peuvent être découpées.

---

### Confondre valeurs et index

Valeurs :

```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

Index :

```bash
for i in "${!fruits[@]}"; do
    echo "${fruits[$i]}"
done
```

---

### Mauvaise syntaxe avec les index

À éviter :

```bash
echo "$fruits[0]"
```

Correct :

```bash
echo "${fruits[0]}"
```

Les accolades sont nécessaires pour accéder correctement à un élément du tableau.

---

## Cheat sheet

| Syntaxe | Effet |
|---|---|
| `tab=("a" "b" "c")` | créer un tableau |
| `${tab[0]}` | accéder à l’élément d’index 0 |
| `${tab[@]}` | tous les éléments |
| `"${tab[@]}"` | tous les éléments en préservant les espaces |
| `${#tab[@]}` | nombre d’éléments |
| `${!tab[@]}` | tous les index |
| `tab+=("d")` | ajouter un élément |
| `tab[1]="x"` | modifier un élément |
| `unset tab[1]` | supprimer un élément |
| `tab=("${tab[@]}")` | réindexer un tableau |
| `tab_args=("$@")` | stocker les arguments dans un tableau |
| `for x in "${tab[@]}"` | parcourir les valeurs |
| `for i in "${!tab[@]}"` | parcourir les index |
| `printf "%s\n" "${tab[@]}" \| sort` | trier les valeurs |
| `declare -A dico` | créer un tableau associatif |
| `${dico[cle]}` | accéder à une valeur par clé |
| `${!dico[@]}` | afficher les clés |

---

## Checklist script propre

Avant de considérer ton usage des tableaux comme propre, vérifie :

- [ ] le tableau est créé avec des parenthèses ;
- [ ] les valeurs contenant des espaces sont entre guillemets ;
- [ ] tu sais si tu parcours les valeurs ou les index ;
- [ ] tu utilises `"${tab[@]}"` pour préserver les espaces ;
- [ ] tu utilises `"${!tab[@]}"` pour parcourir les index ;
- [ ] tu sais que les index commencent à `0` ;
- [ ] tu vérifies que deux tableaux associés ont la même taille ;
- [ ] tu utilises `declare -A` pour les tableaux associatifs ;
- [ ] tu ne supposes pas l’ordre d’un tableau associatif.

---

## Résumé

Un tableau stocke plusieurs valeurs :

```bash
fruits=("pomme" "banane" "cerise")
```

On accède à un élément avec son index :

```bash
echo "${fruits[0]}"
```

On parcourt les valeurs avec :

```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

On parcourt les index avec :

```bash
for i in "${!fruits[@]}"; do
    echo "$i : ${fruits[$i]}"
done
```

Un tableau associatif utilise des clés nommées :

```bash
declare -A mails
mails[Alice]="alice@mail.com"
```

Le réflexe important :

> `"${tab[@]}"` pour les valeurs, `"${!tab[@]}"` pour les index.

---

## Pour aller plus loin

- Fiche précédente : [Manipuler les chaînes](chaines.md).
- Fiche suivante : [Déboguer et écrire des scripts propres](debug.md).