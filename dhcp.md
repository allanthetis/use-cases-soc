# Use cases DHCP

> Le DHCP n'a pas de username : on raisonne par MAC, IP attribuée, hostname (option 12), type de message (DISCOVER/OFFER/REQUEST/ACK/NAK/DECLINE/RELEASE), serveur et relais (giaddr). Seuils indicatifs à calibrer.

| Alerte | Criticité | Etat | Log source | Champs important | Commentaire |
|---|---|---|---|---|---|
| Serveur DHCP pirate (rogue) | Critique | À créer | DHCP | IP serveur, OFFER/ACK, MAC | OFFER/ACK depuis une IP non référencée ; reference set des serveurs DHCP légitimes |
| Épuisement du pool (DHCP starvation) | Haute | À créer | DHCP | type message, MAC, état du pool | Flood de DISCOVER depuis de nombreuses MAC + pool épuisé |
| MAC flooding / spoofing | Haute | À créer | DHCP | MAC, IP, volume | 1 hôte présentant un nombre anormal de MAC distinctes |
| Bail attribué à une MAC sur liste noire | Haute | À créer | DHCP | MAC, reference set blacklist | Équipement interdit qui obtient une IP |
| Pic de requêtes DHCP (DoS) | Moyenne | À créer | DHCP | volume, type message | Volume anormal sur une courte fenêtre |
| Nouvelle MAC inconnue (asset non référencé) | Moyenne | À créer | DHCP | MAC, hostname | Comparaison à un reference set des MAC connues |
| Relais DHCP (giaddr) inattendu | Moyenne | À créer | DHCP | giaddr, source | Relais non référencé = segmentation contournée |
| Conflits/rejets d'adresses en volume (DECLINE/NAK) | Moyenne | À créer | DHCP | type message, IP | Peut trahir un conflit ou une attaque |
| Changement fréquent de hostname pour une MAC | Moyenne | À créer | DHCP | MAC, hostname (option 12) | Spoofing d'identité |
| Option/hostname DHCP suspect (injection) | Moyenne | À créer | DHCP | option 12/60, payload | Payload d'injection dans le hostname |
| Utilisation du pool proche de la saturation | Basse | À créer | DHCP | taux d'utilisation du pool | Alerte capacité/exploitation |
| Requête DHCP hors plage horaire sur segment sensible | Basse | À créer | DHCP | horodatage, segment | Activité réseau anormale la nuit |
