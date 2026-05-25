---
tags:
  - bash
  - scripting
  - cas-pratique
  - automatisation
---

# Automatisation : cron et cas pratiques

Cette fiche rassemble les notions vues dans le pilier Bash pour les appliquer à des cas concrets.

L’objectif n’est plus seulement de comprendre une syntaxe, mais de savoir construire des scripts utiles, réutilisables et automatisables.

L’état d’esprit de l’automaticien tient en trois questions :

1. Qu’est-ce que je fais régulièrement à la main ?
2. Est-ce toujours les mêmes étapes ?
3. Est-ce que cela pourrait tourner tout seul ?

Si la réponse est oui, alors il y a probablement quelque chose à automatiser.

---

## Objectifs

À la fin de cette fiche, tu dois savoir :

- comprendre quand automatiser une tâche ;
- planifier un script avec `cron` ;
- lire une syntaxe cron ;
- journaliser l’exécution d’un script ;
- écrire proprement plusieurs lignes dans un fichier ;
- créer un rapport simple ;
- sauvegarder des dossiers avec `tar` ;
- nommer dynamiquement des archives avec une date ;
- renommer des fichiers en masse ;
- comprendre un cas pratique complet de reconnaissance réseau autorisée ;
- utiliser un template propre pour démarrer tes futurs scripts.

---

## À retenir en 30 secondes

Éditer les tâches cron :

```bash
crontab -e
```

Lister les tâches cron :

```bash
crontab -l
```

Exécuter un script tous les jours à minuit :

```bash
0 0 * * * /home/user/scripts/backup.sh
```

Journaliser une tâche cron :

