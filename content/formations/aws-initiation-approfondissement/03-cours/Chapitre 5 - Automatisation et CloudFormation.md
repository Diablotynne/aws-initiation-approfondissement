# Chapitre 5 — Automatisation, CloudFormation et Well-Architected Framework

---

> [!NOTE]
> Le réseau et les bases de données mis en place au chapitre précédent constituent une architecture fonctionnelle mais encore déployée manuellement ; ce dernier chapitre referme la formation en automatisant ce déploiement et en donnant les clés pour évaluer et faire évoluer une architecture AWS dans la durée.
>
> **Objectifs du chapitre**
>
> À l'issue de ce chapitre, les stagiaires seront capables de :
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
![Boucle DevOps AWS reliant Infrastructure as Code, déploiement, observation et amélioration](formations/aws-initiation-approfondissement/11-images/ch5-carte-automatisation.svg)

<div class="concept-check">
<strong>Réflexe d'automatisation — avant de poursuivre</strong>
<p>Un template CloudFormation déjà déployé est relancé sans aucune modification. Quel comportement recherche-t-on ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Un résultat idempotent : l'état réel correspond déjà à l'état déclaré, donc aucune ressource supplémentaire ne doit être créée. Les changements futurs doivent être évalués avant application.</p></details>
</div>

---

## 1. RTO/RPO et Récupération de Sauvegarde

### 1.1 Définitions essentielles

Avant d'automatiser une infrastructure, il faut comprendre deux concepts critiques pour la **continuité de service** :

#### RTO (Recovery Time Objective) — Temps d'Indisponibilité Acceptable

**RTO** = **combien de temps maximum l'application peut-elle rester indisponible avant que l'impact métier devienne intolérable ?**

```
Exemples concrets :

Service                    | RTO typical | Raison
--------------------------|-------------|----------------------------------------
Site e-commerce (Amazon)   | 5 minutes   | Chaque minute sans vente = perte
Application interne (RH)   | 8 heures    | Métier critique mais moins urgent
Service vidéo (Netflix)    | 30 minutes  | Perte d'utilisateurs, mais pas urgent
Système bancaire           | 15 minutes  | Réglementation stricte
API partenaire (non-vital) | 4 heures    | Impact mineur sur le business
```

#### RPO (Recovery Point Objective) — Quantité de Données Perdable

**RPO** = **combien de données suis-je prêt à perdre en cas de sinistre ?**

```
Exemples concrets :

Application              | RPO        | Raison
------------------------|------------|----------------------------------------
E-commerce actif         | 5 minutes  | Transactions en temps réel = critique
Logs d'application       | 1 jour     | Données historiques, non urgentes
Données de client (CRM)  | 1 heure    | Important pour relancer les clients
Backups archivés         | 30 jours   | Archive long terme, peu critique
```

#### Relation RTO ↔ RPO

```
Scénario : Serveur RDS tombe en panne à 10:00

Stratégie 1 (RTO court, RPO court)
  - Sauvegarde automatique toutes les 10 minutes
  - Multi-AZ activé (failover < 2 minutes)
  - RTO = 2 minutes, RPO = 10 minutes
  - Coût : ⭐⭐⭐⭐ (cher)

Stratégie 2 (RTO moyen, RPO moyen)
  - Sauvegarde quotidienne (minuit)
  - Backup lisible rapidement (1 heure pour restaurer)
  - RTO = 1 heure, RPO = 24 heures
  - Coût : ⭐⭐ (raisonnable)

Stratégie 3 (RTO long, RPO long)
  - Sauvegarde hebdomadaire
  - Pas de failover automatique
  - RTO = 8 heures, RPO = 7 jours
  - Coût : ⭐ (très bon marché)
```

---

### 1.2 Stratégies AWS pour Atteindre RTO/RPO

| Technologie | RTO | RPO | Coût | Cas d'usage |
|-------------|-----|-----|------|-----------|
| **Multi-AZ** | < 2 min | ≈ 0 min | Moyen | Haute dispo critique |
| **Snapshots EBS** | 15-30 min | 1 jour | Faible | Backup régulier |
| **AWS Backup** | 1-4 heures | 1-24 heures | Faible-Moyen | Backup centralisé |
| **Read Replicas (RDS)** | 5-10 min | ≈ 0 min | Moyen | Failover rapide BD |
| **AWS Glacier** | 1-12 heures | Sans limite | Très faible | Archive long terme |
| **Lambda + S3** | 10-60 min | 1 heure | Très faible | Backup custom |

---

### 1.3 AWS Backup — Service Centralisé de Sauvegarde

**AWS Backup** est un **service managé** pour centraliser et automatiser les sauvegardes de ressources AWS.

#### Ressources sauvegardables par AWS Backup

```
Compute & Storage:
  ✓ Amazon EC2 instances
  ✓ Amazon EBS volumes
  ✓ Amazon EFS (Elastic File System)

Database:
  ✓ Amazon RDS databases (MySQL, PostgreSQL, Oracle, SQL Server)
  ✓ Amazon DynamoDB tables
  ✓ Amazon Aurora databases

Backup Store:
  ✓ AWS Storage Gateway
  ✓ VMware vSphere
```

#### Déployer AWS Backup via CLI

On crée d'abord un **Backup Vault** (le coffre où seront stockées les sauvegardes), puis un **Backup Plan** (la planification) avec ses règles de rétention, et enfin une **Backup Selection** (les ressources à sauvegarder). Ces trois objets forment un système complet.

```bash
# 1. Créer un Backup Vault (conteneur pour les sauvegardes)
aws backup create-backup-vault \
  --backup-vault-name mon-vault-production \
  --region eu-west-3

# Output : BackupVaultArn

# 2. Créer un Backup Plan (planification automatique)
# Sauvegarder toutes les instances EC2 chaque jour à minuit
cat > backup-plan.json << 'EOF'
{
  "BackupPlanName": "SauvegardeQuotidienne",
  "Rules": [
    {
      "RuleName": "Sauvegarde Quotidienne",
      "TargetBackupVaultName": "mon-vault-production",
      "ScheduleExpression": "cron(0 0 ? * * *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 120,
      "Lifecycle": {
        "DeleteAfterDays": 30,
        "MoveToColdStorageAfterDays": 7
      }
    }
  ]
}
EOF

# 3. Créer le plan
aws backup create-backup-plan \
  --backup-plan file://backup-plan.json \
  --region eu-west-3
```

> [!TIP]
> **Résultat attendu :**
> ```
> {
>     "BackupPlanId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
>     "BackupPlanArn": "arn:aws:backup:eu-west-3:123456789012:backup-plan:a1b2c3d4-e5f6-7890-abcd-ef1234567890",
>     "CreationDate": "2026-03-24T10:15:00.000Z",
>     "VersionId": "NDEzMWVlNzgtMTk2My00NjMxLWJlOTYt"
> }
> ```
```bash
# 4. Assigner des ressources au plan
aws backup create-backup-selection \
  --backup-plan-id mon-plan-123 \
  --backup-selection file://selection.json \
  --region eu-west-3

# 5. Vérifier les backups créés
aws backup list-recovery-points-by-resource \
  --resource-arn "arn:aws:ec2:eu-west-3:123456789:instance/i-12345678" \
  --region eu-west-3

# 6. Restaurer une instance EC2 à partir d'un backup
aws backup start-restore-job \
  --recovery-point-arn "arn:aws:backup:eu-west-3:123456789:recovery-point:..." \
  --iam-role-arn "arn:aws:iam::123456789:role/AWSBackupDefaultRole" \
  --metadata key1=value1,key2=value2 \
  --region eu-west-3
```

> [!TIP]
> **Résultat attendu :**
> ```
> # list-recovery-points-by-resource :
> {
>     "RecoveryPoints": [
>         {
>             "RecoveryPointArn": "arn:aws:backup:eu-west-3:123456789012:recovery-point:abc123",
>             "CreationDate": "2026-03-24T00:05:12.000Z",
>             "Status": "COMPLETED",
>             "BackupSizeInBytes": 8589934592,
>             "BackupVaultName": "mon-vault-production"
>         }
>     ]
> }
>
> # start-restore-job :
> {
>     "RestoreJobId": "restore-job-0abc123def456"
> }
> ```
> [!WARNING]
> **Coûts AWS Backup :** Le stockage des sauvegardes est facturé selon le volume stocké (~0,05 $/Go/mois en stockage chaud). Le déplacement vers Glacier (cold storage) réduit le coût à ~0,01 $/Go/mois après 7 jours. Vérifiez régulièrement les coûts dans **Cost Explorer** et ajustez les politiques de rétention.
#### Bonnes pratiques AWS Backup

```
✓ Planifier les sauvegardes en dehors des heures de pic
✓ Utiliser des Backup Vaults séparés pour Prod/Staging/Dev
✓ Tester régulièrement les restaurations (RTO réel)
✓ Définir une rétention appropriée (30j prod, 7j dev)
✓ Combiner avec CloudWatch Events pour alertes
```

---

### 1.4 Snapshots EBS et RDS — Sauvegardes Point-in-Time

**Snapshot EBS** = photo point-in-time d'un volume EC2

Un snapshot EBS capture l'état d'un disque à un instant donné. Il est stocké dans S3 (géré par AWS) et peut servir à restaurer un volume ou à déplacer une instance vers une autre région.

