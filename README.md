# SOC — Détection Samba, DHCP et IDS : parcours complet

Ce dépôt est à la fois un **mémoire** (la démarche de bout en bout) et un **tuto à suivre** pour monter, au sein d'un SOC, la détection sur l'**AD Samba** (+ SSO LemonLDAP), le **DHCP** et l'**IDS Firepower**, à partir des logs et avec un SIEM en lecture seule.

Contenu **générique** : aucune donnée réelle (IP, compte, hostname, organisation).

## Parcours (dans l'ordre)
1. [00 - Contexte et objectifs](00-contexte-objectifs.md)
2. [01 - Architecture : Samba AD, LemonLDAP, DHCP, IDS Firepower](01-architecture-samba-dhcp-ids.md)
3. [02 - Cartographier l'infra à partir des logs](02-cartographier-depuis-logs.md)
4. [03 - Lire les logs, évaluer la complétude, créer les propriétés/regex](03-lire-et-evaluer-les-logs.md)
5. [04 - Référentiel de use cases](04-referentiel-use-cases.md)
6. [05 - Du use case à la règle (workflow)](05-de-use-case-a-regle.md)
7. [06 - Couverture MITRE ATT&CK](06-couverture-mitre.md)
8. [07 - Bon SIEM et amélioration collecte/parsing IDS](07-bon-siem-et-collecte-ids.md)
9. [08 - Roadmap et suite](08-roadmap-et-suite.md)
10. [09 - Glossaire](09-glossaire.md)

## Référentiels de détection
- [referentiel-alertes.md](referentiel-alertes.md) / [referentiel-alertes.csv](referentiel-alertes.csv) — catalogue unifié (format ID | Alerte | Criticité | Etat | Log source | Champs importants | Commentaire).
- Catalogues par domaine : [samba.md](samba.md), [dhcp.md](dhcp.md), [ids.md](ids.md).

## Démarche en une phrase
Comprendre l'infra depuis les logs → cartographier et repérer les angles morts → lire et compléter les logs (parsing/regex) → écrire les use cases → les transformer en règles (données, exclusions, test, tuning, playbook) → piloter (MITRE, KPI) et améliorer en continu.

## Principe directeur sur chaque source
Qu'est-ce que le log contient ? Est-ce complet pour détecter ? Est-ce au bon endroit (bon SIEM) ?

## Bonnes pratiques du dépôt
- Garder ce dépôt **privé**.
- Ne **jamais** y mettre de valeur réelle (IP, compte, hostname, organisation).
- Faire vivre la colonne **Etat** des use cases (Etude → En cours → Actif).
