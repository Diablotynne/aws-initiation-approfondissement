---
title: "Objectifs du chapitre"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - Objectifs du chapitre"
---

<nav class="page-sequence"><a href="cours/chapitre-4/index">Sommaire du chapitre</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/vocabulaire">Suivant</a></nav>

> [!info]
> Les instances EC2 et les buckets S3 déployés au chapitre précédent doivent maintenant s'intégrer dans un réseau maîtrisé et s'appuyer sur des bases de données managées : ce chapitre couvre les deux piliers d'une architecture AWS mature, le réseau (VPC) et la donnée persistante (RDS, Aurora, DynamoDB).
>
> À l'issue de ce chapitre, vous saurez :
>
> - **Expliquer** l'intérêt des bases de données managées face à une base auto-administrée
> - **Déployer** une base Amazon RDS en haute disponibilité (Multi-AZ) et en sécuriser l'accès
> - **Différencier** Amazon Aurora d'une base RDS classique en termes de performance et de résilience
> - **Utiliser** Amazon DynamoDB pour un cas d'usage NoSQL à forte scalabilité
> - **Planifier** une migration de base de données avec AWS DMS
> - **Concevoir** une VPC avec subnets publics et privés, table de routage et passerelle Internet/NAT
> - **Sécuriser** le trafic réseau avec des Security Groups et des Network ACLs
> - **Interconnecter** plusieurs VPC avec le VPC Peering et AWS Transit Gateway
> - **Utiliser** un VPC Endpoint pour accéder à un service AWS sans transiter par Internet
> - **Configurer** une zone DNS et des enregistrements avec Amazon Route 53, dont des politiques de routage avancées
> - **Mettre en place** un cluster ElastiCache (Redis) pour accélérer l'accès aux données fréquemment lues

<a class="schema-zoom" href="assets/schemas/architecture-vpc-donnees.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/architecture-vpc-donnees.svg" alt="Architecture VPC segmentée avec subnets publics, applicatifs privés et base RDS privée"></a>

**Lecture du schéma.** Le routage détermine les destinations joignables ; les groupes de sécurité déterminent les flux autorisés. La base reste privée et n'accepte que la couche applicative prévue.

---

<nav class="page-sequence"><a href="cours/chapitre-4/index">Sommaire du chapitre</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/vocabulaire">Suivant</a></nav>
