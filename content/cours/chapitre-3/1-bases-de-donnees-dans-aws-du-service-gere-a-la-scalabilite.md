---
title: "1. Bases de données dans AWS — Du service géré à la scalabilité"
description: "\"Chapitre 3 — Amazon VPC et bases de données AWS\" - 1. Bases de données dans AWS — Du service géré à la scalabilité"
---

<nav class="page-sequence"><a href="cours/chapitre-3/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/2-amazon-vpc-concevoir-un-reseau-prive-securise">Suivant</a></nav>

### 1.1 La révolution des bases managées

📹 **Vidéo** : [AWS Database Services Overview](https://www.youtube.com/watch?v=adB--KhJ95w)

Quand on parle de **bases de données dans le Cloud**, la question centrale n'est plus « Où vais-je installer un serveur ? » mais plutôt « Quel type de données vais-je stocker, et quel accès dois-je offrir ? ».

Dans un environnement traditionnel (**on-premise**), administrer une base de données impliquait :

- **Installation manuelle** du moteur (MySQL, PostgreSQL, Oracle...)
- **Configuration** des paramètres (mémoire, cache, compression...)
- **Sauvegardes régulières** et tests de restauration
- **Maintenance** des patchs de sécurité
- **Réplication** pour la haute disponibilité
- **Surveillance** 24h/24 (CPU, mémoire, disque, connexions)
- **Escalade** : augmenter la taille du disque, du CPU, de la mémoire
- **Gestion des droits** d'accès pour chaque application

Autant de tâches qui détournent les équipes de leur **valeur métier réelle** : créer des applications performantes et sécurisées.

Avec **Amazon RDS** et les services managés AWS, ce paradigme s'inverse : **AWS administre l'infrastructure, vous administrez vos données**.

#### L'abstraction du matériel

Imaginez une **maison en location** vs. une **maison en propriété** :

- **En propriété** (on-prem) : vous gérez tout — le toit, la plomberie, l'électricité, les réparations. Si la toiture fuit, c'est à vos frais et vos équipes doivent intervenir.
- **En location** (RDS) : le propriétaire assure la structure, le toit, les murs. Vous ne vous occupez que du décor intérieur (vos données).

Avec RDS :
- Vous définissez simplement : quel moteur ? (MySQL, PostgreSQL, Oracle, SQL Server, Aurora) — quelle taille ? (t3.small, m5.xlarge...) — combien de stockage ? (100 Go, 5 To...)
- AWS crée l'instance, configure le stockage EBS, met en place le monitoring, gère les snapshots, applique les patches.
- Vous accédez à votre base via un endpoint standard (`mondb.xxxxx.eu-west-1.rds.amazonaws.com:3306`).

Ce changement de paradigme se concrétise par un catalogue de moteurs relationnels managés, détaillé dans la section suivante.

---

### 1.2 Amazon RDS — Bases relationnelles managées

![](assets/schemas/ch4-capture-01-92e4a28c.png)

**Amazon RDS (Relational Database Service)** est le service managé AWS pour les **bases de données structurées** (SQL). Il supporte plusieurs moteurs :

| Moteur | Compatibilité | Cas d'usage |
|--------|--------------|-----------|
| **MySQL** | Open source, très répandu | Applications web classiques |
| **PostgreSQL** | Open source, très avancé | Applications critiques, PostGIS, données complexes |
| **MariaDB** | Fork MySQL, meilleure performance | Alternative MySQL |
| **Oracle** | Propriétaire, très coûteux en on-prem | Migrations legacy, applications critiques |
| **SQL Server** | Windows, intégration Active Directory | Environnements Microsoft |
| **Amazon Aurora** | Natif AWS, ultra-performant | Haute disponibilité, haute scalabilité |

Le choix du moteur ne change rien à l'expérience de gestion : quel que soit le moteur retenu, RDS apporte le même socle de fonctionnalités managées, détaillé ci-dessous.

#### Caractéristiques clés

Quel que soit le moteur choisi, RDS apporte un socle commun de fonctionnalités managées qui le distingue d'une base installée manuellement sur un serveur :

| Fonctionnalité | Description |
|---|---|
| **Haute disponibilité** | Multi-AZ : réplication synchrone sur une autre zone de disponibilité avec basculement automatique |
| **Sauvegardes automatisées** | Sauvegardes quotidiennes, conservées jusqu'à 35 jours, restauration à un instant T |
| **Sécurité intégrée** | Chiffrement au repos (AWS KMS) et en transit (SSL/TLS), isolation réseau (VPC, Security Groups), audit (CloudTrail) |
| **Scalabilité verticale** | Augmenter CPU/RAM sans interruption (dans certains cas) |
| **Read Replicas** | Jusqu'à 5 réplicas de lecture asynchrones pour répartir les lectures |
| **Maintenance automatisée** | Patches et mises à jour sans intervention manuelle |

La haute disponibilité (Multi-AZ) et les Read Replicas sont souvent confondues alors qu'elles répondent à des besoins différents : la première protège contre une panne, la seconde répartit la charge de lecture — les sections suivantes détaillent chacune.

#### Types de stockage EBS pour RDS

RDS s'appuie sur **Amazon EBS** pour son stockage sous-jacent. Vous choisissez le type de disque selon vos besoins :

| Type | Performance | Coût | Cas d'usage |
|------|---|---|---|
| **General Purpose (gp3)** | 3 000 à 16 000 IOPS | Modéré | Production standards, dev/test |
| **Provisioned IOPS (io1/io2)** | Jusqu'à 64 000 IOPS | Élevé | Bases critiques à fort volume transactionnel |
| **Magnetic (standard)** | 100-200 IOPS | Bas | Archivage, données froides (déprécié) |

**IOPS (Input/Output Operations Per Second)** = nombre d'opérations de lecture/écriture par seconde. Un IOPS élevé garantit des temps de réponse courts pour les transactions.

#### Exemple concret — Plateforme de vidéo à la demande (VOD)

Supposons une plateforme qui stocke **les métadonnées** de ses contenus : titres, descriptions, dates de diffusion, catégories, droits d'accès. Les **requêtes sont complexes** (jointures sur plusieurs tables), les données sont structurées, et la **cohérence** est critique.

**Avec on-prem** (avant cloud) :
```
- Installation MySQL manuelle (4h)
- Scripts de sauvegarde réseau + test restauration (8h)
- Monitoring 24h/24 avec alertes (contrat SLA)
- Augmentation disque lors des pics → risque de downtime
- Réplication vers DataCenter secondaire (investissement)
→ Coût total 5 ans : ~100 000 €, équipe IT 2 personnes
```

**Avec RDS AWS** :
```
- Déploiement en 3 minutes via console AWS
- Multi-AZ automatique (zéro downtime en panne)
- Snapshots automatiques (jusqu'à 35 jours)
- Augmentation CPU/stockage sans interruption
- Read Replicas pour diffuser les lectures (reports analytiques)
→ Coût annuel : ~1 500 $, gestion minimal (0,1 FTE)
```

Le gain ne se limite pas au coût : c'est surtout la disponibilité qui change de nature, passant d'un risque géré manuellement à un mécanisme automatique intégré au service, comme détaillé ci-dessous.

---

### 1.3 Haute disponibilité et résilience dans RDS

#### Multi-AZ (Availability Zones) — Résilience automatique

**Multi-AZ** signifie que votre base est automatiquement répliquée **de manière synchrone** sur une autre zone de disponibilité (AZ) de la **même région**.

**Avant failover** : `AZ 1a` héberge le RDS Primary (Master), qui accepte lectures et écritures sur `mysql.xxxxx.rds.amazonaws.com`. Il réplique de façon synchrone vers un RDS Standby en lecture seule dans `AZ 1b`, sur un stockage EBS séparé.

**Pendant le failover** (~30 secondes), une fois la panne d'`AZ 1a` détectée :
1. CloudWatch déclenche une alarme de détection de panne.
2. Route 53 bascule le DNS : l'endpoint redirige vers le Standby.
3. Le Standby est promu Master dans `AZ 1b`.
4. Les nouvelles écritures sont acceptées en `AZ 1b`.
5. Perte de données = 0 (réplication synchrone).
6. Downtime réseau ≈ 30-60 secondes (le temps de la reconnexion).

**Après failover** : `AZ 1a` est en panne. `AZ 1b` héberge désormais le RDS Primary (ex-Standby), qui accepte lectures et écritures sur le **même endpoint** `mysql.xxxxx.rds.amazonaws.com` — aucune reconfiguration applicative n'est nécessaire. Un nouveau Standby est recréé et répliqué depuis ce nouveau Primary, ce qui résorbe la panne en environ 5 minutes.

**Bénéfice** : zéro downtime applicatif (reconnexion auto après failover), zéro perte données.
**Coût** : +50 % sur la facture RDS.
**À savoir** : L'endpoint DNS ne change pas → applications se reconnectent automatiquement.

> [!warning]
> **Coût Multi-AZ RDS — Attention au budget**
>
> Activer Multi-AZ **double le coût de l'instance** RDS : AWS provisionne silencieusement une instance Standby dans une autre AZ. Cette instance n'est pas accessible en lecture (contrairement aux Read Replicas) — elle sert uniquement au failover automatique.
>
> - db.t3.micro RDS MySQL : ~12 $/mois → **~24 $/mois** avec Multi-AZ
> - db.r6g.large RDS MySQL : ~175 $/mois → **~350 $/mois** avec Multi-AZ
>
> **Règle** : activez Multi-AZ uniquement en production. Pour dev/test, une instance simple suffit.

#### Read Replicas — Répartition de la charge de lecture

Un **Read Replica** est une **copie asynchrone** de votre base, destinée à répartir les **lectures** (SELECT) sans surcharger la Primary.

Cas d'usage : Reporting, Analytics, Exports.

Le RDS Primary (`eu-west-1a`) gère les écritures OLTP (500 SELECT/s + 100 INSERT/s) sur son endpoint `mysql.xxxx.rds...`. Il réplique de façon asynchrone (~100ms, avec un lag possible de quelques secondes) vers trois Read Replicas en lecture seule, chacun dédié à un usage distinct :

| Replica | Zone | Charge | Usage |
|---|---|---|---|
| Read Replica 1 | AZ 1b | 1000 GET/s | Reporting DB |
| Read Replica 2 | AZ 1c | 1000 GET/s | Analytics DB |
| Read Replica 3 | Region 2 | 1000 GET/s | Disaster recovery |

**Limite** : jusqu'à **5 Read Replicas** par base MySQL/PostgreSQL (15 sur Aurora).

**Différence clé Read Replica vs Multi-AZ** :
- **Multi-AZ** : synchrone, failover automatique, latence identique
- **Read Replica** : asynchrone, pas failover, peut avoir lag de 1-30s, endpoint séparé

**Commande AWS CLI** :
```bash
# Créer un Read Replica pour reportings
aws rds create-db-instance-read-replica \
  --db-instance-identifier formation-db-analytics \
  --source-db-instance-identifier formation-db \
  --db-instance-class db.t3.small \
  --availability-zone eu-west-1b

# Créer un Replica inter-région (DR)
aws rds create-db-instance-read-replica \
  --db-instance-identifier formation-db-dr \
  --source-db-instance-identifier formation-db \
  --source-region eu-west-1 \
  --region us-east-1 \
  --db-instance-class db.t3.small
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>     "DBInstance": {
>         "DBInstanceIdentifier": "formation-db-analytics",
>         "DBInstanceClass": "db.t3.small",
>         "Engine": "mysql",
>         "DBInstanceStatus": "creating",
>         "ReadReplicaSourceDBInstanceIdentifier": "formation-db",
>         "AvailabilityZone": "eu-west-1b",
>         "MultiAZ": false,
>         "StorageType": "gp3",
>         "Endpoint": {
>             "Address": "formation-db-analytics.abc123.eu-west-1.rds.amazonaws.com",
>             "Port": 3306
>         }
>     }
> }
> ```

---

### 1.4 Sécurité et supervision RDS

Une base de données RDS contient généralement des données sensibles (informations clients, identifiants, données métier) : elle doit donc être protégée par du chiffrement et surveillée en continu pour détecter toute anomalie ou tentative d'accès non autorisé. RDS propose deux volets complémentaires : le **chiffrement** des données elles-mêmes, et la **supervision** de l'activité et des performances de l'instance.

#### Chiffrement

RDS distingue deux états dans lesquels une donnée doit être protégée : au repos (stockée sur disque) et en transit (lors de son transfert entre le client et la base).

| Type | Description |
|------|---|
| **Au repos** | Les données sur disque EBS sont chiffrées via AWS KMS |
| **En transit** | SSL/TLS obligatoire entre client et base |

Le chiffrement au repos s'appuie sur **AWS KMS** (Key Management Service) : la clé chiffre le volume de stockage, les instantanés (snapshots) et les réplicas, de façon totalement transparente pour les applications. Le chiffrement en transit, lui, impose l'usage de SSL/TLS pour toute connexion au moteur de base de données, ce qui protège les données contre l'interception sur le réseau.

⚠️ **Important** : le chiffrement **doit être activé à la création** de l'instance. Il est impossible de chiffrer après coup une instance RDS existante : la seule solution consiste à créer un instantané, puis à restaurer ce dernier vers une nouvelle instance chiffrée.

#### Supervision et alertes

Une fois l'instance en production, plusieurs outils AWS permettent de suivre son état de santé, ses performances et les accès qui y sont effectués. Chacun couvre un besoin différent : métriques d'infrastructure, analyse des requêtes, audit de configuration ou détail au niveau du système d'exploitation.

| Outil | Rôle |
|---|---|
| **CloudWatch** | Métriques de base (CPU, RAM, connexions, IOPS) |
| **Performance Insights** | Requêtes coûteuses, sessions actives |
| **CloudTrail** | Audit : qui a modifié la configuration |
| **Enhanced Monitoring** | Metrics granulaires de l'OS |

En pratique, **CloudWatch** suffit pour une supervision de base (alertes sur le CPU ou l'espace disque), tandis que **Performance Insights** devient indispensable dès qu'un problème de lenteur applicative nécessite d'identifier les requêtes SQL les plus coûteuses. **CloudTrail** répond à un besoin de traçabilité et de conformité (qui a modifié quoi et quand), et **Enhanced Monitoring** apporte une vue fine des ressources de l'OS sous-jacent (jusqu'à la seconde), utile pour du diagnostic système avancé.

---

### 1.5 Amazon Aurora — Performance et résilience supérieures

**Amazon Aurora** est le **moteur de base de données propriétaire AWS**, conçu d'emblée pour le Cloud. Compatible avec **MySQL** et **PostgreSQL** mais offre des performances bien supérieures.

#### Architecture distribuée

Aurora découple le **calcul** (instances) du **stockage** (volume distribué). Les écritures sont répliquées **4 fois** sur 3 AZ.

#### Performances

Ce découplage calcul/stockage se traduit par des gains mesurables par rapport aux moteurs open source qu'Aurora reste compatible avec :

| Métrique | vs MySQL | vs PostgreSQL |
|----------|----------|---|
| Throughput max | **5×** | **3×** |
| Basculement automatique | 30 sec | 30 sec |
| Réplicas de lecture | 15 (vs 5) | 15 (vs 5) |

Ces chiffres ne sont pas de simples arguments marketing : ils découlent directement de l'architecture distribuée décrite plus haut, qui permet à Aurora de répliquer les écritures et de servir davantage de réplicas sans passer par le stockage EBS classique.

#### Comparaison détaillée : RDS MySQL/PostgreSQL vs Aurora

Pour choisir entre les deux moteurs en connaissance de cause, voici une comparaison exhaustive critère par critère :

| Aspect | RDS MySQL/PostgreSQL | Aurora |
|--------|---|---|
| **Architecture** | Stockage EBS couplé à l'instance | Calcul découplé du stockage distribué |
| **Réplication** | Synchrone (Multi-AZ) / Asynchrone (Replicas) | 4 copies sur 3 AZ (natif) |
| **Failover automatique** | 30 sec si Multi-AZ activé | 30 sec (inclus, pas surcoût) |
| **Réplicas de lecture** | 5 max | 15 max |
| **Coût Multi-AZ** | +50 % sur l'instance | 0 € (inclus) |
| **Scaling en écriture** | Impossible (Master unique) | Impossible (Master unique) |
| **Scaling en lecture** | Avec Replicas (asynchrones) | Avec Replicas (synchrones, <1ms latence) |
| **Serverless** | Non (ne s'adapte pas) | Oui (Aurora Serverless v2) |
| **Performance requêtes complexes** | Bonne | Excellente (optimisation native AWS) |
| **Coût stockage** | Paiement par Go allocué | Paiement à l'utilisation (auto-scaling) |
| **Certification AWS** | Sujet SAA-C03 | Sujet SAA-C03 (fortement recommandé) |

**SAA-C03** est le code de l'examen AWS Certified Solutions Architect – Associate, la certification détaillée au Chapitre 1 (section 1.3) et reprise au Chapitre 5 (section 9) — le comparatif RDS/Aurora ci-dessus fait partie des sujets fréquemment évalués dans cet examen.

#### Mode Serverless et Provisioned

Aurora offre **deux modèles de déploiement** :

| Modèle | Principe | Avantages | Cas d'usage |
|---|---|---|---|
| **Provisioned** (classique) | Vous choisissez la taille (ex. `db.r6g.xlarge`) | Coût fixe, performance prévisible | Application de reporting critique, charge stable (12h/jour) |
| **Serverless V2** (moderne) | Scaling auto de 0.5 à 1000+ ACU (Aurora Compute Units) | Paiement à l'utilisation, scaling en ≈5 sec | API imprévisible, pics aléatoires, environnements de développement |

Le choix entre les deux n'est pas figé : une même charge peut démarrer en Serverless V2 pendant sa phase de croissance imprévisible, puis basculer en Provisioned une fois le trafic stabilisé et prévisible.

#### Créer un cluster Aurora en CLI

> [!info]
> Une activité pratique permet d’approfondir le déploiement d’un cluster Aurora.

Cette séquence crée un cluster Aurora MySQL complet avec une instance writer, une instance reader, du chiffrement activé et une fenêtre de sauvegarde automatique. Aurora est un cluster — pas une instance unique — ce qui explique les deux commandes distinctes (cluster + instance).

```bash
# 1. Créer un Aurora MySQL Cluster (2 instances : writer + reader)
aws rds create-db-cluster \
  --db-cluster-identifier formation-aurora-cluster \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.02.0 \
  --master-username admin \
  --master-user-password 'SecurePassword123!' \
  --database-name appdb \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group \
  --storage-encrypted \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "mon:04:00-mon:05:00" \
  --tag-specifications 'ResourceType=cluster,Tags=[{Key=Name,Value=FormationAuroraCluster}]'

# 2. Créer les instances Aurora (Writer)
aws rds create-db-instance \
  --db-instance-identifier formation-aurora-writer \
  --db-instance-class db.r6g.large \
  --engine aurora-mysql \
  --db-cluster-identifier formation-aurora-cluster \
  --publicly-accessible false

# 3. Créer instance Reader (lecture seulement)
aws rds create-db-instance \
  --db-instance-identifier formation-aurora-reader \
  --db-instance-class db.r6g.large \
  --engine aurora-mysql \
  --db-cluster-identifier formation-aurora-cluster \
  --promotion-tier 2 \
  --publicly-accessible false

# 4. Créer Aurora Serverless V2 (auto-scaling)
aws rds create-db-cluster \
  --db-cluster-identifier formation-aurora-serverless \
  --engine aurora-postgresql \
  --engine-version 14.6 \
  --engine-mode provisioned \
  --serverlessv2-scaling-configuration 'MinCapacity=0.5,MaxCapacity=16' \
  --master-username admin \
  --master-user-password 'SecurePassword123!' \
  --database-name appdb \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group

# 5. Attendre disponibilité
aws rds wait db-cluster-available \
  --db-cluster-identifier formation-aurora-cluster

# 6. Récupérer les endpoints
aws rds describe-db-clusters \
  --db-cluster-identifier formation-aurora-cluster \
  --query 'DBClusters[0].[DBClusterEndpoint,ReaderEndpoint]'
# Résultat :
# Writer : formation-aurora-cluster.xxxx.eu-west-1.rds.amazonaws.com (écritures)
# Reader : formation-aurora-cluster-ro.xxxx.eu-west-1.rds.amazonaws.com (lectures)

# 7. Modifier le scaling Serverless (augmenter capacité max)
aws rds modify-db-cluster \
  --db-cluster-identifier formation-aurora-serverless \
  --serverlessv2-scaling-configuration 'MinCapacity=1,MaxCapacity=32' \
  --apply-immediately

# 8. Ajouter un Read Replica dans une autre région
aws rds create-db-cluster \
  --db-cluster-identifier formation-aurora-dr \
  --engine aurora-mysql \
  --source-region eu-west-1 \
  --region us-east-1 \
  --replication-source-identifier arn:aws:rds:eu-west-1:123456789012:cluster:formation-aurora-cluster
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>     "DBCluster": {
>         "DBClusterIdentifier": "formation-aurora-cluster",
>         "Status": "creating",
>         "Engine": "aurora-mysql",
>         "EngineVersion": "8.0.mysql_aurora.3.02.0",
>         "DBClusterEndpoint": "formation-aurora-cluster.cluster-abc123.eu-west-1.rds.amazonaws.com",
>         "ReaderEndpoint": "formation-aurora-cluster.cluster-ro-abc123.eu-west-1.rds.amazonaws.com",
>         "MultiAZ": true,
>         "StorageEncrypted": true,
>         "BackupRetentionPeriod": 7
>     }
> }
> ```

> **Résultat attendu :** `create-db-cluster` retourne le JSON du cluster (état `creating`). `rds wait db-cluster-available` bloque jusqu'à ce que le cluster soit prêt (peut prendre 5-10 min). `describe-db-clusters` affiche les endpoints writer et reader — deux URLs distinctes.

**À retenir** : Aurora est **le choix standard** pour les productions critiques AWS — meilleur ROI que RDS classique quand on inclut les coûts Multi-AZ.


#### Comparatif de coûts réel — RDS vs Aurora (région Paris, eu-west-3)

> Tarifs indicatifs au 1er avril 2026 — à vérifier sur [https://aws.amazon.com/rds/pricing/](https://aws.amazon.com/rds/pricing/)

##### Instance (calcul)

Premier poste de coût à comparer : le prix horaire de l'instance elle-même, qui varie sensiblement selon la taille choisie :

| Type | RDS MySQL | Aurora MySQL | Écart |
|------|-----------|-------------|-------|
| db.t3.micro | **0,017 $/h** (~12 $/mois) | ❌ Non disponible | — |
| db.t3.small | 0,034 $/h (~25 $/mois) | ❌ Non disponible | — |
| db.t3.medium | 0,068 $/h (~49 $/mois) | **0,073 $/h** (~53 $/mois) | +7 % |
| db.r6g.large | 0,240 $/h (~175 $/mois) | 0,260 $/h (~190 $/mois) | +8 % |
| db.r6g.2xlarge | 0,960 $/h (~700 $/mois) | 1,040 $/h (~760 $/mois) | +8 % |

> ⚠️ **Aurora ne propose pas de t3.micro ou t3.small.** L'entrée de gamme est le t3.medium (~53 $/mois). Pour un usage de formation ou de très petite charge, RDS est **nettement moins cher**.

##### Stockage et I/O

Second poste de coût, souvent négligé lors d'un premier chiffrage : le stockage et les I/O, où Aurora inverse la tendance observée sur le calcul en devenant moins cher au Go, réplication comprise :

| | RDS (gp3) | Aurora |
|--|-----------|--------|
| Prix stockage | 0,115 $/Go/mois | 0,10 $/Go/mois |
| Minimum | 20 Go (provisionné) | 10 Go (auto-extensible) |
| Maximum | 64 To | 128 To |
| Réplication | 1 copie (Multi-AZ = 2×) | **6 copies dans 3 AZ** (inclus) |
| I/O | Inclus (gp3) | 0,20 $ / million de requêtes (Standard) ou inclus (I/O-Optimized +25%) |

À volume équivalent, Aurora reste donc plus cher sur le calcul mais moins cher sur le stockage — le choix final dépend du ratio entre les deux pour votre charge de travail, ce que l'exemple chiffré ci-dessous permet d'illustrer.

##### Aurora Serverless v2 — facturation à l'utilisation

Contrairement au mode Provisioned facturé à l'heure d'instance, Serverless v2 facture la capacité réellement consommée, exprimée en ACU (Aurora Capacity Unit) :

```
Facturation : ACU-heure (Aurora Capacity Unit)
1 ACU = ~2 Go de RAM + CPU proportionnel

Tarif : 0,12 $ / ACU-heure (Paris)
Minimum : 0,5 ACU  →  0,06 $/h  →  ~43 $/mois au repos
Maximum : 128 ACU  →  15,36 $/h

Avantage : scale automatiquement de 0,5 à 128 ACU en quelques secondes
Cas idéal : applications avec trafic très variable (pics journaliers, saisonnalité)
```

Ces tarifs unitaires restent abstraits tant qu'on ne les applique pas à un cas concret comparé côte à côte avec RDS classique.

##### Exemple comparatif — Application web standard, 50 Go de données

Pour donner un sens concret à ces tarifs unitaires, voici comment ils se traduisent sur un cas réel comparé côte à côte avec RDS classique :

```
                    RDS MySQL           Aurora MySQL (Serverless v2)
Instance        db.t3.medium            0,5–4 ACU (charge variable)
                49 $/mois               ~65–120 $/mois

Stockage        50 Go × 0,115           50 Go × 0,10
                = 5,75 $/mois           = 5,00 $/mois

I/O             Inclus                  ~5 $/mois (applis légères)

Multi-AZ        +100 % instance         Inclus (6 copies)
                = +49 $/mois            = 0 $

TOTAL Multi-AZ  ~104 $/mois             ~75–125 $/mois
                (coût fixe, prévisible) (variable, mais HA incluse)
```

> 💡 **Conclusion** : Aurora est plus cher à l'instance mais inclut la haute disponibilité sur 6 copies. Pour une charge faible et un budget serré : **RDS**. Pour une charge critique nécessitant de la résilience et de la scalabilité : **Aurora**.

📎 [AWS RDS Pricing](https://aws.amazon.com/rds/pricing/)
📎 [AWS Aurora Pricing](https://aws.amazon.com/rds/aurora/pricing/)


---

### 1.6 Amazon DynamoDB — NoSQL à ultra-haute scalabilité

**DynamoDB** est une **base NoSQL managée** d'AWS. Elle stocke des **documents JSON** ou des **paires clé-valeur** sans schéma figé. Elle garantit des temps de réponse **en millisecondes**, même à l'échelle de millions de requêtes par seconde.

#### Structure d'une table DynamoDB

Contrairement à une table SQL, une table DynamoDB n'impose pas de colonnes fixes : seule la clé primaire est obligatoire, chaque type d'attribut étant identifié par un code court utilisé dans l'API et les requêtes :

```
Chaque attribut peut être :
- Chaîne (S)
- Nombre (N)
- Binaire (B)
- Ensemble (SS, NS, BS)
- Map (document imbriqué)
- Liste

Aucune contrainte de schéma : vous pouvez ajouter des attributs par ligne.
```

Cette absence de schéma figé permet de faire évoluer la structure des données au fil du temps sans migration, mais déplace la responsabilité de la cohérence des données depuis la base de données vers le code applicatif.

#### Modèles de tarification

DynamoDB propose deux façons de payer selon la prévisibilité de votre trafic :

| Modèle | Débit garanti | Facturation | Cas d'usage |
|--------|---|---|---|
| **Provisionné** | À définir (RCU/WCU) | Fixe + dépassement | Charge prédictible |
| **À la demande** | Illimité | Au débit réel | Charge variable |

- **RCU** (Read Capacity Unit) : 1 RCU = une lecture fortement cohérente d'un item ≤ 4 Ko
- **WCU** (Write Capacity Unit) : 1 WCU = une écriture d'un item ≤ 1 Ko

**Exemple chiffré — mode à la demande (région Paris)** : facturé 0,32 $ par million de lectures et 1,60 $ par million d'écritures. Une application avec 500 000 lectures/jour et 50 000 écritures/jour coûte environ (500 000 × 30 × 0,32 / 1 000 000) + (50 000 × 30 × 1,60 / 1 000 000) ≈ 4,80 $ + 2,40 $ = **~7,20 $/mois** rien que pour les requêtes, plus le stockage (0,25 $/Go/mois). Pour une charge stable et prévisible, le mode provisionné revient souvent moins cher — mais il facture même en l'absence de trafic, contrairement au mode à la demande.

📎 [Amazon DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)

#### DynamoDB vs RDS

Ce choix de modèle de tarification n'est qu'un aspect parmi d'autres différences structurelles entre DynamoDB et une base relationnelle comme RDS :

| Aspect | RDS (SQL) | DynamoDB (NoSQL) |
|--------|-----------|---|
| **Schéma** | Structuré, relationnel | Flexible, semi-structuré |
| **Requêtes** | Complexes (jointures) | Simples (accès par clé) |
| **Scalabilité** | Verticale surtout | Horizontale, ultra-massive |
| **Latence** | ms-s | ms |

Cette scalabilité horizontale « ultra-massive » n'est pas magique : elle repose sur un découpage des données en partitions, et un mauvais choix de clé de partition peut annuler cet avantage en concentrant tout le trafic sur une seule partition, comme détaillé ci-dessous.

> [!danger]
> **Hot Partitions DynamoDB — Erreur de conception fréquente**
>
> DynamoDB distribue vos données sur des partitions selon la **Partition Key**. Si vous choisissez une clé avec peu de valeurs distinctes (ex. : `status = "active"/"inactive"`, ou une date comme `2026-05-17`), toutes les requêtes frappent la **même partition** → throttling, latence explosive.
>
> **Symptômes** : erreurs `ProvisionedThroughputExceededException`, latences P99 > 500ms.
>
> **Solutions** :
> - Choisir une Partition Key à **haute cardinalité** (UUID utilisateur, ID produit unique)
> - Ajouter un **suffixe aléatoire** (write sharding) : `user_id#1`, `user_id#2`
> - Utiliser un **Global Secondary Index** avec une clé mieux distribuée
> - Passer en mode **On-Demand** pour absorber les pics sans throttling

---

### 1.7 Migration de bases de données avec AWS DMS

**AWS DMS (Database Migration Service)** permet la **migration sans interruption** d'une base existante vers AWS. C'est un service managé qui gère le transport, la transformation et la synchronisation de données.

#### Scénario réaliste de migration

Imaginez une **entreprise audiovisuelle** avec une **base Oracle on-premise** stockant les **métadonnées de contenus** (titres, droits, calendriers). Actuellement :
- **5 To de données** (10 ans d'historique)
- **Uptime critique** : 24h/24
- **Migration programmée** : 1 week-end

**Sans DMS** : arrêt services, export/import manuel, validation longue, risque perte données.
**Avec DMS** : continuité service, réplication live, bascule maîtrisée.

#### Architecture de migration DMS

La source (Oracle Database on-premise/RDS, 5 To, 50 tables, actif à 1000 txn/s) passe par une tâche DMS (instance `dms.c6i.xlg`) qui exécute d'abord un Full Load (4-6 heures) vers la cible (Aurora/RDS PostgreSQL, 5 To), puis bascule en CDC (Change Data Capture) continu avec un lag inférieur à 1 seconde pour maintenir la cible synchronisée en temps réel.

**Timeline type** :
- J-1 : configuration DMS, full load lancé de nuit.
- J+0 : full load terminé, CDC actif, l'application reste encore on-premise.
- J+0 20h00 : validation des données, préparation du cutover.
- J+0 22h00 : cutover — redirection de l'application vers Aurora.
- J+1 : validation complète, monitoring 24h.

Cette timeline repose sur trois phases techniques distinctes, détaillées ci-dessous.

#### Processus de migration par étapes

**Phase 1 — Full Load (copie complète)** : les données transitent de la source vers la cible via un DMS Agent. Tous les types de données, tous les indices et les contraintes PRIMARY sont copiés automatiquement ; en revanche, les triggers et procédures stockées doivent être recréés manuellement.

**Phase 2 — CDC (Change Data Capture)** : DMS lit les logs natifs de la source (redo logs pour Oracle, binary logs pour MySQL, WAL pour PostgreSQL) et les applique sur la cible en quasi temps réel (latence sous la milliseconde).

**Phase 3 — Cutover (basculement applicatif)** :
1. Arrêter l'application (2-5 min).
2. Valider que le lag de réplication est proche de 0.
3. Rediriger les connexions vers la cible.
4. Vérifier les logs applicatifs.
5. Garder un plan de rollback armé.

La difficulté de ce cutover varie fortement selon le type de migration entrepris, comme le montre le tableau suivant.

#### Types de migrations DMS

La complexité d'une migration DMS dépend surtout de l'écart entre le moteur source et le moteur cible :

| Type | Exemple | Complexité | Coût |
|------|---------|---|---|
| **Homogène (même moteur)** | MySQL → RDS MySQL | ⭐ Très facile | Bas |
| **Hétérogène (moteurs diff)** | Oracle → Aurora PostgreSQL | ⭐⭐⭐ Moyen | Moyen |
| **Schéma complexe** | DB2 → PostgreSQL (types custom) | ⭐⭐⭐⭐ Élevé | Élevé |

L'exemple Oracle → Aurora PostgreSQL de ce chapitre se situe dans la catégorie hétérogène : DMS gère la conversion des types de données courants, mais les procédures stockées et triggers spécifiques à Oracle doivent être portés manuellement en PL/pgSQL.

#### Créer une tâche DMS en CLI

> [!info]
> Une activité pratique permet d’approfondir AWS Database Migration Service.

DMS fonctionne en 3 objets : un **endpoint source** (base existante), un **endpoint cible** (base AWS), et une **instance de réplication** (le moteur qui exécute la migration). On les crée dans cet ordre, puis on démarre la tâche.

```bash
# ═══════════════════════════════════════════════════════════
# 1. Créer les Endpoints source et cible
# ═══════════════════════════════════════════════════════════

# Endpoint SOURCE (Base existante on-prem/RDS)
aws dms create-endpoint \
  --endpoint-identifier oracle-source \
  --endpoint-type source \
  --engine-name oracle \
  --server-name oracle.company.local \
  --port 1521 \
  --database-name PRODDB \
  --username migration_user \
  --password 'SourcePassword123!' \
  --ssl-mode require \
  --extra-connection-attributes 'trustServerCertificate=false'

# Endpoint TARGET (Aurora PostgreSQL)
aws dms create-endpoint \
  --endpoint-identifier aurora-target \
  --endpoint-type target \
  --engine-name aurora-postgresql \
  --server-name formation-aurora.xxxxx.eu-west-1.rds.amazonaws.com \
  --port 5432 \
  --database-name appdb \
  --username migration_user \
  --password 'TargetPassword123!' \
  --ssl-mode require

# ═══════════════════════════════════════════════════════════
# 2. Créer une instance DMS (compute qui exécute la migration)
# ═══════════════════════════════════════════════════════════

aws dms create-replication-instance \
  --replication-instance-identifier formation-dms-instance \
  --replication-instance-class dms.c6i.xlarge \
  --allocated-storage 200 \
  --vpc-security-group-ids sg-12345678 \
  --replication-subnet-group-identifier my-dms-subnet-group \
  --multi-az \
  --engine-version 3.4.7 \
  --publicly-accessible false \
  --tag-specifications 'ResourceType=rep,Tags=[{Key=Name,Value=FormationDMS}]'

# Attendre disponibilité (5-10 min)
aws dms wait replication-instance-available \
  --filters 'Name=replication-instance-id,Values=formation-dms-instance'

# ═══════════════════════════════════════════════════════════
# 3. Valider connexions (test avant migration)
# ═══════════════════════════════════════════════════════════

# Tester accès source
aws dms test-connection \
  --replication-instance-arn arn:aws:dms:eu-west-1:123456789012:rep:formation-dms-instance \
  --endpoint-arn arn:aws:dms:eu-west-1:123456789012:ep:oracle-source

# Tester accès cible
aws dms test-connection \
  --replication-instance-arn arn:aws:dms:eu-west-1:123456789012:rep:formation-dms-instance \
  --endpoint-arn arn:aws:dms:eu-west-1:123456789012:ep:aurora-target

# ═══════════════════════════════════════════════════════════
# 4. Créer tâche DMS (FULL LOAD + CDC)
# ═══════════════════════════════════════════════════════════

aws dms create-replication-task \
  --replication-task-identifier oracle-to-aurora-migration \
  --source-endpoint-arn arn:aws:dms:eu-west-1:123456789012:ep:oracle-source \
  --target-endpoint-arn arn:aws:dms:eu-west-1:123456789012:ep:aurora-target \
  --replication-instance-arn arn:aws:dms:eu-west-1:123456789012:rep:formation-dms-instance \
  --migration-type cdc \
  --table-mappings '{
    "rules": [
      {
        "rule-type": "selection",
        "rule-name": "include-all-tables",
        "object-locator": {
          "schema-name": "%",
          "table-name": "%"
        },
        "rule-action": "include"
      }
    ]
  }' \
  --replication-task-settings '{
    "TargetMetadata": {
      "TargetSchema": "",
      "SupportsCascadeDelete": true,
      "FullLobMode": false,
      "LobChunkSize": 64,
      "LobMaxSize": 32
    },
    "FullLoadSettings": {
      "TargetSchema": "",
      "CreatePkAfterFullLoad": false,
      "StopTaskCachedChangesNotApplied": false,
      "StopTaskCachedChangesApplied": false,
      "MaxFullLoadSubTasks": 8,
      "TransactionConsistencyTimeout": 600,
      "CommitRate": 50000
    },
    "Logging": {
      "EnableLogging": true,
      "LogComponents": [
        {
          "Id": "SOURCE_UNLOAD",
          "Severity": "LOGGER_SEVERITY_DEFAULT"
        }
      ]
    },
    "ChangeProcessingDdlHandlingPolicy": {
      "HandleSourceTableDropped": true,
      "HandleSourceTableTruncated": true,
      "HandleSourceTableAltered": true
    }
  }' \
  --tags Key=Project,Value=Migration Key=Type,Value=Oracle2Aurora

# ═══════════════════════════════════════════════════════════
# 5. Attendre tâche et monitorer statut
# ═══════════════════════════════════════════════════════════

# Status : creating → modifying → ready → running → stopped
aws dms describe-replication-tasks \
  --filters 'Name=replication-task-arn,Values=arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration'
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>     "ReplicationTasks": [{
>         "ReplicationTaskIdentifier": "oracle-to-aurora-migration",
>         "Status": "running",
>         "MigrationType": "cdc",
>         "ReplicationTaskStats": {
>             "FullLoadProgressPercent": 100,
>             "ElapsedTimeMillis": 18000000,
>             "TablesLoaded": 50,
>             "TablesQueued": 0,
>             "TablesErrored": 0,
>             "TablesLoading": 0
>         }
>     }]
> }
> ```

```bash
# Voir les tables migrées (status, nb rows)
aws dms describe-table-statistics \
  --replication-task-arn arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration \
  --query 'TableStatistics[*].[SchemaName,TableName,FullLoadRows,FullLoadErrorRows,Updates,Inserts,Deletes]'
```

> [!tip]
> **Résultat attendu :**
> ```json
> [
>     ["PRODDB", "CONTENT_META", 2500000, 0, 1250, 340, 12],
>     ["PRODDB", "RIGHTS_TABLE", 180000, 0, 45, 8, 1],
>     ["PRODDB", "CALENDAR_SLOTS", 95000, 0, 320, 120, 5],
>     ["PRODDB", "USERS", 50000, 0, 88, 15, 0]
> ]
> ```

```bash
# ═══════════════════════════════════════════════════════════
# 6. Commandes gestion tâche
# ═══════════════════════════════════════════════════════════

# Arrêter tâche (sans supprimer)
aws dms stop-replication-task \
  --replication-task-arn arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration

# Relancer tâche
aws dms start-replication-task \
  --replication-task-arn arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration \
  --start-replication-task-type resume-processing

# Redémarrer complet (FULL LOAD + CDC)
aws dms start-replication-task \
  --replication-task-arn arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration \
  --start-replication-task-type cdc

# Supprimer tâche
aws dms delete-replication-task \
  --replication-task-arn arn:aws:dms:eu-west-1:123456789012:task:oracle-to-aurora-migration
```

Ces commandes de gestion suffisent pour piloter le cycle de vie d'une tâche DMS ; en pratique, quelques erreurs reviennent régulièrement lors des migrations réelles.

#### Pièges et bonnes pratiques DMS

Voici les erreurs les plus courantes constatées lors de migrations DMS en production, et comment les éviter :

| Piège | Solution |
|-------|----------|
| **Oublier full load avant CDC** | Toujours : `migration-type cdc` (inclut full load) |
| **Schema incomplet après migration** | Procs stockées, triggers = manuels. DMS copie juste data |
| **CDC lag important** | Vérifier capacité DMS instance, réduire `MaxFullLoadSubTasks` |
| **Incompatibilités types données** | Utiliser **Schema Conversion Tool (SCT)** avant DMS pour préparer |
| **Oublier transaction logs source** | Source doit activer binary logs (MySQL) / redo logs (Oracle) |
| **Cutover sans validation données** | Test requêtes applicatives sur cible avant redirection |

Le piège le plus coûteux reste le schéma incomplet : DMS donne l'illusion d'une migration totale alors qu'il ne copie que les données, laissant la logique métier embarquée dans la base (procédures, triggers, vues) entièrement à la charge de l'équipe de migration.

---

<nav class="page-sequence"><a href="cours/chapitre-3/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/2-amazon-vpc-concevoir-un-reseau-prive-securise">Suivant</a></nav>
