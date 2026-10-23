# 05 - Du use case à la règle (workflow complet)

## Les 8 étapes
1. **Vérifier la source** : le log source est-il onboardé et envoie-t-il les événements ? (sinon, rien à détecter → demande d'onboarding).
2. **Vérifier les champs** : les champs de la colonne « Champs importants » sont-ils **parsés** ? Si non → **propriété personnalisée (regex)** d'abord (voir [03](03-lire-et-evaluer-les-logs.md)).
3. **Traduire la logique** : transformer le « Commentaire » (condition + seuil + fenêtre) en règle (building blocks, tests).
4. **Exclusions** : brancher les **reference sets** (voir plus bas) pour éviter les faux positifs.
5. **Tester** : rejouer / simuler pour vérifier que la règle se déclenche (voir [08](08-roadmap-et-suite.md), tests de détection).
6. **Tuner** : calibrer les seuils sur le bruit réel, ajuster les exclusions.
7. **Activer** : passer la règle en production (fait par l'admin ; on est en lecture seule).
8. **Playbook SOAR** : définir la réponse de l'analyste (triage, investigation, confinement, escalade).

Puis faire vivre la colonne **Etat** du référentiel (Etude → En cours → Actif).

## Reference sets à constituer (prérequis des règles)
- **Contrôleurs de domaine** (pour DCSync, rogue DC, réplication).
- **Comptes à privilèges** (Domain/Enterprise Admins...).
- **Comptes de service** (dont le compte LemonLDAP qui binde en LDAP) = exclusions.
- **Serveurs DHCP légitimes** (rogue DHCP).
- **MAC / équipements connus** (nouvelle MAC, liste noire).
- **Actifs critiques** (signature IDS sur actif sensible).
- **Comptes compromis** (threat intel) : alimenté par les feeds.
- **Plages d'administration autorisées**, **scanners de vulnérabilité autorisés**.

## Modèle de playbook (par alerte)
1. **Triage** : vrai/faux positif (contexte, exclusions, baseline).
2. **Investigation** : qui, quoi, d'où, depuis quand ; actifs et comptes liés ; corréler avec d'autres sources.
3. **Qualification** : incident ou non ; sévérité.
4. **Confinement** : révoquer les sessions (pas seulement reset mdp), isoler l'actif, bloquer l'IP.
5. **Escalade** : vers N3 / RSSI selon gravité.
6. **Capitalisation** : enrichir reference sets / threat intel, ajuster la règle.

## Tuning et faux positifs
- Baseline du bruit avant d'activer ; démarrer en **observation** si possible.
- Exclusions explicites (comptes de service, scanners, sauvegardes).
- **Revue périodique** : une règle non tunée est coupée en une semaine.
- Mesurer le **taux de faux positifs** par règle (voir KPI dans [08](08-roadmap-et-suite.md)).
