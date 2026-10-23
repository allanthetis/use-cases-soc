# 00 - Contexte et objectifs

## Mission
Monter, au sein d'un SOC, la détection sur trois périmètres d'infrastructure :
- **AD Samba** (annuaire Linux : Kerberos, LDAP, SMB, réplication) avec une couche SSO **LemonLDAP** en façade,
- **DHCP** (attribution d'adresses),
- **IDS/IPS Firepower** (sondes Snort pilotées par un FMC).

La contrainte posée : **comprendre l'infrastructure seul, à partir des logs et des incidents**, puis être capable de dresser un schéma réseau, d'identifier qui est lié à quoi, et de construire la détection.

## Contraintes
- Accès **SIEM en lecture seule** (QRadar) + **SOAR** (QRadar SOAR). On **spécifie et prépare** les règles ; l'équipe admin les **met en production**.
- Coexistence **QRadar + ELK** : l'infra va plutôt dans QRadar, l'applicatif dans ELK (voir [07](07-bon-siem-et-collecte-ids.md)).
- Données sensibles : aucun identifiant, IP, hostname ou donnée réelle dans ce dépôt. Tout est **générique**.

## Objectifs
1. Comprendre le fonctionnement de chaque brique (Samba, LemonLDAP, DHCP, Firepower). Voir [01](01-architecture-samba-dhcp-ids.md).
2. Reconstruire l'infra et le schéma réseau à partir des logs. Voir [02](02-cartographier-depuis-logs.md).
3. Lire les logs, évaluer leur complétude, créer les propriétés/regex manquantes. Voir [03](03-lire-et-evaluer-les-logs.md).
4. Construire un **référentiel de use cases** de détection. Voir [04](04-referentiel-use-cases.md).
5. Passer du use case à la **règle** (données, regex, logique, reference sets, test, tuning, playbook). Voir [05](05-de-use-case-a-regle.md).
6. Piloter : couverture MITRE, bon SIEM, KPI, amélioration continue. Voir [06](06-couverture-mitre.md), [07](07-bon-siem-et-collecte-ids.md), [08](08-roadmap-et-suite.md).

## Fil rouge sur chaque source
Trois questions, répétées pour Samba, DHCP et IDS :
1. **Qu'est-ce que le log contient ?** (format, champs)
2. **Est-ce complet pour détecter ?** (champs présents / manquants)
3. **Est-ce au bon endroit ?** (bon SIEM : infra vs applicatif)

## Leçons d'incident (générique)
Les grands incidents d'exfiltration partagent les mêmes angles morts : supervision applicative absente, pas de corrélation des signaux faibles (volume, horaire, IP, compte compromis), sessions non révoquées après compromission, SOC en heures ouvrées. La détection construite ici vise explicitement ces causes (voir les use cases de corrélation dans le [référentiel](referentiel-alertes.md)).
