---
title: "9. Certifications AWS — Objectif SAA-C03"
description: "\"Chapitre 5 — Automatisation, supervision et reprise d'activité\" - 9. Certifications AWS — Objectif SAA-C03"
---

<nav class="page-sequence"><a href="cours/chapitre-5/8-services-complementaires-queues-et-evenements">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/ressources">Suivant</a></nav>

Le Chapitre 1 a présenté les quatre niveaux de certification AWS (Fondamental, Associate, Professional, Specialty) et pourquoi cette formation cible la **Solutions Architect Associate (SAA-C03)**. Maintenant que vous avez vu l'ensemble des services du programme, voici ce qui compte vraiment pour préparer concrètement cet examen : la pondération réelle des domaines testés.

<a class="schema-zoom" href="assets/schemas/aws-certification-path.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-certification-path.svg"
     alt="Parcours de certification AWS : Foundational, puis trois Associate (dont SAA-C03 ciblé par cette formation), puis Professional, puis Specialty"
     style="display:block; margin:auto; width:90%"></a>

> [!info]
> **Besoin d'un rappel des 4 niveaux de certification ?** Retournez au Chapitre 1, section 1.3.

---

### 9.3 Domaines couverts par la SAA-C03

| Domaine | Pondération |
|---------|-------------|
| Design d'architectures résilientes | 26 % |
| Design d'architectures haute performance | 24 % |
| Design d'architectures sécurisées | 30 % |
| Design d'architectures optimisées en coût | 20 % |

> Les quatre piliers de cette formation correspondent exactement à ces quatre domaines.

---

### 9.4 Certifications spécialisées

Les certifications **Specialty** valident une expertise approfondie sur un domaine précis. Elles nécessitent généralement une expérience pratique significative.

| Specialty | Code | Domaine |
|-----------|------|---------|
| Security | SCS-C02 | IAM, chiffrement, conformité, détection des menaces |
| Machine Learning | MLS-C01 | SageMaker, data pipelines, modèles ML |
| Advanced Networking | ANS-C01 | VPC avancé, Direct Connect, Transit Gateway |
| Data Analytics | DAS-C01 | Redshift, Athena, EMR, Glue, QuickSight |
| Database | DBS-C01 | RDS, DynamoDB, Neptune, ElastiCache |
| SAP on AWS | PAS-C01 | Déploiement SAP sur infrastructure AWS |

Ces codes suivent une logique simple : les deux ou trois premières lettres identifient le domaine (SCS = Security, MLS = Machine Learning, ANS = Advanced Networking, DAS = Data Analytics, DBS = Database, PAS = SAP), et le suffixe `-C0x` numérote la version de l'examen — comme pour SAA-C03 vu plus haut, un chiffre plus élevé signale une version plus récente qui remplace la précédente. Deux services mentionnés dans la colonne Domaine n'ont pas encore été détaillés dans cette formation : **QuickSight** est le service AWS de tableaux de bord et de visualisation de données (BI) qui se branche directement sur Redshift, Athena ou S3 pour construire des graphiques interactifs sans infrastructure à gérer ; **Neptune** est une base de données de graphes managée, pensée pour les données fortement connectées entre elles (réseaux sociaux, moteurs de recommandation, détection de fraude) là où RDS ou DynamoDB modélisent plutôt des tables ou des documents indépendants.

> Pour aller plus loin : [Parcours de certifications AWS](https://aws.amazon.com/certification/) — programme officiel AWS (Solution Architect, Developer, SysOps, DevOps…)

---

La boucle est bouclée : IAM sécurise, VPC isole, EC2/S3/ECS exécutent, CloudFormation automatise. Ce que les cinq chapitres ont posé un par un, c'est maintenant à vous de le recomposer sur des cas réels — dans les ateliers Go Deploy et AWS Academy, puis en conditions d'examen si vous visez la certification Solutions Architect Associate.

<nav class="page-sequence"><a href="cours/chapitre-5/8-services-complementaires-queues-et-evenements">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/ressources">Suivant</a></nav>
