---
title: "3. Amazon Route 53 — DNS Intelligent"
description: "\"Chapitre 3 — Amazon VPC et bases de données AWS\" - 3. Amazon Route 53 — DNS Intelligent"
---

<nav class="page-sequence"><a href="cours/chapitre-3/2-amazon-vpc-concevoir-un-reseau-prive-securise">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/4-amazon-elasticache-mise-en-cache-distribuee">Suivant</a></nav>

### 3.1 Fondamentaux du DNS

#### Qu'est-ce que le DNS ?

Le **DNS (Domain Name System)** est un système **mondial décentralisé** qui traduit des **noms lisibles** (`www.example.com`) en **adresses IP** (`93.184.216.34`).

---

### 3.2 Amazon Route 53 — Service DNS managé

#### Définition

> **Amazon Route 53** est le **service DNS managé** d'AWS. Il permet de **résoudre les noms de domaine**, de **diriger le trafic intelligemment**, et d'**assurer la haute disponibilité**.

Le nom « Route 53 » vient du **port 53**, utilisé par le protocole DNS.

#### Capacités

Route 53 combine plusieurs fonctionnalités :

| Fonctionnalité | Description |
|---|---|
| **Registrar** | Acheter et gérer des domaines (.com, .fr, .io, etc.) |
| **Résolution DNS** | Traduire noms → IP |
| **Health Checks** | Vérifier si une ressource est disponible |
| **Routage intelligent** | Diriger le trafic selon latence, géolocalisation, poids, failover |
| **Alias Records** | Lier un domaine à une ressource AWS (ALB, CloudFront, S3) |

#### Types d'enregistrements DNS courants

| Type | Exemple | Rôle |
|------|---------|------|
| **A** | `example.com` → `93.184.216.34` | IPv4 |
| **AAAA** | `example.com` → `2606:2800:...` | IPv6 |
| **CNAME** | `www.example.com` → `example.com` | Alias |
| **MX** | `example.com` → `mail.example.com` | Serveur mail |
| **TXT** | `example.com` → `v=spf1...` | Enregistrement texte |
| **NS** | `example.com` → `ns1.route53...` | Serveurs DNS |

---

### 3.3 Politiques de routage — Diriger le trafic intelligemment

Route 53 n'est pas un simple DNS classique. C'est un **routeur de trafic applicatif**.

#### Politique Simple

Cas de base : un domaine pointe vers **une seule adresse IP**.

#### Politique Pondérée

Distribuer le trafic en pourcentage entre plusieurs ressources.

**Cas d'usage** : déploiement progressif, A/B testing, migration progressive.

#### Politique Latence

Router les utilisateurs vers la ressource **la plus rapide** (latence réseau minimale).

**Cas d'usage** : applications globales, réduction latence.

#### Politique Failover (Basculement)

En cas de panne détectée, router vers une ressource de secours.

**Cas d'usage** : haute disponibilité, reprise après sinistre.

---

### 3.4 Health Checks et Monitoring

Route 53 peut **monitorer la santé** des ressources et basculer automatiquement.

#### Types de Health Checks

| Type | Description | Fréquence |
|------|---|---|
| **HTTP/HTTPS** | Effectue une requête GET, attend 2xx/3xx | Toutes les 30s |
| **TCP** | Établit une connexion TCP | Toutes les 10s |
| **Calculated** | Combine plusieurs health checks | Toutes les 30s |
| **CloudWatch** | Déclenché par une alarme CloudWatch | Variable |

![](assets/schemas/ch4-capture-09-0f658ba9.png)

![](assets/schemas/ch4-capture-10-042036a6.png)

---

<nav class="page-sequence"><a href="cours/chapitre-3/2-amazon-vpc-concevoir-un-reseau-prive-securise">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/4-amazon-elasticache-mise-en-cache-distribuee">Suivant</a></nav>
