---
created_at: 2026-04-16
updated_at: 2026-04-16
aliases:
  - Rapport d'audit
---

↑ [README](README.md)

---

# RAPPORT D'AUDIT DE SÉCURITÉ

**Client :** NexaTech\
**Périmètre :** Infrastructure Réseau, Serveur Windows AD, Serveur Linux Web, Application RH\
**Version :** 1.0 (Finale)\
**Date :** 16 Avril 2026\
**Classification :** Confidentiel

---

## 1. Résumé Exécutif

**Objet :** Évaluation de la posture de sécurité de NexaTech suite à une alerte d'intrusion signalée par l'hébergeur.

Le cabinet de conseil **SecureOps** a été mandaté pour réaliser un audit de sécurité approfondi de l'infrastructure de NexaTech. La mission s'est déroulée sur 5 jours intensifs et a couvert l'analyse du contrôleur de domaine Active Directory (Windows Server 2022), du serveur d'applications Web (Debian Linux) et de l'application de Gestion RH développée en PHP.

**Synthèse de l'Évaluation :**

L'audit a révélé une **posture de sécurité initialement très faible et critique**. L'application Web présentait des vulnérabilités de type **Injection SQL (CVSS 9.8)** et **Contournement d'authentification IDOR (CVSS 8.1)** permettant une compromission totale des données RH sensibles et une prise de contrôle des comptes utilisateurs.

L'infrastructure système présentait une surface d'attaque excessive avec des services obsolètes (SMBv1) et des configurations par défaut non sécurisées (SSH Root accessible).

**Résultat Final Post-Audit :**

Au cours de la mission, l'équipe SecureOps a accompagné NexaTech dans la correction immédiate des failles critiques et le durcissement des systèmes. Le niveau de risque global est passé de **CRITIQUE à MODÉRÉ**.

- **Les failles critiques (SQLi, IDOR) ont été corrigées.**
- **Les accès distants (SSH, AD) ont été sécurisés (Clés SSH, Refus NTLMv1).**
- **Un script d'audit automatisé a été déployé pour maintenir cette posture dans le temps.**

## 2. Périmètre et Méthodologie

| Élément              | Détail                                                                                                                                                                                                                                                |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Client**           | NexaTech (PME 80 collaborateurs)                                                                                                                                                                                                                      |
| **Contexte**         | Investigation suite à une alerte de tentative d'intrusion détectée par l'hébergeur.                                                                                                                                                                   |
| **Systèmes Audités** | **Infrastructure :** Windows Server 2022 (AD) / Debian 12 (Web) <br>**Application :** Portail RH (PHP)                                                                                                                                                |
| **Méthodologie**     | 1. **Cartographie** (Scan réseau Nmap) <br>2. **Analyse de Vulnérabilités** (OWASP Top 10, Analyse de config) <br>3. **Exploitation (PoC)** pour validation des risques <br>4. **Remédiation & Hardening** <br>5. **Automatisation** (Scripting Bash) |

## 3. Constats Détaillés et Évaluation des Risques

### 3.1. Audit Applicatif : Portail Gestion RH (DVWA)

Cette partie de l'audit a mis en évidence des défaillances majeures de développement sécurisé (Secure Coding).

| Vulnérabilité                               |     Score CVSS     | Preuve Technique (PoC)                                                                   | Statut Initial |  Statut Final  |
| :------------------------------------------ | :----------------: | :--------------------------------------------------------------------------------------- | :------------: | :------------: |
| **Injection SQL (SQLi)**                    | **9.8 (CRITIQUE)** | Saisie de `' OR '1'='1` permettant le bypass d'authentification.                         |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Contournement d'Autorisation (IDOR)**     |  **8.1 (HAUTE)**   | Accès direct à l'URL `/vulnerabilities/authbypass/` avec un cookie utilisateur standard. |  🔴 **Actif**  |  🔴 **Actif**  |
| **Cross-Site Scripting (XSS) Réfléchi**     | **6.1 (MOYENNE)**  | Exécution de `<script>alert("XSS_Exploited")</script>` via paramètre URL.                |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Faiblesse des Identifiants de Session**   |  **7.5 (HAUTE)**   | Prédictibilité incrémentale : `dvwaSession=7` puis `=8`.                                 |  🔴 **Actif**  |  🔴 **Actif**  |
| **Fuites d'Information (Bannière Serveur)** | **5.0 (MOYENNE)**  | En-tête `Server: Apache/2.4.66 (Debian)`.                                                |  🟡 **Actif**  | 🟢 **Corrigé** |

**Actions de Remédiation Appliquées :**

- **SQLi :** Refonte du code avec des **Requêtes Préparées (PDO)**.
- **XSS :** Implémentation de la fonction **`htmlspecialchars()`** pour neutraliser les scripts.
- **IDOR :** Ajout d'un contrôle de session strict sur les endpoints sensibles.
- **Headers :** Ajout de `X-Frame-Options`, `X-Content-Type-Options`.

### 3.2. Audit d'Infrastructure

#### A. Serveur Linux (Debian 12 - Hébergement Web)

| Vulnérabilité                         |     Score CVSS     | Preuve Technique                          | Statut Initial |  Statut Final  |
| :------------------------------------ | :----------------: | :---------------------------------------- | :------------: | :------------: |
| **Connexion SSH Root Autorisée**      | **9.1 (CRITIQUE)** | `PermitRootLogin yes` dans `sshd_config`. |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Pare-feu (UFW) Inactif**            |  **7.5 (HAUTE)**   | `ufw status` retournant `inactive`.       |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Port SSH par défaut (22)**          | **5.0 (MOYENNE)**  | Service écoutant sur le port standard.    |  🟡 **Actif**  | 🟢 **Corrigé** |
| **Permissions `/etc/shadow` faibles** | **6.5 (MOYENNE)**  | Fichier lisible par d'autres groupes.     |  🟡 **Actif**  | 🟢 **Corrigé** |

