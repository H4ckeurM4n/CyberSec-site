# Bash / Terminal

Le terminal de A à Z, puis tout le scripting Bash. Une fois à l’aise ici, le pentest, l’OSINT, le SOC et Python deviennent beaucoup plus rapides.

Ce pilier sert à construire une base solide : manipuler le terminal, écrire des scripts propres, automatiser des tâches simples, puis réutiliser des blocs de code concrets dans des cas pratiques.

---

## Le terminal

- [Naviguer dans le terminal](naviguer-terminal.md) — se repérer, changer de dossier, lister les fichiers et comprendre les chemins.

---

## Scripting Bash

Une série progressive, une notion par fiche, de zéro jusqu’à l’automatisation.

1. [Écrire son premier script](premier-script.md) — shell, shebang, `chmod`, `exit`.
2. [Variables, affichage et saisie](variables.md) — variables, guillemets, `read`, `$(...)`.
3. [Arguments et variables spéciales](arguments.md) — `$0`, `$1`, `$#`, `$?`, `shift`.
4. [Redirections, erreurs et pipes](redirections.md) — `>`, `>>`, `2>`, `2>&1`, `|`, `tee`.
5. [Opérateurs et calculs](operateurs.md) — `$(( ))`, comparaisons, logique, tests.
6. [Conditions et tests](conditions.md) — `if`, `elif`, `else`, tests de fichiers.
7. [Les boucles](boucles.md) — `for`, `while`, `until`, `break`, `continue`, menus.
8. [Fonctions et case](fonctions.md) — fonctions, `local`, `return`, `case`, `select`.
9. [Manipuler les chaînes](chaines.md) — longueur, extraction, remplacement, extensions.
10. [Les tableaux](tableaux.md) — tableaux indexés, associatifs, parcours par index.
11. [Déboguer et écrire des scripts propres](debug.md) — `bash -n`, `bash -x`, `set -euo pipefail`.

---

## Cas pratiques

Cette section rassemble les fiches de synthèse et les recettes réutilisables.

12. [Automatisation : cron et cas pratiques](automatisation.md) — `cron`, logs, sauvegardes, renommage, rapport, script CIDR.
13. [Recettes Bash utiles](recettes-bash.md) — petits blocs prêts à réutiliser : menus, dates, archives, `tee`, `find`, `shift`, Nmap en lab.

---