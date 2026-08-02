---
title: "Objectifs du chapitre"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - Objectifs du chapitre"
---

<nav class="page-sequence"><a href="cours/chapitre-5/index">Sommaire du chapitre</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/reprise">Suivant</a></nav>

> [!info]
> Le réseau et les bases de données étudiés précédemment constituent une architecture fonctionnelle. Ce chapitre ajoute l'automatisation, la supervision et les mécanismes nécessaires pour évaluer et faire évoluer cette architecture.
>
> À l'issue de ce chapitre, vous saurez :
>
> - **Définir** les notions de RTO/RPO et mettre en œuvre une stratégie de sauvegarde avec AWS Backup et les snapshots EBS/RDS
> - **Expliquer** les limites du déploiement manuel et l'intérêt de l'Infrastructure as Code
> - **Écrire** un template CloudFormation en YAML et le déployer via la CLI
> - **Utiliser** AWS Systems Manager pour administrer des instances à distance sans SSH
> - **Déployer** une application avec Elastic Beanstalk et la comparer à une architecture serverless (Lambda)
> - **Créer** des alarmes et des tableaux de bord CloudWatch pour superviser une infrastructure
> - **Appliquer** les six piliers du AWS Well-Architected Framework à une étude de cas concrète
> - **Utiliser** AWS Compute Optimizer pour ajuster le dimensionnement des ressources
> - **Découpler** des composants applicatifs avec Amazon SQS et Amazon SNS
> - **Concevoir** une architecture microservices sans serveur avec API Gateway et Step Functions, et justifier les choix de découplage
> - **Situer** les certifications AWS (Cloud Practitioner à Solutions Architect Professional) et les domaines couverts par la SAA-C03

<a class="schema-zoom" href="assets/schemas/boucle-exploitation-aws.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/boucle-exploitation-aws.svg" alt="Boucle de déploiement, observation, intervention et amélioration d'une architecture AWS"></a>

**Lecture du schéma.** L'exploitation n'est pas une étape finale : les métriques et incidents alimentent une nouvelle décision d'architecture, puis une modification versionnée de l'infrastructure.

---

<nav class="page-sequence"><a href="cours/chapitre-5/index">Sommaire du chapitre</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/reprise">Suivant</a></nav>
