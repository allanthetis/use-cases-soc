# 04 - Référentiel de use cases

Le référentiel est la **spécification** de la détection : chaque ligne = une règle à construire.

## Fichiers
- **[`referentiel-alertes.md`](referentiel-alertes.md)** / **[`referentiel-alertes.csv`](referentiel-alertes.csv)** : le référentiel unifié (Samba + DHCP + IDS + SSO), au format ID | Alerte | Criticité | Etat | Log source | Champs importants | Commentaire.
- Catalogues par domaine (vue détaillée) : [`samba.md`](samba.md), [`dhcp.md`](dhcp.md), [`ids.md`](ids.md).

## Convention
- **Nommage** : `* – SOC — <thème> — <précision>`.
- **Criticité** : Majeure / Mineure / Indicateur.
- **Etat** : Etude -> En cours -> Actif (à faire vivre au fil de la mise en production).
- Les **seuils sont indicatifs**, à calibrer sur le bruit réel.

## Couverture
- **Samba** : auth (bruteforce, spraying, voyage impossible, downgrade NTLM), Kerberos (Kerberoasting RC4, AS-REP, Golden/Silver ticket), LDAP/dsdb (DCSync, groupes privilégiés, reset mdp, GPO, SPN ajouté), SMB (partages admin, accès/suppression massifs), comptes, infra DC (rogue DC, réplication, FSMO, DC muet), corrélation (compte compromis, pic de volume, multi-zones).
- **SSO LemonLDAP** : bruteforce portail, accès hors population, anomalie de session.
- **DHCP** : rogue DHCP, starvation, MAC flooding, liste noire, nouvelle MAC, relais inattendu.
- **IDS Firepower** : signature critique, exploitation de CVE, RCE web, montée alerte->blocage, réputation, beaconing/C2, exfiltration, malware, sonde muette, scan, JA3/SNI, flux inter-zones.

## D'un use case à une règle
Voir [05](05-de-use-case-a-regle.md). En résumé : vérifier que la **source** et les **champs** existent (sinon regex d'abord), traduire le **Commentaire** en logique de règle, ajouter **exclusions** (reference sets) et **playbook**, tester, tuner, activer.