```bash
# Créer un snapshot EBS manuel
aws ec2 create-snapshot \
  --volume-id vol-12345678 \
  --description "Sauvegarde avant migration" \
  --region eu-west-3

# Lister les snapshots
aws ec2 describe-snapshots \
  --owner-ids self \
  --region eu-west-3

# Créer un volume à partir du snapshot
aws ec2 create-volume \
  --snapshot-id snap-12345678 \
  --availability-zone eu-west-3a \
  --region eu-west-3
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-snapshot :
> {
>     "SnapshotId": "snap-0abc123def456789a",
>     "VolumeId": "vol-12345678",
>     "State": "pending",
>     "StartTime": "2026-03-24T09:00:00.000Z",
>     "Progress": "0%",
>     "Description": "Sauvegarde avant migration"
> }
>
> # describe-snapshots (après quelques minutes) :
> {
>     "Snapshots": [
>         {
>             "SnapshotId": "snap-0abc123def456789a",
>             "State": "completed",
>             "Progress": "100%",
>             "VolumeSize": 20
>         }
>     ]
> }
> ```
**Snapshot RDS** = sauvegarde complète de base de données

```bash
# Créer un snapshot RDS manuel
aws rds create-db-snapshot \
  --db-instance-identifier production-db \
  --db-snapshot-identifier prod-db-backup-2026-03-24 \
  --region eu-west-3

# Lister les snapshots
aws rds describe-db-snapshots \
  --region eu-west-3

# Restaurer une BD à partir d'un snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier production-db-restored \
  --db-snapshot-identifier prod-db-backup-2026-03-24 \
  --region eu-west-3
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-db-snapshot :
> {
>     "DBSnapshot": {
>         "DBSnapshotIdentifier": "prod-db-backup-2026-03-24",
>         "DBInstanceIdentifier": "production-db",
>         "Status": "creating",
>         "Engine": "mysql",
>         "AllocatedStorage": 100,
>         "SnapshotCreateTime": "2026-03-24T09:30:00.000Z"
>     }
> }
>
> # describe-db-snapshots (après quelques minutes) :
> {
>     "DBSnapshots": [
>         {
>             "DBSnapshotIdentifier": "prod-db-backup-2026-03-24",
>             "Status": "available",
>             "PercentProgress": 100
>         }
>     ]
> }
> ```
> **Résultat attendu :** `create-db-snapshot` retourne un JSON avec le `DBSnapshotIdentifier` et le statut `creating`. Quelques minutes plus tard, `describe-db-snapshots` affiche le statut `available`. La restauration (`restore-db-instance-from-db-snapshot`) crée une **nouvelle instance** RDS — pas un remplacement de l'existante.

---

## 2. Pourquoi automatiser dans le Cloud ?

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
  Jour 1 : Écrire un template YAML (1-2 heures)
  Jour 2 : Déployer en Prod exactement identique au Dev (2 minutes)
  Jour 3 : Déployer en Test (2 minutes)
  Jour 4 : Dépliquer pour un client différent (2 minutes)

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

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "StackId": "arn:aws:cloudformation:eu-west-3:123456789012:stack/wordpress-production/a1b2c3d4-e5f6-7890-abcd-ef1234567890"
> }
> ```
> La stack `wordpress-production` est en cours de création. Suivez la progression dans la console CloudFormation → Events, ou via `aws cloudformation describe-stack-events --stack-name wordpress-production`. Après ~10 minutes, le statut passe à `CREATE_COMPLETE` et l'URL WordPress est disponible dans les outputs de la stack.
---


## 3. AWS CloudFormation — Infrastructure as Code

### 3.1 Qu'est-ce que CloudFormation ?

**AWS CloudFormation** est un service qui permet de **modéliser et déployer des ressources AWS** sous forme de code.

Plutôt que de créer manuellement une VPC, des sous-réseaux, des groupes de sécurité ou des instances EC2 dans la console, on les **décrit dans un template JSON ou YAML** et CloudFormation s'occupe du reste.

```
Analogie : CloudFormation est comme une RECETTE DE CUISINE pour construire une infrastructure

Recette classique :
  Ingrédients : 1 VPC, 2 subnets, 1 IGW, 3 EC2
  Étapes :
    1. Créer la VPC avec CIDR 10.0.0.0/16
    2. Créer subnet public 10.0.1.0/24
    3. Créer subnet privé 10.0.2.0/24
    4. Ajouter une IGW et l'attacher à la VPC
    5. Créer 3 instances EC2 dans le subnet public
    6. Configurer les groupes de sécurité

Avantage : la recette peut être réutilisée 100 fois identiquement
          et versionnée dans Git
```

---

### 3.2 Fonctionnement simplifié de CloudFormation

1. **Rédiger un template** (JSON/YAML) décrivant les ressources à créer
2. **Créer une stack** dans CloudFormation (via console ou CLI)
3. **AWS provisionne** toutes les ressources **dans le bon ordre** (résout les dépendances automatiquement)
4. **Mettre à jour** la stack pour faire évoluer l'infrastructure (ajout, suppression, modification de ressources)
5. **Supprimer** la stack si plus besoin (CloudFormation supprime toutes les ressources associées)

<img src="formations/aws-initiation-approfondissement/11-images/cloudformation-flow.svg"
     alt="Flux simplifié de CloudFormation"
     style="display:block; margin:auto; width:90%">

---

### 3.3 Exemple simple de template CloudFormation (YAML)

Voici un template minimaliste créant une VPC, un subnet, et une instance EC2 :

```yaml
# Version du format CloudFormation (toujours 2010-09-09)
AWSTemplateFormatVersion: '2010-09-09'

# Description brève du template
Description: |
  Infrastructure AWS simple pour débutants
  Crée une VPC, un subnet public, une IGW et une instance EC2

# Paramètres (permet de rendre le template réutilisable)
# Ici, on paramètre le type d'instance EC2 pour pouvoir changer facilement
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    Description: Type d'instance EC2 (t2.micro, t2.small, etc.)
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro

# Les ressources à créer
Resources:
  # Ressource 1 : Créer une VPC
  # Chaque ressource a un identifiant logique (MonVPC) et un type AWS
  MonVPC:
    # Type de ressource AWS
    Type: AWS::EC2::VPC
    # Propriétés spécifiques
    Properties:
      # CIDR block : plage d'adresses IP pour cette VPC
      CidrBlock: 10.0.0.0/16
      # Activer le hostname DNS
      EnableDnsHostnames: true
      # Tags pour identifier facilement
      Tags:
        - Key: Name
          Value: MonVPC-Formation

  # Ressource 2 : Créer un subnet public dans la VPC
  # !Ref est une fonction intrinsèque qui referéce une autre ressource
  MonSubnetPublic:
    Type: AWS::EC2::Subnet
    Properties:
      # !Ref MonVPC = ID logique de la VPC créée au-dessus
      VpcId: !Ref MonVPC
      # Plage d'adresses pour ce subnet (doit être dans le CIDR de la VPC)
      CidrBlock: 10.0.1.0/24
      # Zone de disponibilité (peut varier, AWS en assigne une par défaut)
      AvailabilityZone: eu-west-1a
      # Assigner automatiquement une IP publique aux instances
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: MonSubnet-Public

  # Ressource 3 : Créer une Internet Gateway (permet l'accès à Internet)
  MonIGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MonIGW

  # Ressource 4 : Attacher l'IGW à la VPC
  # CloudFormation comprend automatiquement que cette ressource dépend de MonVPC et MonIGW
  AttachIGW:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      # Référence à la VPC créée
      VpcId: !Ref MonVPC
      # Référence à l'IGW créé
      InternetGatewayId: !Ref MonIGW

  # Ressource 5 : Groupe de sécurité (pare-feu simplifié)
  MonSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      # Description obligatoire
      GroupDescription: Autorise SSH et HTTP
      # Associer au VPC
      VpcId: !Ref MonVPC
      # Règles de trafic entrant
      SecurityGroupIngress:
        # Permettre SSH depuis n'importe quelle IP (⚠️ À ÉVITER EN PROD)
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: SSH access
        # Permettre HTTP
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP access
      Tags:
        - Key: Name
          Value: MonSG-Formation

  # Ressource 6 : Instance EC2
  MonInstance:
    Type: AWS::EC2::Instance
    Properties:
      # AMI ID (Ubuntu 24.04 LTS en eu-west-1)
      ImageId: ami-0d71ea30463e0ff8d
      # Type d'instance (utilise le paramètre défini au-dessus)
      InstanceType: !Ref InstanceType
      # Placer dans le subnet créé
      SubnetId: !Ref MonSubnetPublic
      # Associer le security group
      SecurityGroupIds:
        - !Ref MonSecurityGroup
      # Script à exécuter au démarrage (user data)
      UserData:
        Fn::Base64: |
          #!/bin/bash
          # Mises à jour système
          apt-get update
          apt-get install -y nginx curl
          # Démarrer Nginx
          systemctl start nginx
          systemctl enable nginx
          # Créer une page de test
          echo "<h1>Bonjour du serveur créé par CloudFormation</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: MonServeur-Formation

# Outputs : affiche les résultats après déploiement
# Utile pour récupérer les IP, URLs, etc.
Outputs:
  # Affiche l'ID de la VPC créée
  VPCId:
    Description: ID de la VPC
    # !Ref récupère l'ID physique de la ressource
    Value: !Ref MonVPC
    # Export permet à d'autres stacks de référencer cette valeur
    Export:
      Name: MonVPC-Id

  # Affiche l'IP publique de l'instance
  PublicIP:
    Description: Adresse IP publique de l'instance EC2
    # !GetAtt récupère un attribut spécifique d'une ressource
    Value: !GetAtt MonInstance.PublicIp
    Export:
      Name: MonServeur-PublicIP

  # Affiche l'URL HTTP pour accéder au serveur
  WebServerURL:
    Description: URL pour accéder au serveur web
    # !Sub remplace les variables ${...} par leurs valeurs
    Value: !Sub 'http://${MonInstance.PublicIp}'
