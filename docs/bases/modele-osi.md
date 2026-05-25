---
tags:
  - réseau
  - fondamentaux
---

# Le modèle OSI

Le modèle OSI découpe une communication réseau en **7 couches**, de la plus
basse (le câble physique) à la plus haute (l'application). Comprendre ce
découpage aide à savoir *où* se situe un problème ou une attaque.

## Les 7 couches

| N° | Couche | Rôle | Exemple |
|----|--------|------|---------|
| 7 | Application | Interface avec l'utilisateur | HTTP, DNS, SMTP |
| 6 | Présentation | Format et chiffrement des données | TLS, JPEG |
| 5 | Session | Ouverture/fermeture des dialogues | Sockets |
| 4 | Transport | Livraison fiable bout en bout | TCP, UDP |
| 3 | Réseau | Routage entre réseaux | IP, ICMP |
| 2 | Liaison | Communication sur le réseau local | Ethernet, MAC |
| 1 | Physique | Le signal lui-même | Câble, Wi-Fi |

!!! tip "Moyen mnémotechnique"
    De la couche 7 à 1 : **A**pplication, **P**résentation, **S**ession,
    **T**ransport, **R**éseau, **L**iaison, **P**hysique.

## Pourquoi c'est utile en sécurité

Chaque couche a ses propres attaques et ses propres défenses :

- **Couche 2** : usurpation d'adresse MAC, attaques ARP.
- **Couche 3** : usurpation d'adresse IP, scan réseau.
- **Couche 4** : scan de ports, attaques sur TCP.
- **Couche 7** : injections, attaques web.

Quand tu analyses un incident, situer le problème sur une couche te dit
immédiatement quels outils utiliser.

## Pour aller plus loin

- [Bash : naviguer dans le terminal](../bash/naviguer-terminal.md) pour lancer
  tes premiers outils réseau.
- Pilier [Pentest](../pentest/index.md) pour passer à la pratique.
