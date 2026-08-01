---
title: "1. RTO/RPO et Récupération de Sauvegarde"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 1. RTO/RPO et Récupération de Sauvegarde"
---

<nav class="page-sequence"><a href="cours/chapitre-5/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/automatisation">Suivant</a></nav>

### 1.1 Définitions essentielles

Avant d'automatiser une infrastructure, il faut comprendre deux concepts critiques pour la **continuité de service** :

#### RTO (Recovery Time Objective) — Objectif de rétablissement

Le **RTO** est la durée maximale visée pour rétablir le service après une interruption. Il ne s'agit pas d'une valeur fournie automatiquement par AWS : l'organisation la fixe à partir de l'impact métier, puis vérifie par des tests que l'architecture permet de l'atteindre.

#### RPO (Recovery Point Objective) — Objectif de point de reprise

Le **RPO** exprime la quantité maximale de données que l'organisation accepte de perdre, mesurée dans le temps. Un RPO de quatre heures signifie, par exemple, que le mécanisme de protection doit permettre de revenir à un état vieux de quatre heures au maximum.

#### Relier les objectifs aux mécanismes techniques

| Question à trancher | Conséquence d'architecture |
|---|---|
| Quel délai de rétablissement est acceptable ? | Choisir entre restauration, capacité maintenue en attente ou service actif sur plusieurs emplacements. |
| Quelle perte de données est acceptable ? | Définir la fréquence des points de reprise et, si nécessaire, une réplication synchrone ou continue. |
| Quel périmètre de panne faut-il couvrir ? | Tester la perte d'une ressource, d'une zone de disponibilité ou d'une région selon le besoin métier. |
| Comment prouver que l'objectif est atteignable ? | Exécuter régulièrement une restauration ou un basculement et mesurer le résultat. |

Un objectif ambitieux augmente généralement la capacité, l'automatisation et les tests nécessaires. Une sauvegarde non restaurée et non chronométrée ne démontre donc ni le RPO ni le RTO.

---

### 1.2 Mécanismes AWS à combiner

Les valeurs de RTO et de RPO dépendent du volume de données, de la configuration, du scénario de panne et des procédures testées. Le tableau suivant décrit donc le rôle des mécanismes, sans promettre de délai générique.

| Mécanisme | Rôle | Point d'attention |
|---|---|---|
| **Déploiement RDS Multi-AZ** | Maintenir une instance de secours synchrone et permettre un basculement géré. | C'est un mécanisme de disponibilité, pas un remplacement des sauvegardes. |
| **Sauvegardes automatiques RDS et restauration à un instant donné** | Restaurer une nouvelle base dans la fenêtre de conservation configurée. | Le temps de restauration doit être mesuré avec un volume représentatif. |
| **Snapshots EBS** | Conserver un point de reprise d'un volume. | La fréquence de création pilote le RPO ; la restauration et l'initialisation du volume influencent le RTO. |
| **AWS Backup** | Centraliser les plans, calendriers, règles de conservation et coffres de sauvegarde. | La couverture exacte des fonctions dépend du type de ressource et de la région. |
| **Versioning et réplication Amazon S3** | Conserver plusieurs versions et, si configuré, répliquer des objets vers un autre compartiment. | La réplication seule ne protège pas de toutes les suppressions ou erreurs logiques ; les règles doivent être testées. |
| **Architecture pilot light, warm standby ou active/active** | Maintenir plus ou moins de capacité prête à reprendre le trafic. | Plus la reprise doit être rapide, plus le coût et la complexité opérationnelle augmentent. |

---

### 1.3 AWS Backup — Service Centralisé de Sauvegarde

**AWS Backup** est un **service managé** pour centraliser et automatiser les sauvegardes de ressources AWS.

#### Ressources sauvegardables par AWS Backup

```text
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

> [!tip]
> **Résultat attendu :**
> ```json
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

> [!tip]
> **Résultat attendu :**
> ```json
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


> [!warning]
> **Coûts AWS Backup :** la facture dépend du volume protégé, du type de stockage, des restaurations, des copies interrégions ou intercomptes et de la durée de rétention. Relevez ces paramètres dans le plan de sauvegarde, puis appliquez les tarifs officiels de la région. Vérifiez régulièrement la consommation dans **Cost Explorer** et supprimez les rétentions sans justification métier.


#### Bonnes pratiques AWS Backup

```text
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

> [!tip]
> **Résultat attendu :**
> ```json
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

> [!tip]
> **Résultat attendu :**
> ```json
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

<nav class="page-sequence"><a href="cours/chapitre-5/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/automatisation">Suivant</a></nav>
