---
title: "8. AWS Management Console"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 8. AWS Management Console"
---

<nav class="page-sequence"><a href="cours/chapitre-1/7-aws-well-architected-framework">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/9-les-services-aws-les-plus-utilises">Suivant</a></nav>

La **console AWS** est une interface Web centralisée qui permet de gérer tous les services AWS depuis un seul endroit.

### 8.1 Principales fonctionnalités

La console web regroupe en un seul endroit tout ce dont un administrateur a besoin pour piloter son compte AWS au quotidien, et c'est justement ce qui la rend accessible aux débutants malgré la richesse du catalogue de services.

Elle donne d'abord accès à l'ensemble des services AWS disponibles dans la région sélectionnée : EC2, S3, RDS, Lambda et les centaines d'autres services se retrouvent derrière une barre de recherche unique, sans avoir à mémoriser des noms de commandes ou des endpoints d'API. Elle permet ensuite de visualiser en temps réel l'état des ressources déjà créées — combien d'instances EC2 tournent, quels buckets S3 existent, comment est configuré le VPC — ce qui en fait le point d'entrée naturel pour diagnostiquer un problème avant de creuser en ligne de commande. Elle sert aussi de porte d'entrée vers **AWS IAM (Identity and Access Management)**, où se gèrent les comptes, les utilisateurs et leurs permissions : c'est depuis la console que l'on crée les premiers utilisateurs IAM et qu'on leur attribue des droits, avant même de savoir écrire une policy JSON. Enfin, elle donne un accès direct aux métriques d'usage et à la facturation, ce qui permet de surveiller les coûts sans quitter l'interface — un réflexe indispensable pour éviter les mauvaises surprises en fin de mois.

### 8.2 Structure de la console

| Zone | Rôle |
|------|------|
| **Barre de recherche** | Trouver rapidement un service (ex : EC2, S3, IAM) |
| **Panneau de navigation** | Accéder aux sections : Services, Ressources, Facturation |
| **Région** | Indique la localisation actuelle (ex : Paris `eu-west-3`) |
| **Compte utilisateur** | Informations, factures, connexions IAM |
| **Notifications** | Alertes et mises à jour |
| **Support** | Documentation, centre d'aide, tickets AWS |