```

---
##### 3.3 (suite) — Exemple Production : VPC + Serveur Web Apache

Voici un template **production-ready** déployant une **VPC complète avec serveur web Apache** et un Security Group. C'est le template utilisé dans les ateliers Dawan.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: |
  Déploiement d'une VPC + serveur web Apache via CloudFormation
  - VPC avec CIDR 10.0.0.0/16
  - 1 subnet public (10.0.1.0/24)
  - Internet Gateway + route vers l'extérieur
  - Security Group (SSH + HTTP)
  - Instance EC2 t3.micro avec Apache2 automatiquement installé
  - Output : URL publique du serveur web

# Paramètres pour rendre le template réutilisable
Parameters:
  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nom de la paire de clés EC2 existante (pour SSH)
  
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t2.micro
    Description: Type d'instance EC2

# Les ressources à créer
Resources:
  # VPC
  FormationVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Formation-VPC-WebLab

  # Subnet public
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref FormationVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: eu-west-3a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: Formation-Subnet-Public

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Formation-IGW

  # Attacher IGW à la VPC
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref FormationVPC
      InternetGatewayId: !Ref InternetGateway

  # Table de routage publique
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref FormationVPC
      Tags:
        - Key: Name
          Value: Formation-PublicRouteTable

  # Route vers l'IGW (tout ce qui va en dehors du VPC)
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # Associer la route table au subnet
  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # Security Group (pare-feu)
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permettre HTTP et SSH
      VpcId: !Ref FormationVPC
      SecurityGroupIngress:
        # SSH (port 22) - accès depuis n'importe où (⚠️ en production : limiter à votre IP)
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: SSH access
        # HTTP (port 80) - serveur web public
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP web server
      Tags:
        - Key: Name
          Value: Formation-WebServerSG

  # Instance EC2
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-04a92520784b94538  # Amazon Linux 2023 (eu-west-3)
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      # Script pour configurer Apache2
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          # Mise à jour du système
          yum update -y
          
          # Installer Apache HTTP Server
          yum install -y httpd
          
          # Activer et démarrer Apache
          systemctl enable httpd
          systemctl start httpd
          
          # Créer une page HTML de test
          cat > /var/www/html/index.html << 'ENDHTML'
          <!DOCTYPE html>
          <html>
          <head>
              <title>CloudFormation - Formation AWS</title>
              <style>
                  body { font-family: Arial, sans-serif; margin: 40px; }
                  h1 { color: #FF9900; }
              </style>
          </head>
          <body>
              <h1>Bienvenue sur votre serveur web CloudFormation!</h1>
              <p>Cette instance EC2 a été déployée automatiquement via CloudFormation.</p>
              <p><strong>Informations serveur :</strong></p>
              <ul>
                  <li>Instance ID : ${AWS::StackId}</li>
                  <li>Région : ${AWS::Region}</li>
                  <li>Instance Type : ${InstanceType}</li>
              </ul>
          </body>
          </html>
          ENDHTML
      
      Tags:
        - Key: Name
          Value: Formation-WebServer

# Outputs - affiche les résultats utiles après déploiement
Outputs:
  WebServerURL:
    Description: URL publique du serveur web
    Value: !Sub 'http://${WebServerInstance.PublicDnsName}'
    Export:
      Name: !Sub '${AWS::StackName}-WebServerURL'
  
  WebServerPublicIP:
    Description: Adresse IP publique de l'instance
    Value: !Sub '${WebServerInstance.PublicIp}'
  
  SSHCommand:
    Description: Commande SSH pour se connecter au serveur
    Value: !Sub 'ssh -i /chemin/vers/cle.pem ec2-user@${WebServerInstance.PublicDnsName}'
  
  SecurityGroupId:
    Description: ID du Security Group
    Value: !Ref WebServerSecurityGroup
```

##### Déployer ce template

> [!NOTE]
> La pratique dans **AWS Academy** (module Automation, section CloudFormation) vous permettra d'approfondir le déploiement de ce template.
```bash
# 1. Créer la stack depuis le fichier YAML local
aws cloudformation create-stack \
  --stack-name formation-webserver-lab \
  --template-body file://vpc-webserver.yaml \
  --parameters \
    ParameterKey=KeyPairName,ParameterValue=ma-clé-ssh \
    ParameterKey=InstanceType,ParameterValue=t3.micro \
  --region eu-west-3

# Attendre que la stack soit créée (statut CREATE_COMPLETE)
aws cloudformation wait stack-create-complete \
  --stack-name formation-webserver-lab \
  --region eu-west-3

# 2. Récupérer l'URL publique du serveur
aws cloudformation describe-stacks \
  --stack-name formation-webserver-lab \
  --query 'Stacks[0].Outputs[?OutputKey==`WebServerURL`].OutputValue' \
  --output text \
  --region eu-west-3

# Output : http://ec2-12-34-56-78.eu-west-3.compute.amazonaws.com
# → Ouvrir cette URL dans un navigateur pour voir le serveur Apache

# 3. Supprimer toute l'infrastructure quand vous avez terminé
aws cloudformation delete-stack \
  --stack-name formation-webserver-lab \
  --region eu-west-3

# Attendre la suppression complète
aws cloudformation wait stack-delete-complete \
  --stack-name formation-webserver-lab \
  --region eu-west-3
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-stack :
> {
>     "StackId": "arn:aws:cloudformation:eu-west-3:123456789012:stack/formation-webserver-lab/b2c3d4e5-f6a7-8901-bcde-f12345678901"
> }
>
> # (après wait stack-create-complete — ~3 à 5 minutes)
> # describe-stacks → WebServerURL :
> http://ec2-15-236-78-42.eu-west-3.compute.amazonaws.com
>
> # Le navigateur affiche la page HTML avec "Bienvenue sur votre serveur web CloudFormation!"
> ```
> [!WARNING]
> **IAM requis pour CloudFormation :** Pour déployer ce template, l'utilisateur (ou le rôle IAM) doit avoir les permissions de créer des ressources EC2, VPC, Security Groups. En entreprise, créer un rôle IAM dédié `CloudFormationDeployRole` avec les permissions nécessaires, plutôt que d'utiliser un compte admin.
> [!IMPORTANT]
> **`delete-stack` supprime toutes les ressources :** La commande `delete-stack` détruit la VPC, le subnet, l'IGW, le Security Group ET l'instance EC2 de manière irréversible. Toutes les données stockées sur l'instance EBS seront perdues. Toujours vérifier `--stack-name` avant d'exécuter.
##### Points clés de ce template production

| Élément | Explication | Bonne pratique |
|---------|------------|----------------|
| **VPC CIDR 10.0.0.0/16** | Classe privée standard pour les VPC | Utiliser RFC 1918 (10.x, 172.16.x, 192.168.x) |
| **Subnet 10.0.1.0/24** | Plage de 251 adresses disponibles | Laisser de l'espace pour futurs subnets |
| **IGW + Route 0.0.0.0/0** | Rend le subnet public et accessible d'Internet | Nécessaire pour un serveur web public |
| **Security Group restrictif** | HTTP (80) + SSH (22) explicites | En prod : limiter SSH à des IPs spécifiques |
| **UserData script** | Configure Apache automatiquement au lancement | Évite la configuration manuelle post-lancement |
| **Outputs** | Affiche l'URL et l'IP après déploiement | Indispensable pour que l'utilisateur sache comment accéder |
| **Tags** | Identification et traçabilité | Permet le suivi des ressources pour la facturation |

---


### 3.4 Déployer le template avec AWS CLI

Une fois le template rédigé, on peut le déployer depuis la ligne de commande :

