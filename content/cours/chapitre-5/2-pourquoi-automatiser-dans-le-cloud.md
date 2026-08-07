---
title: "2. Pourquoi automatiser dans le Cloud ?"
description: "\"Chapitre 5 — Automatisation, supervision et reprise d'activité\" - 2. Pourquoi automatiser dans le Cloud ?"
---

<nav class="page-sequence"><a href="cours/chapitre-5/1-rto-rpo-et-recuperation-de-sauvegarde">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/3-aws-cloudformation-infrastructure-as-code">Suivant</a></nav>

### 2.1 Le déploiement manuel : une source d'erreurs

Dans un environnement traditionnel, déployer une infrastructure peut prendre **des jours, voire des semaines**. Les équipes IT doivent :

- Commander du matériel physique
- Installer les systèmes d'exploitation
- Configurer le réseau et les accès
- Mettre à jour les pare-feu et les politiques de sécurité
- Documenter tous les changements (quand c'est fait...)

**Le problème** : chaque déploiement manuel génère des **incohérences**. Deux administrateurs ne font jamais exactement la même chose. Certains oublis passent inaperçus :

```
Infrastructure créée manuellement :
  Admin Alice crée un VPC le 10 janvier
  → Configure 2 subnets, 1 IGW, 1 NAT Gateway
  → Oublie de documenter les CIDR utilisés
  → Quitte l'entreprise 6 mois après

  Admin Bob doit dépliquer l'infrastructure
  → Cherche la documentation (introuvable)
  → Recrée un nouveau VPC similaire MAIS différent
  → Incompatibilité lors du peering : PERTE DE TEMPS

Résultat : deux "mêmes" infrastructures qui ne sont PAS identiques
```

---

### 2.2 L'automatisation dans le Cloud : standardisation et rapidité

Dans le cloud AWS, l'**automatisation** permet de :

- **Standardiser** les environnements → Prod et Dev sont identiques
- **Réduire** les délais de déploiement → Quelques minutes au lieu de semaines
- **Limiter** les erreurs humaines → Le code est testé avant déploiement
- **Faciliter** la montée en charge → Spawner 100 instances avec un clic

```
Infrastructure automatisée avec CloudFormation :
  Étape 1 : Écrire un template YAML (1-2 heures)
  Étape 2 : Déployer en Prod exactement identique au Dev (2 minutes)
  Étape 3 : Déployer en Test (2 minutes)
  Étape 4 : Dupliquer pour un client différent (2 minutes)

  Avantage : zéro divergence entre les environnements
            zéro oubli de configuration
            traçabilité complète (versionning Git du template)
```

---

### 2.3 Les outils AWS pour l'automatisation

| Outil | Fonction | Cas d'usage |
|-------|----------|-----------|
| **CloudFormation** | Déploiement d'infrastructures as code (IaC) | Créer VPC, EC2, RDS, S3 en une seule opération |
| **Quick Starts** | Templates CloudFormation préconfigurés par AWS | Déployer rapidement une architecture éprouvée (VPC + ELB + RDS) |
| **Systems Manager** | Gestion centralisée et automatisation opérationnelle | Exécuter des scripts, patcher les serveurs, inventorier les ressources |
| **Elastic Beanstalk** | Déploiement PaaS simplifié d'applications web | Déployer une app Node.js sans gérer l'infrastructure |
| **CLI / SDK** | Automatisation par scripts et développement | Orchestrer plusieurs services AWS via Python/Bash |

> **Référence** : [AWS Automation Overview](https://aws.amazon.com/automation/)
> **Référence** : [AWS CloudFormation Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)

---
### 2.4 AWS Quick Starts — Templates Éprouvés

**AWS Quick Starts** sont des **templates CloudFormation préconfigurés et validés par AWS** pour déployer des architectures complètes en quelques clics.

#### Qu'est-ce qu'un Quick Start ?

```
Quick Start = CloudFormation template complet + documentation + bonnes pratiques

Exemplar :
  - Déployer une architecture WordPress hautement disponible
    (VPC + ALB + Auto Scaling + RDS Multi-AZ + CloudFront)
  - Sans avoir à écrire 500 lignes YAML
  - Basé sur des best practices AWS
  - Testé et validé en production
```

#### Quick Starts courants (exemples)

| Quick Start | Qu'il déploie | Temps |
|-------------|---------------|-------|
| **WordPress on AWS** | VPC, ALB, Auto Scaling, RDS, CloudFront | 10-15 min |
| **Kubernetes on AWS** | EKS cluster complet avec worker nodes | 20-30 min |
| **SQL Server on EC2** | Instance EC2 + RDS SQL Server + backup | 15 min |
| **Jenkins on AWS** | Jenkins Master + Agent instances + monitoring | 10 min |
| **Hadoop on AWS** | Cluster EMR complet multi-node | 20 min |

**EKS** (Elastic Kubernetes Service, déjà présenté au Chapitre 1 section 5) est le service AWS de Kubernetes managé. **Hadoop** est un framework open source historique de traitement de données massives (Big Data) : il répartit le calcul et le stockage sur un grand nombre de machines pour traiter des volumes trop importants pour un seul serveur. **EMR** (Elastic MapReduce) est le service managé AWS qui déploie et exploite des clusters Hadoop (ainsi que des outils compatibles comme Spark) sans que vous ayez à installer et administrer vous-même les serveurs du cluster.

#### Accéder aux Quick Starts

Les Quick Starts sont accessibles depuis la console CloudFormation ou directement via CLI. Voici comment les trouver et les utiliser :

```bash
# 1. Via la console AWS → CloudFormation → Quick Starts
#    https://aws.amazon.com/quickstarts/

# 2. Chercher un Quick Start par domaine :
#    - VPC / Networking
#    - Databases & Analytics
#    - Business Applications
#    - DevOps Tools

# 3. Cliquer sur "Launch" → remplir les paramètres → Create Stack
```

#### Avantages des Quick Starts

```
✓ Gain de temps : architecture complète en 15 min au lieu de 3-4 heures
✓ Bonnes pratiques : design conforme au Well-Architected Framework
✓ Haute disponibilité : Multi-AZ, load balancing, failover automatique
✓ Maintenance : AWS met à jour les templates régulièrement
✓ Support : documentation et troubleshooting fournis
✓ Flexibilité : vous pouvez modifier les templates après déploiement
```

#### Exemple : Déployer WordPress via Quick Start

Plutôt que de créer manuellement VPC, ALB, RDS et Auto Scaling, on lance une seule commande CloudFormation qui déploie toute l'architecture WordPress en ~10 minutes :

```bash
# 2. Chercher "WordPress"
# 3. Cliquer sur "Launch Quick Start"
# 4. Remplir les paramètres (taille VPC, taille RDS, DNS, etc.)
# 5. Vérifier les options VPC et subnet
# 6. Cliquer "Create Stack"
# 7. Attendre ~10 minutes
# 8. CloudFormation affiche l'URL du site WordPress

# Vous pouvez aussi utiliser AWS CLI :
aws cloudformation create-stack \
  --stack-name wordpress-production \
  --template-url "https://s3.amazonaws.com/quickstart-reference/wordpress/latest/templates/wordpress.yaml" \
  --parameters \
    ParameterKey=KeyName,ParameterValue=ma-clé-ssh \
    ParameterKey=InstanceType,ParameterValue=t3.small \
    ParameterKey=DBInstanceClass,ParameterValue=db.t3.micro \
  --region eu-west-3
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>     "StackId": "arn:aws:cloudformation:eu-west-3:123456789012:stack/wordpress-production/a1b2c3d4-e5f6-7890-abcd-ef1234567890"
> }
> ```
> La stack `wordpress-production` est en cours de création. Suivez la progression dans la console CloudFormation → Events, ou via `aws cloudformation describe-stack-events --stack-name wordpress-production`. Après ~10 minutes, le statut passe à `CREATE_COMPLETE` et l'URL WordPress est disponible dans les outputs de la stack.

---

<nav class="page-sequence"><a href="cours/chapitre-5/1-rto-rpo-et-recuperation-de-sauvegarde">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/3-aws-cloudformation-infrastructure-as-code">Suivant</a></nav>
