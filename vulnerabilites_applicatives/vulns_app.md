---
created_at: 2026-04-13
updated_at: 2026-04-14
---

↑ [vulnérabilités applicatives](vulnerabilites_applicatives.md)

---

# Vulnérabilités applicatives DVWA

## 1. SQL Injection

Module : SQL Injection  
Payload :

```sql
1' or '1'='1
```

![](../assets/vulns_app-1776160018192.webp)

Résultat :

- récupération de l'utilisateur admin
- fuite d'informations en base

Impact :

- accès non autorisé aux données
- extraction d'utilisateurs

---

## 2. XSS Reflected

Module : XSS (Reflected)  
Payload :

```html
<script>
  alert(1);
</script>
```

![](../assets/vulns_app-1776160040003.webp)

![](../assets/vulns_app-1776160046164.webp)

Résultat :

- popup JavaScript exécutée

Impact :

- exécution JS côté client
- vol de session possible

---

## 3. Mauvaise gestion d'authentification et IDOR

Module : Authorisation Bypass

Preuve :

Accès  à L’URL `/vulnerabilities/authbypass/` par l'utilisateur `gordonb` et modification d'un utilisateur

![](../assets/vulns_app-1776160071030.webp)

Résultat :

- accès à une page réservée à l'administrateur
- modification possible des utilisateurs

Impact :

- élévation de privilèges
- accès non autorisé

---

## 4. Weak Session IDs

Module : Weak Session IDs

Preuve :

- cookie dvwaSession = 7
- valeur incrémentale et prévisible

Impact :

- prédiction de session
- session hijacking possible

---

## 5. Mauvais headers HTTP

Commande :

```bash
curl -v http://10.69.7.2
```

![](../assets/vulns_app-1776160239639.webp)

Résultat :

- Server: Apache/2.4.66 (Debian)
- divulgation de version Apache
- absence de headers de sécurité visibles
