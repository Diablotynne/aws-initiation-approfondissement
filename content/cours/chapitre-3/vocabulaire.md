---
title: "Vocabulaire du chapitre"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - Vocabulaire du chapitre"
---

# Vocabulaire du chapitre

<nav class="page-sequence"><a href="cours/chapitre-3/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/stockage-aws">Suivant</a></nav>

| Terme | Définition |
|---|---|
| Stockage objet | Stockage dans lequel chaque donnée est enregistrée comme un objet identifié par une clé et accompagné de métadonnées. |
| Bucket S3 | Conteneur logique Amazon S3 qui reçoit des objets et porte une partie de leur configuration. |
| AMI | Amazon Machine Image : modèle utilisé pour lancer une instance EC2 avec un système et une configuration initiale. |
| Instance EC2 | Serveur virtuel fourni par Amazon Elastic Compute Cloud. |
| EBS | Elastic Block Store : stockage bloc persistant attaché à une instance EC2. |
| EFS | Elastic File System : système de fichiers réseau managé pouvant être monté par plusieurs clients. |
| Load balancer | Répartiteur qui distribue les requêtes entre plusieurs cibles disponibles. |
| Auto Scaling | Mécanisme qui ajuste le nombre d'instances selon des règles, une planification ou des métriques. |

---

:::info
Après avoir sécurisé les accès avec IAM au chapitre précédent, les stagiaires disposent des bases nécessaires pour créer et protéger de vraies ressources AWS : ce chapitre aborde les deux briques les plus utilisées du Cloud AWS, le stockage objet S3 et le calcul EC2.

**Objectifs du chapitre**

À l'issue de ce chapitre, les stagiaires seront capables de :

- **Créer** et configurer un bucket Amazon S3 (chiffrement, versioning, politiques d'accès)
- **Mettre en œuvre** des règles de lifecycle S3 pour optimiser le coût du stockage
- **Manipuler** S3 via la CLI (upload, download, synchronisation, gestion des permissions)
- **Choisir** un type d'instance EC2, une AMI et un mode de stockage adaptés à un besoin donné
- **Configurer** des Security Groups pour contrôler le trafic réseau d'une instance EC2
- **Utiliser** AWS Compute Optimizer pour dimensionner correctement une instance
- **Comparer** les modèles de tarification EC2 (On-Demand, Reserved, Spot, Savings Plans)
- **Lancer et administrer** une instance EC2 via la CLI
- **Déployer** un Elastic Load Balancer pour répartir le trafic entre plusieurs instances
- **Configurer** un groupe Auto Scaling pour adapter dynamiquement la capacité aux besoins
- **Concevoir** une architecture haute disponibilité combinant S3, EC2, ELB et Auto Scaling
:::

---

<nav class="page-sequence"><a href="cours/chapitre-3/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/stockage-aws">Suivant</a></nav>
