---
title: "7. Options de tarification AWS EC2"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 7. Options de tarification AWS EC2"
---

# 7. Options de tarification AWS EC2

<nav class="page-sequence"><a href="cours/chapitre-3/compute-optimizer">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/elb">Suivant</a></nav>

AWS propose plusieurs modèles de tarification pour s'adapter aux besoins techniques et budgétaires des entreprises. Le choix dépend du niveau de prévisibilité des workloads, du budget disponible, et de la tolérance aux interruptions.

### 7.1 On-Demand (À la demande)

- **Paiement à l'heure** ou à la seconde pour la capacité de calcul utilisée, sans engagement à long terme.
- **Idéal pour** les charges de travail à court terme, les tests et le développement.
- **Pas de paiement anticipé** ni d'engagement minimum.
- **Prix plus élevé** que les autres options mais offre une flexibilité maximale.
- **Recommandé pour** les applications ne pouvant pas être interrompues et ayant des charges de travail imprévisibles.

**Exemple** : Vous avez un pic de trafic imprévu. Vous lancez des instances On-Demand pour répondre à la demande, puis les arrêtez après le pic.

### 7.2 Savings Plans

- **Engagement de consommation horaire** sur une période de 1 ou 3 ans.
- **Réduction variable** par rapport au tarif à la demande selon le plan, la durée et le mode de paiement.
- **Deux types principaux** :
  - **Compute Savings Plans** : Flexibilité maximale couvrant EC2, Fargate et Lambda, avec support multi-familles d'instances, tailles et régions.
  - **EC2 Instance Savings Plans** : Réductions plus importantes mais limité à une famille d'instances dans une région spécifique.
- **Options de paiement** flexibles impactant le taux de réduction :
  - **No Upfront** : Aucun paiement initial.
  - **Partial Upfront** : Paiement partiel initial.
  - **Full Upfront** : Paiement total initial offrant les meilleures réductions.

### 7.3 Instances Spot (À prix réduit)

- **Utilisation de la capacité EC2 inutilisée** d'AWS.
- **Remise variable** par rapport au prix à la demande, en échange d'un risque d'interruption.
- **Les instances peuvent être interrompues** avec un préavis de 2 minutes si AWS a besoin de la capacité.
- **Idéal pour** :
  - Les charges de travail tolérantes aux interruptions.
  - Le calcul haute performance (HPC).
  - Les jobs batch (traitement par lots).
  - Les workloads flexibles en termes de début et de fin.

**Best practices pour la résilience** :
- Utiliser les **groupes d'auto-scaling** pour gérer les interruptions automatiquement.
- Implémenter via **EC2 Spot Fleet** ou **EC2 Spot Instances Requests**.
- Concevoir l'application pour tolérer les interruptions.

:::warning
**Instances Spot : interruption en 2 minutes** — AWS peut récupérer vos instances Spot avec seulement **2 minutes de préavis** lorsque la capacité est nécessaire. Ne jamais utiliser des instances Spot pour des workloads critiques sans tolérance aux interruptions (bases de données de production, serveurs web sans état de session externalisé). Toujours prévoir un mécanisme de sauvegarde ou de checkpoint des données en cours de traitement.
:::

### 7.4 Reserved Instances (RI)

- **Engagement** sur une instance spécifique pour **1 ou 3 ans**.
- **Remise variable** par rapport au prix à la demande, contre un engagement de durée et de configuration.
- **Différences principales** avec les Savings Plans :
  - Les RI sont liées à une instance spécifique (type, taille, région, zone).
  - Les Savings Plans sont basés sur un engagement de consommation en dollars (plus flexibles).
  - Les RI peuvent être vendues sur le **AWS RI Marketplace**, pas les Savings Plans.

### 7.5 Comparatif synthétique

| Critère | On-Demand | Reserved | Spot | Savings Plans |
|---------|-----------|----------|------|----------------|
| **Engagement** | Aucun | 1 ou 3 ans | Aucun | 1 ou 3 ans |
| **Réduction potentielle** | Référence | Variable selon l'engagement | Variable selon la capacité disponible | Variable selon l'engagement |
| **Flexibilité** | ✅✅✅ | ❌ | ✅✅ | ✅✅ |
| **Risque d'interruption** | ❌ | ❌ | ✅✅✅ | ❌ |
| **Idéal pour** | Dev/Test | Prod stable | Batch/CI | Prod optimisée |

**Décision** : il n'existe pas de modèle universellement meilleur. La charge stable favorise un engagement ; la charge interruptible favorise Spot ; l'incertitude favorise le paiement à la demande. La décision doit s'appuyer sur les métriques réelles.

### 7.6 Cas métier : Choisir la meilleure option tarifaire

#### Cas 1 : Site e-commerce avec trafic prévisible

- **Charge** : trafic stable, pics prévisibles en fin d'année.
- **Infrastructure** : 10 instances t3.large en continu, +20 during soldes.
- **Recommandation** : **Savings Plans (Compute)** pour les 10 instances permanentes + **Spot** pour les 20 supplémentaires pendant les soldes.
- **Validation économique** : comparer dans AWS Pricing Calculator le socle engagé, la capacité à la demande et le renfort Spot avec les tarifs du jour.

#### Cas 2 : Environnement de développement/test

- **Charge** : variable, utilisation heures de travail uniquement.
- **Infrastructure** : 2-4 instances selon le sprint en cours.
- **Recommandation** : **On-Demand** uniquement (pas d'engagement, flexibilité totale).
- **Économie** : aucune, mais coûts minimaux et liberté maximale.

#### Cas 3 : Job batch nocturne de traitement

- **Charge** : lance chaque nuit des instances pour 4h, puis arrêt.
- **Infrastructure** : 50 instances c5.2xlarge pour le parallélisme.
- **Recommandation** : **Spot instances** avec **Spot Fleet** (demande auto-scaling de remplacement).
- **Économie attendue** : à estimer avec le tarif Spot observé, le tarif à la demande de référence et le coût des interruptions. Le pourcentage varie selon la famille, la région et la capacité disponible.

```bash
# Configuration Spot Fleet pour job batch
aws ec2 request-spot-fleet \
    --spot-fleet-request-config '{
        "IamFleetRole": "arn:aws:iam::xxxxx:role/fleet",
        "SpotPrice": "0.10",
        "TargetCapacity": 50,
        "LaunchSpecifications": [{
            "ImageId": "ami-xxxxx",
            "InstanceType": "c5.2xlarge",
            "KeyName": "ma-cle"
        }]
    }'
```

:::success
**Résultat attendu :**
```json
{
    "SpotFleetRequestId": "sfr-0a1b2c3d4e5f6789a",
    "SpotFleetRequestState": "submitted"
}
```
:::

#### Cas 4 : Application critiques 24/7 avec charge non prévisible

- **Charge** : pas de pattern clair, augmentations soudaines.
- **Infrastructure** : 4-30 instances selon la demande.
- **Recommandation** : **Savings Plans (mélange)** pour la charge de base + **On-Demand** pour les pics.
- **Avantage** : si charge explose au-delà des prévisions, On-Demand absorbe sans coupure.

### 7.7 Outil : AWS Pricing Calculator

```text
URL : https://calculator.aws/
1. Sélectionner la région
2. Ajouter EC2 : type, nombre, durée
3. Sélectionner option (On-Demand, Reserved, Spot)
4. Voir l'estimation mensuelle/annuelle
5. Exporter en PDF pour justifier budgets
```

📎 [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

<nav class="page-sequence"><a href="cours/chapitre-3/compute-optimizer">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/elb">Suivant</a></nav>
