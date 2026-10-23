# 07 - Bon SIEM (routage) et amélioration de la collecte / parsing IDS

## Le bon SIEM : infra vs applicatif
Principe de routage :
- **Infra / sécurité -> QRadar** : AD Samba, DHCP, DNS, IDS/IPS, pare-feu, VPN, WAF, authentification. Corrélation, offenses, SOAR.
- **Applicatif -> ELK** : logs des applis métier, gros volume, recherche/fouille.

Le critère n'est pas la techno mais l'**usage** : ai-je besoin de **corréler** pour détecter (QRadar) ou de **fouiller en masse** pour comprendre une appli (ELK) ?

Matrice à remplir : `source | nature (infra/appli) | SIEM actuel | SIEM cible | écart`. Les lignes où actuel ≠ cible sont les recommandations (appli qui sature QRadar pour rien, brique infra visible seulement dans ELK donc non corrélée).

## Améliorations collecte (IDS Firepower)
- **Dédoublonner l'ingestion** : choisir une seule voie par type d'événement (eStreamer **ou** syslog). Les deux actifs = offenses doublées + licence EPS consommée deux fois.
- **Transport fiable** : syslog en **TCP/TLS 6514** plutôt qu'UDP 514 (pertes silencieuses, non chiffré).
- **Horodatage** : NTP partout, sinon corrélation faussée (comparer l'heure sonde et l'heure SIEM).
- **Couverture** : toutes les sondes remontent ? Alerter sur l'**absence d'événements** (sonde muette).
- **Bons types activés** : surtout les **Connection events** (souvent coupés car volumineux, mais essentiels pour cartographier) + Security Intelligence + File/Malware.
- **Qualité de capture** : SPAN saturé = perte de paquets ; préférer un **TAP**, mirrorer les deux sens, couvrir le VLAN entier.

## Améliorations parsing
- **DSM à jour** (eStreamer) pour parser les nouveaux champs.
- **Champs clés extraits et indexés** : Ingress/Egress Zone & Interface, Inline Result, Signature/SID, Application, Policy, Device. Sans eux, impossible de filtrer/grouper.
- **Chasser les événements « Stored »/Unknown** (non parsés) et faire corriger.
- **Normaliser l'identité** (username cohérent, IP machine commune) via propriétés personnalisées.

## Angles morts à traiter (IDS)
- **Trafic chiffré non inspecté** : proposer le déchiffrement TLS (dans le cadre légal) ou exploiter SNI/JA3.
- **Est-ouest interne** : la sonde voit-elle le latéral, ou seulement le nord-sud ?
- **Passif -> inline** là où pertinent (les « would have dropped » montrent ce qui aurait été bloqué).

## Méthode : constater, chiffré
Toute proposition s'appuie sur une mesure tirée du SIEM (doublons mesurés, events non parsés comptés, champs manquants, décalage horaire, sondes muettes), pas sur une opinion. Formaliser dans une note : constat / risque / proposition / effort / responsable.