> [!NOTE]
> **Valider le template avant déploiement :** Avant de créer une stack, il est conseillé de valider la syntaxe YAML avec `aws cloudformation validate-template --template-body file://mon-fichier.yaml`. Cette commande vérifie la syntaxe mais pas la validité des valeurs (AMI ID, type d'instance, etc.).
```bash
# 1. Créer la stack (remplacer mon-fichier.yaml par le chemin réel)
aws cloudformation create-stack \
  --stack-name ma-premiere-stack \
  --template-body file://mon-fichier.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
  --region eu-west-1

# Résultat attendu : StackId
# Output : arn:aws:cloudformation:eu-west-1:123456789:stack/ma-premiere-stack/guid

# 2. Suivre la progression du déploiement
aws cloudformation describe-stack-events \
  --stack-name ma-premiere-stack \
  --region eu-west-1

# 3. Une fois CREATE_COMPLETE, afficher les outputs
aws cloudformation describe-stacks \
  --stack-name ma-premiere-stack \
  --query 'Stacks[0].Outputs' \
  --region eu-west-1

# 4. Pour mettre à jour la stack (ex. changer le type d'instance)
aws cloudformation update-stack \
  --stack-name ma-premiere-stack \
  --template-body file://mon-fichier.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small \
  --region eu-west-1

# 5. Supprimer la stack (attention : cela supprime TOUTES les ressources)
aws cloudformation delete-stack \
  --stack-name ma-premiere-stack \
  --region eu-west-1
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-stack :
> {
>     "StackId": "arn:aws:cloudformation:eu-west-1:123456789012:stack/ma-premiere-stack/a1b2c3d4-e5f6-7890-abcd-ef1234567890"
> }
>
> # describe-stack-events (extrait) :
> {
>     "StackEvents": [
>         {
>             "StackId": "arn:aws:cloudformation:eu-west-1:...",
>             "EventId": "...",
>             "ResourceStatus": "CREATE_COMPLETE",
>             "ResourceType": "AWS::EC2::Instance",
>             "LogicalResourceId": "MonInstance",
>             "Timestamp": "2026-03-24T10:32:15.000Z"
>         },
>         {
>             "ResourceStatus": "CREATE_COMPLETE",
>             "ResourceType": "AWS::CloudFormation::Stack",
>             "LogicalResourceId": "ma-premiere-stack"
>         }
>     ]
> }
>
> # describe-stacks (Outputs) :
> [
>     {
>         "OutputKey": "PublicIP",
>         "OutputValue": "54.12.34.56",
>         "Description": "Adresse IP publique de l'instance EC2"
>     },
>     {
>         "OutputKey": "WebServerURL",
>         "OutputValue": "http://54.12.34.56",
>         "Description": "URL pour accéder au serveur web"
>     }
> ]
> ```
> [!IMPORTANT]
> **Attention — `delete-stack` supprime TOUTES les ressources !** La commande `aws cloudformation delete-stack` détruit définitivement toutes les ressources créées par la stack (instances EC2, VPC, RDS, S3…). Il n'y a pas de corbeille. Assurez-vous d'avoir des sauvegardes et de cibler la bonne stack avant d'exécuter cette commande.
---

### 3.5 Avantages de CloudFormation

| Avantage | Bénéfice pédagogique |
|----------|----------------------|
| **Automation complète** | Créer/détruire une infrastructure complexe en quelques minutes |
| **Gestion des dépendances** | CloudFormation sait que l'IGW doit être créée AVANT d'être attachée à la VPC |
| **Versionning** | Stocker les templates dans Git, tracer tous les changements |
| **Reproductibilité** | Déployer exactement la même infrastructure en 10 environnements différents |
| **Rollback** | Si une erreur survient, CloudFormation annule les changements automatiquement |
| **Coût** | CloudFormation est gratuit (on paie seulement les ressources créées) |

> [!WARNING]
> **CloudFormation Drift — Divergence de configuration :** Si vous modifiez manuellement des ressources gérées par CloudFormation (via la console ou la CLI), elles entrent en état de **drift** — elles ne correspondent plus au template. CloudFormation ne détecte pas ces écarts automatiquement. Utilisez `aws cloudformation detect-stack-drift --stack-name <nom>` pour identifier les ressources divergentes. Règle d'or : **ne jamais modifier manuellement une ressource gérée par CloudFormation**.
> **Référence** : [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)

---

## 4. AWS Systems Manager — Automatisation opérationnelle

### 4.1 Qu'est-ce que Systems Manager ?

**AWS Systems Manager (SSM)** est un service d'**administration centralisée** et d'**automatisation opérationnelle** pour les ressources AWS et hybrides.

Il remplace le service obsolète **OpsWorks** et fournit des outils modernes pour :

- Exécuter des **scripts à distance** sur des instances EC2 (**Run Command**)
- **Patcher** automatiquement les systèmes (**Patch Manager**)
- Stocker des **configurations centralisées** (**Parameter Store**)
- Automatiser des **tâches complexes** (**Automation Documents**)
- Collectionner un **inventaire** de toutes les ressources (**Inventory**)

```
Analogie : Systems Manager est comme un TABLEAU DE CONTRÔLE À DISTANCE

Avant (sans SSM) :
  Admin doit se connecter à chaque serveur manuellement :
    ssh ubuntu@10.0.1.100
    ssh ubuntu@10.0.1.101
    ssh ubuntu@10.0.1.102
    ... exécuter la même commande 100 fois

Avec Systems Manager Run Command :
  Admin exécute UNE SEULE commande :
    aws ssm send-command --document-name "AWS-RunShellScript" \
                          --parameters commands=["apt-get update"]
  → Appliquée automatiquement à 100 instances simultaneously
```

---

### 4.2 Fonctionnalités clés de Systems Manager

#### Run Command — Exécuter des scripts à distance

**Run Command** permet d'exécuter des scripts sans accès SSH direct.

**Prérequis** :
- Instance EC2 doit avoir le rôle IAM `AmazonSSMManagedInstanceCore`
- L'agent SSM est pré-installé sur les AMI récentes

```bash
# Exemple : Installer Apache HTTP Server sur 5 instances
aws ssm send-command \
  --instance-ids i-12345 i-67890 i-abcde i-fghij i-klmno \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=[
    "apt-get update",
    "apt-get install -y apache2",
    "systemctl start apache2",
    "systemctl enable apache2"
  ]'

# Résultat : les 5 instances exécutent les commandes EN PARALLÈLE
# Voir les résultats dans CloudWatch Logs ou via CLI :
aws ssm get-command-invocation \
  --command-id <command-id> \
  --instance-id i-12345
```

> [!TIP]
> **Résultat attendu :**
> ```
> # send-command :
> {
>     "Command": {
>         "CommandId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
>         "DocumentName": "AWS-RunShellScript",
>         "Status": "Pending",
>         "TargetCount": 5,
>         "CompletedCount": 0
>     }
> }
>
> # get-command-invocation (après exécution) :
> {
>     "CommandId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
>     "InstanceId": "i-12345",
>     "Status": "Success",
>     "StatusDetails": "Success",
>     "StandardOutputContent": "Reading package lists...\nBuilding dependency tree...\nThe following NEW packages will be installed: apache2\nSetting up apache2 (2.4.52-1ubuntu4)...\n",
>     "StandardErrorContent": ""
> }
> ```
> **Résultat attendu :** `send-command` retourne un `CommandId`. `get-command-invocation` affiche le `Status` (`InProgress` → `Success`) et les `StandardOutputContent` avec la sortie de chaque commande exécutée sur l'instance.

---

#### Parameter Store — Stocker des configurations centralisées

**Parameter Store** permet de stocker des variables (secrets, chemins d'accès, configurations) de manière centralisée. L'avantage clé : vos applications lisent leurs secrets via l'API SSM, sans jamais avoir de valeur en clair dans le code ou un fichier `.env`.

```bash
# Stocker un secret (ex. mot de passe database)
aws ssm put-parameter \
  --name /prod/database/password \
  --value "SecurePassword123!" \
  --type "SecureString" \
  --description "Mot de passe RDS pour production"

# Récupérer la valeur dans une application
aws ssm get-parameter \
  --name /prod/database/password \
  --with-decryption

# Stocker une configuration simple
aws ssm put-parameter \
  --name /app/api-url \
  --value "https://api.example.com" \
  --type "String"
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-parameter :
> {
>     "Version": 1,
>     "Tier": "Standard"
> }
>
> # get-parameter (avec --with-decryption) :
> {
>     "Parameter": {
>         "Name": "/prod/database/password",
>         "Type": "SecureString",
>         "Value": "SecurePassword123!",
>         "Version": 1,
>         "LastModifiedDate": "2026-03-24T10:00:00.000Z",
>         "ARN": "arn:aws:ssm:eu-west-3:123456789012:parameter/prod/database/password",
>         "DataType": "text"
>     }
> }
> ```
> **Résultat attendu :** `put-parameter` retourne un numéro de version (`"Version": 1`). `get-parameter` retourne le JSON avec `"Value"` déchiffré (grâce à `--with-decryption`). Sans ce flag, `SecureString` serait masqué.

---

#### Session Manager — Accès shell sécurisé sans SSH

**Session Manager** permet de se connecter à une instance EC2 **sans port SSH ouvert**, via la console AWS ou CLI. C'est la méthode recommandée pour accéder aux instances en production — zéro clé SSH à gérer, audit complet automatique.

```bash
# Démarrer une session interactive sur une instance
aws ssm start-session \
  --target i-12345

# AWS configure le tunnel securely et vous connecte au shell
```

> [!TIP]
> **Résultat attendu :**
> ```
> Starting session with SessionId: stagiaire-demo-0abc123def456789
> sh-4.2$
> ```
> Un shell bash s'ouvre directement sur l'instance sans passer par SSH. Toutes les commandes saisies sont journalisées dans CloudTrail. Si la commande échoue avec `TargetNotConnected`, vérifiez que l'agent SSM est actif (`systemctl status amazon-ssm-agent`) et que le rôle IAM `AmazonSSMManagedInstanceCore` est attaché à l'instance.
> **Note :** Un shell bash s'ouvre sur l'instance (`sh-4.2$`). Toutes les commandes sont journalisées dans CloudTrail. Si la commande échoue avec `TargetNotConnected`, vérifiez que l'agent SSM est actif et que le rôle IAM `AmazonSSMManagedInstanceCore` est attaché à l'instance.

**Avantages** :
- Pas besoin d'ouvrir le port 22 → Sécurité renforcée
- Audit complet des sessions dans CloudTrail
- Gestion centralisée des accès via IAM

---

#### Patch Manager — Appliquer les mises à jour automatiquement

**Patch Manager** scanne les instances et applique les patchs de sécurité/OS. On définit d'abord une **Patch Baseline** (quels patchs approuver et dans quel délai), puis on associe cette baseline aux instances.

```bash
# Scanner les instances pour les mises à jour manquantes
aws ssm describe-instance-patches \
  --instance-id i-12345

# Créer un plan de patch automatique
aws ssm create-patch-baseline \
  --name "MonLieuxPatchLineMonthly" \
  --operating-system "UBUNTU" \
  --approval-rules 'PatchRules=[{PatchFilterGroup={PatchFilters=[{Key=CLASSIFICATION,Values=[SECURITY,BUGFIX]}]},ApproveAfterDays=7}]'
```

> [!TIP]
> **Résultat attendu :**
> ```
> # describe-instance-patches :
> {
>     "Patches": [
>         {
>             "Title": "linux-aws-headers-5.15.0-1056",
>             "KBId": "USN-6819-1",
>             "Classification": "SECURITY",
>             "Severity": "Important",
>             "State": "Missing",
>             "InstalledTime": null
>         },
>         {
>             "Title": "libssl3",
>             "Classification": "SECURITY",
>             "Severity": "Critical",
>             "State": "Missing"
>         }
>     ]
> }
>
> # create-patch-baseline :
> {
>     "BaselineId": "pb-0abc123def456789a",
>     "Name": "MonLieuxPatchLineMonthly",
>     "OperatingSystem": "UBUNTU",
>     "CreatedDate": "2026-03-24T10:00:00.000Z"
> }
> ```
> **Résultat attendu :** `describe-instance-patches` liste les patchs manquants avec leur sévérité (`Critical`, `Important`…). `create-patch-baseline` retourne un `BaselineId` (ex. `pb-0abc123`). Cette baseline s'applique ensuite via une **Maintenance Window** planifiée.

> **Référence** : [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)

---
### 4.3 AWS OpsWorks — Gestion de configuration avec Chef et Puppet (Contexte Historique)

#### 🚫 Important : AWS OpsWorks en fin de vie

**AWS OpsWorks** était un service de **gestion de configuration** basé sur les outils open source **Chef** et **Puppet**. Depuis 2023, AWS recommande **fortement** de migrer vers **AWS Systems Manager** pour les nouvelles infrastructures.

**Statut actuel :**
- ❌ OpsWorks for Chef Automate : **fin de vie et désactivé depuis le 5 mai 2024**
- ❌ OpsWorks for Puppet Enterprise : **fin de vie et désactivé depuis le 5 mai 2024**
- ❌ OpsWorks Stacks : **fin de vie et désactivé depuis le 26 mai 2024**

#### Qu'était AWS OpsWorks ?

OpsWorks permettait de **déployer et configurer des applications** sur des instances EC2 en utilisant des **scripts de configuration déclaratifs** :

<img src="formations/aws-initiation-approfondissement/11-images/opsworks-vs-ssm.svg"
     alt="CloudFormation vs OpsWorks vs Systems Manager"
     style="display:block; margin:auto; width:90%">

#### Migration depuis OpsWorks vers Systems Manager

Si vous héritez d'une infrastructure avec OpsWorks :

```bash
# 1. Exporter les recipes Chef / configurations Puppet
#    → Convertir en shell scripts / PowerShell pour Systems Manager

# 2. Utiliser Systems Manager Run Command pour exécuter les scripts
aws ssm send-command \
  --instance-ids i-12345678 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["yum install httpd -y","systemctl start httpd"]'

# 3. Utiliser Patch Manager pour appliquer les correctifs
aws ssm create-patch-baseline \
  --name "MonLignePatchMigree" \
  --operating-system "UBUNTU" \
  --approval-rules ...

# 4. Archiver les ressources OpsWorks
# Tous les stacks OpsWorks doivent être supprimés avant le 26 janvier 2024
```

> [!TIP]
> **Résultat attendu :**
> ```
> # send-command (migration depuis OpsWorks) :
> {
>     "Command": {
>         "CommandId": "c3d4e5f6-a7b8-9012-cdef-a12345678901",
>         "DocumentName": "AWS-RunShellScript",
>         "Status": "Pending",
>         "TargetCount": 1
>     }
> }
> ```
#### Ressources de migration

```
📎 [AWS OpsWorks → Systems Manager Migration Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/opsworks-migration.html)
📎 [Pourquoi OpsWorks est obsolète](https://aws.amazon.com/fr/blogs/france/migration-opsworks-systems-manager/)
```

**Conclusion pour les stagiaires :** Vous ne créerez JAMAIS un nouvel OpsWorks stack. Si vous le rencontrez en production, c'est un signal pour moderniser vers Systems Manager.

---


## 5. AWS Elastic Beanstalk — Déploiement simplifié d'applications

### 5.1 Qu'est-ce que Elastic Beanstalk ?

**AWS Elastic Beanstalk** est une plateforme PaaS managée qui permet de **déployer des applications web** sans gérer l'infrastructure sous-jacente.

Contrairement à CloudFormation où vous décrivez **chaque ressource manuellement**, Beanstalk **abstrait** la complexité : vous uploadez simplement votre code, et Beanstalk s'occupe de :

- Créer/gérer les instances EC2
- Configurer l'Auto Scaling
- Mettre en place le Load Balancer
- Activer le monitoring CloudWatch
- Gérer les mises à jour de l'OS et du runtime

```
Analogie : Elastic Beanstalk vs CloudFormation

CloudFormation = "Je veux décrire exactement mon infrastructure"
  - Créer VPC, subnets, security groups, EC2, ALB, RDS, etc.
  - Contrôle total mais responsabilité complète

Elastic Beanstalk = "Je veux juste déployer mon app, pas m'embêter avec l'infra"
  - Upload le code (Node.js, Python, Java, .NET)
  - Beanstalk crée l'infrastructure automatiquement
  - Mise en échelle automatique en cas de charge
  - Moins de contrôle mais moins de friction
```

---

### 5.2 Runtimes et plateformes supportées

Elastic Beanstalk supporte plusieurs langages et frameworks :

| Langage | Framework | Exemple |
|---------|-----------|---------|
| **Node.js** | Express, Fastify | Application web Node.js classique |
| **Python** | Flask, Django | Application Flask avec routes |
| **Java** | Spring Boot | Application Spring Boot JAR |
| **.NET** | ASP.NET Core | Application web .NET |
| **PHP** | Laravel, Symfony | Brochure web PHP |
| **Go** | Gin, Echo | Microservice Go |
| **Docker** | N'importe quel container | Flexibilité maximale |

> **Référence** : [Elastic Beanstalk Platforms](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html)

---

### 5.3 Déployer une app Node.js avec Elastic Beanstalk (CLI)

Elastic Beanstalk gère toute l'infrastructure à votre place : vous fournissez juste le code. La commande `eb create` provisionne automatiquement une instance EC2, un load balancer, un Auto Scaling Group et CloudWatch. Vous n'avez qu'à pousser votre code.

```bash
# 1. Créer une application Node.js simple (app.js)
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Bonjour depuis Elastic Beanstalk !');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# 2. Créer un package.json
cat > package.json << 'EOF'
{
  "name": "monappbeanstalk",
  "version": "1.0.0",
  "description": "Petite app Express sur Beanstalk",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# 3. Initialiser un environnement Elastic Beanstalk
eb init -p node.js-18 monappbeanstalk --region eu-west-1

# 4. Créer et déployer l'environnement
# AWS crée automatiquement :
#   - Une application Beanstalk
#   - Un environnement (Dev, Staging, Prod, etc.)
#   - 1+ instances EC2 (t2.micro par défaut)
#   - Un Load Balancer
#   - Auto Scaling Group
#   - Monitoring CloudWatch
eb create monappbeanstalk-env --instance-type t2.micro --envvars NODE_ENV=production

# 5. Vérifier l'état du déploiement
eb status

# 6. Voir l'URL publique de l'application
eb open

# 7. Voir les logs en temps réel
eb logs -f

# 8. Augmenter le nombre minimum d'instances
eb scale 3

# 9. Deployer une nouvelle version du code
# (après modifi app.js)
eb deploy

# 10. Supprimer l'application et l'environnement
eb terminate monappbeanstalk-env
```

> [!TIP]
> **Résultat attendu :**
> ```
> # eb create (extrait de la progression) :
> Creating application version archive "app-v1".
> Uploading monappbeanstalk/app-v1.zip to S3. This may take a while.
> Upload Complete.
> Environment details for: monappbeanstalk-env
>   Application name: monappbeanstalk
>   Region: eu-west-1
>   Deployed Version: app-v1
>   Environment ID: e-abc123defg
>   Platform: arn:aws:elasticbeanstalk:eu-west-1::platform/Node.js 18 running on 64bit Amazon Linux 2023
>   Tier: WebServer-Standard-1.0
>   CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
>   Updated: 2026-03-24 10:45:00.000000+00:00
>   Status: Launching
>   Health: Grey
> ...
> INFO: Successfully launched environment: monappbeanstalk-env
>
> # eb status :
> Environment details for: monappbeanstalk-env
>   Status: Ready
>   Health: Green
>   CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
>
> # L'application répond "Bonjour depuis Elastic Beanstalk !" à http://monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
> ```
> [!WARNING]
> **Cold start Elastic Beanstalk :** Le premier déploiement (`eb create`) prend généralement 5 à 10 minutes car AWS provisionne l'infrastructure complète (EC2, ELB, Auto Scaling Group). Les déploiements suivants (`eb deploy`) sont plus rapides (1 à 3 minutes). Si `eb status` reste sur `Launching` trop longtemps, consultez les logs avec `eb logs`.
> **Résultat attendu :** `eb create` affiche la progression en temps réel (création VPC, EC2, Load Balancer…) et se termine avec l'URL publique de l'application. `eb open` ouvre cette URL dans votre navigateur — vous devez voir "Bonjour depuis Elastic Beanstalk !".

---

### 5.4 Avantages et limitations

| Avantage | Limitation |
|----------|-----------|
| Déploiement simple du code | Contrôle limité sur l'infrastructure |
| Scalabilité automatique | Moins flexible que CloudFormation |
| Monitoring intégré | Pas adapté aux architectures très complexes |
| Mises à jour OS automatiques | Coûts potentiellement plus élevés |

> **Référence** : [Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)


### 5.5 Elastic Beanstalk vs Lambda — Quand choisir quoi ?

Ces deux services répondent à la même question — "comment déployer du code sans gérer de serveurs" — mais avec des philosophies radicalement différentes.

| | Beanstalk | Lambda |
|---|---|---|
| Exécution | EC2 toujours allumé, Load Balancer, Auto Scaling Group, CloudWatch | Conteneur éphémère, créé à la demande et détruit après exécution |
| Paradigme | Lift & Shift | Event-Driven |

#### Tableau de comparaison

| Critère | Elastic Beanstalk | Lambda |
|---------|------------------|--------|
| **Paradigme** | PaaS — application toujours en cours | FaaS — fonction déclenchée à la demande |
| **Infrastructure** | EC2 + ELB + ASG (gérés automatiquement) | Aucune instance visible |
| **Démarrage** | Toujours chaud | Cold start possible (ms à quelques s) |

> [!WARNING]
> **Lambda Cold Starts :** Lors du premier appel d'une fonction Lambda (ou après une longue période d'inactivité), AWS doit initialiser le conteneur d'exécution — c'est le **cold start**. La latence peut aller de quelques dizaines de ms (Node.js/Python) à plusieurs secondes (Java). Solutions : **Provisioned Concurrency** (maintient N conteneurs chauds en permanence, payant), ou choisir un runtime léger (Node.js/Python) pour les APIs sensibles à la latence.
| **Durée max d'exécution** | Illimitée | **15 minutes** |
| **Mémoire max** | Celle de l'instance (jusqu'à 384 Go) | **10 Go** |
| **Stockage local** | EBS persistent | **/tmp : 10 Go seulement** |
| **Langages supportés** | Java, Node.js, Python, Ruby, PHP, Go, .NET | Java, Node.js, Python, Ruby, Go, .NET, Rust + custom runtime |
| **Modèle de coût** | EC2 à la seconde (même si pas de requêtes) | À l'invocation (1M req gratuites/mois) |
| **Scaling** | Auto Scaling Group (minutes) | Instantané, jusqu'à 1 000 exécutions parallèles |
| **État** | Stateful possible (session, fichiers) | Stateless obligatoire |
| **Réseau** | VPC natif, Security Groups | VPC optionnel |
| **Déploiement** | ZIP, WAR, Docker, `eb deploy` | ZIP, container, `aws lambda update-function-code` |

