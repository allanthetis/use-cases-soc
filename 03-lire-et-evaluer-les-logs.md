# 03 - Lire les logs, évaluer la complétude, créer les propriétés/regex

## Lire le brut (comprendre le format)
Pour chaque source, sortir un échantillon de **log brut** et décortiquer les champs.

### Samba (JSON audit)
Plusieurs journaux : **authentification** (JSON), **KDC** (Kerberos), **smbd/full_audit** (accès fichiers), **dsdb** (modifs d'annuaire). Champs clés du JSON d'authentification :
- `serviceDescription` (SMB2, LDAP, NETLOGON...), `authDescription` (simple bind, interactive, network, bare-NTLM, « LDAP Modify » = changement de mdp),
- `clientAccount` / `clientDomain` (fournis par le client), `becameAccount` / `becameDomain` / `becameSid` (compte réellement obtenu),
- `remoteAddress` / `localAddress`, `status` (`NT_STATUS_OK` ou erreur), `passwordType`, `workstation`.
KDC : type de requête (AS-REQ/TGS-REQ), **etype** (0x17 = RC4), erreurs de pré-auth. dsdb : opération, objet, attribut, auteur.

### DHCP
Pas de username. Champs : type de message (DISCOVER/OFFER/REQUEST/ACK/NAK/DECLINE/RELEASE), **MAC**, IP attribuée, hostname (option 12), serveur, relais (giaddr).

### IDS Firepower
Intrusion : signature/SID, action (alert/drop/would-have-dropped), source/dest, zones, sévérité, CVE, Device. Connection : src/dst, ports, application, octets, zones, JA3/SNI.

## Évaluer la complétude (les 3 colonnes)
Pour chaque champ utile : **présent dans le brut ?** / **parsé par le SIEM ?** / **nécessaire à la détection ?**
- présent + parsé + utile -> exploitable directement,
- présent mais **non parsé** -> **propriété personnalisée (regex)**,
- **absent du brut** -> augmenter la verbosité à la source (activer full_audit, audit dsdb, niveau de log IDS).

Les 5 champs minimum d'un bon log : **qui / quoi / d'où / résultat / quand**.

## Événements non parsés
Catégorie **« Stored »** (ou Unknown) = événement **reçu mais non normalisé** (aucun DSM ne le reconnaît) : les champs ne sont pas extraits, tout reste dans le payload. C'est un **trou de parsing** : mesurer le volume (filtre « Event is Unparsed » / Low Level Category = Stored), identifier la source, et faire corriger (mise à jour DSM, Log Source Extension, mapping QID).

## Créer une propriété personnalisée (regex)
Quand un champ utile est dans le brut mais non parsé :
1. Ouvrir un événement représentatif, afficher le **Payload**.
2. **Extract Property** : écrire une regex avec un **groupe de capture** `(...)`, tester en direct.
3. Nommer, associer au bon type de log source, activer (parsing/optimisation/indexation).
Bonnes pratiques : capture non gourmande `(.+?)`, ancrer sur le texte stable (`user=([^,\s]+)`), tester sur succès ET échec.
Réserve lecture seule : on **prépare et valide** la regex, l'admin la crée.

Exemples de départ : compte `"account":\s*"([^"]+)"`, IP `(\d{1,3}(\.\d{1,3}){3})`, MAC `([0-9A-Fa-f]{2}(:[0-9A-Fa-f]{2}){5})`, etype `etype\s*[:=]\s*(\w+)`.