**Actions de Hardening Appliquées :**

- **SSH :** Déplacement sur le port `707`, interdiction de `PermitRootLogin`, activation exclusive de l'authentification par **Clé Publique (Ed25519)**.
- **Pare-feu (UFW) :** Politique de **Whitelist**. Règles actives : `707/tcp (SSH)`, `80/tcp (HTTP)`, `443/tcp (HTTPS)`.

#### B. Serveur Windows (Windows Server 2022 - Active Directory)

| Vulnérabilité                        |    Score CVSS     | Preuve Technique                                                            | Statut Initial |  Statut Final  |
| :----------------------------------- | :---------------: | :-------------------------------------------------------------------------- | :------------: | :------------: |
| **Protocole SMBv1 Activé**           |  **8.0 (HAUTE)**  | `Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` = `Enabled`. |  🔴 **Actif**  | 🟢 **Corrigé** |
| **NTLMv1 Autorisé**                  |  **8.4 (HAUTE)**  | `LmCompatibilityLevel` < 4.                                                 |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Politique de mot de passe faible** |  **7.0 (HAUTE)**  | Longueur < 8, complexité non forcée.                                        |  🔴 **Actif**  | 🟢 **Corrigé** |
| **Verrouillage de compte absent**    | **6.0 (MOYENNE)** | `LockoutThreshold` = 0.                                                     |  🔴 **Actif**  | 🟢 **Corrigé** |

**Actions de Hardening Appliquées :**

- **Protocoles :** **Désactivation complète de SMBv1**. Passage au niveau de compatibilité **LM Compatibility Level 5** (Refus total de NTLMv1).
- **Politique de Sécurité (GPO) :**
  - Longueur minimale du mot de passe : **14 caractères** avec complexité.
  - Historique : **24 mots de passe mémorisés**.
  - Verrouillage : **5 tentatives / 15 minutes**.

## 4. Automatisation de la Surveillance

Afin de pérenniser les corrections et d'assurer une hygiène de sécurité continue, un script Bash (`audit_nexatech.sh`) a été développé et déployé sur le serveur Linux.

**Fonctionnalités clés du script :**

- Vérification de l'état du pare-feu (UFW).
- Contrôle du port d'écoute SSH.
- Audit des permissions des fichiers sensibles (`/etc/shadow`).
- Recherche de binaires SUID suspects.

**Recommandation :** Intégrer l'exécution de ce script dans une tâche planifiée (**cron**) avec envoi de rapport par email à l'équipe IT.

## 5. Recommandations et Feuille de Route

Bien que les failles critiques aient été colmatées, des actions complémentaires sont nécessaires pour atteindre un niveau de maturité conforme aux standards de l'industrie.

|  Priorité   | Action                                                        | Périmètre       | Délai suggéré |
| :---------: | :------------------------------------------------------------ | :-------------- | :------------ |
|  **HAUTE**  | **Déploiement de LAPS** (Local Admin Password Solution)       | Windows AD      | 1 Semaine     |
|  **HAUTE**  | **Activation de Fail2Ban** (Protection Anti-Brute Force)      | Linux           | 1 Semaine     |
| **MOYENNE** | **Mise en place d'une Content Security Policy (CSP)** stricte | Application Web | 2 Semaines    |
| **MOYENNE** | **Audit de la rotation du mot de passe du compte KRBTGT**     | Windows AD      | 1 Mois        |
|  **BASSE**  | Mise en place d'une solution de **SIEM** (Wazuh/Graylog)      | Global          | 3 Mois        |

## 6. Conclusion

L'intervention de SecureOps a permis de traiter efficacement l'urgence sécuritaire liée à l'alerte de l'hébergeur. **Le risque immédiat de compromission des données RH est écarté.**

NexaTech dispose désormais d'une base saine :

- **Application** sécurisée contre les injections et vols de session.
- **Infrastructure** durcie contre les attaques réseau communes.

Il est impératif que l'équipe interne s'approprie le script d'audit et suive la feuille de route de recommandations pour maintenir ce niveau de sécurité dans la durée.

**Bilan de la posture finale de NexaTech :**

- **SSH :** Accès root désactivé, port déplacé (`707`), authentification par clé uniquement.
- **Réseau :** Pare-feu UFW actif en mode `deny incoming`.
- **Windows :** GPO sécurisées, SMBv1 supprimé, NTLMv2 strict.
- **Applicatif :** Vulnérabilités OWASP Top 10 critiques neutralisées (SQLi, IDOR).
- **Monitoring :** Scripts Bash et Python de contrôle déployés et prêts à être automatisés.

**Risques Résiduels Acceptés (Documentés) :**

- L'application DVWA reste en environnement de test à des fins de démonstration.
- Les scripts d'audit nécessitent une planification automatisée (non incluse dans le périmètre de la mission initiale).

**Équipe d'Audit SecureOps :**

- Clarisse DAUDEL, Consultante Cybersécurité
- Maxime CASTEL, Ingénieur Sécurité Systèmes & Réseaux

---

**Annexes :**

- [cartographie de l'infrastructure](cartographie_infrastructure/cartographie_infrastructure.md)
- [vulnérabilités applicatives](vulnerabilites_applicatives/vulnerabilites_applicatives.md)
- [Corrections applicatives](corrections_applicatives.md)
- [hardening serveur linux](hardening_serveur_linux.md)
- [hardening serveur windows](hardening_server_windows.md)
- [script bash v1](script_bash_v1.md)
- [script bash v2](script_bash_v2.md)
- [script audit complet](script_audit_complet.md)
