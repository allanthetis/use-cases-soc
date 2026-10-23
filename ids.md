# Use cases IDS / IPS Firepower

> Firepower (Snort) remonté via le FMC en eStreamer. Types d'événements : Intrusion, Connection, Security Intelligence, File/Malware. Champs clés : signature/SID, action (alert / drop / would have dropped), source/dest IP, zones ingress/egress, application, sévérité, CVE, device (sonde). Seuils indicatifs à calibrer.

| Alerte | Criticité | Etat | Log source | Champs important | Commentaire |
|---|---|---|---|---|---|
| Signature critique sur actif sensible | Critique | À créer | Firepower Intrusion | SID/signature, dest IP, sévérité | Croiser avec un reference set des actifs critiques |
| Exploitation d'une CVE détectée | Haute | À créer | Firepower Intrusion | SID, CVE, dest IP | Ex. PHP2Shell (CVE-2026-109618/109619) |
| Tentative de RCE web (webshell, upload) | Haute | À créer | Firepower Intrusion | SID, URL, dest IP | PHP2Shell, BeanShell, deserialization |
| Montée alerte vers blocage (même source) | Haute | À créer | Firepower Intrusion | source IP, inline result | Passage de alert à drop = escalade |
| Source récurrente multi-signatures / multi-cibles | Haute | À créer | Firepower Intrusion | source IP, SID, dest IP | Scan ou attaque ciblée |
| Trafic vers IP/URL/DNS de mauvaise réputation | Haute | À créer | Firepower Security Intelligence | source/dest, catégorie SI | Flux Talos |
| Beaconing / communication C2 | Haute | À créer | Firepower Connection | dest IP, intervalle, JA3 | Connexions régulières, JA3 suspect |
| Exfiltration de données (volume sortant) | Haute | À créer | Firepower Connection | octets, dest IP, zone | Inclut le DNS tunneling |
| Détection de malware / fichier (AMP) | Haute | À créer | Firepower File/Malware | hash, disposition, hôte | Fichier malveillant transféré |
| Sonde IDS muette (perte de visibilité) | Haute | À créer | Firepower (device) | device/sensor, dernière activité | Absence d'événements d'une sonde |
| « Would have dropped » récurrent sur actif critique | Moyenne | À créer | Firepower Intrusion | inline result, dest IP | Argument pour passer la sonde en inline |
| Scan de ports / reconnaissance | Moyenne | À créer | Firepower Connection | source IP, ports, nb de cibles | Phase de reconnaissance |
| Trafic chiffré suspect (JA3 / SNI anormal) | Moyenne | À créer | Firepower Connection | JA3, SNI, dest IP | Visibilité sur le chiffré non déchiffré |
| Flux inter-zones non autorisé | Moyenne | À créer | Firepower Connection | ingress zone, egress zone | Violation de segmentation |
| Anomalie de volume de flux | Moyenne | À créer | Firepower Connection | volume, baseline | Pic par rapport à la normale |