#### Modèle de coût détaillé

**Elastic Beanstalk :**
```
Beanstalk en lui-même = GRATUIT
Vous payez les ressources qu'il crée :

Exemple : app Node.js standard
  1× EC2 t3.small      → 0,023 $/h  → ~17 $/mois
  1× ELB Application   → 0,008 $/h  → ~6 $/mois  + 0,008 $/LCU
  Stockage EBS 20 Go   →            → ~2,5 $/mois
  ─────────────────────────────────────────────────
  TOTAL (1 instance)   →            → ~26 $/mois
  (même si 0 utilisateur cette nuit-là)
```

**Lambda :**
```
Free Tier permanent : 1 000 000 requêtes/mois + 400 000 Go-secondes/mois

Au-delà :
  Requêtes : 0,20 $ / million
  Durée    : 0,0000000167 $ / Go-seconde

Exemple : API Lambda 128 Mo RAM, 200 ms d'exécution, 1M requêtes/mois
  Durée : 1 000 000 × 0,128 Go × 0,2 s = 25 600 Go-secondes → GRATUIT (< 400 000)
  Requêtes : 1 000 000 → GRATUIT (< 1M)
  TOTAL : 0 $ (dans le Free Tier)

Exemple : 10M requêtes/mois
  Durée : 256 000 Go-s supplémentaires → 256 000 × 0,0000000167 = ~0,004 $
  Requêtes : 9M supplémentaires → 9 × 0,20 = 1,80 $
  TOTAL : ~1,80 $/mois
```

