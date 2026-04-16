# Projet SecureOps : Audit de Sécurité NexaTech

## Présentation du projet

Ce projet a été réalisé dans le cadre d'un bootcamp intensif de 5 jours. Il simule une mission d'audit de sécurité réelle confiée à un cabinet de conseil cyber par la PME **NexaTech**.

L'objectif est d'évaluer la posture de sécurité de l'entreprise face à une tentative d'intrusion détectée par son hébergeur, en analysant ses infrastructures critiques (Active Directory) et ses applications web (Gestion RH).

## Scénario d'Audit

**Client :** NexaTech (PME de 80 collaborateurs).

**Contexte :** NexaTech gère des documents RH sensibles via une application web PHP. Suite à une alerte de sécurité émanant de leur hébergeur signalant une tentative d'intrusion, une mission d'audit approfondie a été mandatée.

**Objectifs de la mission :**

1. **Audit d'Infrastructure :** Analyse de la configuration Windows Server 2022 et de l'Active Directory.
2. **Audit Applicatif :** Recherche de vulnérabilités critiques (OWASP Top 10) sur l'application RH.
3. **Automatisation :** Mise en place d'un script d'audit automatisé pour assurer une surveillance continue.
4. **Remédiation :** Proposition de correctifs et de bonnes pratiques de durcissement (_hardening_).

## Environnement Technique & Compétences

### Technologies utilisées

- **Systèmes :** Linux, Windows Server 2022.
- **Annuaire :** Active Directory (AD).
- **Web :** PHP (Application de gestion RH).
- **Sécurité :** Analyse des vulnérabilités OWASP (XSS, SQLi, IDOR).
- **Automatisation :** Scripting Bash pour l'audit système.

### Compétences mobilisées

- **Pentesting & Audit :** Identification et exploitation de failles (Injection SQL, Cross-Site Scripting, Insecure Direct Object Reference).
- **Sécurité Réseau & Système :** Analyse de configuration et durcissement d'infrastructure.
- **Gestion de crise :** Réponse à un incident et analyse de traces.
- **Reporting :** Rédaction de rapports techniques et présentation de livrables.

## Équipe

- DAUDEL Clarisse
- CASTEL Maxime

## Livrables

> [!NOTE]
> L'ensemble de ce projet est consultable avec l'outil [obsidian.md](https://obsidian.md/) pour faciliter la lecture et la navigation.

| Jour   | Livrable                                                                                                                                                                                                   |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **J1** | [Cartographie de l'infrastructure](./cartographie_infrastructure/cartographie_infrastructure.md) et liste des [vulnérabilités applicatives​](./vulnerabilites_applicatives/vulnerabilites_applicatives.md) |
| **J2** | [Corrections applicatives documentées](./corrections_applicatives.md) et [hardening serveur linux](hardening_serveur_linux.md)                                                                             |
| **J3** | Script Bash v1 fonctionnel                                                                                                                                                                                 |
| **J4** | Rapport d'audit technique détaillé                                                                                                                                                                         |
| **J5** | Démonstration des vulnérabilités (PoC) et soutenance orale                                                                                                                                                 |

## Statut du projet

| Phase                      | Statut      |
| -------------------------- | ----------- |
| Cartographie               | Terminé     |
| Analyse des vulnérabilités | Terminé     |
| Corrections applicatives   | Terminé     |
| Hardening serveur Linux    | Terminé     |
| Script d'automatisation    | En cours    |
| Rapport final              | En cours    |
| Soutenance                 | À livrer J5 |
