---
title: "Objectifs du chapitre"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - Objectifs du chapitre"
---

<nav class="page-sequence"><a href="cours/chapitre-3/index">Sommaire du chapitre</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/stockage-aws">Suivant</a></nav>

> [!info]
> Après l'étude des accès IAM, ce chapitre aborde deux composants structurants d'une architecture AWS : le stockage objet avec Amazon S3 et la capacité de calcul avec Amazon EC2.
>
> À l'issue de ce chapitre, vous saurez :
>
> - **Créer** et configurer un bucket Amazon S3 (chiffrement, versioning, politiques d'accès)
> - **Mettre en œuvre** des règles de lifecycle S3 pour optimiser le coût du stockage
> - **Manipuler** S3 via la CLI (upload, download, synchronisation, gestion des permissions)
> - **Choisir** un type d'instance EC2, une AMI et un mode de stockage adaptés à un besoin donné
> - **Configurer** des Security Groups pour contrôler le trafic réseau d'une instance EC2
> - **Utiliser** AWS Compute Optimizer pour dimensionner correctement une instance
> - **Comparer** les modèles de tarification EC2 (On-Demand, Reserved, Spot, Savings Plans)
> - **Lancer et administrer** une instance EC2 via la CLI
> - **Déployer** un Elastic Load Balancer pour répartir le trafic entre plusieurs instances
> - **Configurer** un groupe Auto Scaling pour adapter dynamiquement la capacité aux besoins
> - **Concevoir** une architecture haute disponibilité combinant S3, EC2, ELB et Auto Scaling

<a class="schema-zoom" href="assets/schemas/architecture-web-elastique.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/architecture-web-elastique.svg" alt="Architecture web élastique avec Route 53, un load balancer et des instances EC2 réparties sur deux zones"></a>

**Lecture du schéma.** Le load balancer distribue le trafic vers plusieurs instances gérées par Auto Scaling. La répartition sur plusieurs zones évite qu'une défaillance unique d'AZ interrompe toute la couche de calcul.

---

<nav class="page-sequence"><a href="cours/chapitre-3/index">Sommaire du chapitre</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/stockage-aws">Suivant</a></nav>
