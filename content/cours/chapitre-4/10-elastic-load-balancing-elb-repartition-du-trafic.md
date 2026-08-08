---
title: "10. Elastic Load Balancing (ELB) — Répartition du trafic"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 10. Elastic Load Balancing (ELB) — Répartition du trafic"
---

<nav class="page-sequence"><a href="cours/chapitre-4/9-lancer-une-instance-ec2-en-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/11-auto-scaling-adaptation-dynamique-des-ressources">Suivant</a></nav>

### 10.1 Pourquoi un Load Balancer ?

Un **Load Balancer** agit comme un répartiteur de trafic. Il reçoit les requêtes des clients et les distribue vers les instances EC2 disponibles, selon des règles de routage et de santé (**health checks**).

#### Architecture simple

<a class="schema-zoom" href="assets/schemas/elb-architecture.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/elb-architecture.svg"
     alt="Elastic Load Balancing — Architecture"
     style="display:block; margin:auto; width:90%"></a>

### 10.2 Types de Load Balancer AWS

AWS propose trois types de Load Balancer, qui n'opèrent pas au même niveau du modèle réseau OSI :

| Type | Cas d'usage typique | Protocole | Niveau OSI |
|---|---|---|---|
| **ALB (Application Load Balancer)** | Applications web, microservices | HTTP/HTTPS | Couche 7 (Application) |
| **NLB (Network Load Balancer)** | Faible latence, TCP | TCP/UDP | Couche 4 (Transport) |
| **GLB (Gateway Load Balancer)** | Appliances réseau (firewall, inspection) | IP | Couche 3 (Réseau) |

**ALB (Application Load Balancer)** : conçu pour les applications web. Il fonctionne au niveau **HTTP/HTTPS** (couche 7 du modèle OSI) et permet un **routage avancé** (par URL, en-tête, hostname, etc.).

**NLB (Network Load Balancer)** : adapté aux applications nécessitant une **faible latence**. Il fonctionne au niveau **TCP** (couche 4), idéal pour les bases de données ou les services temps réel.

**GLB (Gateway Load Balancer)** : utilisé pour intégrer des **appliances réseau** comme des pare-feu ou des outils d'inspection. Il fonctionne au niveau **IP**.

![](assets/schemas/ch3-capture-03-43719ff5.png)

### 10.3 Fonctionnement du Load Balancer

Concrètement, le Load Balancer répète un cycle simple pour maintenir le trafic dirigé uniquement vers des instances fonctionnelles :

- Le Load Balancer **vérifie l'état** des instances via des **health checks** (tests de disponibilité).
- Il **répartit les requêtes** vers les instances **saines** uniquement.
- Il s'**adapte automatiquement** à l'ajout ou la suppression d'instances via **Auto Scaling**.

**Health Check - Exemple** :
```
Chaque 30 secondes :
1. LB envoie requête GET http://instance:80/health
2. Instance répond HTTP 200 OK
3. Instance est marquée "saine"

Si pas de réponse ou erreur 5xx :
1. Instance marquée "défaillante"
2. Pas plus de trafic envoyé vers elle
```

📎 [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)

---

<nav class="page-sequence"><a href="cours/chapitre-4/9-lancer-une-instance-ec2-en-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/11-auto-scaling-adaptation-dynamique-des-ressources">Suivant</a></nav>
