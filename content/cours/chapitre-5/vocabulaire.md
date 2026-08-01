---
title: "Vocabulaire du chapitre"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - Vocabulaire du chapitre"
---

# Vocabulaire du chapitre

<nav class="page-sequence"><a href="cours/chapitre-5/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/reprise">Suivant</a></nav>

| Terme | Définition |
|---|---|
| RPO | Recovery Point Objective : quantité maximale de données que l'organisation accepte de perdre, exprimée comme un point de reprise dans le temps. |
| RTO | Recovery Time Objective : délai maximal visé pour rétablir un service après un incident. |
| Infrastructure as Code | Description versionnée d'une infrastructure dans des fichiers interprétés par un outil de déploiement. |
| Template CloudFormation | Document JSON ou YAML qui décrit les ressources et leurs propriétés. |
| Stack CloudFormation | Ensemble de ressources créé et géré comme une unité à partir d'un template. |
| Drift | Écart entre la configuration déclarée dans le template et l'état réel des ressources. |
| Métrique | Série de valeurs numériques horodatées utilisée pour observer un système. |
| Alarme CloudWatch | Règle qui surveille une métrique et change d'état lorsqu'un seuil ou une condition est atteint. |

---

:::info
Le réseau et les bases de données mis en place au chapitre précédent constituent une architecture fonctionnelle mais encore déployée manuellement ; ce dernier chapitre referme la formation en automatisant ce déploiement et en donnant les clés pour évaluer et faire évoluer une architecture AWS dans la durée.

**Objectifs du chapitre**

À l'issue de ce chapitre, les stagiaires seront capables de :

- **Définir** les notions de RTO/RPO et mettre en œuvre une stratégie de sauvegarde avec AWS Backup et les snapshots EBS/RDS
- **Expliquer** les limites du déploiement manuel et l'intérêt de l'Infrastructure as Code
- **Écrire** un template CloudFormation en YAML et le déployer via la CLI
- **Utiliser** AWS Systems Manager pour administrer des instances à distance sans SSH
- **Déployer** une application avec Elastic Beanstalk et la comparer à une architecture serverless (Lambda)
- **Créer** des alarmes et des tableaux de bord CloudWatch pour superviser une infrastructure
- **Appliquer** les six piliers du AWS Well-Architected Framework à une étude de cas concrète
- **Utiliser** AWS Compute Optimizer pour ajuster le dimensionnement des ressources
- **Découpler** des composants applicatifs avec Amazon SQS et Amazon SNS
- **Concevoir** une architecture microservices sans serveur avec API Gateway et Step Functions, et justifier les choix de découplage
- **Situer** les certifications AWS (Cloud Practitioner à Solutions Architect Professional) et les domaines couverts par la SAA-C03
:::

---

<nav class="page-sequence"><a href="cours/chapitre-5/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/reprise">Suivant</a></nav>
