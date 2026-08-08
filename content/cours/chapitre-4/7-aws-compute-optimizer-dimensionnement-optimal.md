---
title: "7. AWS Compute Optimizer — Dimensionnement optimal"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 7. AWS Compute Optimizer — Dimensionnement optimal"
---

<nav class="page-sequence"><a href="cours/chapitre-4/6-choisir-le-bon-type-dinstance-ami-stockage-et-securite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/8-options-de-tarification-aws-ec2">Suivant</a></nav>

### 7.1 Qu'est-ce que AWS Compute Optimizer ?

**AWS Compute Optimizer** est un service qui **analyse vos patterns d'utilisation** des instances EC2 et recommande des types plus optimisés en coût et performance.

| | Instance actuelle | Recommandation |
|---|---|---|
| Type | t3.large (trop puissante) | t3.small (plus économique) |
| CPU utilisation | 5% | — |
| Mémoire | 12% | — |
| Coût mensuel | 80 $ | ~48 $ (économie 60%) |
| Performance | — | identique |
| Risque | — | très faible (marges CPU) |

Compute Optimizer applique cette analyse automatiquement à partir des métriques CloudWatch collectées.

### 7.2 Fonctionnement

1. **Collecte** : Compute Optimizer récupère les métriques CloudWatch (CPU, mémoire, réseau) sur **14 jours minimum**.
2. **Analyse** : Machine Learning compare votre utilisation réelle avec les capabilities des autres types.
3. **Recommandation** : Propose des types économiquement viables.
4. **Confiance** : Indique un score de confiance (low, medium, high).

Ce score de confiance conditionne directement la nature de la recommandation reçue, qui se décline en quatre catégories.

### 7.3 Types de recommandations

Compute Optimizer ne se limite pas à proposer une taille plus petite : il distingue quatre types de recommandations selon ce que révèle l'analyse de vos métriques :

| Recommandation | Bénéfice | Risque | Exemple |
|---|---|---|---|
| **Downsizer** | Économies importantes | Risque d'augmenter le CPU > 100% | t3.large → t3.small |
| **Upgrade** | Meilleure performance | Légère augmentation de coût | m5.large → m5.xlarge |
| **Switch Family** | Meilleure performance/$ | Changement d'architecture | t3.large → m6i.large |
| **Aucune recommandation** | Instance bien dimensionnée | N/A | ✅ Garder tel quel |

Voyons maintenant comment activer le service et récupérer concrètement ces recommandations via la CLI.

### 7.4 Activation et utilisation

Compute Optimizer analyse l'utilisation réelle de vos instances (CPU, mémoire, réseau) sur 14 jours et suggère le type le mieux adapté. La commande suivante affiche ces recommandations sous forme de tableau comparatif.

```bash
# Vérifier les recommandations Compute Optimizer
aws compute-optimizer get-ec2-instance-recommendations \
    --region eu-west-1 \
    --query 'instanceRecommendations[].{
        Instance:instanceArn,
        Current:currentInstanceType,
        Recommended:recommendationOptions[0].instanceType,
        Savings:recommendationOptions[0].savingsOpportunity.estimatedMonthlySavings.value,
        ConfidenceLevel:currentInstanceType
    }' \
    --output table
```

> [!tip]
> **Résultat attendu :**
> ```
> -------------------------------------------------------------------------------------------
> |                         GetEc2InstanceRecommendations                                   |
> +-------------------------------------+----------+------------+---------+------------------+
> |              Instance               | Current  | Recommended| Savings | ConfidenceLevel  |
> +-------------------------------------+----------+------------+---------+------------------+
> |  arn:aws:ec2:eu-west-1:123:instance | t3.large | t3.small   |  47.82  |  t3.large        |
> |  arn:aws:ec2:eu-west-1:123:instance | m5.xlarge| m5.large   |  62.40  |  m5.xlarge       |
> +-------------------------------------+----------+------------+---------+------------------+
> ```
> Si aucune recommandation n'apparaît, Compute Optimizer manque encore de données (il lui faut au minimum 30h d'activité sur les instances).

### 7.5 Cas d'usage

Au-delà de la simple réduction de facture, Compute Optimizer s'intègre dans plusieurs démarches organisationnelles :

- **Optimisation de coûts** : identifier toutes les instances surdimensionnées.
- **Gouvernance cloud** : politiques de rightsizing automatisées.
- **Migration** : recommandations pour basculer vers une architecture nouvelle.
- **Audit FinOps** : justification des dépenses EC2.

**Avantage clé** : Compute Optimizer s'appuie sur **12-14 jours de données réelles**, pas sur des hypothèses théoriques.

---

<nav class="page-sequence"><a href="cours/chapitre-4/6-choisir-le-bon-type-dinstance-ami-stockage-et-securite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/8-options-de-tarification-aws-ec2">Suivant</a></nav>
