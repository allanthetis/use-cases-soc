# 02 - Cartographier l'infra à partir des logs

Objectif : reconstruire « qui est lié à quoi » et un schéma réseau, à partir des seuls logs (SIEM lecture seule). Méthode : **tableau d'abord, schéma ensuite**.

## Les sources d'or dans le SIEM
- **Log Activity (events)** : authentification, web, DNS, firewall, process.
- **Network Activity (flows)** : qui parle à qui, sur quel port, combien d'octets = la matière du schéma.
- **Asset Model** : IP, hostname, OS, ports/services, dernier vu.
- **Offenses + SOAR** : incidents déjà traités = actifs et relations confirmées.

## Méthode en Add Filter (filtres structurés)
Rappel : en Add Filter, deux paramètres différents se combinent en **ET** ; le **OU** n'existe que dans un même paramètre via **« Is any of »**.

1. **Inventaire de la collecte** : Group By **Log Source Type** puis **Log Source**. Dresse la liste des briques qui parlent (AD, DHCP, DNS, firewall, IDS, applis). Une source attendue mais absente = **angle mort**.
2. **Serveurs centraux** : côté flows, Group By **Destination IP + Destination Port**, trier par nombre de **clients distincts**. Les plus contactés = AD, DNS, applis majeures, proxy.
3. **Zones** : Group By **Source Network / Destination Network** pour dégager les segments et les passages entre zones.
4. **Par application** (du nom au périmètre) : chercher le nom dans les Log Sources / URL / DNS → IP serveurs → pivoter dans les flows (clients, dépendances par port : 1433 SQL, 389/88 LDAP/Kerberos, 445 SMB, 443 web) → Asset Model pour nommer → utilisateurs et exposition Internet.

## Cas particulier IDS (sondes Firepower)
- **Log source** = le FMC ; la **sonde** = le champ **Device**.
- Pour situer une sonde : Group By **Device** + **Ingress Interface** + **Ingress Security Zone** → les zones qu'elle observe (sa couverture). En passif, c'est un point d'écoute, pas un lien entre 2 zones.
- Reconstruire le réseau : les **Connection events** (src/dst + zones + application) donnent qui parle à qui.

## Construire le schéma (3 couches)
- **Données** : zones + pare-feu + serveurs, arêtes étiquetées des ports.
- **Supervision IDS** : sondes posées en écoute à côté des liens (passif).
- **Collecte** : sondes -> FMC -> SIEM (dessinée à part).

## Le livrable qui valorise : les angles morts
Lister toutes les zones/sous-réseaux, puis marquer ceux **couverts par aucune source / aucune sonde** = trafic non inspecté. C'est le constat le plus attendu par un manager.

| Zone / sous-réseau | Couvert par | Angle mort ? | Remarque |
|---|---|---|---|
| | | | |
