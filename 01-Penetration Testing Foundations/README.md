# Module 1 — Test d'intrusion guidé : Web / Penetration Testing Foundations

## Objectif du module

S'initier à une méthodologie complète de test d'intrusion web, depuis la reconnaissance jusqu'à l'exploitation de failles logiques et applicatives courantes, sur une machine de lab TryHackMe.

>  Toutes les commandes ci-dessous ont été exécutées sur une machine de lab TryHackMe, avec autorisation explicite dans ce cadre. `<TARGET_IP>` remplace l'adresse IP réelle du lab.

---

## 1. Reconnaissance réseau — Nmap

```
nmap -sV -sT -p- <TARGET_IP>
```

- `-sV` : détection des versions de services (bannières)
- `-sT` : scan TCP connect (handshake complet, plus fiable que `-sS` sans privilèges root)
- `-p-` (ou `-p 1-65535`) : scan de tous les ports

**Ce que ça m'a appris** : commencer par un scan complet des ports plutôt que de se limiter aux ports "communs" évite de rater un service web sur un port non standard (8080, 8443...).

## 2. Énumération web — Gobuster

```
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

- `dir` : mode de brute-force de répertoires/fichiers
- `-u` : URL cible
- `-w` : wordlist utilisée

**Ce que ça m'a appris** : Gobuster permet de découvrir des chemins non référencés (panels d'admin, répertoires d'upload, fichiers de sauvegarde) qui ne sont pas visibles depuis la navigation normale du site.

## 3. Vérification des en-têtes HTTP — curl

```
curl -I http://<TARGET_IP>
```

- `-I` : récupère uniquement les en-têtes HTTP (pas le corps de la réponse)

**Ce que ça m'a appris** : les en-têtes révèlent souvent la stack technique (serveur web, framework, cookies de session) sans avoir à charger toute la page — utile pour affiner la recherche de vulnérabilités connues.

## 4. Contournement de la réinitialisation de mot de passe

**Principe** : certains formulaires de "mot de passe oublié" font confiance à des valeurs contrôlées par le client (identifiant utilisateur dans l'URL, jeton prévisible, ou étape de vérification contournable côté client) plutôt que de valider strictement l'identité côté serveur.

**Méthodologie suivie** :
1. Intercepter la requête de réinitialisation avec Burp Suite.
2. Identifier quel paramètre détermine le compte ciblé (ex. `user_id`, `email` passé en clair).
3. Vérifier si ce paramètre peut être modifié pour cibler un autre compte sans que le serveur ne revérifie la propriété du compte.

**Point de vigilance côté défense** : toujours valider côté serveur que l'utilisateur authentifié (ou le token envoyé par e-mail) correspond bien au compte ciblé par la réinitialisation.

## 5. IDOR (Insecure Direct Object Reference)

**Principe** : l'application expose une référence directe à un objet (ID de document, de facture, de profil...) dans l'URL ou une requête, sans vérifier que l'utilisateur courant a le droit d'accéder à cet objet précis.

**Méthodologie suivie** :
1. Repérer un identifiant numérique/séquentiel dans une URL ou un paramètre (ex. `?doc_id=15`).
2. Modifier cet identifiant (incrémenter, décrémenter) tout en restant connecté avec mon propre compte.
3. Observer si l'application renvoie les données d'un autre utilisateur.

**Point de vigilance côté défense** : implémenter un contrôle d'autorisation systématique (vérifier que l'objet demandé appartient bien à l'utilisateur authentifié), plutôt que de se reposer sur l'absence de visibilité de l'ID.

## 6. Vol de session via cookie

**Principe** : si le cookie de session n'est pas correctement protégé (pas de flag `HttpOnly`/`Secure`, transmis en clair, ou simplement réutilisable), un attaquant en possession de ce cookie peut usurper la session d'un utilisateur sans connaître son mot de passe.

**Méthodologie suivie** :
1. Récupérer la valeur du cookie de session (via l'inspecteur du navigateur ou l'historique Burp).
2. Injecter cette valeur de cookie dans mes propres requêtes vers l'application.
3. Constater l'accès à la session de l'utilisateur ciblé sans authentification classique.

**Point de vigilance côté défense** : flags `HttpOnly`, `Secure`, `SameSite` sur les cookies, rotation de session après authentification, et transport exclusivement en HTTPS.

## 7. Upload de fichier malveillant → exécution de code à distance

**Principe** : lorsqu'un formulaire d'upload ne valide pas correctement le type de fichier côté serveur (filtrage uniquement côté client, ou sur l'extension sans vérifier le contenu réel), il est possible d'uploader un script interprétable par le serveur (ex. `.phtml`, `.php5`) dans un répertoire accessible publiquement, puis de le déclencher via une requête HTTP pour exécuter des commandes sur le système.

**Méthodologie suivie** (résumé, sans one-liner d'exploitation) :
1. Identifier un formulaire d'upload et tester les filtres de type de fichier (extension, MIME type, magic bytes).
2. Uploader un fichier avec une extension exécutable par le serveur web, déguisé pour passer les contrôles côté client.
3. Localiser le fichier uploadé (souvent via Gobuster, étape 2) et y accéder via le navigateur/curl pour déclencher l'exécution du code qu'il contient.
4. Utiliser cette exécution de code pour établir une connexion shell inversée (reverse shell) vers une machine d'écoute que je contrôle (`nc -lvnp <PORT>`), afin d'obtenir un accès interactif à la machine cible.

**Point de vigilance côté défense** : valider le contenu réel des fichiers uploadés (pas seulement l'extension), stocker les uploads hors de la racine web ou dans un répertoire sans droit d'exécution, et whitelister strictement les types de fichiers autorisés.

---

## Synthèse

| Faille | Cause racine | Impact |
|--------|---------------|--------|
| Reset password bypass | Validation côté client / paramètre de confiance mal vérifié | Prise de contrôle de compte |
| IDOR | Absence de contrôle d'autorisation sur les objets | Fuite de données d'autres utilisateurs |
| Vol de session | Cookie mal protégé / rejouable | Usurpation de session |
| Upload → RCE | Validation insuffisante des fichiers uploadés | Exécution de code à distance |

## À retenir pour la suite

- Toujours combiner recon automatisée (nmap, gobuster) et vérification manuelle (curl, navigateur, Burp).
- La majorité des failles de ce module reposent sur une **confiance excessive envers les données côté client** (IDs, cookies, type de fichier) : réflexe à chercher systématiquement.
- Documenter chaque étape comme pour un rapport de pentest réel (reproductibilité, impact, remédiation).
