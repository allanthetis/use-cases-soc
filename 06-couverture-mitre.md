# 06 - Couverture MITRE ATT&CK

Mapper les use cases sur ATT&CK permet de **visualiser la couverture et les trous**, et c'est un livrable apprécié du management.

## Correspondance use cases -> techniques

| Technique ATT&CK | ID | Use cases couvrants |
|---|---|---|
| Brute Force | T1110 | Bruteforce compte, verrouillages en série, bruteforce portail LemonLDAP |
| Password Spraying | T1110.003 | Password spraying |
| Kerberoasting | T1558.003 | Kerberoasting (RC4), ticket SPN sensible |
| AS-REP Roasting | T1558.004 | AS-REP Roasting |
| Golden / Silver Ticket | T1558.001 / .002 | Golden Ticket, Silver Ticket |
| DCSync | T1003.006 | DCSync (réplication par non-DC) |
| Account Manipulation | T1098 | Modif groupe privilégié, reset mdp, SPN ajouté, compte réactivé |
| Create Account | T1136 | Création compte, création/suppression hors process |
| Account Discovery | T1087 | Lecture massive annuaire, énumération LDAP |
| Valid Accounts | T1078 | Connexion nocturne, voyage impossible, compte compromis actif, admin KDC, multi-zones, accès appli hors population |
| Valid Accounts: Service | T1078.003 | Compte de service anormal |
| Remote Services: SMB | T1021.002 | Accès partage admin, SMB source non autorisée |
| Exfiltration over network | T1048 | Accès massif fichiers, exfiltration IDS |
| Data Destruction | T1485 | Suppression massive (rançongiciel) |
| Domain Policy Modification | T1484 | Modification de GPO |
| Rogue Domain Controller | T1207 | Rogue DC |
| Use Alternate Auth Material | T1550 | Repli NTLM |
| Network DoS | T1498 | DHCP starvation, pic de requêtes DHCP |
| Command and Control | TA0011 | Beaconing / C2, réputation, JA3/SNI |
| Exploitation (RCE) | T1190 | Exploitation CVE, RCE web (ex. PHP2Shell) |

## Lecture de la couverture
- **Bien couvert** : Credential Access (Kerberos), Persistence/Privilege (annuaire), Lateral Movement (SMB).
- **À renforcer / trous typiques** : trafic chiffré non inspecté (déchiffrement), est-ouest interne, UEBA/anomalie comportementale, détection sur les applis métier (côté ELK).
- Entretenir une **matrice de couverture** (ATT&CK Navigator) et l'actualiser à chaque nouvelle règle.
