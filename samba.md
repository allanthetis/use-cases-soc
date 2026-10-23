# Use cases Samba (AD : Kerberos, LDAP, SMB, réplication) + LemonLDAP

> AD Samba * : 80 serveurs, 2 primaires (FSMO), SSO LemonLDAP en façade. Seuils indicatifs à calibrer.

| Alerte | Criticité | Etat | Log source | Champs important | Commentaire |
|---|---|---|---|---|---|
| Bruteforce sur un compte | Moyenne | À créer | Samba Auth | compte, résultat, horodatage | Seuil ~10 échecs/5 min ; exclure comptes de service |
| Password spraying | Haute | À créer | Samba Auth | IP source, compte, résultat | 1 IP vers >=10 comptes distincts/10 min |
| Succès après rafale d'échecs | Haute | À créer | Samba Auth | compte, IP, résultat | Compromission probable |
| Auth hors horaires compte sensible | Moyenne | À créer | Samba Auth | compte, groupe, horodatage | Exclure astreintes déclarées |
| Voyage impossible | Haute | À créer | Samba Auth | compte, IP/zone source | Exclure VPN et NAT sortant |
| Repli NTLM (downgrade) | Moyenne | À créer | Samba Auth | type d'auth, compte | Exclure vieux clients legacy |
| Bind LDAP anonyme ou en clair | Moyenne | À créer | Samba LDAP/dsdb | type de bind, port, chiffrement | 389 non TLS ou bind anonyme |
| Kerberoasting (RC4) | Haute | À créer | Samba KDC | compte, etype, type de requête | Champ etype à parser ; etype 23 = RC4 |
| AS-REP Roasting | Haute | À créer | Samba KDC | flag preauth, compte | Comptes sans pré-authentification |
| Golden Ticket suspecté | Haute | À créer | Samba KDC | durée ticket, krbtgt, compte | Durée de vie anormale |
| Silver Ticket suspecté | Haute | À créer | Samba KDC | TGS vs usage service | Ticket de service sans TGS correspondant |
| Pic d'échecs de pré-auth | Moyenne | À créer | Samba KDC | erreur preauth, compte, IP | ~20 erreurs/10 min |
| Ticket pour SPN sensible | Moyenne | À créer | Samba KDC | SPN cible, compte demandeur | Exclure comptes admin légitimes |
| SPN ajouté sur un compte user | Haute | À créer | Samba LDAP/dsdb | attribut servicePrincipalName, objet | Préparation Kerberoast |
| DCSync (réplication par non-DC) | Critique | À créer | Samba Réplication | opération réplication, source, compte | Reference set des DC obligatoire |
| Lecture massive de l'annuaire | Moyenne | À créer | Samba LDAP/dsdb | compte, volume de requêtes | >500 objets/10 min ; exclure outils d'inventaire |
| Modification d'un groupe privilégié | Haute | À créer | Samba LDAP/dsdb | opération, groupe cible | Domain/Enterprise Admins |
| Ajout d'un compte à un groupe privilégié | Haute | À créer | Samba LDAP/dsdb | opération, groupe, compte | - |
| Reset de mot de passe sensible | Haute | À créer | Samba LDAP/dsdb | opération, compte cible, auteur | Exclure helpdesk autorisé |
| Création/suppression de compte hors process | Moyenne | À créer | Samba LDAP/dsdb | opération, objet, auteur | Exclure provisioning automatisé |
| Modification de GPO/stratégie | Haute | À créer | Samba LDAP/dsdb | opération, objet GPO | Admins habilités |
| Énumération LDAP par non-admin | Moyenne | À créer | Samba LDAP/dsdb | compte, requêtes LDAP | - |
| Accès à un partage administratif | Moyenne | À créer | Samba SMB | partage (C$/ADMIN$/IPC$), compte, IP | Exclure admins et outils de gestion |
| Accès massif à des fichiers | Haute | À créer | Samba SMB | session, événements full_audit | >1000 fichiers/10 min ; exclure sauvegardes |
| Suppression massive (rançongiciel) | Critique | À créer | Samba SMB | suppressions, session | >200 suppressions/5 min |
| SMB depuis une source non autorisée | Moyenne | À créer | Samba SMB | IP source, partage | - |
| Verrouillages de comptes en série | Moyenne | À créer | Samba Auth | événement lockout, compte | ~5 verrouillages/10 min |
| Compte désactivé réactivé | Moyenne | À créer | Samba LDAP/dsdb | état du compte, opération | Exclure réintégration RH validée |
| Création d'un compte à privilèges | Haute | À créer | Samba LDAP/dsdb | opération, groupe | - |
| Compte de service anormal | Haute | À créer | Samba Auth | compte, type de logon, source | Logon interactif ou nouvelle source |
| Réveil d'un compte dormant | Moyenne | À créer | Samba Auth | compte, dernière activité | Inactif >90 j |
| Rogue DC | Critique | À créer | Samba Réplication | source, type d'événement | Reference set des DC |
| Réplication anormale | Haute | À créer | Samba Réplication | source, horodatage de réplication | Hors fenêtre ou source inattendue |
| Opération FSMO inattendue | Haute | À créer | Samba Réplication | opération de rôle | Exclure migration validée |
| DC muet (perte de visibilité) | Haute | À créer | Samba (toutes sources) | log source, dernière activité | Absence d'événements >2 h |
| Compte signalé compromis actif | Haute | À créer | Samba Auth | compte, reference set threat intel | Alimenté par la threat intel |
| Pic de volume d'authentification | Moyenne | À créer | Samba Auth | compte/appli, volume, baseline | Exclure pics métier connus |
| Admin du KDC hors population | Haute | À créer | Samba KDC | compte, IP | Reference set des admins |
| Connexions concomitantes multi-zones | Haute | À créer | Samba Auth | compte, zone, horodatage | Comptes partagés à proscrire |
| Bruteforce portail LemonLDAP | Moyenne | À créer | LemonLDAP | compte, IP, résultat portail | Credential stuffing |
| Accès appli hors population autorisée | Haute | À créer | LemonLDAP | user, application, décision | Évolution de droits à valider |
| Anomalie de session SSO | Moyenne | À créer | LemonLDAP | session, user, IP | Réutilisation ou durée anormale |
