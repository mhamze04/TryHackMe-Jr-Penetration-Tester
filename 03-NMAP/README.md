# Module 3: Nmap

Ce module est entièrement dédié à Nmap, l'outil de référence pour la découverte d'hôtes et le scan de ports. On passe des techniques de base aux techniques avancées utilisées pour contourner la détection.

---

## 📡 Découverte d'hôtes en direct avec Nmap

Déterminer quels hôtes sont actifs sur un réseau avant de scanner quoi que ce soit en détail.

- **Logique** : Nmap envoie une sonde (ARP, ICMP, TCP SYN/ACK, ou UDP) et considère l'hôte actif si une réponse revient.
- **ARP** (même sous-réseau uniquement) : le plus fiable localement.
- **ICMP** (echo, timestamp, netmask) : classique, mais souvent filtré par les pare-feux modernes.
- **TCP SYN/ACK sur ports choisis** : contourne les pare-feux qui bloquent l'ICMP.
- **UDP vers un port fermé** : utile en dernier recours.
- `-sn` est l'option clé : découverte d'hôtes **sans** scan de ports (sinon Nmap scanne par défaut les ports des hôtes actifs trouvés).

## 🔓 Analyses de ports de base avec Nmap

Les 3 types de scan de port fondamentaux, et les options pour cibler précisément et ajuster la vitesse.

- **TCP Connect Scan** (`-sT`) : handshake TCP complet, le plus fiable, ne nécessite pas de privilèges root, mais le plus "bruyant" (visible dans les logs).
- **TCP SYN Scan** (`-sS`) : scan "furtif" (half-open), nécessite les privilèges root.
- **UDP Scan** (`-sU`) : plus lent et moins fiable (UDP n'a pas d'accusé de réception).
- **Ciblage des ports** : tous les ports, une plage, les 100 plus courants, ou dans l'ordre séquentiel plutôt qu'aléatoire.
- **Vitesse** : templates de timing de `T0` (le plus lent, furtif) à `T5` (le plus rapide), ou contrôle fin du débit de paquets/parallélisme.

## 🥷 Analyses de ports avancées avec Nmap

Techniques avancées basées sur des indicateurs TCP non standards, pour contourner des pare-feux/IDS ou rester difficile à tracer.

- **Scans Null / FIN / Xmas** : exploitent une particularité de la RFC TCP — un port fermé répond par RST, un port ouvert/filtré ne répond rien. Utile contre un pare-feu stateless qui ne bloque que les paquets avec le flag SYN.
- **Scan Maimon** : cible les anciens systèmes dérivés de BSD.
- **Scan ACK / Fenêtre (Window)** : ne détecte pas "ouvert/fermé" mais **autorisé par un pare-feu** vs service réellement actif.
- **Usurpation (spoofing)** : adresse IP source usurpée, adresse MAC usurpée, "leurres" (decoys) pour noyer l'origine réelle du scan, scan "zombie" (idle scan) qui fait porter le scan par une machine tierce discrète.
- **Fragmentation de paquets** : découpe les données IP en petits fragments (8 ou 16 octets) pour passer sous le radar d'un pare-feu/IDS basique.
- **Règle générale de choix de technique** : Null/FIN/Xmas si pare-feu stateless suspecté ; Maimon sur cible ancienne type BSD ; ACK/Window pour cartographier les règles d'un pare-feu ; leurres/fragmentation si on doit rester discret ou illisible ; `--reason` pour comprendre comment Nmap est arrivé à sa conclusion.

---

## 🔬 Analyses de ports Nmap après analyse

Une fois les ports ouverts identifiés, cette room couvre comment creuser plus loin : identifier précisément ce qui tourne derrière chaque port, et organiser les résultats pour le rapport.

- **Détection de service/version** (`-sV`) : identifie le logiciel et sa version exacte derrière un port ouvert, avec un niveau d'intensité réglable (sondes rapides vs exhaustives).
- **Détection d'OS** (`-O`) : identifie le système d'exploitation de la cible via le fingerprinting de la pile TCP/IP.
- **Traceroute intégré** (`--traceroute`) : cartographie le chemin réseau en même temps que le scan, sans commande séparée.
- **NSE (Nmap Scripting Engine)** : exécution de scripts pour automatiser des vérifications plus poussées (par défaut ou personnalisés).
- **`-A`** : le combo "tout en un" (version + OS + scripts par défaut + traceroute) — pratique en reconnaissance rapide, mais plus bruyant/détectable.
- **Formats de sortie** : indispensables pour documenter un test d'intrusion et réutiliser les résultats (grep, parsing automatisé, rapport).

---

## 📝 Ce que j'ai retenu

Nmap n'est pas "un outil de scan", c'est une boîte à outils entière avec une logique en couches : d'abord savoir qui est en ligne (`-sn`), puis quels ports sont ouverts (`-sT`/`-sS`/`-sU`), puis, si besoin, contourner un pare-feu ou rester furtif avec les techniques avancées. Le point commun à tout le module : chaque type de scan repose sur une règle précise du protocole TCP/IP (comment un hôte est censé répondre), donc comprendre *pourquoi* une option fonctionne compte plus que la mémoriser.
notes.md — Fiche de notes Nmap (commandes et options)
markdown
# Fiche de notes — Module 3 : Nmap

