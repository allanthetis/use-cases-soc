# 10 - Objectifs opérationnels FMC (IDS Firepower)

> Guide d'actions précises dans la console FMC (Secure Firewall Management Center) pour remplir les objectifs IDS : maîtriser les signatures, les règles par application, proposer de nouvelles règles, sauvegarder config/PCAP, vérifier stockage/bande passante, identifier les plages réseaux. Plus deux procédures transverses : suivre une IP publique de bout en bout (FMC + SIEM + Kibana) et dédupliquer la signature d'une application.
>
> Contexte : Firepower/FTD en **mode passif** (SPAN/TAP) -> les règles génèrent des événements, elles ne bloquent pas. Chemins donnés pour FMC 7.x ; les libellés varient selon la version. Toute donnée sensible est à traiter **en local** et anonymisée (noms d'organisation -> `*`).
>
> Chaque section indique **où cliquer**, **quoi lire**, et **quoi reporter dans l'Excel** (structure en annexe).

---

## 1. Maîtriser les signatures

Une signature = une **règle d'intrusion Snort**, identifiée par un **GID:SID**, portant un message, une catégorie, une sévérité et parfois une référence CVE.

Actions :
1. `Policies > Intrusion` : ouvrir la/les **politiques d'intrusion** appliquées. Noter la **base policy** (Connectivity over Security / Balanced / Security over Connectivity / Maximum Detection) : elle fixe le jeu de règles actif par défaut.
2. `Objects > Intrusion Rules` (Snort 3) : parcourir les règles par **catégorie**. Pour une règle, lire : **GID:SID**, **message**, **classtype**, **référence (CVE/URL)**, **état** (Generate Events / Drop and Generate / Disabled).
3. `Devices > Device Management > [sonde] > Inspection` (ou Device) : confirmer la **version de Snort** (2 ou 3).
4. `System > Updates > Rule Updates` : date et fréquence des mises à jour de signatures (**LSP** pour Snort 3, SRU pour Snort 2) = fraîcheur de la détection.
5. Identifier les **overrides** : règles dont l'état a été modifié par rapport à la base policy (filtrer sur « Overridden »).

En passif : vérifier que les règles sont bien en **Generate Events** (et non Drop, inopérant sans inline).

-> Reporter dans l'Excel : onglet **Signatures**.

---

## 2. Maîtriser les règles par application

« Application » = identification L7 via **AppID** (détecteurs d'application). Trois champs distincts : **Application Protocol**, **Client**, **Web Application**.

Actions :
1. `Policies > Access Control > [politique]` : repérer les **règles qui matchent sur Applications** (onglet **Applications** de chaque règle). Noter l'application, l'action, les zones source/destination.
2. `Policies > Application Detectors` : liste des **détecteurs**. Pour chaque application, voir le(s) détecteur(s), le **Type** (Cisco-provided / User-Defined) et l'**État** (Active / Inactive).
3. `Objects > Object Management > Application Filters` : filtres applicatifs réutilisés dans les règles (par risque, type, catégorie).
4. `Analysis > Connections > Events` : colonnes **Application Protocol / Client / Web Application** pour voir ce qui est réellement identifié sur le trafic.

-> Reporter dans l'Excel : onglet **Regles_Application**.

---

## 3. Proposer de nouvelles règles

Deux leviers : règle d'**intrusion custom** (Snort) ou règle **Access Control** custom (L3/L7).

Actions - règle d'intrusion custom :
1. `Objects > Intrusion Rules > Create Rule` (syntaxe Snort 3). Base de départ : un trafic observé ou un **PCAP** (voir section 4).
2. Renseigner : action `alert` (passif), protocole, flux (source/dest/ports), options de contenu (`content`, `pcre`, `flow`, `http_uri`...), `msg`, `classtype`, `sid` (plage locale), `rev`.
3. Ajouter la règle à la **politique d'intrusion** et la mettre en **Generate Events** (jamais Drop directement en passif).
4. **Deploy**, puis mesurer le **taux de faux positifs** sur du trafic réel avant d'industrialiser.

Actions - règle Access Control custom (si filtrage/visibilité L7) :
1. `Policies > Access Control > [politique] > Add Rule` : conditions (zones, réseaux, ports, applications), action **Monitor** (en passif, pour générer l'événement sans bloquer).
2. **Deploy**.

Méthode : partir du besoin (use case du référentiel) -> trafic/PCAP -> règle en alerte -> validation faux positifs -> documentation.

-> Reporter dans l'Excel : onglet **Nouvelles_regles**.

---

## 4. Sauvegarder config / PCAP (partie FMC)

### Config
1. `System > Tools > Backup/Restore` : créer un **Backup Profile FMC** (sauvegarde de la configuration du FMC).
2. Ajouter la sauvegarde des **devices gérés** (managed device backup).
3. **Planifier** la sauvegarde (`System > Tools > Scheduling`) et définir la **destination de stockage** (local + export distant SCP/NFS).

### PCAP - deux niveaux
- **Paquet déclencheur d'un événement IPS** : activer **Store packets** dans la politique d'intrusion (Advanced Settings). Il devient téléchargeable depuis `Analysis > Intrusions > Events` (détail de l'événement -> Download Packet).
- **Capture complète sur la sonde FTD** : `Devices > Troubleshoot` (outil de capture) ou CLI FTD `capture <nom> interface <if> match ip host <ip> ...` puis `copy /pcap capture:...` pour exporter.

-> Reporter dans l'Excel : onglet **Sauvegardes**.

### 4a. Vérifier le stockage et la bande passante
1. `System > Health > Monitor` : **disk usage** du FMC et des sondes, **throughput** par sonde, stats interfaces.
2. `System > Configuration > Database` : **rétention** de la base d'événements (nb de jours / nb d'enregistrements) = combien de temps on garde les événements et PCAP.
3. Estimer l'espace PCAP nécessaire (volume x rétention) ; vérifier qu'il tient dans le stockage.
4. **Bande passante du sftunnel** (port **8305**) FTD <-> FMC : vérifier qu'elle absorbe le débit d'événements (sinon perte/retard). Health Monitor + débit par sonde.

-> Reporter dans l'Excel : onglet **Stockage_BP**.

### 4b. Identifier les plages réseaux (cartographie) -> voir section 5.

---

## 5. Cartographier le réseau depuis l'IDS

Levier principal : la **découverte réseau (Network Discovery)**, qui construit la carte **passivement** depuis le trafic vu par les sondes.

1. **Localiser les sondes** : `Devices > Device Management > [chaque FTD] > Interfaces` -> interfaces passives (SPAN/TAP), **Security Zone** et mode. On sait quelle patte de quelle sonde écoute quelle zone.
2. **Vérifier/activer la discovery** : `Policies > Network Discovery` -> règle **Discover Hosts/Applications/Users** couvrant les plages (souvent `0.0.0.0/0`). Sans elle, pas de Network Map.
3. **Inventaire auto** : `Analysis > Hosts > Network Map` (hôtes, OS, hiérarchie par sous-réseau) et `Analysis > Hosts > Hosts` / `> Servers` (services, ports, applis).
4. **Plages réelles depuis le trafic** : `Analysis > Connections > Events` avec colonnes **Initiator IP, Responder IP, Ingress Interface, Ingress Security Zone, Application** ; agréger par **/24** -> sous-réseaux vus et par quelle sonde ; les flux Initiator->Responder donnent la **segmentation**.
5. **Confronter au déclaré** : `Objects > Object Management > Network` (plages déclarées) vs ce que révèle la discovery (écarts = zones oubliées / shadow IT).
6. **Vue rapide** : `Analysis > Context Explorer`.

Limites (passif) : seul le trafic **traversant le SPAN** et **non chiffré** est vu ; l'intra-sous-réseau non spanné est aveugle.

-> Reporter dans l'Excel : onglet **Cartographie_reseau**.

---

## 6. Suivre une IP publique de bout en bout (FMC + SIEM + Kibana)

Principe : **pivoter sur IP publique + horodatage + ports** ; le **NAT** relie le public à l'interne. Chaque outil couvre un étage : FMC = réseau/IDS (zones, sondes, NAT au FTD), SIEM = logs agrégés (firewall + NAT, proxy, WAF), Kibana = couche applicative (access logs, X-Forwarded-For).

1. **Point de départ** : IP publique + fenêtre de temps + port destination (443).
2. **FMC - entrée** : `Analysis > Connections > Events`, filtre **Initiator IP = <IP publique>**. Colonnes : Initiator IP, Responder IP, **Ingress Interface, Ingress/Egress Security Zone**, Source/Dest Port, Application, **Device (sonde)**, Access Control Rule, **NAT Source IP/Port**, **NAT Destination IP/Port**. L'Ingress Zone + Device = sonde/zone périmètre ; le même flux sur une autre sonde = hop suivant ; les champs NAT donnent la cible interne réelle. Croiser `Analysis > Intrusions > Events`.
3. **SIEM - chemin logué** : rechercher Source IP = <IP publique> sur **toutes les log sources**, trié par temps -> firewall (accept + translation pre/post-NAT), reverse proxy / WAF, app. Utiliser la **Network Hierarchy** pour situer les IP internes.
4. **Kibana - couche appli** : chercher l'IP dans le champ **client IP / `X-Forwarded-For`** des access logs -> endpoint applicatif atteint, session, backend. Corréler sur l'horodatage.
5. **Reconstituer** : `IP publique -> [sonde/zone périmètre (FMC)] -> [firewall + NAT (SIEM)] -> [reverse proxy/WAF (SIEM/Kibana)] -> [serveur applicatif (Kibana)]`.

-> Reporter dans l'Excel : onglet **Suivi_IP**.

---

## 7. Identifier la signature d'une application et n'en garder qu'une

La « signature d'application » = un **détecteur d'application (AppID)**. Une appli peut être identifiée par **plusieurs détecteurs** (Cisco built-in + custom) -> doublons à consolider.

1. **Identifier l'appli** : `Analysis > Connections > Events` -> colonnes Application Protocol / Client / Web Application.
2. **Trouver ses détecteurs** : `Policies > Application Detectors`, recherche sur le nom de l'application -> tous les détecteurs, leur **Type** (Cisco-provided / User-Defined) et **État** (Active / Inactive).
3. **N'en garder qu'un** : choisir le détecteur à conserver actif ; basculer les autres en **Inactive**. Nuances : un détecteur **Cisco built-in** n'est pas toujours désactivable ; un détecteur **référencé par une règle/politique active** doit d'abord être retiré de ces références ; les détecteurs **custom** peuvent être désactivés ou supprimés.
4. **Deploy**, laisser tourner la discovery, puis vérifier sur du **nouveau trafic** que l'appli est identifiée par **le seul détecteur voulu**.

-> Reporter dans l'Excel : onglet **Detecteurs_Application**.

---

## Annexe - Structure Excel à remplir (onglets + colonnes)

**Signatures**
`GID:SID | Message | Catégorie | Classtype | Sévérité | Référence CVE | État (Generate/Drop/Disabled) | Base policy | Overridden (O/N) | Sonde(s) | Commentaire`

**Regles_Application**
`Nom règle AC | Application (Protocol/Client/WebApp) | Action | Zone source | Zone dest | Réseaux | Filtre applicatif | Commentaire`

**Nouvelles_regles**
`ID local (SID) | Type (Intrusion/AC) | Use case visé | Besoin / menace | Contenu règle (résumé) | État (alerte) | Faux positifs observés | Statut (proposée/testée/validée) | Commentaire`

**Sauvegardes**
`Objet (FMC/Device) | Type (config/PCAP) | Planification | Destination stockage | Dernière sauvegarde | Taille | Statut | Commentaire`

**Stockage_BP**
`Composant (FMC/Sonde) | Disk usage % | Rétention événements (jours) | Volume événements/j | Espace PCAP estimé | Throughput sonde | BP sftunnel (8305) | Alerte capacité (O/N) | Commentaire`

**Cartographie_reseau**
`Sous-réseau (/24) | VLAN | Security Zone | Sonde (FTD) qui le voit | Interface ingress | Hôtes clés | Services/ports | Rôle (DC/Samba/KBC/...) | Déclaré dans Network objects (O/N) | Commentaire`

**Suivi_IP**
`IP publique | Horodatage | Port dest | Sonde/zone périmètre (FMC) | NAT -> IP interne | Firewall (règle/accept) | Reverse proxy/WAF | Serveur applicatif | Endpoint (Kibana) | Outils utilisés | Verdict`

**Detecteurs_Application**
`Application | Détecteur | Type (Cisco/Custom) | État avant | Action (garder/désactiver/supprimer) | Référencé par règle (O/N) | État après | Déployé (O/N) | Commentaire`
