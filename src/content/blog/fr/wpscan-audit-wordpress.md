---
title: "WPScan : auditer la sécurité d'un site WordPress en ligne de commande"
description: "Guide pratique de WPScan, l'outil open source de référence pour détecter les vulnérabilités WordPress : installation, commandes essentielles, interprétation des résultats."
pubDate: 2026-02-10
tags: ["Sécurité", "WordPress", "WPScan"]
lang: fr
slug: wpscan-audit-wordpress
---

## Introduction : WordPress et la surface d'attaque

WordPress propulse environ 43 % des sites web mondiaux. Cette popularité en fait une cible de choix pour les attaquants : les plugins abandonnés, les thèmes obsolètes et les mots de passe d'administration faibles sont exploités en masse par des bots automatisés.

Avant d'auditer manuellement un site WordPress, j'utilise systématiquement WPScan pour obtenir une cartographie rapide de la surface d'attaque. C'est un outil open source écrit en Ruby, maintenu par la communauté de sécurité, avec une base de données de vulnérabilités régulièrement mise à jour.

## Présentation de WPScan

WPScan est un scanner de sécurité spécialisé WordPress. Il peut :

- Détecter la version de WordPress et les fichiers exposés
- Lister les plugins et thèmes installés (actifs ou non)
- Identifier les vulnérabilités connues (CVE) pour chaque composant
- Énumérer les utilisateurs
- Tester des mots de passe par force brute

Il utilise la [WPScan Vulnerability Database](https://wpscan.com/), une base de données communautaire et professionnelle qui recense des milliers de vulnérabilités WordPress. L'API gratuite permet 25 requêtes par jour, suffisant pour des audits ponctuels.

## Installation

### Via RubyGems

WPScan nécessite Ruby 2.7+. Sur Debian/Ubuntu :

```bash
sudo apt install ruby ruby-dev libcurl4-openssl-dev make zlib1g-dev
sudo gem install wpscan
wpscan --version
```

### Via Docker (recommandé)

La méthode Docker évite les problèmes de dépendances et garantit d'utiliser la dernière version :

```bash
docker pull wpscanteam/wpscan
docker run -it --rm wpscanteam/wpscan --url https://target.example.com
```

## Les commandes essentielles

### Scan de base

```bash
wpscan --url https://target.example.com
```

Détecte la version WordPress, les fichiers courants exposés (`readme.html`, `xmlrpc.php`, `wp-cron.php`) et les headers de sécurité.

### Énumération des plugins vulnérables

```bash
wpscan --url https://target.example.com \
  --enumerate vp \
  --api-token VOTRE_TOKEN_API
```

`vp` = *vulnerable plugins*. Avec un token API, WPScan croise chaque plugin détecté avec la base de données de vulnérabilités et affiche les CVE correspondantes avec leur score CVSS.

### Énumération des thèmes

```bash
wpscan --url https://target.example.com \
  --enumerate vt \
  --api-token VOTRE_TOKEN_API
```

`vt` = *vulnerable themes*. Les thèmes premium abandonnés par leur auteur sont une source fréquente de vulnérabilités non corrigées.

### Énumération des utilisateurs

```bash
wpscan --url https://target.example.com --enumerate u
```

WPScan exploite plusieurs vecteurs pour trouver les noms d'utilisateurs : l'API REST WordPress, les pages auteurs, les flux RSS. Un nom d'utilisateur `admin` encore actif est un signal d'alerte.

### Test de mots de passe (brute force)

```bash
wpscan --url https://target.example.com \
  --usernames admin \
  --passwords /usr/share/wordlists/rockyou.txt \
  --max-threads 10
```

**Attention** : cette commande ne doit être utilisée qu'avec l'autorisation explicite du propriétaire du site. Elle génère du trafic significatif et peut déclencher des protections (ban IP, lockout de compte).

## Interpréter les résultats et prioriser

WPScan produit un rapport structuré. Les points d'attention principaux :

**Version WordPress non masquée** : WPScan affiche la version détectée. Si elle est inférieure à la version stable courante, c'est une priorité de mise à jour.

**Plugins avec CVE** : chaque vulnérabilité affiche le score CVSS, le type (XSS, SQLi, CSRF...) et un lien vers les détails. Je priorise les scores > 7 et les vulnérabilités exploitables sans authentification.

**`xmlrpc.php` activé** : ce fichier est un vecteur d'attaque par force brute et DDoS. À désactiver s'il n'est pas utilisé.

**`readme.html` accessible** : révèle la version de WordPress. Supprimer ou restreindre l'accès.

**Utilisateurs énumérables** : renommer `admin`, activer la double authentification, mettre en place un plugin de limitation de tentatives de connexion (Limit Login Attempts Reloaded, par exemple).

## Limitations : scan passif vs actif

WPScan fonctionne principalement en **détection passive** : il analyse les réponses HTTP, les métadonnées publiques, les fichiers accessibles. Il ne teste pas activement l'exploitation des vulnérabilités.

Pour aller plus loin :

- **OWASP ZAP** ou **Burp Suite** pour les tests d'injection actifs
- **Metasploit** pour valider l'exploitabilité de vulnérabilités spécifiques (en environnement contrôlé)
- Revue manuelle du code des plugins custom

WPScan est donc un excellent **premier filtre**, mais ne remplace pas un audit complet.

## Conclusion

WPScan est un outil indispensable dans la boîte à outils de tout développeur ou auditeur travaillant avec WordPress. En quelques minutes, il donne une image claire de l'état de sécurité d'un site et identifie les vulnérabilités les plus urgentes à corriger.

L'intégrer dans une routine de maintenance mensuelle — aux côtés des mises à jour WordPress, plugins et thèmes — est une bonne pratique accessible à tous, même sans expertise poussée en sécurité.
