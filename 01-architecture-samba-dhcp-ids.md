# 01 - Architecture : Samba AD, LemonLDAP, DHCP, IDS Firepower

## Samba AD = l'annuaire et l'authentification
Samba AD est une réimplémentation open source d'Active Directory sur Linux. Il **stocke et valide les identités** :
- **LDAP** (comptes, groupes, machines, OU) sur 389/636,
- **KDC Kerberos** (tickets) sur 88,
- **NTLM**, **DNS** du domaine, **jonction de domaine**, **réplication** entre DC.
Équivalent Microsoft : **AD DS**.

**Multi-maître** : tous les DC sont inscriptibles et se répliquent. Les « primaires » détiennent les **rôles FSMO** et servent de référence ; les autres sont des **réplicas** (haute dispo, charge, multi-site). On repère les primaires car eux seuls émettent des événements de **réplication / opérations de rôle**.

## LemonLDAP::NG = le SSO web devant les applications
LemonLDAP **ne stocke pas les identités**. C'est un portail d'authentification unique (SSO) et de fédération pour les **applications web** : il authentifie l'utilisateur **contre Samba (bind LDAP)**, récupère ses groupes, applique **ses propres règles d'autorisation**, crée une session et donne accès aux applis (SAML/OIDC/en-têtes). Équivalent Microsoft : **AD FS**.

Distinction clé : **Samba authentifie** (qui tu es) ; **LemonLDAP autorise** (à quelle appli tu accèdes). Plusieurs nœuds LemonLDAP = haute dispo derrière un répartiteur, sessions partagées. Le **compte de service LemonLDAP** qui binde en LDAP sur les DC est le fil qui relie les deux couches.

## DHCP = la première marche
Le DHCP attribue l'IP et distribue les options réseau, dont le **DNS = les DC**. C'est lui qui permet à une machine de **trouver le domaine** (via les enregistrements SRV) puis de s'authentifier. Il n'a **pas de username** : on raisonne par **MAC, IP attribuée, hostname (option 12), type de message**. Pour le SOC, le DHCP est le **pont MAC ↔ IP ↔ hostname** qui permet d'attribuer une IP à une machine à un instant T (crucial en investigation, les IP tournant).

Chaîne complète d'un poste qui arrive :
```
DHCP (IP + DNS=les DC) -> DNS (trouve les DC) -> Samba (Kerberos/LDAP) -> LemonLDAP (accès applis web)
```

## IDS Firepower (Snort) + FMC
Le moteur de détection est **Snort**, intégré aux boîtiers **FTD** (Firepower Threat Defense), pilotés par un **FMC** (Firepower Management Center).
- **Interfaces et zones** : les interfaces sont regroupées en **Security Zones**. Une sonde **passive** (SPAN/TAP) peut écouter **plusieurs zones** à la fois (SPAN agrégé ou sous-interfaces VLAN) ; une interface de monitoring n'a **pas d'IP** sur le segment surveillé. « Entre 2 interfaces » ne vaut que pour le mode **inline**.
- **Collecte** : chaque sonde envoie au **FMC** par le **sftunnel (TCP 8305, TLS)** ; QRadar **tire** les événements du FMC en **eStreamer (TCP 8302, TLS + certificat)**. Donc **tout passe par le FMC** : dans QRadar, un **seul log source = le FMC**, et la **sonde est un champ** (Device), pas l'émetteur réseau. Les IP/ports du **trafic analysé** sont bien présents comme champs.
- **Types d'événements** : Intrusion (alertes Snort), Connection (flux : src/dst, zones, application, octets), Security Intelligence (réputation), File/Malware.
- **« drop » en passif** : c'est un « would have dropped » (action que Snort aurait prise), pas un blocage réel ; un vrai drop = sonde inline ou pare-feu.

## Vue d'ensemble
```
Utilisateurs
   | HTTPS            +-----------+     LDAP 389/636     +------------------+
   +---> LemonLDAP ---| bind svc  |-------------------->|  Samba AD (DC)   |
   |     (SSO web)    +-----------+                      | 2 primaires+réplicas
   |                                                     |  Kerberos 88 / DNS
   +---> DHCP (IP + DNS=les DC) ------------------------>+  réplication DC<->DC
                                                            ^
   Trafic réseau --- (SPAN/TAP) ---> Sondes FTD --sftunnel 8305--> FMC --eStreamer 8302--> QRadar
```