📎 [AWS Console](https://aws.amazon.com/console/)
📎 [Guide de l'interface AWS Console](https://docs.aws.amazon.com/awsconsole)

> [!tip]
> **Vérification AWS CLI — lister les régions disponibles :**
> ```bash
> aws ec2 describe-regions --output table
> ```
> ```
> -------------------------------------------------------
> |                    DescribeRegions                  |
> +-------------------+---------------------------------+
> |   RegionName      |   Endpoint                      |
> +-------------------+---------------------------------+
> |  eu-west-3        |  ec2.eu-west-3.amazonaws.com   |  ← Paris
> |  eu-west-1        |  ec2.eu-west-1.amazonaws.com   |  ← Irlande
> |  eu-central-1     |  ec2.eu-central-1.amazonaws.com|  ← Francfort
> |  us-east-1        |  ec2.us-east-1.amazonaws.com   |  ← Virginie du Nord
> |  ap-southeast-1   |  ec2.ap-southeast-1.amazonaws.com| ← Singapour
> +-------------------+---------------------------------+
> ```
> La région `eu-west-3` (Paris) est votre région de travail par défaut pour cette formation. Vérifiez toujours que vous êtes bien dans la bonne région avant de créer une ressource — une instance EC2 créée dans `us-east-1` par erreur sera difficile à retrouver.

Rappelons que la console est un moyen parmi d'autres d'interagir avec AWS — il existe également des API, des SDK et la **CLI AWS** pour les automatisations.

### 8.3 Bonnes pratiques de base

- Toujours vérifier la **région active** avant de créer des ressources.
- Fermer les ressources non utilisées pour éviter des coûts inutiles.
- Utiliser **IAM** pour créer des comptes utilisateurs distincts plutôt que d'utiliser le compte root — rappel : voir l'encart sur le compte root en section 2.6 de ce chapitre.
- Activer la **MFA (Multi-Factor Authentication)** pour sécuriser les accès.
- Surveiller les coûts dans **Billing & Cost Management**.

📎 [Best Practices AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

### 8.4 Gestion du budget et des coûts AWS

L'un des **risques majeurs** du cloud est la **perte de contrôle des coûts**. Un bucket S3 public par erreur, une instance EC2 oubliée, un transfert de données non optimisé — et la facture peut exploser en quelques jours.

AWS fournit des outils puissants pour **monitorer, budgéter et optimiser** les coûts.

### Principes clés de la tarification AWS

1. **Pay-as-you-go** : payer ce qu'on utilise réellement.
2. **Granularité** : chaque service a ses propres métriques — EC2 facturé par heure (ou seconde), S3 par Go stocké + requêtes, RDS par heure + stockage + transfert.
3. **Réductions possibles** : Reserved Instances (engagement 1 ou 3 ans), Savings Plans (engagement flexible), Spot Instances (capacité excédentaire à -70%).
4. **AWS Free Tier** : pour les comptes créés depuis le 15 juillet 2025, AWS propose un système de crédits — 100 USD à l'inscription et jusqu'à 100 USD supplémentaires via des activités — ainsi qu'un plan gratuit limité à six mois. Les comptes plus anciens restent soumis au régime historique, qui comportait notamment certaines offres gratuites pendant douze mois. Dans tous les cas, un service hors périmètre, un dépassement de crédit ou un dépassement de quota est facturé au tarif normal.

> [!tip]
> **Exemple de commande AWS CLI pour vérifier votre consommation du Free Tier :**
> ```bash
> aws ce get-cost-and-usage \
>   --time-period Start=2024-01-01,End=2024-01-31 \
>   --granularity MONTHLY \
>   --metrics BlendedCost \
>   --group-by Type=DIMENSION,Key=SERVICE
> ```
> ```json
> {
>     "ResultsByTime": [{
>         "TimePeriod": { "Start": "2024-01-01", "End": "2024-01-31" },
>         "Groups": [
>             { "Keys": ["Amazon EC2"], "Metrics": { "BlendedCost": { "Amount": "0.00", "Unit": "USD" } } },
>             { "Keys": ["Amazon S3"], "Metrics": { "BlendedCost": { "Amount": "0.23", "Unit": "USD" } } },
>             { "Keys": ["Amazon RDS"], "Metrics": { "BlendedCost": { "Amount": "0.00", "Unit": "USD" } } }
>         ]
>     }]
> }
> ```
> Dans cet exemple fictif, les crédits promotionnels absorbent encore la consommation EC2 et RDS. S3 affiche un coût car l'usage concerné n'est plus entièrement couvert. Le résultat réel dépend de la date de création du compte, du plan choisi, du solde de crédits, de la région et des services utilisés : il faut toujours vérifier la page **Free Tier** et la facturation du compte actif.

### Outils de monitoring des coûts

#### AWS Billing Dashboard

Accessible depuis la console AWS, le **Billing Dashboard** affiche :
- Facture du mois en cours
- Prévisions de coûts (estimé) jusqu'à fin du mois
- Services coûtant le plus

C'est un premier coup d'œil rapide, mais peu détaillé.

#### AWS Cost Explorer

**Cost Explorer** permet une **analyse fine des dépenses** :
- Filtrer par service (EC2, S3, RDS…)
- Filtrer par période (jour, mois, année)
- Filtrer par région
- Voir l'historique et les tendances

**Cas d'usage :** « Pourquoi ma facture a-t-elle explosé en janvier ? Quel service en est responsable ? »

#### AWS Budgets

**AWS Budgets** permet de **définir des seuils d'alerte** :
- Budget mensuel maximum (ex : 500 €/mois)
- Alertes si dépassement imminent
- Alertes si usage atypique détecté

**Cas d'usage :** Recevoir une notification avant d'exploser le budget plutôt qu'une mauvaise surprise à la fin du mois.

#### AWS Cost Anomaly Detection

Utilise l'**apprentissage automatique** pour détecter les **anomalies de dépenses** :
- Si vous dépensez soudain 10x plus que normal dans une région, alerter.
- Si un service coûte étrangement cher, signaler.

C'est très utile pour détecter des ressources oubliées ou des fuites de données.

#### AWS Pricing Calculator

**AWS Pricing Calculator** permet d'**estimer les coûts** avant de déployer :
- Configurez votre architecture (1 instance EC2 t3.micro, 100 Go S3…)
- Voir le coût mensuel/annuel
- Comparer différentes configurations

C'est idéal pour les **POC (Proof of Concept)** et les **RFQ** (appels d'offre).

### Bonnes pratiques pour maîtriser les coûts

| Pratique | Description | Impact |
|----------|-------------|--------|
| **Utiliser des Reserved Instances** | Engagement 1 ou 3 ans pour réduction tarifaire (~30-60%) | Très utile pour charges stables 24/7 |
| **Utiliser Spot Instances** | Capacité excédentaire à -70% (mais peut être interrompue) | Idéal pour tâches batch, tests |
| **Éteindre les ressources non utilisées** | Arrêter EC2 dev/test en fin de journée, supprimer RDS inutilisés | Économies rapides et simples |
| **Auto-scaling** | Adapter le nombre d'instances au trafic | Évite surprovisionnement |
| **Optimiser le stockage** | S3 Standard pour accès fréquent, S3 Glacier pour archive | Différences tarifaires importantes |
| **Utiliser des réductions volantes** | Savings Plans, commitments, consolidated billing | ~30-40% de réduction possible |
| **Monitorer régulièrement** | Cost Explorer, Budgets, anomaly detection | Détection précoce des dérives |

### Exemple concret : budget mal maîtrisé

Imaginons une startup qui lance une application web sur AWS.

**Scénario optimiste :**
- Petite instance EC2 dont la consommation est couverte par le solde de crédits du compte de démonstration
- 50 Go S3 (~1 € / mois)
- **Coût mensuel : ~1 €**

**Scénario problématique (sans monitoring) :**
- Instance EC2 mal fermée le week-end : +50 €
- Bucket S3 avec version history excessive : +100 €
- Transfert de données cross-région oublié : +200 €
- **Coût mensuel : ~350 € !**

**Avec les outils AWS Budgets :**
- Budget défini : 50 €/mois
- Alerte à 40 € : Action rapide
- Découverte de l'instance oubliée → suppression
- Découverte des versions excessives → nettoyage
- **Coût final : 5 €/mois**

> [!tip]
> **Résultat attendu — alerte AWS Budgets reçue par email :**
> ```
> De : no-reply@notifications.aws
> Objet : [AWS Budgets] Alerte — Mon Budget Mensuel (80% atteint)
>
> Bonjour,
>
> Votre budget "Mon Budget Mensuel" a atteint 80% de votre seuil d'alerte.
>
> Budget : 50,00 USD
> Dépensé : 40,23 USD
> Prévu ce mois : 52,31 USD
>
> Service le plus coûteux : Amazon EC2 (28,50 USD)
> Recommandation : vérifiez les instances EC2 actives dans toutes les régions.
>
> → Accéder au Cost Explorer : https://console.aws.amazon.com/cost-management/
> ```
> Grâce à cette alerte précoce, vous pouvez arrêter l'instance oubliée avant que la facture n'explose. Sans ce budget configuré, vous n'auriez découvert le problème qu'à la réception de la facture mensuelle.

C'est pourquoi **mettre en place un monitoring budgétaire dès le départ est essentiel**.

📎 [AWS Billing Documentation](https://docs.aws.amazon.com/billing/)
📎 [AWS Cost Explorer Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
📎 [AWS Cost Optimization Best Practices](https://aws.amazon.com/aws-cost-management/cost-optimization/)

---

<nav class="page-sequence"><a href="cours/chapitre-1/7-aws-well-architected-framework">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/9-les-services-aws-les-plus-utilises">Suivant</a></nav>
