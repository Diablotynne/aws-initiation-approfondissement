---
title: "1. RTO/RPO et Récupération de Sauvegarde"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 1. RTO/RPO et Récupération de Sauvegarde"
---

# 1. RTO/RPO et Récupération de Sauvegarde

<nav class="page-sequence"><a href="cours/chapitre-5/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/automatisation">Suivant</a></nav>

### 1.1 Définitions essentielles

Avant d'automatiser une infrastructure, il faut comprendre deux concepts critiques pour la **continuité de service** :

#### RTO (Recovery Time Objective) — Temps d'Indisponibilité Acceptable

**RTO** = **combien de temps maximum l'application peut-elle rester indisponible avant que l'impact métier devienne intolérable ?**

```text
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

```text
Exemples concrets :

Application              | RPO        | Raison
------------------------|------------|----------------------------------------
E-commerce actif         | 5 minutes  | Transactions en temps réel = critique
Logs d'application       | 1 jour     | Données historiques, non urgentes
Données de client (CRM)  | 1 heure    | Important pour relancer les clients
Backups archivés         | 30 jours   | Archive long terme, peu critique
```

#### Relation RTO ↔ RPO

```text
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

:::success
**Résultat attendu :**
```json
{
    "BackupPlanId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "BackupPlanArn": "arn:aws:backup:eu-west-3:123456789012:backup-plan:a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "CreationDate": "2026-03-24T10:15:00.000Z",
    "VersionId": "NDEzMWVlNzgtMTk2My00NjMxLWJlOTYt"
}
```
:::

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

:::success
**Résultat attendu :**
```json
# list-recovery-points-by-resource :
{
    "RecoveryPoints": [
        {
            "RecoveryPointArn": "arn:aws:backup:eu-west-3:123456789012:recovery-point:abc123",
            "CreationDate": "2026-03-24T00:05:12.000Z",
            "Status": "COMPLETED",
            "BackupSizeInBytes": 8589934592,
            "BackupVaultName": "mon-vault-production"
        }
    ]
}

# start-restore-job :
{
    "RestoreJobId": "restore-job-0abc123def456"
}
```
:::

:::warning
**Coûts AWS Backup :** la facture dépend du volume protégé, du type de stockage, des restaurations, des copies interrégions ou intercomptes et de la durée de rétention. Relevez ces paramètres dans le plan de sauvegarde, puis appliquez les tarifs officiels de la région. Vérifiez régulièrement la consommation dans **Cost Explorer** et supprimez les rétentions sans justification métier.
:::

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

:::success
**Résultat attendu :**
```json
# create-snapshot :
{
    "SnapshotId": "snap-0abc123def456789a",
    "VolumeId": "vol-12345678",
    "State": "pending",
    "StartTime": "2026-03-24T09:00:00.000Z",
    "Progress": "0%",
    "Description": "Sauvegarde avant migration"
}

# describe-snapshots (après quelques minutes) :
{
    "Snapshots": [
        {
            "SnapshotId": "snap-0abc123def456789a",
            "State": "completed",
            "Progress": "100%",
            "VolumeSize": 20
        }
    ]
}
```
:::

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

:::success
**Résultat attendu :**
```json
# create-db-snapshot :
{
    "DBSnapshot": {
        "DBSnapshotIdentifier": "prod-db-backup-2026-03-24",
        "DBInstanceIdentifier": "production-db",
        "Status": "creating",
        "Engine": "mysql",
        "AllocatedStorage": 100,
        "SnapshotCreateTime": "2026-03-24T09:30:00.000Z"
    }
}

# describe-db-snapshots (après quelques minutes) :
{
    "DBSnapshots": [
        {
            "DBSnapshotIdentifier": "prod-db-backup-2026-03-24",
            "Status": "available",
            "PercentProgress": 100
        }
    ]
}
```
:::

> **Résultat attendu :** `create-db-snapshot` retourne un JSON avec le `DBSnapshotIdentifier` et le statut `creating`. Quelques minutes plus tard, `describe-db-snapshots` affiche le statut `available`. La restauration (`restore-db-instance-from-db-snapshot`) crée une **nouvelle instance** RDS — pas un remplacement de l'existante.

---

<nav class="page-sequence"><a href="cours/chapitre-5/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/automatisation">Suivant</a></nav>