#### Quand utiliser lequel ?

```
CHOISIR BEANSTALK si :
  ✅ Application web traditionnelle (Django, Express, Spring Boot, WordPress)
  ✅ Traitement long (> 15 minutes)
  ✅ Besoin de sessions persistantes côté serveur
  ✅ Migration d'une app existante ("lift & shift")
  ✅ Besoin d'accès à des fichiers locaux entre requêtes
  ✅ Équipe non familière avec l'architecture event-driven

CHOISIR LAMBDA si :
  ✅ API REST légère (avec API Gateway)
  ✅ Traitement d'événements (upload S3, message SQS, stream DynamoDB)
  ✅ Tâches planifiées (cron CloudWatch Events)
  ✅ Traitement de fichiers (redimensionnement images, parsing CSV)
  ✅ Webhooks, notifications, automatisations
  ✅ Trafic très variable (pics et creux importants)
  ✅ Budget serré avec faible volumétrie
```

> 💡 **En pratique** : beaucoup d'architectures modernes combinent les deux. Beanstalk pour le frontend/API principale, Lambda pour les traitements en arrière-plan (envoi d'emails, génération de rapports, nettoyage de données).

📎 [Elastic Beanstalk vs Lambda — AWS Blog](https://aws.amazon.com/compare/the-difference-between-aws-lambda-and-elastic-beanstalk/)

---

## 6. Amazon CloudWatch — Supervision et alarmes

### 6.1 Qu'est-ce que CloudWatch ?

**Amazon CloudWatch** est le service de **monitoring centralisé** d'AWS. Il collecte, stocke et affiche des métriques sur :

- **Instances EC2** : CPU, mémoire réseau, I/O disque
- **Bases RDS** : connexions actives, CPU, I/O
- **Load Balancers** : requêtes/seconde, latence
- **Applications custom** : envoi de métriques via API

<img src="formations/aws-initiation-approfondissement/11-images/cloudwatch-architecture.svg"
     alt="Architecture Amazon CloudWatch"
     style="display:block; margin:auto; width:90%">

---

### 6.2 Créer une alarme CloudWatch (CLI)

```bash
# 1. Créer une alarme sur la métrique CPU d'une instance EC2
# L'alarme se déclenche si CPU > 80% pendant 2 périodes consécutives (10 min)
aws cloudwatch put-metric-alarm \
  --alarm-name "MonInstance-CPU-Élevé" \
  --alarm-description "Alerte CPU élevé instance EC2" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-12345

# 2. Ajouter une action SNS (envoyer un email)
# D'abord, créer un sujet SNS
aws sns create-topic --name MonTopicAlarmes
# Output : TopicArn: arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes

# S'abonner au sujet (recevoir les notifications)
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --protocol email \
  --notification-endpoint admin@example.com

# 3. Modifier l'alarme pour envoyer une notification
aws cloudwatch put-metric-alarm \
  --alarm-name "MonInstance-CPU-Élevé" \
  --alarm-description "Alerte CPU élevé instance EC2" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-12345 \
  --alarm-actions arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes

# 4. Lister toutes les alarmes
aws cloudwatch describe-alarms

# 5. Supprimer une alarme
aws cloudwatch delete-alarms --alarm-names "MonInstance-CPU-Élevé"
```

> [!TIP]
> **Résultat attendu :**
> ```
> # sns create-topic :
> {
>     "TopicArn": "arn:aws:sns:eu-west-1:123456789012:MonTopicAlarmes"
> }
>
> # sns subscribe :
> {
>     "SubscriptionArn": "pending confirmation"
> }
> # → Un email est envoyé à admin@example.com avec un lien de confirmation
>
> # put-metric-alarm : (pas de sortie si succès — code HTTP 200)
>
> # describe-alarms (extrait) :
> {
>     "MetricAlarms": [
>         {
>             "AlarmName": "MonInstance-CPU-Élevé",
>             "AlarmDescription": "Alerte CPU élevé instance EC2",
>             "StateValue": "OK",
>             "MetricName": "CPUUtilization",
>             "Threshold": 80.0,
>             "Period": 300,
>             "EvaluationPeriods": 2
>         }
>     ]
> }
> ```
---

### 6.3 Envoyer des métriques custom depuis une application

Applications peuvent envoyer des métriques CloudWatch pour monitorer des KPI métier :

```bash
# Exemple : envoyer une métrique custom "OrdersPerMinute"
aws cloudwatch put-metric-data \
  --namespace "MonApplication" \
  --metric-name "OrdersPerMinute" \
  --value 42 \
  --unit Count \
  --timestamp 2025-03-24T14:30:00Z
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-metric-data : pas de sortie si succès (HTTP 200)
> # La métrique est visible dans CloudWatch Console sous "MonApplication > OrdersPerMinute"
> # après environ 1 minute de délai d'ingestion.
> ```
```bash
# Ou dans un script Python :
import boto3

cloudwatch = boto3.client('cloudwatch')

# Envoyer une métrique custom
cloudwatch.put_metric_data(
    Namespace='MonApplication',
    MetricData=[
        {
            'MetricName': 'PedidosProcessados',
            'Value': 42,
            'Unit': 'Count',
            'Timestamp': datetime.utcnow()
        }
    ]
)
```

---

### 6.4 Tableaux de bord CloudWatch

CloudWatch permet de créer des **dashboards** personnalisés affichant plusieurs métriques :

```bash
# Créer un dashboard JSON
cat > dashboard.json << 'EOF'
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          [ "AWS/EC2", "CPUUtilization", { "stat": "Average" } ],
          [ ".", "NetworkIn", { "stat": "Sum" } ]
        ],
        "period": 300,
        "stat": "Average",
        "region": "eu-west-1",
        "title": "Métriques EC2"
      }
    }
  ]
}
EOF

# Créer le dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "MonDashboard" \
  --dashboard-body file://dashboard.json
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-dashboard :
> {
>     "DashboardValidationMessages": []
> }
> # Le tableau de bord "MonDashboard" est maintenant visible dans la console CloudWatch.
> # Accès : CloudWatch → Dashboards → MonDashboard
> ```
> **Référence** : [CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)

---
### 6.5 AWS SDK pour Développeurs — Automatisation Programmatique

Jusque-là, nous avons utilisé la **CLI AWS** pour exécuter des commandes manuellement. Mais les **SDK AWS** permettent d'**intégrer AWS directement dans du code applicatif** (Python, Node.js, Java, Go, etc.).

#### Qu'est-ce que le SDK AWS ?

**SDK** = **kit de développement** fourni par AWS dans plusieurs langages pour interagir avec les services AWS par programmation.

```
Analogie : CLI vs SDK

CLI (AWS CLI)
  ↓
Outil en ligne de commande
Exécute des commandes manuellement ou dans des scripts bash
Exemple : aws ec2 describe-instances

SDK (boto3, SDK.js, SDK.java)
  ↓
Bibliothèque logicielle intégrée dans votre code
Votre application Python/Node/Java appelle AWS directement
Exemple : ec2_client.describe_instances()
```

#### SDK AWS Disponibles

| Langage | Nom SDK | Cas d'usage |
|---------|---------|-----------|
| **Python** | `boto3` | Data science, Lambda, backend | |
| **JavaScript/Node.js** | `AWS SDK for JavaScript` | Applications web, serverless | |
| **Java** | `AWS SDK for Java` | Entreprise, Spring Boot | |
| **Go** | `AWS SDK for Go` | CLI tools, microservices | |
| **C#/.NET** | `AWS SDK for .NET` | Windows, applications d'entreprise | |
| **PHP** | `AWS SDK for PHP** | Applications web, Laravel | |

#### Introduction à boto3 (Python)

**boto3** est la **SDK AWS officielle pour Python**. Elle est utilisée dans :
- Scripts d'automatisation
- Applications Lambda
- Tâches cron de maintenance
- Outils de gestion d'infrastructure

##### Installation de boto3

```bash
# Installer boto3
pip install boto3

# Vérifier l'installation
python3 -c "import boto3; print(boto3.__version__)"
```

> [!TIP]
> **Résultat attendu :**
> ```
> Collecting boto3
>   Downloading boto3-1.34.69-py3-none-any.whl (139 kB)
> Successfully installed boto3-1.34.69 botocore-1.34.69 s3transfer-0.10.1
> 1.34.69
> ```
##### Exemple 1 : Lister les instances EC2

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Lister toutes les instances
response = ec2_client.describe_instances()

# Parcourir les instances
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        instance_id = instance['InstanceId']
        instance_type = instance['InstanceType']
        state = instance['State']['Name']
        
        # Afficher les informations
        print(f"Instance : {instance_id}")
        print(f"  Type : {instance_type}")
        print(f"  État : {state}")
        print()
```

**Résultat :**
```
Instance : i-0123456789abcdef0
  Type : t3.micro
  État : running

Instance : i-0987654321abcdef0
  Type : t3.small
  État : stopped
```

##### Exemple 2 : Créer une snapshot EBS

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Créer un snapshot du volume vol-12345678
response = ec2_client.create_snapshot(
    VolumeId='vol-12345678',
    Description='Sauvegarde avant migration',
    TagSpecifications=[
        {
            'ResourceType': 'snapshot',
            'Tags': [
                {'Key': 'Name', 'Value': 'backup-migration-2026-03-24'},
                {'Key': 'Environment', 'Value': 'production'}
            ]
        }
    ]
)

# Afficher l'ID du snapshot créé
snapshot_id = response['SnapshotId']
progress = response['Progress']

print(f"Snapshot créé : {snapshot_id}")
print(f"Progression : {progress}")
```

##### Exemple 3 : Arrêter une instance EC2

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Arrêter une instance
instance_id = 'i-0123456789abcdef0'

response = ec2_client.stop_instances(InstanceIds=[instance_id])

# Vérifier que l'arrêt est en cours
for instance in response['StoppingInstances']:
    print(f"Instance {instance['InstanceId']} est en cours d'arrêt")
    print(f"État précédent : {instance['PreviousState']['Name']}")
    print(f"État courant : {instance['CurrentState']['Name']}")
```

##### Exemple 4 : Créer une alarme CloudWatch

```python
import boto3

# Créer un client CloudWatch
cloudwatch_client = boto3.client('cloudwatch', region_name='eu-west-3')

# Créer une alarme si CPU > 80%
cloudwatch_client.put_metric_alarm(
    AlarmName='CPU-Haute-Production',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Period=300,  # 5 minutes
    Statistic='Average',
    Threshold=80.0,
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:eu-west-3:123456789:AlertesProduction'],
    Dimensions=[
        {
            'Name': 'InstanceId',
            'Value': 'i-0123456789abcdef0'
        }
    ]
)

print("Alarme CloudWatch créée avec succès")
```

#### Bonnes pratiques boto3

```
✓ Utiliser des variables d'environnement ou des profils AWS pour les credentials
✓ Gérer les erreurs avec try/except
✓ Utiliser des context managers ou des sessions boto3
✓ Documenter chaque appel API avec un commentaire
✓ Tester en environnement non-production d'abord
✓ Utiliser des rôles IAM appropriés (pas de clés d'accès root)
```

---


## 7. AWS Well-Architected Framework — Mise en pratique

Le Chapitre 1 a introduit les six piliers du **AWS Well-Architected Framework** (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability) avec leurs bonnes pratiques respectives. Maintenant que vous avez manipulé IAM, S3, EC2, Lambda, RDS, VPC et CloudFormation, vous disposez de tous les services nécessaires pour appliquer concrètement ce framework à un cas réel.

> [!NOTE]
> **Besoin d'un rappel des 6 piliers ?** Retournez au Chapitre 1, section 7 — définitions, questions clés et bonnes pratiques par pilier y sont détaillées.
### 7.1 Cas d'étude : Application "CloudPizza"

Imaginons une application de commande de pizzas. Appliquons les 6 piliers :

| Pilier | Décision | Justification |
|--------|----------|---------------|
| **Operational Excellence** | Déployer via CloudFormation + CI/CD | Répétabilité, traçabilité |
| **Security** | IAM par service, KMS pour BDD, bucket S3 privé | Moindre privilège, conformité |
| **Reliability** | Multi-AZ, ALB, RDS Multi-AZ, snapshots EBS quotidiens | RTO 1h, RPO 1h |
| **Performance** | Lambda + DynamoDB au lieu de serveurs | Scalabilité illimitée automatique |
| **Cost Optimization** | Reserved Instances pour serveurs stables, Spot pour batch | Réduire 40% des coûts |
| **Sustainability** | Déployer en Irlande (énergies renouvelables), Lambda sans serveur | Réduire l'empreinte carbone |

---

### 7.2 Rappel — AWS Compute Optimizer et le pilier Cost Optimization

Le Chapitre 3 a détaillé le fonctionnement d'**AWS Compute Optimizer** (collecte CloudWatch, analyse ML, recommandations chiffrées) — c'est l'outil concret qui alimente le pilier **Cost Optimization** vu ci-dessus : dans le cas CloudPizza, c'est lui qui permettrait de vérifier a posteriori que les Reserved Instances choisies sont bien dimensionnées à l'usage réel.

> [!NOTE]
> **Besoin d'un rappel du fonctionnement de Compute Optimizer ?** Retournez au Chapitre 3, section 7 — commande CLI complète, exemple de sortie JSON et cas concret `t3.large → t3.small` y sont détaillés.
---

## 8. Services complémentaires — Queues et événements

### 8.1 Amazon SQS — File d'attente de messages

**Amazon SQS** (Simple Queue Service) est une **file d'attente de messages** complètement gérée.

**Cas d'usage** : Découpler des composants d'une application.

```
Analogie : SQS est comme une BOÎTE AUX LETTRES

Sans SQS (couplage fort) :
  Producteur (app web) → appelle directement Consommateur (worker)
  Si worker est en panne → producteur attend → demande utilisateur bloquée

Avec SQS (découplage) :
  Producteur → envoie message dans SQS → retour immédiat
  Consommateur → consomme messages quand il est prêt (même en panne, pas de perte)
```

Voici la séquence complète : créer la queue, envoyer un message, le lire, puis le supprimer. La suppression explicite est obligatoire — SQS ne supprime pas automatiquement un message après lecture (pour éviter la perte en cas d'échec).

```bash
# Créer une queue SQS
aws sqs create-queue --queue-name MonQueue

# Envoyer un message
aws sqs send-message \
  --queue-url https://sqs.eu-west-1.amazonaws.com/123456789/MonQueue \
  --message-body "Bonjour depuis CloudFormation"

# Consommer un message (avec delete)
aws sqs receive-message \
  --queue-url https://sqs.eu-west-1.amazonaws.com/123456789/MonQueue \
  --max-number-of-messages 1

# Supprimer le message de la queue
aws sqs delete-message \
  --receipt-handle <receipt-handle>
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-queue :
> {
>     "QueueUrl": "https://sqs.eu-west-1.amazonaws.com/123456789012/MonQueue"
> }
>
> # send-message :
> {
>     "MD5OfMessageBody": "9c7c6f0f3f748bdfa5a5e6e7c8d9e0f1",
>     "MessageId": "msg-0abc123def456789a"
> }
>
> # receive-message :
> {
>     "Messages": [
>         {
>             "MessageId": "msg-0abc123def456789a",
>             "ReceiptHandle": "AQEB...longstring...",
>             "MD5OfBody": "9c7c6f0f3f748bdfa5a5e6e7c8d9e0f1",
>             "Body": "Bonjour depuis CloudFormation"
>         }
>     ]
> }
>
> # delete-message : pas de sortie si succès (HTTP 200)
> ```
---

### 8.2 Amazon SNS — Notifications pubsub

**Amazon SNS** (Simple Notification Service) est un service de **notifications pub/sub**.

```
Différence SQS vs SNS :
  SQS : 1 producteur → 1 queue → 1 consommateur (FIFO ou parallèle)
  SNS : 1 producteur → N abonnés (email, SMS, SQS, Lambda, HTTP)
```

Voici comment créer un topic SNS, y abonner une adresse email, et publier un message qui sera envoyé à tous les abonnés simultanément :

```bash
# Créer un sujet SNS
aws sns create-topic --name MonTopicAlarmes

# Ajouter un abonnement email
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --protocol email \
  --notification-endpoint admin@example.com

# Publier un message
aws sns publish \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --message "Alerte : CPU élevé détecté !"
```

> [!TIP]
> **Résultat attendu :**
> ```
> # create-topic :
> {
>     "TopicArn": "arn:aws:sns:eu-west-1:123456789012:MonTopicAlarmes"
> }
>
> # subscribe :
> {
>     "SubscriptionArn": "pending confirmation"
> }
> # → Un email est envoyé avec un lien de confirmation
>
> # publish :
> {
>     "MessageId": "pub-0abc123def456789a"
> }
> # → Tous les abonnés (email, SQS, Lambda) reçoivent le message instantanément
> ```
> **Référence** : [Amazon SQS](https://docs.aws.amazon.com/sqs/)
> **Référence** : [Amazon SNS](https://docs.aws.amazon.com/sns/)

---

### 8.3 Architectures découplées, microservices et sans serveur

Le schéma suivant rassemble les briques d'une architecture sans serveur orientée événements. L'objectif n'est pas de mémoriser une succession d'icônes : suivez le trajet d'une requête, puis identifiez les points où l'application absorbe un pic, conserve un état ou isole une panne.

![Architecture AWS sans serveur avec API Gateway, Lambda, files de messages et services de données](formations/aws-initiation-approfondissement/11-images/aws-serverless-architecture.svg)

API Gateway reçoit les appels synchrones, tandis qu'une file ou un bus d'événements permet de différer certains traitements. Lambda exécute le code sans serveur à administrer, mais le client reste responsable des permissions IAM, des dépendances, de l'idempotence, des erreurs partielles et du cycle de vie des données.

SQS et SNS résolvent le découplage entre deux composants pris isolément. Une architecture applicative complète va plus loin : elle assemble plusieurs services managés pour qu'aucun composant ne dépende directement de la disponibilité d'un autre, et que chaque brique puisse évoluer, tomber en panne ou être remplacée sans effet domino sur le reste du système. C'est le principe des **microservices** : découper une application monolithique en plusieurs services indépendants, chacun responsable d'une capacité métier précise (paiement, catalogue, notifications), communiquant entre eux par API ou par messages plutôt que par appels de fonction directs en mémoire.

**Couplage fort vs couplage faible — le test décisif**

```
Couplage fort (monolithe classique) :
  Service A appelle directement Service B (HTTP synchrone ou appel de fonction)
  → Si B est lent ou en panne, A attend, se bloque, ou échoue en cascade
  → Faire évoluer B (changer sa techno, sa capacité) impose de coordonner A

Couplage faible (architecture découplée) :
  Service A publie un événement/message, sans savoir qui le consommera
  → B (et C, D…) consomment à leur rythme, indépendamment de la disponibilité de A
  → B peut tomber, redémarrer, ou être remplacé sans qu'A ne le sache
```

Le test décisif pour repérer un couplage fort évitable : si le service producteur doit attendre une réponse synchrone du consommateur pour continuer son propre travail, alors qu'il n'a besoin d'aucune information en retour, c'est un candidat naturel au découplage via SQS ou SNS.

**Amazon API Gateway — le point d'entrée unifié**

Dans une architecture microservices, chaque service pourrait exposer sa propre adresse réseau — mais cela oblige les clients (applications mobiles, sites web, partenaires) à connaître et gérer N adresses différentes, et complique la sécurisation (authentification à répliquer partout). **Amazon API Gateway** résout ce problème en offrant un point d'entrée HTTP unique, qui route chaque requête vers le bon service backend (Lambda, conteneur, serveur EC2) selon l'URL et la méthode appelées.

```
Client mobile/web
       │
       ▼
┌─────────────────┐
│  API Gateway     │  ← authentification centralisée, throttling, cache
│  /users   → Lambda A
│  /orders  → Lambda B
│  /catalog → ECS Fargate
└─────────────────┘
```

API Gateway prend en charge, sans code supplémentaire à écrire dans chaque microservice : l'authentification (intégration IAM, Cognito, ou clé API), la limitation de débit (*throttling*) pour protéger les backends d'un pic de trafic, la mise en cache des réponses, et la transformation de requêtes/réponses (mapping de formats).

```bash
# Créer une API REST
aws apigateway create-rest-api --name "MonAPI-Catalogue"

# Créer une ressource sous la racine
aws apigateway create-resource \
  --rest-api-id <api-id> \
  --parent-id <root-resource-id> \
  --path-part "produits"

# Associer une méthode GET à une fonction Lambda (intégration proxy)
aws apigateway put-integration \
  --rest-api-id <api-id> \
  --resource-id <resource-id> \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:eu-west-1:lambda:path/2015-03-31/functions/arn:aws:lambda:eu-west-1:123456789:function:lister-produits/invocations
```

> [!TIP]
> **Résultat attendu (create-rest-api) :**
> ```
> {
>     "id": "a1b2c3d4e5",
>     "name": "MonAPI-Catalogue",
>     "createdDate": "2026-07-27T10:00:00Z",
>     "apiKeySource": "HEADER",
>     "endpointConfiguration": {
>         "types": ["EDGE"]
>     }
> }
> ```
**AWS Step Functions — orchestrer plusieurs services dans un workflow**

Certains traitements ne se résument pas à un simple message transmis d'un service à l'autre : ils enchaînent plusieurs étapes avec de la logique conditionnelle (si le paiement échoue, annuler la réservation), des étapes parallèles (vérifier le stock et calculer les frais de port en même temps), et des reprises sur erreur. **AWS Step Functions** modélise ce type de workflow sous forme de machine à états (*state machine*), où chaque état est typiquement une fonction Lambda, un appel à un autre service AWS, ou une branche de décision.

```json
{
  "Comment": "Traitement d'une commande e-commerce",
  "StartAt": "VerifierStock",
  "States": {
    "VerifierStock": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:verifier-stock",
      "Next": "StockDisponible"
    },
    "StockDisponible": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.stock_ok",
          "BooleanEquals": true,
          "Next": "TraiterPaiement"
        }
      ],
      "Default": "AnnulerCommande"
    },
    "TraiterPaiement": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:traiter-paiement",
      "Catch": [
        {
          "ErrorEquals": ["PaiementRefuse"],
          "Next": "AnnulerCommande"
        }
      ],
      "Next": "ConfirmerCommande"
    },
    "ConfirmerCommande": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:confirmer-commande",
      "End": true
    },
    "AnnulerCommande": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:annuler-commande",
      "End": true
    }
  }
}
```

L'intérêt par rapport à enchaîner ces appels directement dans le code d'une seule fonction Lambda : chaque état est visible, monitorable et rejouable indépendamment dans la console AWS (Step Functions affiche un diagramme visuel de l'exécution, avec l'état exact atteint en cas d'échec) — un débogage bien plus direct qu'une pile d'appels imbriqués dans les logs CloudWatch d'une fonction monolithique.

**Architecture microservices sans serveur complète — exemple**

L'illustration ci-dessous assemble les briques vues dans ce chapitre et les précédents en une architecture microservices sans serveur cohérente pour une application de commande en ligne :

```
                        ┌──────────────┐
   Client (web/mobile) ─▶ API Gateway   │
                        └──────┬───────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        Lambda: Catalogue  Lambda: Commande  Lambda: Compte
              │                │                │
              ▼                ▼                ▼
        DynamoDB          Step Functions    Cognito
        (produits)        (workflow         (utilisateurs)
                            commande)
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                SQS (paiement)      SNS (notifications)
                    │                     │
                    ▼                     ▼
              Lambda: Paiement     Email / SMS client
```

Aucun composant de ce schéma ne connaît directement l'adresse réseau d'un autre composant en amont : le client ne connaît que l'URL d'API Gateway, les Lambdas ne connaissent que les ARN des ressources qu'elles invoquent, et la communication asynchrone (SQS/SNS) élimine toute dépendance temporelle stricte entre le traitement de la commande et l'envoi de la notification. C'est cette absence de dépendance directe qui permet à chaque brique d'être mise à l'échelle, remplacée ou de tomber en panne sans effet domino sur le reste de l'architecture — le principe même du découplage appliqué à l'échelle d'un système complet.

> [!WARNING]
> **Piège fréquent :** multiplier les microservices sans réel besoin métier augmente la complexité opérationnelle (plus de composants à surveiller, plus de latence réseau entre services, plus de scénarios d'échec partiel à gérer) sans bénéfice proportionnel. Le découpage en microservices se justifie quand des équipes différentes doivent déployer indépendamment, quand des composants ont des besoins de mise à l'échelle très différents (le service de paiement encaisse un pic le vendredi soir, le catalogue reste stable), ou quand la résilience d'un composant ne doit jamais bloquer les autres. Un monolithe bien structuré reste souvent le bon choix pour une application simple ou une petite équipe.
> **Référence** : [Amazon API Gateway](https://docs.aws.amazon.com/apigateway/)
> **Référence** : [AWS Step Functions](https://docs.aws.amazon.com/step-functions/)

---

## 9. Certifications AWS — Objectif SAA-C03

Le Chapitre 1 a présenté les quatre niveaux de certification AWS (Fondamental, Associate, Professional, Specialty) et pourquoi cette formation cible la **Solutions Architect Associate (SAA-C03)**. Maintenant que vous avez vu l'ensemble des services du programme, voici ce qui compte vraiment pour préparer concrètement cet examen : la pondération réelle des domaines testés.

<img src="formations/aws-initiation-approfondissement/11-images/aws-certification-path.svg"
     alt="Parcours de certification AWS : Foundational, puis trois Associate (dont SAA-C03 ciblé par cette formation), puis Professional, puis Specialty"
     style="display:block; margin:auto; width:90%">

> [!NOTE]
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

## Ressources

### Documentation officielle AWS
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)
- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Certification](https://aws.amazon.com/certification/)
