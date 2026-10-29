# 11 - Comprendre les signatures IDS (FMC) : guide d'équipe

> Guide de référence pour une équipe SOC : comprendre comment sont organisées les signatures sur un FMC (Secure Firewall Management Center), comment une application y est reliée, comment suivre une attaque ou un incident de bout en bout, et comment rédiger une nouvelle règle.
>
> Contenu **générique** : aucune donnée réelle (organisation, IP, hostname, nom d'interface, nom de policy). Les tableaux sont des **modèles à remplir** dans votre environnement. Mode IDS **passif** (SPAN/TAP) : les règles génèrent des événements, elles ne bloquent pas.

---

## 1. La chaîne d'inspection (vue d'ensemble)

```
   Internet / réseau surveillé
            │
            ▼
   ┌─────────────────────┐
   │  Device (sonde FTD) │   + moteur Snort actif (2 ou 3)
   └─────────┬───────────┘
             │ 1 ACP par device / périmètre
             ▼
   ┌─────────────────────┐
   │ Access Control      │   règles en 2 sections : Mandatory puis Default
   │ Policy (ACP)        │   critères : Zone | Réseau/Range IP | VLAN | Application | Ports | URL
   └─────────┬───────────┘
             │ la règle qui matche ouvre son onglet Inspection
             ▼
   ┌─────────────────────┐        ┌─────────────────────┐
   │  Intrusion Policy   │        │   File Policy        │
   │  = SIGNATURES Snort │        │  fichiers / malware  │  (séparé des signatures)
   └─────────┬───────────┘        └─────────────────────┘
             ▼
        Événements (Intrusion, Connection, Security Intelligence, File)
```

Idée : on part du **device** (par où le trafic est vu), l'**ACP** décide **quelle règle traite quel trafic**, la règle **envoie le trafic vers une intrusion policy**, et c'est cette policy qui porte les **signatures** qui détectent.

---

## 2. Le device (point souvent oublié)

Où le trouver dans FMC : **`Devices > Device Management`**. La liste affiche chaque sonde gérée ; en cliquant sur une sonde on obtient ses interfaces, ses zones, sa version de Snort, sa santé et l'ACP qui lui est assignée.

Ce qu'on capture par device (modèle à remplir) :

| Device (sonde) | Rôle | Moteur Snort | ACP assignée | Interfaces / zones (passives) |
|---|---|---|---|---|
| `<device 1>` | `<ex. périmètre externe>` | `<Snort 2 ou 3>` | `<ACP 1>` | `<zone A, zone B>` |
| `<device 2>` | `<ex. interne / DC>` | `<...>` | `<ACP 2>` | `<...>` |

En passif, les interfaces sont en **SPAN/TAP** : la sonde observe une copie du trafic, elle ne le route pas.

---

## 3. Les Access Control Policies (ACP)

Une ACP par device / périmètre. Modèle d'inventaire :

| ACP | Périmètre | Device(s) | Moteur Snort |
|---|---|---|---|
| `<ACP 1>` | `<à remplir>` | `<...>` | `<...>` |
| `<ACP 2>` | `<à remplir>` | `<...>` | `<...>` |
| `<ACP 3>` | `<à remplir>` | `<...>` | `<...>` |

Pour remplir : `Devices > Device Management` (colonne ACP), ou dans l'ACP, bouton **Policy Assignments**.

---

## 4. Anatomie d'une règle Access Control

### Ordre d'évaluation

```
  ┌──────────────┐   ┌──────────────────┐   ┌──────────────┐   ┌────────────────┐
  │  MANDATORY   │ → │ catégories custom │ → │   DEFAULT    │ → │ Default Action │
  │ (prioritaire)│   │   (optionnel)     │   │              │   │ (intrusion pol.│
  └──────────────┘   └──────────────────┘   └──────────────┘   │  par défaut)   │
                                                                 └────────────────┘
        la 1re règle qui matche gagne (évaluation de haut en bas)
```

### Critères de matching (onglets de la règle)

| Onglet | Ce qu'il filtre |
|---|---|
| Zones | zone ingress/egress = par quelle sonde/interface le trafic est vu |
| Networks | source/destination par IP, sous-réseau, **range IP**, objet réseau, géoloc |
| VLAN Tags | filtrage par **VLAN** (visible seulement si le SPAN transporte les trames taguées) |
| Applications | application L7 (AppID) |
| Ports | ports / protocoles source et destination |
| URLs | filtrage d'URL |
| Users | identité (realm / ISE) |

Règle de combinaison : **ET entre onglets différents**, **OU à l'intérieur d'un onglet**, onglet vide = « any ».

Modèle de nommage courant : **une règle par application / équipement**, et/ou **une règle par VLAN et range IP**.

---

## 5. De la règle à la signature

Sur une règle, onglet **Inspection** :

| Élément | Rôle |
|---|---|
| Intrusion Policy | le jeu de **signatures** qui inspecte ce trafic |
| Variable Set | les variables réseau (HOME_NET, etc.) utilisées par les signatures |
| File Policy | inspection fichiers / malware (séparé des signatures) |

Pour savoir quelles signatures s'appliquent à un trafic : lire **quelle Intrusion Policy** la règle invoque, puis ouvrir cette policy.

---

## 6. Qu'est-ce qu'une signature

Une signature d'intrusion = **une règle Snort** (GID:SID) qui détecte une technique ou une vulnérabilité précise.

| Champ | Signification |
|---|---|
| GID:SID | identifiant unique de la règle |
| Message (msg) | description de ce qui est détecté |
| Service / protocole | protocole visé (http, ssl, smb, dns...). **Le lien au trafic se fait par le service, pas par le catalogue AppID.** |
| Catégorie / Rule Group | regroupement (protocole, produit, type de menace) |
| Référence CVE | la vulnérabilité visée, le cas échéant |
| État / Rule Action | Disabled / **Alert** (Generate Events) / Block (Drop). En passif : **Alert**. |

Une intrusion policy dérive d'une **Base Policy** (Connectivity over Security / Balanced / Security over Connectivity / Maximum Detection) qui active **plusieurs milliers** de signatures. Un total affiché très bas (ex. quelques dizaines) = en général un **filtre actif**, une vue **Rule Overrides** (règles modifiées vs la base), ou un **Rule Group** ; ce n'est pas le total réel.

---

## 7. Snort 2 vs Snort 3

| | Snort 2 | Snort 3 |
|---|---|---|
| Nature | ancienne génération du moteur | nouvelle génération (réécriture) |
| Sur un device | **un seul moteur actif à la fois** | idem |
| Normalisation protocolaire | oui | oui |
| Détection applicative | oui | oui |
| Syntaxe de règles | historique | nouvelle, plus souple |
| Par défaut (FMC 7.x) | — | **oui** |

Points clés :
- **Les deux font normalisation ET détection applicative.** La différence n'est pas « protocoles d'un côté, applications de l'autre ».
- Une intrusion policy garde **deux versions** de ses règles (Snort 2 et Snort 3) ; **seule celle du moteur actif du device est appliquée**.
- Quand on édite une signature, travailler **dans la version du moteur actif**, sinon la modif n'a aucun effet.

À confirmer dans votre environnement : quel moteur tourne chaque device, et si les deux versions coexistent, pourquoi.

---

## 8. Relier une application à ses signatures

```
  Application (AppID)
       │  repérée dans une règle AC (onglet Applications)
       ▼
  Règle Access Control  ──(onglet Inspection)──►  Intrusion Policy
       (zone / réseau / VLAN / range IP)                 │
                                                          │ filtrer par service/protocole
                                                          ▼
                                                   Signatures (GID:SID)
```

Marche à suivre :
1. ACP → repérer la **règle qui cible l'application** (onglet Applications).
2. Règle → onglet **Inspection** → noter l'**Intrusion Policy**.
3. Intrusion Policy (version du moteur actif) → **filtrer par service/protocole** → lister les signatures.

Modèle à remplir :

| Application | Règle AC | Zone / Réseau / VLAN | Intrusion Policy | Signatures clés (GID:SID) |
|---|---|---|---|---|
| `<appli 1>` | `<règle>` | `<...>` | `<policy>` | `<SID, SID...>` |

---

## 9. Suivre une attaque ou un incident (de bout en bout)

Types d'événements FMC : **Intrusion**, **Connection**, **Security Intelligence**, **File/Malware**.

Principe : pivot sur **IP + horodatage + ports** ; le **NAT** relie le public à l'interne. Flux à travers les outils :

```
 IP publique
     │
     ▼
 [ FMC ]  Connections : zone ingress, device, Responder IP, NAT src/dst, application
     │    Intrusions  : signatures déclenchées (GID:SID)
     ▼
 [ SIEM ] toutes log sources par temps : firewall (accept + NAT pre/post), reverse proxy / WAF
     │
     ▼
 [ Kibana ] access logs applicatifs : client IP / X-Forwarded-For → endpoint, session, backend
     │
     ▼
 Chemin reconstitué : IP pub → sonde/zone → firewall+NAT → reverse proxy/WAF → serveur appli
```

Par étage :

| Étage | Outil | Ce qu'on y lit |
|---|---|---|
| Réseau / IDS | FMC | zone ingress, device (sonde), Responder IP, **NAT source/dest**, application, règle AC, signatures (GID:SID) |
| Logs agrégés | SIEM | source IP sur toutes les log sources, triée par temps ; firewall (accept + NAT), reverse proxy / WAF ; hiérarchie réseau |
| Applicatif | Kibana | client IP / `X-Forwarded-For` ; endpoint atteint, session, backend |

Depuis un **événement d'intrusion** : le **GID:SID** donne la signature (quoi), on remonte à la **règle AC** (quel périmètre / appli) et à la **source** (via le pivot ci-dessus).

Limites en passif : seul le trafic **traversant le SPAN** et **non chiffré** est visible.

---

## 10. Rédiger une nouvelle règle (signature custom)

| Étape | Action |
|---|---|
| 1. Besoin | use case, attaque observée, ou PCAP (paquet déclencheur via « Store packets » dans l'intrusion policy, puis `Analysis > Intrusions > Events`) |
| 2. Création | `Objects > Intrusion Rules > Create Rule` (syntaxe **Snort 3**) : action `alert`, protocole, flux, options (`content`, `pcre`, `flow`, `http_uri`...), `msg`, `classtype`, `sid` (plage locale), `rev` |
| 3. Activation | intrusion policy (Edit, version Snort active) → groupe **Local Rules** → **Rule Action = Alert** |
| 4. Mise en service | **Save** puis **Deploy** |
| 5. Tuning | mesurer le taux de **faux positifs** sur trafic réel avant d'industrialiser |

Hygiène : **lire/noter** → filtrer, relever, **sortir sans sauvegarder** ; **modifier/ajouter** → **Save + Deploy** obligatoire.

---

## 11. Checklist à compléter (par l'équipe)

- [ ] Inventaire des devices (rôle, moteur Snort, ACP, zones).
- [ ] Inventaire des ACP et de leur périmètre.
- [ ] Base Policy et nombre de signatures actives par intrusion policy.
- [ ] Version Snort appliquée et raison d'une éventuelle coexistence Snort 2 / 3.
- [ ] Applications critiques → règle AC → intrusion policy → signatures.
- [ ] Convention de SID pour les règles custom (plage locale).
- [ ] Requêtes types SIEM et Kibana pour le suivi d'incident.
