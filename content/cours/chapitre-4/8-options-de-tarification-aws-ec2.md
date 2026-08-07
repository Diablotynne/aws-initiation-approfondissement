---
title: "8. Options de tarification AWS EC2"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 8. Options de tarification AWS EC2"
---

<nav class="page-sequence"><a href="cours/chapitre-4/7-aws-compute-optimizer-dimensionnement-optimal">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/9-lancer-une-instance-ec2-en-cli">Suivant</a></nav>

AWS propose plusieurs modèles de tarification pour s'adapter aux besoins techniques et budgétaires des entreprises. Le choix dépend du niveau de prévisibilité des workloads, du budget disponible, et de la tolérance aux interruptions.

### 8.1 On-Demand (À la demande)

- **Paiement à l'heure** ou à la seconde pour la capacité de calcul utilisée, sans engagement à long terme.
- **Idéal pour** les charges de travail à court terme, les tests et le développement.
- **Pas de paiement anticipé** ni d'engagement minimum.
- **Prix plus élevé** que les autres options mais offre une flexibilité maximale.
- **Recommandé pour** les applications ne pouvant pas être interrompues et ayant des charges de travail imprévisibles.

**Exemple** : Vous avez un pic de trafic imprévu. Vous lancez des instances On-Demand pour répondre à la demande, puis les arrêtez après le pic.

### 8.2 Savings Plans

- **Engagement de consommation horaire en dollars** ($/heure) sur une période de 1 ou 3 ans.
- **Réductions** pouvant atteindre **72%** par rapport au tarif à la demande.
- **Deux types principaux** :
  - **Compute Savings Plans** : Flexibilité maximale couvrant EC2, Fargate et Lambda, avec support multi-familles d'instances, tailles et régions.
  - **EC2 Instance Savings Plans** : Réductions plus importantes mais limité à une famille d'instances dans une région spécifique.
- **Options de paiement** flexibles impactant le taux de réduction :
  - **No Upfront** : Aucun paiement initial.
  - **Partial Upfront** : Paiement partiel initial.
  - **Full Upfront** : Paiement total initial offrant les meilleures réductions.

### 8.3 Instances Spot (À prix réduit)

- **Utilisation de la capacité EC2 inutilisée** d'AWS.
- **Réductions jusqu'à 90%** par rapport au prix à la demande.
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

> [!warning]
> **Instances Spot : interruption en 2 minutes** — AWS peut récupérer vos instances Spot avec seulement **2 minutes de préavis** lorsque la capacité est nécessaire. Ne jamais utiliser des instances Spot pour des workloads critiques sans tolérance aux interruptions (bases de données de production, serveurs web sans état de session externalisé). Toujours prévoir un mécanisme de sauvegarde ou de checkpoint des données en cours de traitement.

### 8.4 Reserved Instances (RI)

- **Engagement** sur une instance spécifique pour **1 ou 3 ans**.
- **Réductions jusqu'à 75%** par rapport au prix à la demande.
- **Différences principales** avec les Savings Plans :
  - Les RI sont liées à une instance spécifique (type, taille, région, zone).
  - Les Savings Plans sont basés sur un engagement de consommation en dollars (plus flexibles).
  - Les RI peuvent être vendues sur le **AWS RI Marketplace**, pas les Savings Plans.

### 8.5 Comparatif synthétique

| Critère | On-Demand | Reserved | Spot | Savings Plans |
|---------|-----------|----------|------|----------------|
| **Engagement** | Aucun | 1 ou 3 ans | Aucun | 1 ou 3 ans |
| **Réduction potentielle** | ❌ | ✅ (75%) | ✅✅✅ (90%) | ✅✅ (72%) |
| **Flexibilité** | ✅✅✅ | ❌ | ✅✅ | ✅✅ |
| **Risque d'interruption** | ❌ | ❌ | ✅✅✅ | ❌ |
| **Idéal pour** | Dev/Test | Prod stable | Batch/CI | Prod optimisée |

**Recommandation** : Pour la plupart des entreprises, **Savings Plans** offre le meilleur compromis entre réduction (72%) et flexibilité.

### 8.6 Cas métier : Choisir la meilleure option tarifaire

#### Cas 1 : Site e-commerce avec trafic prévisible

- **Charge** : trafic stable, pics prévisibles en fin d'année.
- **Infrastructure** : 10 instances t3.large en continu, +20 during soldes.
- **Recommandation** : **Savings Plans (Compute)** pour les 10 instances permanentes + **Spot** pour les 20 supplémentaires pendant les soldes.
- **Économie** : ~72% sur la base, ~90% sur les renforts = **78% global**.

```
Coût mensuel sans optimisation (tout On-Demand) :
  10 × 730h × 0,096$ (t3.large) = 699 $

Coût optimisé (Savings Plans + Spot) :
  10 × (0,096$ × 0,28) + 20 × (0,096$ × 0,10) = 26,88 $ + 19,2 $ = 46,08 $

Économie : 652,92 $ / mois = 7,835 $ / an
```

#### Cas 2 : Environnement de développement/test

- **Charge** : variable, utilisation heures de travail uniquement.
- **Infrastructure** : 2-4 instances selon le sprint en cours.
- **Recommandation** : **On-Demand** uniquement (pas d'engagement, flexibilité totale).
- **Économie** : aucune, mais coûts minimaux et liberté maximale.

#### Cas 3 : Job batch nocturne de traitement

- **Charge** : lance chaque nuit des instances pour 4h, puis arrêt.
- **Infrastructure** : 50 instances c5.2xlarge pour le parallélisme.
- **Recommandation** : **Spot instances** avec **Spot Fleet** (demande auto-scaling de remplacement).
- **Économie** : ~90% vs On-Demand = **81$/j au lieu de 810$**, soit 8,100$/mois.

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

> [!tip]
> **Résultat attendu :**
> ```
> {
>     "SpotFleetRequestId": "sfr-0a1b2c3d4e5f6789a",
>     "SpotFleetRequestState": "submitted"
> }
> ```

#### Cas 4 : Application critiques 24/7 avec charge non prévisible

- **Charge** : pas de pattern clair, augmentations soudaines.
- **Infrastructure** : 4-30 instances selon la demande.
- **Recommandation** : **Savings Plans (mélange)** pour la charge de base + **On-Demand** pour les pics.
- **Avantage** : si charge explose au-delà des prévisions, On-Demand absorbe sans coupure.

### 8.7 Outil : AWS Pricing Calculator

```
URL : https://calculator.aws/
1. Sélectionner la région
2. Ajouter EC2 : type, nombre, durée
3. Sélectionner option (On-Demand, Reserved, Spot)
4. Voir l'estimation mensuelle/annuelle
5. Exporter en PDF pour justifier budgets
```

📎 [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

<nav class="page-sequence"><a href="cours/chapitre-4/7-aws-compute-optimizer-dimensionnement-optimal">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/9-lancer-une-instance-ec2-en-cli">Suivant</a></nav>
