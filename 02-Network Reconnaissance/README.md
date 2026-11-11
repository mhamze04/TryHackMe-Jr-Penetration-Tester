# Module 2: Reconnaissance du réseau ( Network Reconnaissance )

Ce module couvre la collecte de renseignements sur une cible, d'abord de façon passive (sans jamais la toucher), puis active (interaction directe mais légère), avant de sonder en détail les protocoles réseau qui font tourner les infrastructures modernes et leurs failles de sécurité.

---

## 🕶️ Reconnaissance passive

Collecte d'informations **sans aucune interaction directe** avec la cible — la phase la plus furtive, à risque juridique quasi nul.

- **WHOIS** : informations d'enregistrement du domaine (bureau d'enregistrement, dates, serveurs de noms).
- **DNS (nslookup / dig)** : enregistrements A/AAAA (IP), MX (serveurs mail), TXT (SPF/DMARC), interrogés via des résolveurs publics comme `1.1.1.1`.
- **Énumération de sous-domaines** : DNSDumpster et crt.sh (logs de transparence des certificats SSL/TLS) — méthode passive la plus efficace pour trouver des sous-domaines.
- **Services exposés** : Shodan.io pour bannières d'appareils, ports ouverts, infos d'hébergement.

## 🔦 Reconnaissance active

Interaction légère avec la cible via des outils simples, en combinaison pour obtenir une vue structurée avant de passer aux scanners avancés (Nmap).

- **ping** : confirme l'accessibilité d'une cible, donne des indices sur l'OS via le TTL.
- **traceroute** : cartographie le chemin réseau, révèle routeurs intermédiaires et filtrage potentiel.
- **telnet / netcat** : connexion à des ports individuels pour récupérer bannières et versions de services.
- **Navigateur + DevTools** : technologies serveur, en-têtes, code source JS, détails de certificats.

## 🧩 Protocoles et serveurs

Fonctionnement bas niveau des protocoles réseau courants : HTTP, FTP, POP3, SMTP, IMAP.

- **Leçon clé** : tous ces protocoles transmettent les données **en clair**, identifiants compris.
- **Telnet** comme outil de test manuel universel pour dialoguer avec n'importe quel protocole textuel sur son port TCP.
- Chaque protocole en clair a une alternative chiffrée (HTTPS, SFTP/FTPS, IMAPS/POP3S/SMTPS, SSH).

## 🛡️ Protocoles et serveurs 2

Les attaques exploitant ces protocoles en clair, et comment s'en protéger.

- **3 attaques étudiées** : reniflage (sniffing), MITM (homme du milieu), attaque par mot de passe.
- **Mitigation universelle** : chiffrer systématiquement (HTTPS, SSH, SFTP/FTPS, IMAPS/POP3S/SMTPS).
- **Le chiffrement ne suffit pas** : mots de passe robustes, verrouillage de comptes, limitation de débit, MFA, voire authentification sans mot de passe (clé/certificat).
- **Hydra** : outil de brute-force contre les services réseau (identifiants/mots de passe).
- **Checklist défensive** : TLS 1.2/1.3, protocoles en clair désactivés, authentification SSH par clé, politiques de mots de passe, MFA, segmentation réseau, validation de certificats, HSTS, logs/monitoring.

---

## 📝 Ce que j'ai retenu

La reconnaissance n'est pas un bloc unique : il y a une vraie gradation de furtivité, du WHOIS/DNS (zéro contact) jusqu'au sondage actif des ports.
Mais le vrai point à retenir de ce module, c'est que **la quasi-totalité des protocoles historiques (FTP, HTTP, POP3, SMTP, Telnet) transmettent tout en clair** — la reconnaissance active elle-même (`nc`, `telnet`) marche *parce que* ces protocoles ne chiffrent rien.
Le réflexe pro : toujours vérifier si une alternative chiffrée existe (et pourquoi elle ne serait pas utilisée) avant de conclure qu'un service en clair est acceptable.
