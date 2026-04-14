---
created_at: 2026-04-13
updated_at: 2026-04-14
aliases:
  - vulnérabilités applicatives
---

↑ [README](../README.md)

---

# Vulnérabilités applicatives DVWA

## Fichier de travail opérationnel

Pour une traçabilité complète des commandes et des logs bruts, se référer au journal d'exploitation :

- [vulns_app](vulns_app.md)

## Objectif de la phase

1. **Identification** des vecteurs d'attaque sur le module web.
2. **Exploitation (PoC)** pour confirmer la criticité des failles.
3. **Évaluation de l'impact** sur la confidentialité, l'intégrité et la disponibilité (DIC).
4. **Recommandation** de correctifs de sécurité.

## Environnement de Test

L'audit a été réalisé sur une instance isolée de **DVWA (Damn Vulnerable Web Application)** déployée sur le LXC Debian (voir [infrastructure](../cartographie_infrastructure/cartographie_infrastructure.md)).

**Procédure de déploiement :**

```bash
# Mise à jour et installation des dépendances
apt update && apt upgrade -y
apt install wget -y

# Automatisation de l'installation de l'instance cible
wget https://raw.com/IamCarron/DVWA-Script/main/Install-DVWA.sh
chmod +x Install-DVWA.sh
./Install-DVWA.sh
```

## Analyse des Vulnérabilités

### 1. SQL Injection (SQLi)

| Module              | Vecteur d'attaque                                  |
| ------------------- | -------------------------------------------------- |
| SQL Injection (Low) | Paramètre d'entrée non assaini dans la requête SQL |

**Preuve de Concept (PoC) :**

```sql
1' or '1'='1
```

**Résultat de l'exploitation :**

- Contournement de la logique de filtrage.
- Extraction de la table `users` (récupération de l'identifiant `admin`).

**Impact :** **CRITIQUE** - Perte totale de la confidentialité des données et compromission de la base de données.

**Remédiation :** Utiliser des **requêtes préparées (Prepared Statements)** avec PDO et implémenter une validation stricte des types de données.

### 2. Cross-Site Scripting (XSS) - Reflected

| Module          | Vecteur d'attaque                                        |
| --------------- | -------------------------------------------------------- |
| XSS (Reflected) | Injection de scripts malveillants via un paramètre d'URL |

**Preuve de Concept (PoC) :**

```html
<script>
  alert("XSS_Exploited_by_Audit");
</script>
```

**Résultat de l'exploitation :**

Exécution de code JavaScript dans le contexte du navigateur de la victime.

**Impact :** **ÉLEVÉ** - Vol de cookies de session (Session Hijacking), redirection malveillante ou défaçage du contenu.

**Remédiation :** Mise en place d'un **encodage en sortie (Output Encoding)** et application d'une politique de sécurité de contenu (**Content Security Policy - CSP**).

### 3. IDOR - Insecure Direct Object Reference (Contournement d'authentification)

| Module               | Vecteur d'attaque                                                     |
| -------------------- | --------------------------------------------------------------------- |
| Authorization Bypass | Accès direct à une URL administrative sans authentification préalable |

**Description de la faille :**

L'IDOR (Insecure Direct Object Reference) est une faille de sécurité qui survient lorsqu'une application autorise un utilisateur à accéder à des ressources ou à exécuter des actions en modifiant simplement un identifiant dans l'URL, **sans vérifier ses droits d'accès**.

**Contexte :**

Le répertoire `/vulnerabilities/authbypass/` est normalement réservé au compte administrateur. Cependant, lorsqu'un utilisateur standard (par exemple `gordonb`) tente d'y accéder directement, l'application ne vérifie pas ses permissions.

**Preuve de Concept (PoC) :**

```http
GET /vulnerabilities/authbypass/ HTTP/1.1
Host: 10.69.7.2
Cookie: PHPSESSID=<session_utilisateur_standard>
```

**Résultat de l'exploitation :**

Accès direct à des pages administratives sans authentification valide.

**Impact :** **CRITIQUE** - Élévation de privilèges et prise de contrôle de la gestion des utilisateurs.

**Remédiation :** Implémenter un contrôle d'accès basé sur les rôles (**RBAC**) systématique sur chaque point d'entrée (endpoint) sensible.

### 4. Session Management : Weak Session IDs

| Module           | Vecteur d'attaque                          |
| ---------------- | ------------------------------------------ |
| Weak Session IDs | Prédictibilité de l'identifiant de session |

**Preuve de Concept (PoC) :** Observation de l'en-tête `Set-Cookie` lors de sessions successives.

- Session 1 : `dvwaSession=7`
- Session 2 : `dvwaSession=8`

**Impact :** **ÉLEVÉ** - Risque de **Session Hijacking** par prédiction d'ID (attaque par force brute sur les cookies).

**Remédiation :** Utiliser un générateur de tokens cryptographiquement sécurisé et régénérer l'ID de session après chaque changement de privilège.

### 5. Mauvaise configuration HTTP (Security Misconfiguration)

| Module                             | Vecteur d'attaque                    |
| ---------------------------------- | ------------------------------------ |
| Information Disclosure via Headers | Analyse des en-têtes de réponse HTTP |

**Preuve de Concept (PoC) :**

```bash
curl -v http://10.69.7.2
```

**Résultat de l'audit :**

- `Server: Apache/2.4.66 (Debian)` - Divulgation de la version précise.
- Absence de headers de protection (`X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`).

**Impact :** **FAIBLE** - Aide l'attaquant à la phase de reconnaissance en ciblant des vulnérabilités spécifiques à la version du serveur.

**Remédiation :** Désactiver la signature du serveur (`ServerTokens Prod`) et configurer les headers de sécurité indispensables.

## Synthèse des vulnérabilités

| Vulnérabilité          | Niveau de risque | Module concerné      |
| ---------------------- | ---------------- | -------------------- |
| SQL Injection          | **CRITIQUE**     | SQL Injection        |
| IDOR / Auth Bypass     | **CRITIQUE**     | Authorization Bypass |
| Weak Session IDs       | **ÉLEVÉ**        | Weak Session IDs     |
| XSS (Reflected)        | **ÉLEVÉ**        | XSS (Reflected)      |
| Information Disclosure | **FAIBLE**       | Headers HTTP         |

## Conclusion de la phase

L'application DVWA expose plusieurs vulnérabilités critiques et élevées permettant un contournement complet des mécanismes d'authentification, une extraction de données sensibles et une prise de contrôle partielle de l'application. La priorité de correction doit se concentrer sur :

1. La mise en place de **requêtes préparées** (SQLi)
2. L'implémentation d'un **contrôle d'accès systématique** (IDOR)
3. La sécurisation des **identifiants de session** (Weak Session IDs)

Le fichier [`vulns_app.md`](vulns_app.md) contient l'ensemble des traces techniques pour la validation et l'audit interne.
