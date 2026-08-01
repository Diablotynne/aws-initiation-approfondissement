---
title: "Vocabulaire du chapitre"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - Vocabulaire du chapitre"
---

# Vocabulaire du chapitre

<nav class="page-sequence"><a href="cours/chapitre-4/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/bases-donnees">Suivant</a></nav>

| Terme | Définition |
|---|---|
| Base relationnelle | Base organisée en tables liées et interrogée généralement avec SQL. |
| NoSQL | Famille de modèles non relationnels, par exemple clé-valeur ou document, conçus pour des accès spécifiques et une distribution horizontale. |
| RDS | Relational Database Service : service AWS managé pour plusieurs moteurs de bases relationnelles. |
| Aurora | Moteur relationnel managé par AWS, compatible avec MySQL ou PostgreSQL selon l'édition choisie. |
| DynamoDB | Base NoSQL clé-valeur et document entièrement managée par AWS. |
| VPC | Virtual Private Cloud : réseau virtuel logiquement isolé dans AWS. |
| CIDR | Notation qui décrit un bloc d'adresses IP au moyen d'une adresse réseau et d'une longueur de préfixe. |
| Subnet | Sous-réseau d'un VPC, limité à une seule zone de disponibilité. |
| Table de routage | Ensemble de règles qui détermine la prochaine destination d'un paquet réseau. |

---

:::info
Les instances EC2 et les buckets S3 déployés au chapitre précédent doivent maintenant s'intégrer dans un réseau maîtrisé et s'appuyer sur des bases de données managées : ce chapitre couvre les deux piliers d'une architecture AWS mature, le réseau (VPC) et la donnée persistante (RDS, Aurora, DynamoDB).

**Objectifs du chapitre**

À l'issue de ce chapitre, les stagiaires seront capables de :

- **Expliquer** l'intérêt des bases de données managées face à une base auto-administrée
- **Déployer** une base Amazon RDS en haute disponibilité (Multi-AZ) et en sécuriser l'accès
- **Différencier** Amazon Aurora d'une base RDS classique en termes de performance et de résilience
- **Utiliser** Amazon DynamoDB pour un cas d'usage NoSQL à forte scalabilité
- **Planifier** une migration de base de données avec AWS DMS
- **Concevoir** une VPC avec subnets publics et privés, table de routage et passerelle Internet/NAT
- **Sécuriser** le trafic réseau avec des Security Groups et des Network ACLs
- **Interconnecter** plusieurs VPC avec le VPC Peering et AWS Transit Gateway
- **Utiliser** un VPC Endpoint pour accéder à un service AWS sans transiter par Internet
- **Configurer** une zone DNS et des enregistrements avec Amazon Route 53, dont des politiques de routage avancées
- **Mettre en place** un cluster ElastiCache (Redis) pour accélérer l'accès aux données fréquemment lues
:::

---

<nav class="page-sequence"><a href="cours/chapitre-4/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/bases-donnees">Suivant</a></nav>