```bash
0 0 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

Écrire plusieurs lignes dans un fichier :

```bash
{
    echo "Date : $(date)"
    echo "Utilisateur : $USER"
} > rapport.txt
```

Créer une archive datée :

```bash
archive="/tmp/backups/$(date +%Y%m%d-%H%M%S).tar.gz"
tar -czf "$archive" "$dossier"
```

---

## Penser comme un automaticien

Avant d’écrire un script, pose-toi ces questions :

| Question | Exemple |
|---|---|
| Qu’est-ce que je fais souvent à la main ? | compter des fichiers, sauvegarder un dossier, lancer une vérification |
| Est-ce toujours les mêmes étapes ? | vérifier un argument, créer un dossier, générer un fichier |
| Est-ce que ça pourrait tourner tout seul ? | via `cron`, un menu, ou une commande lancée périodiquement |

Exemples de tâches automatisables :

- sauvegarder un dossier tous les jours ;
- générer un rapport d’état ;
- renommer des fichiers en masse ;
- vérifier la présence de fichiers ;
- compter les lignes d’un fichier ;
- lancer une vérification réseau autorisée ;
- journaliser une action ;
- produire un fichier de sortie propre.

!!! tip "Règle simple"
    Une bonne automatisation remplace une suite d’actions répétitives par un script clair, testé et journalisé.

---

## Planifier avec `cron`

`cron` permet de lancer automatiquement une commande ou un script à intervalles réguliers.

Commandes essentielles :

```bash
crontab -e
```

Éditer les tâches planifiées de l’utilisateur courant.

```bash
crontab -l
```

Lister les tâches planifiées existantes.

---

## Syntaxe d’une ligne cron

Une ligne cron contient cinq champs de temps, puis la commande à exécuter.

```txt
┌───────── minute (0-59)
│ ┌─────── heure (0-23)
│ │ ┌───── jour du mois (1-31)
│ │ │ ┌─── mois (1-12)
│ │ │ │ ┌─ jour de la semaine (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * * commande
```

Exemples :

```bash
0 0 * * * /home/user/scripts/backup.sh
```

Tous les jours à minuit.

```bash
0 */6 * * * /home/user/scripts/check_disk.sh
```

Toutes les 6 heures.

```bash
0 8 * * 1 /home/user/scripts/rapport.sh
```

Tous les lundis à 8h.

---

## Exemples de planification

| Expression cron | Signification |
|---|---|
| `* * * * *` | toutes les minutes |
| `0 * * * *` | toutes les heures |
| `0 0 * * *` | tous les jours à minuit |
| `0 8 * * 1` | tous les lundis à 8h |
| `0 */6 * * *` | toutes les 6 heures |
| `30 23 * * *` | tous les jours à 23h30 |
| `0 9 1 * *` | le 1er jour de chaque mois à 9h |

---

## Toujours journaliser une tâche cron

Une tâche cron tourne sans terminal interactif.  
Si ton script affiche une erreur, tu ne la verras pas forcément.

Il faut donc rediriger la sortie vers un fichier de log.

```bash
0 0 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

Explication :

| Élément | Rôle |
|---|---|
| `>> backup.log` | ajoute la sortie normale au fichier de log |
| `2>&1` | envoie aussi les erreurs dans le même fichier |
| `cron` | lance le script automatiquement |

!!! tip "Bonne pratique"
    Pour une tâche automatisée, prévois toujours un fichier de log.  
    Sinon, diagnostiquer une erreur devient beaucoup plus difficile.

---

## Attention aux chemins avec cron

Dans un terminal, ton environnement est chargé : `PATH`, dossier courant, variables, etc.

Avec `cron`, l’environnement est souvent plus minimal.

Évite donc les chemins relatifs :

```bash
./backup.sh
```

Préférer les chemins absolus :

```bash
/home/user/scripts/backup.sh
```

Même chose pour les fichiers de sortie :

```bash
/home/user/logs/backup.log
```

!!! warning "Point important"
    Un script qui fonctionne dans ton terminal peut échouer avec `cron` si les chemins ne sont pas absolus.

---

## Exemple : script journalisé par cron

Script `hello_log.sh` :

```bash
#!/bin/bash

LOG="$HOME/connexions.log"

utilisateur=$(whoami)
date_heure=$(date '+%Y-%m-%d %H:%M:%S')

echo "Bienvenue, $utilisateur !"
echo "[$date_heure] Connexion de $utilisateur" >> "$LOG"
echo "Connexion enregistrée dans $LOG"
```

Exécution manuelle :

```bash
chmod +x hello_log.sh
./hello_log.sh
```

Exemple de ligne cron :

```bash
0 8 * * * /home/user/scripts/hello_log.sh >> /home/user/logs/hello.log 2>&1
```

---

## Formater une date

La commande `date` peut produire différents formats.

Date compacte :

```bash
date +%Y%m%d
```

Résultat possible :

```txt
20260525
```

Date avec heure :

```bash
date '+%Y-%m-%d %H:%M:%S'
```

Résultat possible :

```txt
2026-05-25 21:30:00
```

Date pour nom de fichier :

```bash
date +%Y%m%d-%H%M%S
```

Résultat possible :

```txt
20260525-213000
```

---

## Stocker une date dans une variable

```bash
date_jour=$(date +%Y-%m-%d)
date_heure=$(date '+%Y-%m-%d %H:%M:%S')
timestamp=$(date +%Y%m%d-%H%M%S)

echo "$date_jour"
echo "$date_heure"
echo "$timestamp"
```

Ces formats sont utiles pour :

- nommer un fichier ;
- créer une archive ;
- journaliser une action ;
- horodater un rapport.

---

## Écrire proprement plusieurs lignes dans un fichier

Pour écrire plusieurs lignes dans un fichier, tu peux rediriger un bloc entier.

```bash
{
    echo "Date : $(date)"
    echo "Dossier : $1"
    echo "Nombre de fichiers : $nbr_fichiers"
} > rapport.txt
```

Avantage : tu évites de répéter `>> rapport.txt` à chaque ligne.

---

## Exemple : générer un rapport de dossier

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

Ce script montre plusieurs notions importantes :

- vérification d’argument ;
- test de dossier avec `-d` ;
- compteur avec `wc -l` ;
- génération de rapport ;
- redirection d’un bloc vers un fichier.

---

## Cas pratique : tester des fichiers et dossiers

Ce script prend un ou plusieurs chemins en arguments et indique s’ils correspondent à des fichiers, dossiers ou chemins inexistants.

```bash
#!/bin/bash

if [[ $# -eq 0 ]]; then
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
./check_paths.sh /etc/passwd /home /inexistant
```

Ce script illustre :

- `"$@"` pour parcourir tous les arguments ;
- `-f` pour tester un fichier ;
- `-d` pour tester un dossier ;
- `wc -c` pour mesurer une taille ;
- une boucle `for`.

---

## Cas pratique : sauvegarder des dossiers

Objectif : sauvegarder plusieurs dossiers dans des archives `.tar.gz`.

```bash
#!/bin/bash

# --- Configuration ---
dossiers=("/etc" "/home")
destination="/tmp/backups"
date_du_jour=$(date +%Y-%m-%d)

# --- Préparation ---
mkdir -p "$destination"

echo "=== Sauvegarde du $date_du_jour ==="

for dossier in "${dossiers[@]}"; do
    nom_archive=$(echo "$dossier" | tr '/' '_')
    fichier="${destination}/${nom_archive}-${date_du_jour}.tar.gz"

    echo -n "Sauvegarde de $dossier ... "

    if tar -czf "$fichier" "$dossier" 2>/dev/null; then
        taille=$(du -h "$fichier" | cut -f1)
        echo "OK ($taille)"
    else
        echo "ERREUR" >&2
    fi
done

echo "=== Terminé ==="
```

---

## Explication du script de sauvegarde

### Configuration

```bash
dossiers=("/etc" "/home")
destination="/tmp/backups"
date_du_jour=$(date +%Y-%m-%d)
```

- `dossiers` contient la liste des dossiers à sauvegarder ;
- `destination` indique où stocker les archives ;
- `date_du_jour` permet de nommer les fichiers avec la date.

### Préparation

```bash
mkdir -p "$destination"
```

Crée le dossier de destination s’il n’existe pas.

### Nom d’archive

```bash
nom_archive=$(echo "$dossier" | tr '/' '_')
fichier="${destination}/${nom_archive}-${date_du_jour}.tar.gz"
```

Si :

```bash
dossier="/home/kali/Documents"
```

Alors :

```txt
nom_archive="_home_kali_Documents"
```

Et le fichier final ressemble à :

```txt
/tmp/backups/_home_kali_Documents-2026-05-25.tar.gz
```

---

## Variante : archive avec nom de dossier propre

Une autre approche consiste à récupérer seulement le nom final du dossier avec `basename`.

```bash
dossier="/home/kali/Documents"
nom_dossier=$(basename "$dossier")

echo "$nom_dossier"
```

Résultat :

```txt
Documents
```

Exemple d’archive :

```bash
destination="/tmp/backups"
date_du_jour=$(date +%Y%m%d)
nom_dossier=$(basename "$dossier")
archive="$destination/${date_du_jour}_${nom_dossier}.tar.gz"

tar -czf "$archive" "$dossier"
```

Résultat possible :

```txt
/tmp/backups/20260525_Documents.tar.gz
```

---

## Variante : archive avec timestamp complet

```bash
destination="/tmp/backups"
dossier="$1"

mkdir -p "$destination"

archive="$destination/$(date +%Y%m%d-%H%M%S).tar.gz"

tar -czf "$archive" "$dossier"

echo "Archive créée : $archive"
```

Ce format est utile si plusieurs sauvegardes peuvent être lancées le même jour.

---

## Planifier la sauvegarde avec cron

Exemple : lancer la sauvegarde tous les jours à minuit.

```bash
0 0 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

Avant de l’ajouter à `cron`, teste toujours manuellement :

```bash
/home/user/scripts/backup.sh
```

Puis vérifie le log :

```bash
tail -n 50 /home/user/logs/backup.log
```

---

## Cas pratique : renommer des fichiers en masse

Objectif : renommer tous les fichiers `.jpeg` en `.jpg`.

```bash
#!/bin/bash

dossier=${1:-.}
compteur=0

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

## Explication du renommage

```bash
dossier=${1:-.}
```

Utilise le premier argument si fourni, sinon le dossier courant `.`.

```bash
for fichier in "$dossier"/*.jpeg; do
```

Parcourt les fichiers `.jpeg`.

```bash
[[ -f "$fichier" ]] || continue
```

Ignore ce qui n’est pas un fichier.

```bash
nouveau="${fichier%.jpeg}.jpg"
```

Supprime `.jpeg` à la fin du nom, puis ajoute `.jpg`.

```bash
mv "$fichier" "$nouveau"
```

Renomme le fichier.

!!! warning "Avant un renommage massif"
    Teste toujours sur un dossier de copie avant de lancer sur tes vrais fichiers.

---

## Cas pratique complet : reconnaissance réseau autorisée

Ce script réunit plusieurs notions du cours :

- vérification d’argument ;
- fonctions ;
- boucles `for` ;
- conditions ;
- `case` ;
- pipes ;
- compteurs ;
- redirections ;
- récupération de sortie dans des variables.

Il identifie les adresses IP associées à un domaine, tente d’identifier la plage réseau correspondante, puis propose de pinguer les hôtes découverts.

!!! warning "Cadre légal"
    Ce script doit être utilisé uniquement sur des domaines, systèmes et plages réseau qui t’appartiennent ou pour lesquels tu as une autorisation explicite.

    Outils requis selon l’usage : `host`, `whois`, `grep`, `awk`, `tee`, `prips`, `ping`.

---

## Script CIDR complet

```bash
#!/bin/bash

# =============================================================================
# Nom         : cidr.sh
# Description : Identification d'IP, plage réseau et hôtes joignables
# Utilisation : ./cidr.sh <domain>
# =============================================================================

# 1. Vérifier l'argument
if [[ $# -eq 0 ]]; then
    echo -e "You need to specify the target domain.\n" >&2
    echo -e "Usage:\n\t$0 <domain>" >&2
    exit 1
else
    domain="$1"
fi

# 2. Identifier la plage réseau d'une ou plusieurs IP
network_range() {
    for ip in $ipaddr; do
        netrange=$(whois "$ip" | grep "NetRange\|CIDR" | tee -a CIDR.txt)
        cidr=$(whois "$ip" | grep "CIDR" | awk '{print $2}')

        if [[ -z "$cidr" ]]; then
            echo "Aucun CIDR trouvé pour $ip" >&2
            continue
        fi

        cidr_ips=$(prips "$cidr")

        echo -e "\nNetRange for $ip:"
        echo -e "$netrange"
    done
}

# 3. Pinguer les hôtes découverts et compter le résultat
ping_host() {
    hosts_up=0
    hosts_total=0

    if [[ -z "${cidr_ips:-}" ]]; then
        echo "Erreur : aucune plage CIDR chargée. Lancez d'abord l'option 1." >&2
        return 1
    fi

    echo -e "\nPinging host(s):"

    for host in $cidr_ips; do
        if ping -c 2 "$host" > /dev/null 2>&1; then
            echo "$host is up."
            ((hosts_up++))
        else
            echo "$host is down."
        fi

        ((hosts_total++))
    done

    echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# 4. Identifier l'IP du domaine
hosts=$(host "$domain" | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)
ipaddr=$(host "$domain" | grep "has address" | cut -d" " -f4 | tr "\n" " ")

if [[ -z "$ipaddr" ]]; then
    echo "Erreur : aucune adresse IP trouvée pour $domain" >&2
    exit 1
fi

echo -e "Discovered IP address(es):\n$ipaddr\n"

# 5. Menu d'options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -r -p "Select your option: " opt

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
    *)
        exit 0
        ;;
esac
```

---

## Ce que le script CIDR illustre

| Notion | Exemple dans le script |
|---|---|
| argument obligatoire | `./cidr.sh <domain>` |
| message d’usage | `Usage:\n\t$0 <domain>` |
| fonction | `network_range()` et `ping_host()` |
| boucle `for` | parcours des IP et hôtes |
| condition | vérification de `$cidr`, `$ipaddr`, `$cidr_ips` |
| redirection erreur | `>&2` |
| pipe | `host`, `grep`, `cut`, `tee` |
| compteur | `hosts_up`, `hosts_total` |
| menu | `case "$opt" in ... esac` |
| fichier de sortie | `discovered_hosts.txt`, `CIDR.txt` |

---

## Limites du script CIDR

Ce script est pédagogique. Il n’est pas parfait.

Points à améliorer :

- gérer proprement plusieurs CIDR ;
- vérifier la présence des outils (`whois`, `prips`, `host`) ;
- éviter de pinguer de très grandes plages ;
- ajouter une option de sortie fichier ;
- limiter le bruit réseau ;
- ajouter un mode dry-run ;
- ajouter des logs ;
- mieux gérer les erreurs de résolution DNS ;
- encadrer encore plus clairement le périmètre autorisé.

!!! danger "Important"
    Ne scanne jamais une plage réseau sans autorisation.  
    Même un simple ping sweep peut être considéré comme intrusif selon le contexte.

---

## Template de script propre

Tu peux utiliser ce modèle pour démarrer un script Bash propre.

```bash
#!/bin/bash
# =============================================================================
# Nom         : mon_script.sh
# Description : [Ce que fait le script]
# Utilisation : ./mon_script.sh [options] <arguments>
# =============================================================================

set -euo pipefail

# --- Fonctions ---
afficher_aide() {
    echo "Utilisation : $0 [options] <argument>"
    echo "  -h, --help    Afficher cette aide"
}

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

erreur() {
    echo "Erreur : $1" >&2
}

# --- Vérification des arguments ---
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

# --- Programme principal ---
log "Début du script"

# ...
# Ton code ici
# ...

log "Fin du script"
```

---

## Mini-lab 1 : créer une tâche cron de test

Objectif : écrire la date dans un fichier toutes les minutes.

Créer le script :

```bash
mkdir -p "$HOME/scripts" "$HOME/logs"

cat > "$HOME/scripts/date_test.sh" << 'EOF'
#!/bin/bash
echo "Exécution : $(date '+%Y-%m-%d %H:%M:%S')"
EOF

chmod +x "$HOME/scripts/date_test.sh"
```

Ajouter dans `crontab -e` :

```bash
* * * * * /home/user/scripts/date_test.sh >> /home/user/logs/date_test.log 2>&1
```

Adapter `/home/user` à ton vrai chemin.

Vérifier :

```bash
tail -f "$HOME/logs/date_test.log"
```

---

## Mini-lab 2 : générer un rapport de dossier

Objectif : créer un script `rapport_dossier.sh`.

Contraintes :

1. le script attend un dossier en argument ;
2. il vérifie que le dossier existe ;
3. il compte les éléments ;
4. il écrit un rapport dans `rapport.txt`.

---

### Correction possible

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Erreur : veuillez renseigner un dossier" >&2
    echo -e "Usage :\n\t$0 <dossier>"
    exit 1
fi

if [[ ! -d "$1" ]]; then
    echo "Erreur : $1 n'est pas un dossier valide" >&2
    exit 1
fi

nbr_elements=$(ls "$1" | wc -l)

{
    echo "Date : $(date)"
    echo "Dossier : $1"
    echo "Nombre d'éléments : $nbr_elements"
} > rapport.txt

cat rapport.txt
```

---

## Mini-lab 3 : sauvegarde datée

Objectif : créer un script `backup_one.sh`.

Contraintes :

1. le script attend un dossier ;
2. il vérifie que le dossier existe ;
3. il crée `/tmp/backups` si nécessaire ;
4. il crée une archive datée ;
5. il affiche le chemin de l’archive.

---

### Correction possible

```bash
#!/bin/bash

if [[ -z "$1" ]]; then
    echo "Usage : $0 <dossier>" >&2
    exit 1
fi

dossier="$1"

if [[ ! -d "$dossier" ]]; then
    echo "Erreur : $dossier n'est pas un dossier valide" >&2
    exit 1
fi

destination="/tmp/backups"
mkdir -p "$destination"

nom_dossier=$(basename "$dossier")
date_jour=$(date +%Y%m%d-%H%M%S)
archive="$destination/${date_jour}_${nom_dossier}.tar.gz"

tar -czf "$archive" "$dossier"

echo "Archive créée : $archive"
```

---

## Mini-lab 4 : renommage de fichiers

Objectif : créer un script `jpeg_to_jpg.sh`.

Contraintes :

1. le script accepte un dossier en argument ;
2. s’il n’y a pas d’argument, utiliser le dossier courant ;
3. renommer les fichiers `.jpeg` en `.jpg` ;
4. afficher le nombre de fichiers renommés.

---

### Correction possible

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

echo "$compteur fichier(s) renommé(s)."
```

---

## Erreurs classiques

### Ne pas tester le script avant cron

À éviter :

```bash
0 0 * * * /home/user/scripts/backup.sh
```

sans avoir testé :

```bash
/home/user/scripts/backup.sh
```

Toujours tester manuellement avant de planifier.

---

### Oublier les logs cron

À éviter :

```bash
0 0 * * * /home/user/scripts/backup.sh
```

Préférer :

```bash
0 0 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

---

### Utiliser des chemins relatifs dans cron

À éviter :

```bash
0 0 * * * ./backup.sh
```

Préférer :

```bash
0 0 * * * /home/user/scripts/backup.sh
```

---

### Écraser un rapport sans le vouloir

```bash
commande > rapport.txt
```

écrase le fichier.

Si tu veux ajouter :

```bash
commande >> rapport.txt
```

---

### Lancer un scan sans validation

À éviter :

```bash
nmap -F -sV "$target"
```

sans vérifier que la cible existe ou qu’elle est autorisée.

Préférer au minimum :

```bash
if [[ -z "$target" ]]; then
    echo "Erreur : aucune cible fournie" >&2
    exit 1
fi
```

Et surtout : rester dans un cadre autorisé.

---

## Cheat sheet

| Élément | Usage |
|---|---|
| `crontab -e` | éditer les tâches cron |
| `crontab -l` | lister les tâches cron |
| `* * * * * commande` | syntaxe générale cron |
| `0 0 * * * commande` | tous les jours à minuit |
| `>> log.txt 2>&1` | journaliser sortie + erreurs |
| `date +%Y%m%d` | date compacte |
| `date '+%Y-%m-%d %H:%M:%S'` | date lisible avec heure |
| `mkdir -p "$destination"` | créer un dossier si nécessaire |
| `{ echo ...; } > fichier` | écrire plusieurs lignes dans un fichier |
| `tar -czf archive.tar.gz dossier` | créer une archive compressée |
| `basename "$dossier"` | récupérer le nom final d’un chemin |
| `tr '/' '_'` | remplacer les `/` par `_` |
| `${var:-defaut}` | valeur par défaut |
| `${fichier%.jpeg}.jpg` | remplacer une extension finale |
| `tee fichier.txt` | afficher et sauvegarder |
| `case "$choix" in ... esac` | gérer un menu ou des options |

---

## Checklist d’automatisation

Avant de considérer ton automatisation comme propre, vérifie :

- [ ] le script fonctionne manuellement ;
- [ ] les arguments sont vérifiés ;
- [ ] les chemins sont absolus si le script tourne via cron ;
- [ ] les erreurs sont envoyées vers `stderr` ;
- [ ] la sortie est journalisée ;
- [ ] les fichiers générés ont un nom clair ;
- [ ] les dates sont au bon format ;
- [ ] les dossiers de destination sont créés avec `mkdir -p` ;
- [ ] les fichiers importants ne sont pas écrasés sans raison ;
- [ ] les commandes sensibles sont utilisées uniquement dans un cadre autorisé ;
- [ ] le script a été testé avec un cas normal et un cas d’erreur.

---

## Résumé

Automatiser, c’est transformer une tâche répétitive en script fiable.

`cron` permet de planifier :

```bash
0 0 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

Les logs sont indispensables :

```bash
>> log.txt 2>&1
```

Les dates permettent de nommer proprement les fichiers :

```bash
date +%Y%m%d-%H%M%S
```

Les scripts concrets combinent plusieurs notions :

- arguments ;
- conditions ;
- fonctions ;
- boucles ;
- tableaux ;
- redirections ;
- logs ;
- erreurs ;
- `case`.

Le réflexe important :

> Un script automatisé doit être testé, journalisé et prévisible.

---

## Pour aller plus loin

- Fiche précédente : [Déboguer et écrire des scripts propres](debug.md).
- Fiche suivante : [Recettes Bash utiles](recettes-bash.md).
- Retour au pilier [Bash](index.md).