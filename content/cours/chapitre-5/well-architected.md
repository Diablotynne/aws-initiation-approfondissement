---
title: "7. AWS Well-Architected Framework — Mise en pratique"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 7. AWS Well-Architected Framework — Mise en pratique"
---

# 7. AWS Well-Architected Framework — Mise en pratique

<nav class="page-sequence"><a href="cours/chapitre-5/cloudwatch">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/evenements">Suivant</a></nav>

Le Chapitre 1 a introduit les six piliers du **AWS Well-Architected Framework** (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability) avec leurs bonnes pratiques respectives. Maintenant que vous avez manipulé IAM, S3, EC2, Lambda, RDS, VPC et CloudFormation, vous disposez de tous les services nécessaires pour appliquer concrètement ce framework à un cas réel.

:::info
**Besoin d'un rappel des 6 piliers ?** Retournez au Chapitre 1, section 7 — définitions, questions clés et bonnes pratiques par pilier y sont détaillées.
:::

### 7.1 Cas d'étude : Application "CloudPizza"

Imaginons une application de commande de pizzas. Appliquons les 6 piliers :

| Pilier | Décision | Justification |
|--------|----------|---------------|
| **Operational Excellence** | Déployer via CloudFormation + CI/CD | Répétabilité, traçabilité |
| **Security** | IAM par service, KMS pour BDD, bucket S3 privé | Moindre privilège, conformité |
| **Reliability** | Multi-AZ, ALB, RDS Multi-AZ, snapshots EBS quotidiens | RTO 1h, RPO 1h |
| **Performance** | Lambda et DynamoDB peuvent retirer la gestion de serveurs | Mise à l'échelle gérée, dans les quotas et avec un modèle de données adapté |
| **Cost Optimization** | Reserved Instances pour serveurs stables, Spot pour batch | Réduire 40% des coûts |
| **Sustainability** | Déployer en Irlande (énergies renouvelables), Lambda sans serveur | Réduire l'empreinte carbone |

---

### 7.2 Rappel — AWS Compute Optimizer et le pilier Cost Optimization

Le Chapitre 3 a détaillé le fonctionnement d'**AWS Compute Optimizer** (collecte CloudWatch, analyse ML, recommandations chiffrées) — c'est l'outil concret qui alimente le pilier **Cost Optimization** vu ci-dessus : dans le cas CloudPizza, c'est lui qui permettrait de vérifier a posteriori que les Reserved Instances choisies sont bien dimensionnées à l'usage réel.

:::info
**Besoin d'un rappel du fonctionnement de Compute Optimizer ?** Retournez au Chapitre 3, section 7 — commande CLI complète, exemple de sortie JSON et cas concret `t3.large → t3.small` y sont détaillés.
:::

---

<nav class="page-sequence"><a href="cours/chapitre-5/cloudwatch">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/evenements">Suivant</a></nav>
