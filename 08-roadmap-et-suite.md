# 08 - Roadmap et suite (après les use cases)

## Roadmap de mise en œuvre
1. **Analyse d'écart collecte & parsing** : par use case, la source est-elle onboardée ? le champ parsé ? → liste des propriétés/regex à créer et des sources à brancher. C'est le vrai blocage entre le catalogue et des règles en production.
2. **Priorisation** : impact / effort / disponibilité de la donnée. Commencer par les **Majeure dont les champs sont déjà parsés**.
3. **Reference sets** (voir [05](05-de-use-case-a-regle.md)).
4. **Playbooks de réponse** (SOAR).
5. **Activation progressive** + tuning.

## Ce qu'on peut proposer ensuite (et auquel on ne pense pas toujours)
- **Matrice de couverture MITRE** entretenue (ATT&CK Navigator) → [06](06-couverture-mitre.md).
- **KPI SOC** : MTTD, MTTR, taux de faux positifs, couverture, volume d'offenses, part des sources onboardées. Dashboards par use case et par actif critique.
- **Tests de détection** (purple team / atomic red team) : rejouer les techniques pour **prouver** que les règles se déclenchent, et faire du **détection gap analysis**.
- **UEBA / détection d'anomalie** : baselines comportementales (volume, horaires, accès) en complément des règles à seuil.
- **Threat intelligence** : feeds IOC, reference sets de comptes/IP compromis, enrichissement GeoIP/réputation.
- **Veille CVE et virtual patching** : process pour réagir vite à une CVE critique (ex. une RCE applicative) — détection IDS + règle WAF temporaire + patch.
- **Gouvernance des règles** : workflow spec → test → validation RSSI → prod → revue ; versionnage (ce dépôt) et cycle de vie (colonne Etat).
- **Amélioration continue de la collecte/parsing** → [07](07-bon-siem-et-collecte-ids.md).
- **Traitement des angles morts** : chiffré non inspecté, est-ouest, segments non collectés.
- **Révocation de session et quotas applicatifs** (leçons d'incident) : au-delà de la détection, pousser les mesures de réduction d'impact.
- **Couverture 24/7** : détection et réponse hors heures ouvrées (blocage/alerte automatisés, astreinte).
- **Formation / transfert** : documenter et former les N1/N2 sur ces use cases (ce dépôt sert de support).
- **Restitution** : schéma réseau + matrice angles morts + couverture MITRE + KPI = le dossier à présenter au manager/RSSI.

## Le mot de la fin
Ce dépôt est à la fois un **mémoire** (la démarche, de la compréhension de l'infra à la détection) et un **tuto à suivre** (ordre des étapes). Il se fait vivre : on met à jour la colonne **Etat** des use cases, on ajoute les nouvelles règles, on actualise la couverture MITRE.
