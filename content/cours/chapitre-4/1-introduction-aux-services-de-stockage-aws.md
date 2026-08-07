---
title: "1. Introduction aux services de stockage AWS"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 1. Introduction aux services de stockage AWS"
---

<nav class="page-sequence"><a href="cours/chapitre-4/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/2-amazon-s3-le-stockage-objet-scalable">Suivant</a></nav>

📹 **Vidéo** : [Introduction to Amazon S3](https://www.youtube.com/watch?v=4RI3pDKpx38)

### 1.1 Pourquoi plusieurs services de stockage ?

Le stockage est au cœur de toute infrastructure cloud. AWS propose plusieurs types de stockage, mais **Amazon Simple Storage Service (S3)** est le service le plus emblématique : fiable, scalable et économique.

Créé en 2006, S3 a révolutionné la façon dont les entreprises stockent leurs données en passant d'un modèle de serveur à un modèle **d'espace de stockage à la demande**.

Avant de plonger dans les services techniques, il est essentiel de comprendre **pourquoi AWS propose plusieurs modèles de stockage** et dans quels contextes les utiliser.

Dans une entreprise traditionnelle, le stockage repose sur :
- des **disques durs internes** (pour les postes ou serveurs locaux),
- des **baies NAS/SAN** (pour le stockage partagé),
- parfois des sauvegardes sur bande ou sur site distant.

AWS transpose ces modèles dans le Cloud et les **rend flexibles, évolutifs et disponibles à la demande**.

### 1.2 Les trois modèles de stockage AWS

| Type de stockage | Service AWS        | Cas d'usage typique                              | Analogie utilisateur                 |
|-------------------|--------------------|--------------------------------------------------|---------------------------------------|
| **Objet**         | **Amazon S3**      | Sauvegarde, site statique, logs, Data Lake       | Dropbox / Google Drive               |
| **Bloc**          | **Amazon EBS**     | Disque de VM, base de données, stockage persistant | Disque dur local                     |
| **Fichier**       | **Amazon EFS/FSx** | Partage réseau, systèmes distribués              | NAS / Partage Windows                |

**À retenir** : Chaque type de stockage a ses propres performances, coûts et scénarios d'usage. Nous allons détailler S3 et EBS en priorité, car ce sont les services les plus utilisés par les administrateurs AWS en début de carrière.

📎 [Documentation Amazon S3](https://docs.aws.amazon.com/s3/)

---

<nav class="page-sequence"><a href="cours/chapitre-4/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/2-amazon-s3-le-stockage-objet-scalable">Suivant</a></nav>