## 1. Découverte d'hôtes en direct

| Type de numérisation | Exemple de commande | Sonde envoyée | Hôte actif si... |
|------------------------|----------------------|----------------|---------------------|
| Analyse ARP | `sudo nmap -PR -sn 10.200.6.0/24` | Requête ARP (même sous-réseau) | Réponse ARP |
| Analyse d'écho ICMP | `sudo nmap -PE -sn 10.200.6.0/24` | ICMP Type 8 (Echo) | ICMP Type 0 (réponse d'écho) |
| Analyse d'horodatage ICMP | `sudo nmap -PP -sn 10.200.6.0/24` | ICMP Type 13 | ICMP Type 14 |
| Analyse du masque d'adresse ICMP | `sudo nmap -PM -sn 10.200.6.0/24` | ICMP Type 17 | ICMP Type 18 |
| Ping TCP SYN | `sudo nmap -PS22,80,443 -sn 10.200.6.0/30` | TCP SYN | SYN/ACK ou RST |
| Ping TCP ACK | `sudo nmap -PA22,80,443 -sn 10.200.6.0/30` | TCP ACK | RST |
| Ping UDP | `sudo nmap -PU53,161,162 -sn 10.200.6.0/30` | Datagramme UDP (idéalement port fermé) | Port ICMP inaccessible |

**Options générales** :

| Option | Objectif |
|--------|----------|
| `-n` | Pas de résolution DNS |
| `-R` | Recherche DNS inverse pour tous les hôtes |
| `-sn` | Découverte d'hôte uniquement (pas de scan de ports) |

---

## 2. Analyses de ports de base

| Type de scan de port | Exemple de commande |
|------------------------|----------------------|
| TCP Connect Scan | `nmap -sT MACHINE_IP` |
| TCP SYN Scan | `sudo nmap -sS MACHINE_IP` |
| UDP Scan | `sudo nmap -sU MACHINE_IP` |

**Options de ciblage et performance** :

| Option | Objectif |
|--------|----------|
| `-p-` | Tous les ports |
| `-p1-1023` | Scanner les ports 1 à 1023 |
| `-F` | 100 ports les plus courants |
| `-r` | Scanner les ports dans l'ordre consécutif (pas aléatoire) |
| `-T<0-5>` | Vitesse : `-T0` le plus lent, `-T5` le plus rapide |
| `--max-rate 50` | Débit ≤ 50 paquets/s |
| `--min-rate 15` | Débit ≥ 15 paquets/s |
| `--min-parallelism 100` | Au moins 100 sondes en parallèle |

---

## 3. Analyses de ports avancées

| Type de scan de port | Exemple de commande |
|------------------------|----------------------|
| Analyse Null | `sudo nmap -sN MACHINE_IP` |
| Analyse TCP FIN | `sudo nmap -sF MACHINE_IP` |
| Analyse TCP de Noël (Xmas) | `sudo nmap -sX MACHINE_IP` |
| Analyse TCP Maimon | `sudo nmap -sM MACHINE_IP` |
| Analyse TCP ACK | `sudo nmap -sA MACHINE_IP` |
| Analyse de fenêtre TCP | `sudo nmap -sW MACHINE_IP` |
| Analyse TCP personnalisée | `sudo nmap --scanflags URGACKPSHRSTSYNFIN MACHINE_IP` |
| Adresse IP source usurpée | `sudo nmap -S IP_USURPÉE MACHINE_IP` |
| Adresse MAC usurpée | `nmap --spoof-mac MAC_USÉ MACHINE_IP` |
| Analyse de leurre (decoy) | `nmap -D ENSEMBLE_IP_LEURRE,ME ENSEMBLE_IP MACHINE_IP` |
| Scan inactif (zombie) | `sudo nmap -sI ZOMBIE_IP MACHINE_IP` |
| Fragmenter les données IP (8 octets) | `-f` |
| Fragmenter les données IP (16 octets) | `-ff` |

**Options complémentaires** :

| Option | Objectif |
|--------|----------|
| `--source-port NUM_PORT` | Spécifier le port source |
| `--data-length NUM` | Ajouter des données aléatoires (longueur donnée) |
| `--reason` | Explique comment Nmap est arrivé à sa conclusion |
| `-v` | Verbeux |
| `-vv` | Très verbeux |
| `-d` | Débogage |
| `-dd` | Débogage détaillé |

**Quand utiliser quelle technique** :

| Technique | Utiliser lorsque... |
|-----------|----------------------|
| Null, FIN, Xmas | Pare-feu stateless suspecté, qui ne bloque que les paquets avec le flag SYN activé |
| Scan Maimon | Cible potentiellement un ancien système dérivé de BSD |
| ACK et analyse de fenêtre | Identifier les ports autorisés par un pare-feu, pas les services en cours d'exécution |
| Leurres et paquets fragmentés | Rendre le scan plus difficile à tracer ou le faire passer inaperçu derrière un pare-feu/IDS simple |
| Scan inactif (zombie) | Masquer sa propre adresse IP en utilisant un hébergeur tiers discret |

> ⚠️ Les scans Null/FIN/Xmas/Maimon/ACK/Window nécessitent les privilèges root (`sudo`) car ils manipulent directement les paquets bruts.

---
