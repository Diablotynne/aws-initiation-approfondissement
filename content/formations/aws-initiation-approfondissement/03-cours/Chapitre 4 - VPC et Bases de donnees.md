# Chapitre 4 — Réseaux et Bases de données — VPC, RDS, DynamoDB, Route 53

---

> [!NOTE]
> Les instances EC2 et les buckets S3 déployés au chapitre précédent doivent maintenant s'intégrer dans un réseau maîtrisé et s'appuyer sur des bases de données managées : ce chapitre couvre les deux piliers d'une architecture AWS mature, le réseau (VPC) et la donnée persistante (RDS, Aurora, DynamoDB).
>
> **Objectifs du chapitre**
>
> À l'issue de ce chapitre, les stagiaires seront capables de :
>
> - **Expliquer** l'intérêt des bases de données managées face à une base auto-administrée
> - **Déployer** une base Amazon RDS en haute disponibilité (Multi-AZ) et en sécuriser l'accès
> - **Différencier** Amazon Aurora d'une base RDS classique en termes de performance et de résilience
> - **Utiliser** Amazon DynamoDB pour un cas d'usage NoSQL à forte scalabilité
> - **Planifier** une migration de base de données avec AWS DMS
> - **Concevoir** une VPC avec subnets publics et privés, table de routage et passerelle Internet/NAT
> - **Sécuriser** le trafic réseau avec des Security Groups et des Network ACLs
> - **Interconnecter** plusieurs VPC avec le VPC Peering et AWS Transit Gateway
> - **Utiliser** un VPC Endpoint pour accéder à un service AWS sans transiter par Internet
> - **Configurer** une zone DNS et des enregistrements avec Amazon Route 53, dont des politiques de routage avancées
> - **Mettre en place** un cluster ElastiCache (Redis) pour accélérer l'accès aux données fréquemment lues
![Architecture VPC en couches avec sous-réseaux publics, privés et services de données](formations/aws-initiation-approfondissement/11-images/ch4-carte-reseau-donnees.svg)

<div class="concept-check">
<strong>Diagnostic réseau — avant de poursuivre</strong>
<p>Une instance possède une IP publique et un Security Group autorisant HTTPS, mais reste inaccessible depuis Internet. Que faut-il encore vérifier ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>La table de routage du subnet doit contenir une route vers une Internet Gateway attachée au VPC. Une adresse et une règle de sécurité ne créent pas à elles seules le chemin réseau.</p></details>
</div>

---

## 1. Bases de données dans AWS — Du service géré à la scalabilité

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

---

### 1.2 Amazon RDS — Bases relationnelles managées

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-01-92e4a28c.png)

**Amazon RDS (Relational Database Service)** est le service managé AWS pour les **bases de données structurées** (SQL). Il supporte plusieurs moteurs :

| Moteur | Compatibilité | Cas d'usage |
|--------|--------------|-----------|
| **MySQL** | Open source, très répandu | Applications web classiques |
| **PostgreSQL** | Open source, très avancé | Applications critiques, PostGIS, données complexes |
| **MariaDB** | Fork MySQL, meilleure performance | Alternative MySQL |
| **Oracle** | Propriétaire, très coûteux en on-prem | Migrations legacy, applications critiques |
| **SQL Server** | Windows, intégration Active Directory | Environnements Microsoft |
| **Amazon Aurora** | Natif AWS, ultra-performant | Haute disponibilité, haute scalabilité |

#### Caractéristiques clés

| Fonctionnalité | Description |
|---|---|
| **Haute disponibilité** | Multi-AZ : réplication synchrone sur une autre zone de disponibilité avec basculement automatique |
| **Sauvegardes automatisées** | Sauvegardes quotidiennes, conservées jusqu'à 35 jours, restauration à un instant T |
| **Sécurité intégrée** | Chiffrement au repos (AWS KMS) et en transit (SSL/TLS), isolation réseau (VPC, Security Groups), audit (CloudTrail) |
| **Scalabilité verticale** | Augmenter CPU/RAM sans interruption (dans certains cas) |
| **Read Replicas** | Jusqu'à 5 réplicas de lecture asynchrones pour répartir les lectures |
| **Maintenance automatisée** | Patches et mises à jour sans intervention manuelle |

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

> [!WARNING]
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

> [!TIP]
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

#### Chiffrement

| Type | Description |
|------|---|
| **Au repos** | Les données sur disque EBS sont chiffrées via AWS KMS |
| **En transit** | SSL/TLS obligatoire entre client et base |

⚠️ **Important** : le chiffrement **doit être activé à la création** de l'instance.

#### Supervision et alertes

| Outil | Rôle |
|---|---|
| **CloudWatch** | Métriques de base (CPU, RAM, connexions, IOPS) |
| **Performance Insights** | Requêtes coûteuses, sessions actives |
| **CloudTrail** | Audit : qui a modifié la configuration |
| **Enhanced Monitoring** | Metrics granulaires de l'OS |

---

### 1.5 Amazon Aurora — Performance et résilience supérieures

**Amazon Aurora** est le **moteur de base de données propriétaire AWS**, conçu d'emblée pour le Cloud. Compatible avec **MySQL** et **PostgreSQL** mais offre des performances bien supérieures.

#### Architecture distribuée

Aurora découple le **calcul** (instances) du **stockage** (volume distribué). Les écritures sont répliquées **4 fois** sur 3 AZ.

#### Performances

| Métrique | vs MySQL | vs PostgreSQL |
|----------|----------|---|
| Throughput max | **5×** | **3×** |
| Basculement automatique | 30 sec | 30 sec |
| Réplicas de lecture | 15 (vs 5) | 15 (vs 5) |

#### Comparaison détaillée : RDS MySQL/PostgreSQL vs Aurora

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

#### Créer un cluster Aurora en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Database) vous permettra d'approfondir le déploiement d'un cluster Aurora.
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

> [!TIP]
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

| Type | RDS MySQL | Aurora MySQL | Écart |
|------|-----------|-------------|-------|
| db.t3.micro | **0,017 $/h** (~12 $/mois) | ❌ Non disponible | — |
| db.t3.small | 0,034 $/h (~25 $/mois) | ❌ Non disponible | — |
| db.t3.medium | 0,068 $/h (~49 $/mois) | **0,073 $/h** (~53 $/mois) | +7 % |
| db.r6g.large | 0,240 $/h (~175 $/mois) | 0,260 $/h (~190 $/mois) | +8 % |
| db.r6g.2xlarge | 0,960 $/h (~700 $/mois) | 1,040 $/h (~760 $/mois) | +8 % |

> ⚠️ **Aurora ne propose pas de t3.micro ou t3.small.** L'entrée de gamme est le t3.medium (~53 $/mois). Pour un usage de formation ou de très petite charge, RDS est **nettement moins cher**.

##### Stockage et I/O

| | RDS (gp3) | Aurora |
|--|-----------|--------|
| Prix stockage | 0,115 $/Go/mois | 0,10 $/Go/mois |
| Minimum | 20 Go (provisionné) | 10 Go (auto-extensible) |
| Maximum | 64 To | 128 To |
| Réplication | 1 copie (Multi-AZ = 2×) | **6 copies dans 3 AZ** (inclus) |
| I/O | Inclus (gp3) | 0,20 $ / million de requêtes (Standard) ou inclus (I/O-Optimized +25%) |

##### Aurora Serverless v2 — facturation à l'utilisation

```
Facturation : ACU-heure (Aurora Capacity Unit)
1 ACU = ~2 Go de RAM + CPU proportionnel

Tarif : 0,12 $ / ACU-heure (Paris)
Minimum : 0,5 ACU  →  0,06 $/h  →  ~43 $/mois au repos
Maximum : 128 ACU  →  15,36 $/h

Avantage : scale automatiquement de 0,5 à 128 ACU en quelques secondes
Cas idéal : applications avec trafic très variable (pics journaliers, saisonnalité)
```

##### Exemple comparatif — Application web standard, 50 Go de données

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

#### Modèles de tarification

| Modèle | Débit garanti | Facturation | Cas d'usage |
|--------|---|---|---|
| **Provisionné** | À définir (RCU/WCU) | Fixe + dépassement | Charge prédictible |
| **À la demande** | Illimité | Au débit réel | Charge variable |

- **RCU** (Read Capacity Unit) : 1 RCU = une lecture fortement cohérente d'un item ≤ 4 Ko
- **WCU** (Write Capacity Unit) : 1 WCU = une écriture d'un item ≤ 1 Ko

**Exemple chiffré — mode à la demande (région Paris)** : facturé 0,32 $ par million de lectures et 1,60 $ par million d'écritures. Une application avec 500 000 lectures/jour et 50 000 écritures/jour coûte environ (500 000 × 30 × 0,32 / 1 000 000) + (50 000 × 30 × 1,60 / 1 000 000) ≈ 4,80 $ + 2,40 $ = **~7,20 $/mois** rien que pour les requêtes, plus le stockage (0,25 $/Go/mois). Pour une charge stable et prévisible, le mode provisionné revient souvent moins cher — mais il facture même en l'absence de trafic, contrairement au mode à la demande.

📎 [Amazon DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)

#### DynamoDB vs RDS

| Aspect | RDS (SQL) | DynamoDB (NoSQL) |
|--------|-----------|---|
| **Schéma** | Structuré, relationnel | Flexible, semi-structuré |
| **Requêtes** | Complexes (jointures) | Simples (accès par clé) |
| **Scalabilité** | Verticale surtout | Horizontale, ultra-massive |
| **Latence** | ms-s | ms |

> [!IMPORTANT]
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

#### Processus de migration par étapes

**Phase 1 — Full Load (copie complète)** : les données transitent de la source vers la cible via un DMS Agent. Tous les types de données, tous les indices et les contraintes PRIMARY sont copiés automatiquement ; en revanche, les triggers et procédures stockées doivent être recréés manuellement.

**Phase 2 — CDC (Change Data Capture)** : DMS lit les logs natifs de la source (redo logs pour Oracle, binary logs pour MySQL, WAL pour PostgreSQL) et les applique sur la cible en quasi temps réel (latence sous la milliseconde).

**Phase 3 — Cutover (basculement applicatif)** :
1. Arrêter l'application (2-5 min).
2. Valider que le lag de réplication est proche de 0.
3. Rediriger les connexions vers la cible.
4. Vérifier les logs applicatifs.
5. Garder un plan de rollback armé.

#### Types de migrations DMS

| Type | Exemple | Complexité | Coût |
|------|---------|---|---|
| **Homogène (même moteur)** | MySQL → RDS MySQL | ⭐ Très facile | Bas |
| **Hétérogène (moteurs diff)** | Oracle → Aurora PostgreSQL | ⭐⭐⭐ Moyen | Moyen |
| **Schéma complexe** | DB2 → PostgreSQL (types custom) | ⭐⭐⭐⭐ Élevé | Élevé |

#### Créer une tâche DMS en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Database, section migration) vous permettra d'approfondir DMS.
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

> [!TIP]
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

> [!TIP]
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

#### Pièges et bonnes pratiques DMS

| Piège | Solution |
|-------|----------|
| **Oublier full load avant CDC** | Toujours : `migration-type cdc` (inclut full load) |
| **Schema incomplet après migration** | Procs stockées, triggers = manuels. DMS copie juste data |
| **CDC lag important** | Vérifier capacité DMS instance, réduire `MaxFullLoadSubTasks` |
| **Incompatibilités types données** | Utiliser **Schema Conversion Tool (SCT)** avant DMS pour préparer |
| **Oublier transaction logs source** | Source doit activer binary logs (MySQL) / redo logs (Oracle) |
| **Cutover sans validation données** | Test requêtes applicatives sur cible avant redirection |

---

## 2. Amazon VPC — Concevoir un réseau privé sécurisé

### 2.1 Du réseau on-prem au réseau virtuel

#### Définition — Qu'est-ce qu'une VPC ?

> **Amazon VPC (Virtual Private Cloud)** est un **réseau privé virtuel isolé** que vous créez dans AWS. C'est votre **datacenter logique**, entièrement contrôlé, où vous déployez vos instances EC2, vos bases RDS, vos Load Balancers, etc.

Une VPC vous offre :
- **Isolation logique** : vos ressources ne sont pas visibles aux autres comptes AWS.
- **Contrôle total** : vous définissez les adresses IP, les routes, les pare-feux.
- **Flexibilité** : ajouter ou retirer des subnets, des passerelles à volonté.

---

### 2.2 Composants clés d'une VPC

#### Schéma architectural complet d'une VPC sécurisée

<img src="formations/aws-initiation-approfondissement/11-images/vpc-architecture.svg"
     alt="Architecture VPC — Haute disponibilité multi-AZ"
     style="display:block; margin:auto; width:90%">

**Flux de trafic :**
1. Internet → ALB (IGW ouvre l'accès)
2. ALB → EC2 Web (Security Group + règles subnet)
3. EC2 → RDS (Security Group DB ouvre port 3306)
4. EC2 → Internet (via NAT Gateway, pour updates)

---

#### 1. CIDR Block (Classless Inter-Domain Routing)

Une VPC commence par une **plage d'adresses IP privées**. Par exemple, `10.0.0.0/16` signifie :

```
10.0.0.0/16 = 65 536 adresses disponibles (10.0.0.0 à 10.0.255.255)

Adresses réservées AWS :
10.0.0.0     = Adresse réseau (réservée)
10.0.0.1     = Gateway AWS (réservée)
10.0.0.2     = DNS AWS (réservé)
10.0.0.3     = Réservé pour l'avenir
10.0.0.4–... = EC2, RDS, ALB, etc.
```

**Plages privées standards (RFC 1918)** :
- `10.0.0.0/8` (la plus courante, 16 M d'adresses)
- `172.16.0.0/12` (1 M d'adresses)
- `192.168.0.0/16` (65 536 adresses)

#### 2. Subnets (Sous-réseaux)

Un subnet est une **subdivision logique** d'une VPC, liée à une **zone de disponibilité (AZ)** spécifique.

**Subnet public** : route vers Internet Gateway → instances accessibles depuis Internet.
**Subnet privé** : route vers NAT Gateway → instances qui accèdent Internet, mais non accessibles de l'extérieur.

#### 3. Internet Gateway (IGW)

L'**Internet Gateway** est la **passerelle de sortie vers Internet**.

**Coût** : 0 € (sauf si vous utilisez des adresses Elastic IP).

#### 4. NAT Gateway

Permet aux instances **privées** d'accéder à Internet **de manière sécurisée** sans être exposées.

**Coût** : $0.032/h + $0.032 par Go transféré.

##### Coût des adresses IPv4 publiques — Point important depuis 2024

Depuis le **1er février 2024**, AWS facture **toutes les adresses IPv4 publiques**, y compris celles attachées à une instance EC2 en cours d'exécution.

**Le tarif :** `$0,005 / heure` par adresse IPv4 publique, soit **~3,65 $ / mois** par IP.

```
Avant février 2024 :
  ✅ IP publique attachée à une instance → GRATUIT
  ❌ Elastic IP non attachée             → 0,005 $/h

Depuis février 2024 :
  ❌ IP publique attachée à EC2          → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur ELB                 → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur RDS                 → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur NAT Gateway         → 0,005 $/h  ← NOUVEAU
  ❌ Elastic IP non attachée            → 0,005 $/h  (inchangé)
```

**Pourquoi cette décision ?**

L'épuisement des adresses IPv4 est un problème mondial. Il ne reste plus d'adresses disponibles dans le registre IANA depuis 2011. AWS détient environ **130 millions d'adresses IPv4**, qu'il doit acheter sur le marché secondaire.

```
Prix de marché d'une adresse IPv4 en 2024 : environ 55 $ l'adresse
AWS facture 0,005 $/h × 8 760 h/an = 43,80 $/an par IP
→ AWS récupère son investissement en ~15 mois par adresse
```

**Impact concret :**

| Scénario | Nombre d'IPs | Coût mensuel |
|----------|-------------|-------------|
| 1 instance EC2 avec IP publique | 1 | ~3,65 $ |
| 10 instances EC2 + 1 ELB | ~12 | ~43,80 $ |
| Architecture 3-tiers standard | ~5–8 | ~18–29 $ |

> 💡 **Bonne pratique** : utiliser IPv6 là où c'est possible (gratuit), réduire le nombre de ressources avec IP publique, et regrouper les accès via un seul Load Balancer ou une NAT Gateway.

📎 [AWS — Annonce facturation IPv4 (2023)](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)

##### AWS = plateforme 100 % API — À quoi servent vraiment les Elastic IPs ?

**Vous n'avez jamais besoin d'une IP publique pour piloter AWS.** Créer une instance EC2, configurer un VPC, déployer une Lambda — tout cela se fait via l'API AWS, que vous passiez par la console web, la CLI ou un SDK.

```
Vous (navigateur / terminal / code)
         
           HTTPS → api.aws.amazon.com
           (pas besoin d'IP publique sur vos ressources)
        ▼
   AWS Control Plane  (infrastructure interne AWS)
         
        ▼
   Votre ressource (EC2, RDS, Lambda...)
   dans votre VPC privé
```

Le **Control Plane** est entièrement géré par AWS et accessible depuis Internet sur ses propres IPs. Vos ressources peuvent très bien vivre dans un subnet privé sans aucune IP publique.

> 💡 Une instance EC2 sans IP publique est tout à fait fonctionnelle si elle n'a pas besoin d'être atteinte depuis Internet. Elle peut appeler des services AWS (S3, DynamoDB, SSM…) via des VPC Endpoints, sans jamais exposer la moindre adresse publique.

**Les cas d'usage légitimes d'une Elastic IP :**

| Cas d'usage | Pourquoi une EIP est nécessaire |
|------------|--------------------------------|
| **Whitelist IP chez un partenaire** | Le firewall du client autorise uniquement votre IP fixe. Si l'instance redémarre avec une nouvelle IP, la connexion est bloquée. |
| **Failover manuel rapide** | En cas de panne, vous réassignez l'EIP de l'instance morte vers une instance de remplacement en quelques secondes — sans changer la DNS ni attendre la propagation. |
| **NAT Gateway** | Obligatoire : le NAT Gateway a toujours besoin d'une EIP pour sortir sur Internet au nom des instances privées. |
| **Serveur mail (SMTP)** | Les serveurs de messagerie tiers filtrent par IP source. Une IP fixe est indispensable pour la réputation mail. |
| **VPN ou tunnel IPsec** | L'extrémité du tunnel doit être connue à l'avance et ne pas changer. |

**Les cas où une EIP n'est PAS la bonne réponse :**

| Mauvais réflexe | Meilleure alternative |
|----------------|----------------------|
| "Je veux accéder à mon serveur web depuis Internet" | → **Load Balancer** avec DNS (pas d'IP fixe nécessaire) |
| "Je veux une URL stable pour mon API" | → **Route 53** + nom de domaine (DNS, pas IP) |
| "Je veux accéder à mon instance en SSH" | → **Session Manager** (SSM) : SSH sans IP publique, sans port 22 ouvert |
| "Je veux que mes Lambda puissent appeler une API externe" | → **NAT Gateway** (une seule EIP pour tout le subnet) |

> ⚠️ **Le réflexe à éviter** : assigner une Elastic IP à chaque instance "au cas où". C'est la première source de surcoût inutile et de surface d'attaque élargie. Préférez toujours un accès via Load Balancer, Route 53, ou SSM Session Manager.

```
Architecture naïve (coûteuse et risquée) :
  EC2-web    ←── EIP  ($3,65/mois, port 80/443 ouvert sur toute l'IP)
  EC2-api    ←── EIP  ($3,65/mois, port 8080 ouvert)
  EC2-admin  ←── EIP  ($3,65/mois, port 22 ouvert)
  Total IPs  : 3 × 3,65 = 10,95 $/mois + surface d'attaque maximale

Architecture correcte (économique et sécurisée) :
  ALB        ←── 1 EIP  ($3,65/mois, ports 80/443 uniquement)
    - EC2-web   (privé, pas d'IP publique)
    - EC2-api   (privé, pas d'IP publique)
  EC2-admin  (privé) ←── SSM Session Manager (0 IP publique, 0 port ouvert)
  Total IPs  : 1 × 3,65 = 3,65 $/mois + sécurité maximale
```

📎 [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
📎 [Elastic IP Addresses — Documentation AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)


#### 5. Route Tables (Tables de routage)

Chaque subnet est associé à une **table de routage** qui définit comment le trafic circule.

<img src="formations/aws-initiation-approfondissement/11-images/vpc-route-tables.svg"
     alt="Tables de routage VPC — subnet public vs privé, association subnet/route table"
     style="display:block; margin:auto; width:90%">

---

### 2.3 Sécurité réseau — Security Groups et Network ACLs

La sécurité dans une VPC repose sur **deux couches** complémentaires.

#### Security Groups — Pare-feu au niveau instance

Un **Security Group** est un ensemble de **règles de filtrage** appliquées à une ou plusieurs instances EC2.

**Caractéristiques** :
- **Stateful** : si vous autorisez les demandes entrantes, les réponses sortantes sont automatiquement autorisées.
- Changements appliqués **immédiatement**.
- Peut être modifié sur une instance en cours d'exécution.

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-05-73664d94.png)

📹 [Groupes de sécurité : pourquoi faire ? Comment ?](https://www.youtube.com/watch?v=QwhexkU2ya4)

#### Network ACLs — Pare-feu au niveau subnet

Un **Network ACL** est un ensemble de règles appliquées à un **subnet entier**.

**Caractéristiques** :
- **Stateless** : vous devez définir EXPLICITEMENT les règles entrantes ET sortantes.
- Numérotées (ordre d'évaluation).
- Application par subnet.

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-06-27e106c5.png)

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-05-73664d94.png)

#### Différences clés

| Aspect | Security Group | Network ACL |
|--------|---|---|
| **Portée** | Instance | Subnet |
| **État** | Stateful | Stateless |
| **Défaut** | Tout refusé sauf règles | Tout refusé sauf règles |

**Bonne pratique** : utilisez les **Security Groups** pour les **règles fines** (par instance), et les **ACLs** pour les **règles larges** (par subnet).

#### Comment tout s'imbrique — architecture 3-tiers dans une VPC

Ces briques (subnets, Security Groups, NAT Gateway) ne prennent tout leur sens qu'assemblées avec les ressources RDS/EC2 vues plus haut dans ce chapitre. Voici l'architecture la plus enseignée en SAA-C03 : un serveur web accessible depuis Internet, une base de données qui ne l'est jamais.

```
VPC 10.0.0.0/16
│
├── Subnet PUBLIC (10.0.1.0/24, AZ 1a)
│    ├── Route Table → 0.0.0.0/0 via Internet Gateway
│    ├── Application Load Balancer (ALB)
│    │    Security Group ALB : inbound 443 depuis 0.0.0.0/0
│    └── NAT Gateway (sort vers Internet pour le subnet privé)
│
├── Subnet PRIVÉ — App (10.0.10.0/24, AZ 1a)
│    ├── Route Table → 0.0.0.0/0 via NAT Gateway (pas d'IGW direct)
│    └── Instance EC2 (serveur web)
│         Security Group EC2 : inbound 80 UNIQUEMENT depuis Security Group ALB
│
└── Subnet PRIVÉ — Data (10.0.20.0/24, AZ 1a)
     ├── Route Table → pas de route vers Internet (isolé)
     └── Instance RDS (déployée en section 1 de ce chapitre)
          Security Group RDS : inbound 3306 UNIQUEMENT depuis Security Group EC2
```

**Ce que cette architecture garantit :**
- Seul l'ALB est exposé sur Internet (subnet public, port 443 ouvert à tous).
- L'EC2 n'accepte du trafic que depuis l'ALB — jamais directement depuis Internet, même si son IP était devinée.
- Le RDS n'accepte du trafic que depuis l'EC2 — la base de données n'a **aucune route vers Internet**, donc même une erreur de configuration Security Group ne peut pas l'exposer.
- Chaque Security Group référence un *autre Security Group* comme source (pas une plage d'IP) — c'est la bonne pratique : si l'EC2 change d'adresse IP (redémarrage, remplacement), la règle reste valide.

---

### 2.4 VPC Peering — Connecter plusieurs VPC

> **VPC Peering** établit une **connexion réseau privée** entre deux VPC, permettant aux instances de communiquer comme si elles étaient dans le même réseau.

#### Caractéristiques

| Aspect | Détail |
|--------|--------|
| **Non-transitif** | A ↔ B fonctionne. Mais A ↔ C ne passera pas par B : il faut une peering A-C explicite. |
| **Coût** | $0.01 par million de requêtes |
| **Inter-comptes** | Deux VPC dans deux comptes AWS différents peuvent être peered |
| **Inter-régions** | Deux VPC dans deux régions différentes peuvent être peered |

> [!WARNING]
> **VPC Peering est non-transitif — Piège architectural classique**
>
> Si vous avez 3 VPCs : **Prod ↔ Shared** et **Dev ↔ Shared**, cela ne signifie PAS que Prod peut parler à Dev via Shared. Le trafic ne transite JAMAIS par un VPC intermédiaire.
>
> Pour interconnecter N VPCs avec transitivité, utilisez **AWS Transit Gateway** (hub centralisé). Avec 4 VPCs, VPC Peering crée 6 connexions à gérer — avec 10 VPCs, c'est 45 connexions. Transit Gateway réduit cela à 1 attachement par VPC.
![](formations/aws-initiation-approfondissement/11-images/ch4-capture-07-5d8ca32f.png)

📎 [Documentation VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

---

### 2.5 Modèle Multi-VPC / Multi-Comptes et AWS Transit Gateway

#### Limitation du VPC Peering

Quand vos infrastructures deviennent **complexes** (10+ VPCs, plusieurs comptes AWS), le peering classique crée un problème :

```
AVEC PEERING CLASSIQUE (non-transitif) :
Pour N VPCs : N × (N-1) / 2 peerings = EXPLOSION combinatoire

Exemple 4 VPCs :
VPC-Prod ↔ VPC-Dev        (1)
VPC-Prod ↔ VPC-Staging    (2)
VPC-Prod ↔ VPC-Shared     (3)
VPC-Dev ↔ VPC-Staging     (4)
VPC-Dev ↔ VPC-Shared      (5)
VPC-Staging ↔ VPC-Shared  (6)
             ↓
        6 peerings manuels ! 😰

Problèmes :
- Non-transitif : Prod ↔ Dev ↔ Shared = Prod ne voit pas Shared
- Gestion complexe : chaque nouveau VPC = 3 peerings à créer
- Pas de contrôle centralisé : règles partagées impossibles
```

#### AWS Transit Gateway — La solution

> **AWS Transit Gateway** est un **hub réseau centralisé** qui connecte **toutes vos VPCs, comptes AWS et réseaux on-prem** via une seule interface.

En architecture hub-and-spoke, l'AWS Transit Gateway devient le hub centralisé (policies et routing uniques) auquel se connectent, chacun via un seul attachement : le réseau On-Premise, VPC-Prod (`10.0.0.0/16`), VPC-Dev (`10.1.0.0/16`), VPC-Staging (`10.2.0.0/16`) et VPC-Shared (`10.3.0.0/16`) — une connexion unique par ressource, à l'échelle illimitée.

Attachements possibles :
- ✅ VPC (plusieurs)
- ✅ Comptes AWS (via RAM — Resource Access Manager)
- ✅ On-premise (via VPN ou Direct Connect)
- ✅ Transit Gateway externe (inter-régions)

#### Architecture complète : Multi-Comptes avec Transit Gateway

L'organisation AWS regroupe plusieurs comptes, tous rattachés au même Transit Gateway (`tgw-xxx`, possédé par le Compte Prod) :

| Compte | Ressource | Rattachement |
|---|---|---|
| PROD (123456789012) | VPC-Prod (`10.0.0.0/16`) | Attachement TGW |
| DEV (210987654321) | VPC-Dev (`10.1.0.0/16`) | Attachement TGW |
| SHARED (shared-000) | VPC-Shared (`10.3.0.0/16`) | Attachement TGW |
| ON-PREMISE | Network (`192.168.0.0/16`) | Attachement VPN/Direct Connect |
| MGMT | CloudTrail logs (audit) | — |

Le Transit Gateway maintient des route tables distinctes (Production, Dev, Shared) pour contrôler quel trafic peut transiter entre quels VPC.

#### Avantages Transit Gateway

| Aspect | VPC Peering | Transit Gateway |
|--------|---|---|
| **Connexions N VPCs** | N(N-1)/2 peerings 😱 | 1 attachement par VPC ✅ |
| **Transitif** | Non (A↔B, B↔C ≠ A↔C) | Oui (contrôlé par routing) |
| **Multi-comptes** | Non (même compte seulement) | Oui (via Resource Access Manager) |
| **On-premise** | Non | Oui (VPN + Direct Connect) |
| **Policies centralisées** | Impossible | Oui (Network Policy) |
| **Coût** | $0.01 par million requêtes | $0.05 par attachement/h + data |

#### Configuration AWS CLI — Transit Gateway, principe

Le Transit Gateway est créé une seule fois, puis chaque VPC s'y attache via une **attachment**. La table de routage du TGW détermine quels VPCs peuvent se parler.

```bash
# Créer le Transit Gateway
aws ec2 create-transit-gateway \
  --description "Formation Transit Gateway Hub" \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=FormationTGW}]'
# Résultat : TransitGatewayId = tgw-0123456789abcdef

# Attacher un VPC au Transit Gateway
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-0123456789abcdef \
  --vpc-id vpc-prod-12345 \
  --subnet-ids subnet-prod-1a subnet-prod-1b
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {"TransitGatewayVpcAttachment": {"State": "pending", "TransitGatewayId": "tgw-0123456789abcdef", "VpcId": "vpc-prod-12345"}}
> ```
> [!NOTE]
> **Pour aller plus loin — hors périmètre de cette formation** : partager un Transit Gateway entre plusieurs comptes AWS (via Resource Access Manager) et l'étendre à un réseau on-premise (VPN Site-to-Site) relèvent du niveau Advanced Networking Specialty. Le principe reste le même — un attachement par ressource, une table de routage centralisée — mais la mise en œuvre cross-account est un sujet à part entière.
#### Pièges Transit Gateway

| Piège | Solution |
|-------|----------|
| **TGW par défaut permet tout** | Créer des route tables TGW restrictives par environnement |
| **Coût : $0.05/attachement/h** | Budget pour 20 VPCs = ~$72/mois (0.05 × 20 × 720 h) |
| **Association subnet obligatoire** | Au moins 1 subnet par AZ pour la résilience |
| **CIDR overlap interdit** | VPCs partagés doivent avoir CIDRs différents |

---

### 2.6 VPC Endpoints — Accès privé aux services AWS

#### Définition

> Un **VPC Endpoint** est une **passerelle privée** qui permet à vos ressources d'accéder à des **services AWS sans passer par Internet**.

#### Deux types

| Type | Services | Fonctionnement | Coût |
|------|----------|---|---|
| **Gateway Endpoint** | S3, DynamoDB | Interface simple (ajoute une route) | Gratuit |
| **Interface Endpoint** | EC2, Secrets Manager, Lambda, etc. | Interface élastique privée (ENI) | $0.007/h + transfert |

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-08-302703e2.png)

📎 [Documentation VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)

---

### 2.7 ENI (Elastic Network Interfaces) — Les cartes réseau d'AWS

#### Qu'est-ce qu'une ENI ?

> Une **ENI** est une **carte réseau virtuelle** attachée à une instance EC2. Elle gère vos **adresses IP**, vos **adresses MAC**, vos **Security Groups** et vos **routes réseau**.

Dans une infrastructure **on-prem**, vous aviez des **cartes réseau physiques** (NIC) dans vos serveurs. Sur AWS, c'est exactement la même chose, mais **virtuelle et reconfigurable**.

#### Anatomie d'une ENI

```
Instance EC2 (t3.medium)
 
- Primary ENI (eth0)  [obligatoire]
    
   - Primary IP privée : 10.0.1.42 (CIDR subnet)
    
   - Secondary IP privées : 10.0.1.43, 10.0.1.44 (optionnel)
    
   - Elastic IP publique : 203.0.113.12 (optionnel)
    
   - MAC Address : 02:c1:1f:a0:2b:4d (auto-générée)
    
   - Security Group : sg-12345678
    
   - Source/Dest Check : ✅ activée (drop trafic non-destiné)
```

📹 [Comment conserver son IP sur AWS ?](https://www.youtube.com/watch?v=oSMEQlQDohM)

#### Cas d'usage : Multiple ENIs sur une même instance

Certains scénarios nécessitent **plusieurs ENIs** sur une instance :

```
SCÉNARIO : Serveur pare-feu / VPN / Load Balancer

Instance (m5.xlarge) : 4 ENIs possibles
 
- eth0 (Primary) : connectée à VPC public  → Internet
    - 10.0.1.10
 
- eth1 (Secondary) : connectée à VPC privée → Apps internes
    - 10.1.1.10
 
- eth2 (Secondary) : connectée à VPC client (peering) → Client network
    - 10.2.1.10
 
- eth3 (Secondary) : Management/monitoring
    - 10.0.2.10 (subnet privé)

Résultat : machine routeur/pare-feu multi-réseaux ! 🔥
```

#### Configuration ENI via AWS CLI

On crée ici une ENI indépendante puis on l'attache à une instance existante. Cela permet d'ajouter une interface réseau secondaire sans recréer l'instance — utile pour un serveur bastion ou un routeur multi-réseaux.

```bash
# ═══════════════════════════════════════════════════════════
# 1. Créer une ENI seule (détachée)
# ═══════════════════════════════════════════════════════════

aws ec2 create-network-interface \
  --subnet-id subnet-12345678 \
  --description "Secondary management interface" \
  --private-ip-addresses PrivateIpAddress=10.0.2.100,Primary=true \
  --groups sg-mgmt-12345678 \
  --tag-specifications 'ResourceType=network-interface,Tags=[{Key=Name,Value=ENI-Management}]'

# Résultat : NetworkInterfaceId = eni-0a1b2c3d4e5f6g7h8

# ═══════════════════════════════════════════════════════════
# 2. Attacher une ENI existante à une instance
# ═══════════════════════════════════════════════════════════

aws ec2 attach-network-interface \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --instance-id i-0123456789abcdef0 \
  --device-index 1  # eth1 (0=primary/eth0, 1=eth1, etc.)

# ═══════════════════════════════════════════════════════════
# 3. Allouer une Elastic IP et l'attacher à une ENI
# ═══════════════════════════════════════════════════════════

# Allouer Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --private-ip-address 10.0.2.100

# Résultat : PublicIp = 203.0.113.50

# ═══════════════════════════════════════════════════════════
# 4. Ajouter une IP privée secondaire à une ENI
# ═══════════════════════════════════════════════════════════

aws ec2 assign-private-ip-addresses \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --private-ip-addresses 10.0.2.101 10.0.2.102

# ═══════════════════════════════════════════════════════════
# 5. Changer de Security Group sur une ENI
# ═══════════════════════════════════════════════════════════

aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --groups sg-new-12345678 sg-management-67890

# ═══════════════════════════════════════════════════════════
# 6. Détacher une ENI (reste dans la VPC, peut être réattachée)
# ═══════════════════════════════════════════════════════════

aws ec2 detach-network-interface \
  --attachment-id eni-attach-12345678

# ═══════════════════════════════════════════════════════════
# 7. Voir toutes les ENI d'une instance
# ═══════════════════════════════════════════════════════════

aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --query 'Reservations[0].Instances[0].NetworkInterfaces[*].[NetworkInterfaceId,Attachment.DeviceIndex,PrivateIpAddresses[0].PrivateIpAddress,PrivateIpAddresses[0].Association.PublicIp]'

# Résultat (exemple) :
# [
#   ["eni-primary", 0, "10.0.1.42", "203.0.113.50"],    ← eth0 + Elastic IP
#   ["eni-secondary", 1, "10.0.2.100", null]             ← eth1 (privée)
# ]
```

> [!TIP]
> **Résultat attendu :**
> ```json
> [
>     ["eni-0a1b2c3d4e5f00001", 0, "10.0.1.42", "203.0.113.50"],
>     ["eni-0a1b2c3d4e5f00002", 1, "10.0.2.100", null]
> ]
> ```
#### Cas d'usage réels : ENI multiples

| Scénario | Interfaces | Bénéfice |
|----------|---|---|
| **Routeur/Pare-feu** | 3-4 ENIs | Connexion à plusieurs VPCs/subnets sans NAT |
| **Haute disponibilité** | Primary ENI + failover | IP privée = même, instance change (failover transparent) |
| **Serveur DNS interne** | Primary + management | Trafic DNS sur une interface, logs/monitoring sur autre |
| **Load Balancer maison** | Multiple NICs | Distribution load par interface réseau |
| **Serveur VPN/bastion** | 2+ interfaces | Accès de plusieurs subnets via une machine unique |

#### Piège : Source/Destination Check

Par défaut, AWS bloque tout paquet dont l'IP source ou destination ne correspond pas à l'interface. C'est intentionnel pour la sécurité, mais cela empêche un routeur ou un pare-feu de forwarder le trafic. Il faut donc **désactiver cette vérification** sur les instances qui jouent un rôle de routage.

```bash
# ⚠️ Par défaut, une ENI REFUSE le trafic non-destiné à elle-même.
# Exemple : routeur firewall doit FOWARDER le trafic.

# Désactiver Source/Destination check
aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-12345678 \
  --no-source-dest-check

# Résultat : routeur peut maintenant forwarder (forwarding activé)

# Réactiver (sécurité)
aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-12345678 \
  --source-dest-check
```

> [!TIP]
> **Résultat attendu :**
> ```
> # modify-network-interface-attribute : aucun output si succès
>
> # Vérification : aws ec2 describe-network-interface-attribute --network-interface-id eni-12345678 --attribute sourceDestCheck
> {
>     "NetworkInterfaceId": "eni-12345678",
>     "SourceDestCheck": {
>         "Value": false
>     }
> }
> ```
> La vérification Source/Destination est désactivée (`Value: false`). L'interface peut désormais forwarder du trafic dont elle n'est pas la destination finale — comportement requis pour une instance jouant le rôle de routeur ou de pare-feu.
---

## 3. Amazon Route 53 — DNS Intelligent

### 3.1 Fondamentaux du DNS

#### Qu'est-ce que le DNS ?

Le **DNS (Domain Name System)** est un système **mondial décentralisé** qui traduit des **noms lisibles** (`www.example.com`) en **adresses IP** (`93.184.216.34`).

---

### 3.2 Amazon Route 53 — Service DNS managé

#### Définition

> **Amazon Route 53** est le **service DNS managé** d'AWS. Il permet de **résoudre les noms de domaine**, de **diriger le trafic intelligemment**, et d'**assurer la haute disponibilité**.

Le nom « Route 53 » vient du **port 53**, utilisé par le protocole DNS.

#### Capacités

Route 53 combine plusieurs fonctionnalités :

| Fonctionnalité | Description |
|---|---|
| **Registrar** | Acheter et gérer des domaines (.com, .fr, .io, etc.) |
| **Résolution DNS** | Traduire noms → IP |
| **Health Checks** | Vérifier si une ressource est disponible |
| **Routage intelligent** | Diriger le trafic selon latence, géolocalisation, poids, failover |
| **Alias Records** | Lier un domaine à une ressource AWS (ALB, CloudFront, S3) |

#### Types d'enregistrements DNS courants

| Type | Exemple | Rôle |
|------|---------|------|
| **A** | `example.com` → `93.184.216.34` | IPv4 |
| **AAAA** | `example.com` → `2606:2800:...` | IPv6 |
| **CNAME** | `www.example.com` → `example.com` | Alias |
| **MX** | `example.com` → `mail.example.com` | Serveur mail |
| **TXT** | `example.com` → `v=spf1...` | Enregistrement texte |
| **NS** | `example.com` → `ns1.route53...` | Serveurs DNS |

---

### 3.3 Politiques de routage — Diriger le trafic intelligemment

Route 53 n'est pas un simple DNS classique. C'est un **routeur de trafic applicatif**.

#### Politique Simple

Cas de base : un domaine pointe vers **une seule adresse IP**.

#### Politique Pondérée

Distribuer le trafic en pourcentage entre plusieurs ressources.

**Cas d'usage** : déploiement progressif, A/B testing, migration progressive.

#### Politique Latence

Router les utilisateurs vers la ressource **la plus rapide** (latence réseau minimale).

**Cas d'usage** : applications globales, réduction latence.

#### Politique Failover (Basculement)

En cas de panne détectée, router vers une ressource de secours.

**Cas d'usage** : haute disponibilité, reprise après sinistre.

---

### 3.4 Health Checks et Monitoring

Route 53 peut **monitorer la santé** des ressources et basculer automatiquement.

#### Types de Health Checks

| Type | Description | Fréquence |
|------|---|---|
| **HTTP/HTTPS** | Effectue une requête GET, attend 2xx/3xx | Toutes les 30s |
| **TCP** | Établit une connexion TCP | Toutes les 10s |
| **Calculated** | Combine plusieurs health checks | Toutes les 30s |
| **CloudWatch** | Déclenché par une alarme CloudWatch | Variable |

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-09-0f658ba9.png)

![](formations/aws-initiation-approfondissement/11-images/ch4-capture-10-042036a6.png)

---

## 4. Amazon ElastiCache — Mise en cache distribuée

### 4.1 Pourquoi une couche cache ?

Imaginez une **base de données RDS** qui reçoit **1 000 requêtes par seconde** pour lire les **mêmes 10 utilisateurs**. Chaque requête demande 5–10 ms à la base. Résultat : **goulot d'étranglement**, latence élevée, coût RDS énorme.

**Solution** : placez un **cache rapide** devant la base. Les 1 000 requêtes frappent le cache (**< 1 ms**) au lieu de la base.

**Amazon ElastiCache** est le service managé AWS pour placer un **cache distribuée** haute performance devant vos applications.

---

### 4.2 Deux moteurs : Redis vs Memcached

#### Redis (Remote Dictionary Server)

```
Cas d'usage : Sessions utilisateur, panier e-commerce, rankings, pubsub
Structure : Chaînes, listes, ensembles, hashes, streams, géo-spatial
Persistance : RDB + AOF (journalisation)
Clustering : Oui, avec failover automatique
Transactions : MULTI/EXEC
TTL (durée de vie clé) : Oui
```

Redis peut perdre les données stockées en mémoire lors d'un redémarrage ou d'une panne — la **persistance** est le mécanisme qui les sauvegarde sur disque pour pouvoir les recharger ensuite. AWS ElastiCache pour Redis combine deux méthodes complémentaires : **RDB** (Redis Database, un instantané complet du contenu de la mémoire pris à intervalles réguliers, rapide à recharger mais qui peut perdre les toutes dernières écritures) et **AOF** (Append Only File, un journal qui enregistre chaque commande d'écriture au fil de l'eau, plus lourd mais qui permet de ne perdre quasiment aucune donnée en cas de panne).

**Analogie** : un **dictionnaire magique ultra-rapide** qui se souvient des modifications.

#### Memcached

```
Cas d'usage : Cache objet simple (résultats DB, pages HTML)
Structure : Chaînes et blobs uniquement
Persistance : Non (tout volatil)
Clustering : Oui, mais pas de failover
Transactions : Non
TTL : Oui
```

**Analogie** : un **panier à oublier** ultra-simple — parfait pour des données éphémères.

---

### 4.3 Cas d'usage typiques

| Cas d'usage | Moteur | Raison |
|---|---|---|
| **Session utilisateur** | Redis | Besoin de persistance, expiration TTL |
| **Panier e-commerce** | Redis | Structures complexes (hash), transactions |
| **Leaderboard** | Redis | Opérations set triées (`ZSET`) |
| **Cache HTML statique** | Memcached | Simple, volatil, très rapide |
| **Résultats requête DB** | Redis | Contrôle TTL par clé, publish/subscribe |
| **Real-time counters** | Redis | Opérations atomiques (`INCR`) |

---

### 4.4 Architecture ElastiCache

#### Cluster Mode Disabled (simple, old-school)

L'application (EC2, Lambda) envoie ses requêtes `GET cache_key` vers le nœud ElastiCache Redis primary (`eu-west-1a`), qui réplique de façon asynchrone vers un nœud Replica standby en lecture seule (`eu-west-1b`).

**Limitation** : une seule shard, donc un seul nœud — le CPU de ce nœud unique plafonne la capacité totale.

#### Cluster Mode Enabled (production, sharding)

L'application (EC2, Lambda) hache chaque clé pour la router vers l'une des trois shards : Shard 1 (clés 1-3), Shard 2 (clés 4-6) ou Shard 3 (clés 7-10). Chaque shard a son propre primary node, répliqué vers un Replica correspondant (`eu-west-1b`).

**Bénéfice** : parallélisation et scalabilité linéaire — ajouter des shards augmente la capacité totale, contrairement au mode Cluster Disabled.

---

### 4.5 Commandes Redis essentielles

```bash
# Installation (macOS via Homebrew)
brew install redis

# Lancer serveur Redis local (développement)
redis-server

# Client Redis (dans un autre terminal)
redis-cli

# ──── CHAÎNES (Strings) ────
SET nom "Alice"                    # Stocker
GET nom                            # Récupérer → "Alice"
APPEND nom " Dupont"               # Ajouter → "Alice Dupont"
STRLEN nom                         # Longueur → 12
INCR compteur                      # Incrémenter (atomique)
DECR compteur                      # Décrémenter

# ──── LISTES (Lists) ────
RPUSH queue "tache1"               # Ajouter à droite
RPUSH queue "tache2" "tache3"      # Multiple
LPOP queue                         # Retirer de gauche
LLEN queue                         # Longueur
LRANGE queue 0 -1                  # Tout afficher

# ──── HASHES (Objets) ────
HSET user:100 nom "Alice"          # Stocker champ
HSET user:100 email "alice@ex.com" age 28
HGET user:100 nom                  # Récupérer → "Alice"
HGETALL user:100                   # Tous les champs
HDEL user:100 age                  # Supprimer champ

# ──── ENSEMBLES TRIÉS (Sorted Sets, ZSET) ────
ZADD leaderboard 100 "Alice"       # Score 100 → Alice
ZADD leaderboard 150 "Bob" 200 "Charlie"
ZRANGE leaderboard 0 -1            # Ordre croissant
ZREVRANGE leaderboard 0 -1         # Ordre décroissant (top)
ZRANK leaderboard "Alice"          # Position → 0 (première)

# ──── EXPIRATION ────
SET session:user123 "data"
EXPIRE session:user123 3600        # Expirer dans 1 heure
TTL session:user123                # Temps restant → 3599

# ──── TRANSACTIONS ────
MULTI
SET clé1 "valeur1"
SET clé2 "valeur2"
EXEC                               # Atomique : tout ou rien

# ──── PUBLISH/SUBSCRIBE ────
SUBSCRIBE channel:notifications    # S'abonner
PUBLISH channel:notifications "Hello" # Diffuser
```

---

### 4.6 Créer un cluster Redis en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Database, section ElastiCache) vous permettra d'approfondir le déploiement d'un cluster Redis.
On crée ici le type de cluster le plus simple : un nœud Redis unique, sans réplication. En production, on ajouterait un groupe de réplication (`create-replication-group`) avec un nœud primaire et des replicas, mais ce modèle suffit pour comprendre les concepts.

```bash
# 1. Créer un cluster Redis (simple, mode Cluster Mode Disabled)
aws elasticache create-cache-cluster \
  --cache-cluster-id formation-redis-simple \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --engine-version 7.0 \
  --num-cache-nodes 1 \
  --vpc-security-group-ids sg-12345678 \
  --cache-subnet-group-name my-subnet-group \
  --tags Key=Name,Value=FormationRedis Key=Env,Value=Dev

# 2. Attendre que le cluster soit disponible
aws elasticache wait cache-cluster-available \
  --cache-cluster-id formation-redis-simple

# 3. Récupérer l'endpoint (adresse:port)
aws elasticache describe-cache-clusters \
  --cache-cluster-id formation-redis-simple \
  --query 'CacheClusters[0].CacheNodes[0].Endpoint'
# Résultat exemple : formation-redis-simple.abc123.ng.0001.euw1.cache.amazonaws.com:6379
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "CacheClusters": [{
>         "CacheClusterId": "formation-redis-simple",
>         "CacheClusterStatus": "available",
>         "Engine": "redis",
>         "EngineVersion": "7.0.7",
>         "CacheNodeType": "cache.t3.micro",
>         "CacheNodes": [{
>             "CacheNodeId": "0001",
>             "CacheNodeStatus": "available",
>             "Endpoint": {
>                 "Address": "formation-redis-simple.abc123.ng.0001.euw1.cache.amazonaws.com",
>                 "Port": 6379
>             }
>         }]
>     }]
> }
> ```
```bash
# 4. Créer un Replication Group (Multi-AZ avec failover auto)
aws elasticache create-replication-group \
  --replication-group-id formation-redis-ha \
  --replication-group-description "Redis avec failover" \
  --engine redis \
  --engine-version 7.0 \
  --cache-node-type cache.t3.micro \
  --num-cache-clusters 2 \
  --automatic-failover-enabled \
  --multi-az-enabled \
  --vpc-security-group-ids sg-12345678 \
  --cache-subnet-group-name my-subnet-group

# 5. Décrire le replication group
aws elasticache describe-replication-groups \
  --replication-group-id formation-redis-ha

# 6. Créer un cluster Memcached (simple)
aws elasticache create-cache-cluster \
  --cache-cluster-id formation-memcached \
  --cache-node-type cache.t3.micro \
  --engine memcached \
  --engine-version 1.6.17 \
  --num-cache-nodes 3 \
  --vpc-security-group-ids sg-12345678 \
  --cache-subnet-group-name my-subnet-group

# 7. Supprimer un cluster (attention : perte de données)
aws elasticache delete-cache-cluster \
  --cache-cluster-id formation-redis-simple
```

---

### 4.7 Bonne pratique : Cache-Aside Pattern

Le pattern le plus courant pour intégrer un cache :

```
Requête application :

1. Cache.GET(clé) ?
   - Si HIT → retourner valeur (< 1 ms) ✅
   - Si MISS → aller à 2

2. Requête base de données
   RDS.SELECT(clé) → résultat

3. Stocker en cache
   Cache.SET(clé, résultat, TTL=3600) # Expire en 1 heure

4. Retourner résultat application

Avantage : logique simple, contrôle du cache
Risque : cache stale (données anciennes) pendant TTL
```

### 4.8 Combien coûte ElastiCache ?

ElastiCache se facture à l'heure d'instance active, comme EC2 et RDS — pas de coût "à la requête" contrairement à DynamoDB. Le prix dépend du type de nœud et du nombre de nœuds (primary + replicas).

| Type de nœud | RAM | Usage typique | Prix indicatif (région Paris) |
|---|---|---|---|
| cache.t3.micro | 0,5 Go | Dev/test, très petite charge | ~0,020 $/h (~15 $/mois) |
| cache.t3.medium | 3,09 Go | Sessions utilisateur, petite prod | ~0,080 $/h (~58 $/mois) |
| cache.r6g.large | 13,07 Go | Cache applicatif de production | ~0,226 $/h (~165 $/mois) |

**Ce qui fait varier la facture :**
- **Nombre de nœuds** : un cluster Redis en haute disponibilité (primary + replica) double le coût du nœud seul — exactement comme Multi-AZ sur RDS.
- **Cluster Mode Enabled** (sharding) : chaque shard supplémentaire est un nœud facturé en plus — utile pour la scalabilité, mais le coût grimpe linéairement avec le nombre de shards.
- **Transfert de données** : gratuit entre ElastiCache et EC2 dans la même AZ, facturé au-delà (inter-AZ, inter-région).

**Repère utile** : ElastiCache n'est rentable que si le cache réduit suffisamment la charge sur RDS pour permettre une instance RDS plus petite, ou évite d'ajouter des Read Replicas RDS payants. Sur une charge de lecture très répétitive (mêmes clés interrogées des milliers de fois), le calcul est presque toujours favorable ; sur des requêtes peu répétées, le cache n'apporte rien et n'est qu'un coût supplémentaire.

📎 [Amazon ElastiCache Pricing](https://aws.amazon.com/elasticache/pricing/)

---

## 5. Points importants et pièges fréquents

### 5.1 Pièges RDS et Bases de données

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **RDS n'est pas auto-scalable en stockage** | Faut augmenter manuellement (RDS) ou activer auto-scaling (Aurora) | Saturation disque → downtime | Vérifier "Storage autoscaling" dans RDS config |
| **Multi-AZ RDS ≠ haute disponibilité lue** | Multi-AZ = résilience (failover), pas scalabilité lecture | Bottleneck en lecture malgré Multi-AZ | Ajouter **Read Replicas** (asynchrones) |
| **Chiffrement RDS doit être activé à la création** | On ne peut pas l'activer après coup | Recréer l'instance = downtime | Checker "Encrypt at rest" lors création |
| **Read Replica ≠ Multi-AZ** | Replica = asynchrone, pour lectures. Multi-AZ = synchrone, failover | Confondre les deux gâche design | Multi-AZ pour haute dispo, Replicas pour scalabilité lecture |
| **Snapshot RDS = backup manuel** | Snapshots manuels ne s'auto-suppriment pas | Surcoûts stockage | Supprimer manuellement ou appliquer cycle vie |

### 5.2 Pièges VPC et Réseau

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **VPC Peering n'est pas transitif** | A ↔ B et B ↔ C ne signifie pas A ↔ C | Communication A-C bloquée | Créer peering A-C explicitement ou utiliser Transit Gateway |
| **Security Group ≠ Network ACL** | SG = stateful (instance), NACL = stateless (subnet) | Oublier une règle sortante NACL bloque tout | Vérifier ACL entrée ET sortie |
| **IGW coûte 0 €** | NAT Gateway coûte $0.032/h + transfert | Budget explosif avec gros trafic privé | Minimiser NAT, utiliser VPC Endpoints S3 |
| **Security Group par défaut refuse tout** | Sauf trafic **sortant** (autorisé par défaut) | Instances isolées jusqu'à ouverture ingress | Ajouter règles **ingress** explicites |
| **Security Group : ALLOW vs DENY** | SG = whitelist (ALLOW seulement), pas de DENY | Penser NACL pour bloquer IPs spécifiques | NACL pour explicite DENY |
| **Associer route table incorrect** | Associer table RT publique à subnet privé = accès internet non sécurisé | Instances "privées" exposées à Internet | Vérifier subnet <→ route table |
| **ENI : Source/Dest Check activé par défaut** | ENI refuse le trafic non-destiné à elle (sécurité) | Routeur/pare-feu ne peut pas forwarder | Désactiver `source-dest-check` pour routeurs |
| **Attach ENI = Device index critique** | Device index 0 = primary (obligatoire), 1+ = secondary | Erreur lors attach bloque l'instance | Vérifier device index disponible avant attach |
| **Transit Gateway ≠ gratuit** | $0.05/attachement/h + $0.02 par Go data | Coût pour 20 VPCs = ~$72/mois | Budget transit gateway si 10+ VPCs |
| **Transit Gateway routing par défaut = tous allowed** | TGW fait transiter tous les paquets par défaut | Communication imprévue entre VPCs | Restreindre via Route Tables TGW explicites |
| **VPC CIDR overlap interdit dans Transit Gateway** | Tous les VPCs attachés doivent avoir CIDR différents | Adresses en collision = paquets perdus | Planifier CIDR par VPC avant TGW |

### 5.3 Pièges Route 53 et DNS

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **DNS Route 53 a un TTL** | Les réponses sont cachées pendant le TTL | Changement DNS peut prendre 24h | Baisser TTL avant changement (300s) |
| **Health Check ≠ Failover automatique** | Health check détecte panne, failover redirection | Juste détecter ne suffit pas | Configurer failover + health check |
| **Alias Records ≠ CNAME** | Alias = pointeur AWS (gratuit, flexible), CNAME = alias DNS classique | Confondre risque problèmes CNAME root | Toujours Alias pour AWS resources (ALB, CloudFront) |

> [!WARNING]
> **TTL Route 53 trop court = coût de requêtes élevé**
>
> Un TTL de **30 secondes** sur un enregistrement très consulté (ex. : API avec 1 000 req/s) génère des **résolutions DNS répétées**. Route 53 facture **$0.40 par million de requêtes** pour les zones publiques.
>
> Exemple : 1 000 req/s × 2 résolutions/min (TTL=30s) × 3 600 s/h = ~432 000 requêtes DNS/h → **~$4/jour** juste pour le DNS.
>
> **Règle** :
> - TTL **300s** (5 min) pour la plupart des enregistrements stables
> - TTL **60s** maximum pendant une migration ou un failover planifié
> - Remonter le TTL à **300-3600s** après stabilisation
### 5.4 Pièges DynamoDB

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **DynamoDB provisionned vs on-demand** | Mode provisionné = moins cher si prévisible | Charge imprévisible = throttling ou surcoûts | Choisir on-demand si variable, provisionné si stable |
| **Partition key ≠ Sort key** | Partition = hash (required), Sort = range (optional) | Query sans sort key = full table scan | Bien concevoir partition + sort key |
| **DynamoDB TTL n'est pas immédiat** | TTL supprime dans 24-48h après expiration | Données restent visibles brièvement | Ne pas compter sur TTL pour sécurité |
| **Global Secondary Index (GSI) coûte** | GSI = throughput supplémentaire à provisionner | Surcoûts si GSI mal utilisés | Bien planifier projections, ne créer que GSI utiles |

### 5.5 Pièges ElastiCache

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **Redis vs Memcached** | Redis = persistance, structures complexes. Memcached = volatil, simple | Choisir Memcached pour données durables = perte | Redis pour sessions, Memcached pour cache éphémère |
| **Cache-Aside pattern = possibilité cache stale** | TTL peut garder données anciennes 1h | Utilisateurs voient données obsolètes | Réduire TTL ou implémenter invalidation manuelle |
| **ElastiCache dans VPC ≠ accessible depuis EC2 autre subnet** | Besoin Security Group + route table | EC2 ne peut pas accéder cache | Vérifier SG ElastiCache permet EC2, même VPC |
| **Cluster mode disabled : une seule shard** | Pas de sharding = un seul nœud max CPU | Bottleneck CPU même avec plusieurs replicas | Cluster mode enabled pour scalabilité |

### 5.6 Pièges Architecture Générale

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **Aurora Serverless = scaling pas instantané** | Scaling automatique peut durer 30-60s | Latence pics pendant scaling | Pas idéal real-time, mieux RDS provisionné |
| **VPC Endpoint S3 évite la NAT** | Mais doit configurer policies explicites | S3 accès reste privé mais règles complexes | Créer endpoint + bucket policy restrictive |
| **VPC Endpoint = interface privée (coût)** | Interface endpoint = ENI = $0.007/h | Nombreux endpoints = facture élevée | Gateway endpoint pour S3/DynamoDB (gratuit) |

---

## 6. Construire une VPC en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Networking) vous permettra d'approfondir la construction d'une VPC complète.
### 6.1 Créer une VPC complète avec AWS CLI

Cette séquence construit une VPC de zéro, pièce par pièce : VPC → subnets public/privé → Internet Gateway → NAT Gateway → tables de routage. C'est l'ordre obligatoire — chaque ressource dépend de la précédente.

```bash
# ═══════════════════════════════════════════════════════════
# ÉTAPE 1 : Créer la VPC avec CIDR /16
# ═══════════════════════════════════════════════════════════

aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=FormationVPC},{Key=Env,Value=Training}]'

# Récupérer l'ID VPC depuis la sortie précédente
VPC_ID="vpc-12345678"

# Valider la VPC créée
aws ec2 describe-vpcs --vpc-ids $VPC_ID
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "Vpcs": [{
>         "VpcId": "vpc-12345678",
>         "CidrBlock": "10.0.0.0/16",
>         "State": "available",
>         "IsDefault": false,
>         "Tags": [{"Key": "Name", "Value": "FormationVPC"}, {"Key": "Env", "Value": "Training"}]
>     }]
> }
> ```
```bash
# ═══════════════════════════════════════════════════════════
# ÉTAPE 2 : Créer subnets PUBLICS (pour ALB, NAT Gateway)
# ═══════════════════════════════════════════════════════════

# Subnet public en AZ 1a (10.0.1.0/24 = 256 IPs)
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone eu-west-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=SubnetPublic-AZa},{Key=Type,Value=Public}]'

SUBNET_PUBLIC_AZa="subnet-public-a"

# Subnet public en AZ 1b (10.0.2.0/24 = 256 IPs)
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone eu-west-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=SubnetPublic-AZb},{Key=Type,Value=Public}]'

SUBNET_PUBLIC_AZb="subnet-public-b"

# ═══════════════════════════════════════════════════════════
# ÉTAPE 3 : Créer subnets PRIVÉS (pour EC2, RDS)
# ═══════════════════════════════════════════════════════════

# Subnet privé en AZ 1a (10.0.10.0/24)
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.10.0/24 \
  --availability-zone eu-west-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=SubnetPrivate-AZa},{Key=Type,Value=Private}]'

SUBNET_PRIVATE_AZa="subnet-private-a"

# Subnet privé en AZ 1b (10.0.20.0/24) pour haute dispo
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.20.0/24 \
  --availability-zone eu-west-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=SubnetPrivate-AZb},{Key=Type,Value=Private}]'

SUBNET_PRIVATE_AZb="subnet-private-b"

# Lister tous les subnets créés
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID"
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "Subnets": [
>         {"SubnetId": "subnet-public-a", "CidrBlock": "10.0.1.0/24", "AvailabilityZone": "eu-west-1a", "State": "available", "Tags": [{"Key": "Name", "Value": "SubnetPublic-AZa"}]},
>         {"SubnetId": "subnet-public-b", "CidrBlock": "10.0.2.0/24", "AvailabilityZone": "eu-west-1b", "State": "available", "Tags": [{"Key": "Name", "Value": "SubnetPublic-AZb"}]},
>         {"SubnetId": "subnet-private-a", "CidrBlock": "10.0.10.0/24", "AvailabilityZone": "eu-west-1a", "State": "available", "Tags": [{"Key": "Name", "Value": "SubnetPrivate-AZa"}]},
>         {"SubnetId": "subnet-private-b", "CidrBlock": "10.0.20.0/24", "AvailabilityZone": "eu-west-1b", "State": "available", "Tags": [{"Key": "Name", "Value": "SubnetPrivate-AZb"}]}
>     ]
> }
> ```
```bash
# ═══════════════════════════════════════════════════════════
# ÉTAPE 4 : Créer Internet Gateway (accès Internet pour VPC)
# ═══════════════════════════════════════════════════════════

aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=FormationIGW}]'

IGW_ID="igw-12345678"

# Attacher IGW à la VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id $IGW_ID \
  --vpc-id $VPC_ID

# Valider l'attachement
aws ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "InternetGateways": [{
>         "InternetGatewayId": "igw-12345678",
>         "State": "available",
>         "Attachments": [{"State": "available", "VpcId": "vpc-12345678"}],
>         "Tags": [{"Key": "Name", "Value": "FormationIGW"}]
>     }]
> }
> ```
```bash
# ═══════════════════════════════════════════════════════════
# ÉTAPE 5 : Créer Elastic IPs et NAT Gateways
# ═══════════════════════════════════════════════════════════

# Allouer une Elastic IP pour NAT Gateway AZ 1a
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=NAT-EIP-AZa}]'

ALLOCATION_ID_AZa="eipalloc-12345678"

# Allouer une Elastic IP pour NAT Gateway AZ 1b
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=NAT-EIP-AZb}]'

ALLOCATION_ID_AZb="eipalloc-87654321"

# Créer NAT Gateway en AZ 1a (place dans subnet PUBLIC)
aws ec2 create-nat-gateway \
  --subnet-id $SUBNET_PUBLIC_AZa \
  --allocation-id $ALLOCATION_ID_AZa \
  --tag-specifications 'ResourceType=nat-gateway,Tags=[{Key=Name,Value=NAT-GW-AZa}]'

NAT_GW_ID_AZa="nat-12345678"

# Créer NAT Gateway en AZ 1b (pour HA)
aws ec2 create-nat-gateway \
  --subnet-id $SUBNET_PUBLIC_AZb \
  --allocation-id $ALLOCATION_ID_AZb \
  --tag-specifications 'ResourceType=nat-gateway,Tags=[{Key=Name,Value=NAT-GW-AZb}]'

NAT_GW_ID_AZb="nat-87654321"

# Attendre que les NAT Gateways soient disponibles
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID_AZa $NAT_GW_ID_AZb

# ═══════════════════════════════════════════════════════════
# ÉTAPE 6 : Créer et configurer ROUTE TABLES PUBLIQUES
# ═══════════════════════════════════════════════════════════

# Créer route table pour subnets PUBLICS
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=RT-Public},{Key=Type,Value=Public}]'

RT_PUBLIC="rtb-public-12345"

# Ajouter route par défaut vers IGW (0.0.0.0/0 → IGW)
aws ec2 create-route \
  --route-table-id $RT_PUBLIC \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associer subnet PUBLIC AZa avec route table PUBLIC
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PUBLIC_AZa \
  --route-table-id $RT_PUBLIC

# Associer subnet PUBLIC AZb avec route table PUBLIC
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PUBLIC_AZb \
  --route-table-id $RT_PUBLIC

# ═══════════════════════════════════════════════════════════
# ÉTAPE 7 : Créer et configurer ROUTE TABLES PRIVÉES
# ═══════════════════════════════════════════════════════════

# Créer route table pour subnets PRIVÉS en AZ 1a
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=RT-Private-AZa},{Key=Type,Value=Private}]'

RT_PRIVATE_AZa="rtb-private-a"

# Ajouter route par défaut vers NAT Gateway AZ 1a
aws ec2 create-route \
  --route-table-id $RT_PRIVATE_AZa \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW_ID_AZa

# Associer subnet PRIVÉ AZa avec sa route table
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PRIVATE_AZa \
  --route-table-id $RT_PRIVATE_AZa

# Créer route table pour subnets PRIVÉS en AZ 1b
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=RT-Private-AZb},{Key=Type,Value=Private}]'

RT_PRIVATE_AZb="rtb-private-b"

# Ajouter route vers NAT Gateway AZ 1b
aws ec2 create-route \
  --route-table-id $RT_PRIVATE_AZb \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW_ID_AZb

# Associer subnet PRIVÉ AZb
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PRIVATE_AZb \
  --route-table-id $RT_PRIVATE_AZb

# ═══════════════════════════════════════════════════════════
# RÉSUMÉ : Vérifier la VPC complète
# ═══════════════════════════════════════════════════════════

echo "✅ VPC FormationVPC créée avec succès !"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "VPC ID : $VPC_ID"
echo "Subnets publics : $SUBNET_PUBLIC_AZa, $SUBNET_PUBLIC_AZb"
echo "Subnets privés : $SUBNET_PRIVATE_AZa, $SUBNET_PRIVATE_AZb"
echo "Internet Gateway : $IGW_ID"
echo "NAT Gateways : $NAT_GW_ID_AZa (AZa), $NAT_GW_ID_AZb (AZb)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Lister la configuration complète
aws ec2 describe-vpcs --vpc-ids $VPC_ID --query 'Vpcs[0]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID"
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "VpcId": "vpc-12345678",
>     "CidrBlock": "10.0.0.0/16",
>     "State": "available",
>     "Tags": [{"Key": "Name", "Value": "FormationVPC"}]
> }
> ```
---

### 6.2 Créer et configurer un RDS Multi-AZ

Cette démo crée une instance RDS MySQL en production-ready : Multi-AZ activé (réplication synchrone vers une AZ de secours), chiffrement KMS, logs CloudWatch, et authentification IAM. Chaque option a son impact sur la résilience et la sécurité.

```bash
# 1. Créer une instance RDS MySQL avec Multi-AZ ET chiffrement
aws rds create-db-instance \
  --db-instance-identifier formation-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password 'MonMotDePasseSecurise123!' \
  --allocated-storage 20 \
  --storage-type gp3 \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:eu-west-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group \
  --multi-az \
  --backup-retention-period 7 \
  --backup-window "03:00-04:00" \
  --maintenance-window "mon:04:00-mon:05:00" \
  --enable-cloudwatch-logs-exports '["error","general","slowquery"]' \
  --enable-iam-database-authentication \
  --tag-specifications 'ResourceType=db,Tags=[{Key=Name,Value=FormationDB},{Key=Env,Value=Training}]'

# 2. Attendre que l'instance soit disponible (peut prendre 5-10 min)
aws rds wait db-instance-available \
  --db-instance-identifier formation-db

# 3. Récupérer l'endpoint RDS (adresse pour connexion)
aws rds describe-db-instances \
  --db-instance-identifier formation-db \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
# Résultat : formation-db.abc123.eu-west-1.rds.amazonaws.com
```

> [!TIP]
> **Résultat attendu :**
> ```
> formation-db.abc123.eu-west-1.rds.amazonaws.com
> ```
```bash
# 4. Vérifier la configuration Multi-AZ
aws rds describe-db-instances \
  --db-instance-identifier formation-db \
  --query 'DBInstances[0].[DBInstanceIdentifier,MultiAZ,AvailabilityZone,PreferredBackupWindow]'
```

> [!TIP]
> **Résultat attendu :**
> ```json
> [
>     "formation-db",
>     true,
>     "eu-west-1a",
>     "03:00-04:00"
> ]
> ```
```bash
# 5. Modifier l'instance pour augmenter la classe (scaling vertical)
aws rds modify-db-instance \
  --db-instance-identifier formation-db \
  --db-instance-class db.t3.small \
  --apply-immediately

# 6. Créer un Read Replica pour analytics/reporting
aws rds create-db-instance-read-replica \
  --db-instance-identifier formation-db-analytics \
  --source-db-instance-identifier formation-db \
  --db-instance-class db.t3.micro \
  --availability-zone eu-west-1b \
  --storage-type gp3

# 7. Créer un Read Replica inter-région pour Disaster Recovery
aws rds create-db-instance-read-replica \
  --db-instance-identifier formation-db-dr \
  --source-db-instance-identifier formation-db \
  --source-region eu-west-1 \
  --region us-east-1 \
  --db-instance-class db.t3.micro \
  --tag-specifications 'ResourceType=db,Tags=[{Key=Name,Value=FormationDB-DR}]'

# 8. Promouvoir un Read Replica en base indépendante (cas failover manual)
# ⚠️ Une fois promu, il n'est plus répliqué = devient Master indépendant
aws rds promote-read-replica \
  --db-instance-identifier formation-db-analytics

# 9. Créer une sauvegarde manuelle (snapshot) avec timestamp
SNAPSHOT_ID="formation-db-backup-$(date +%Y%m%d-%H%M%S)"
aws rds create-db-snapshot \
  --db-instance-identifier formation-db \
  --db-snapshot-identifier $SNAPSHOT_ID \
  --tags Key=Name,Value="Manual backup" Key=Version,Value="pre-update"

# 10. Lister tous les snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier formation-db \
  --query 'DBSnapshots[*].[DBSnapshotIdentifier,SnapshotCreateTime,DBInstanceIdentifier,AllocatedStorage]'
```

> [!TIP]
> **Résultat attendu :**
> ```json
> [
>     ["formation-db-backup-20260315-143022", "2026-03-15T14:30:22.000Z", "formation-db", 20],
>     ["formation-db-backup-20260316-031500", "2026-03-16T03:15:00.000Z", "formation-db", 20]
> ]
> ```
```bash
# 11. Restaurer une base à partir d'un snapshot (crée une nouvelle instance)
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier formation-db-restored \
  --db-snapshot-identifier formation-db-backup-20240324 \
  --db-instance-class db.t3.micro

# 12. Copier un snapshot vers une autre région (pour DR)
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier arn:aws:rds:eu-west-1:123456789012:snapshot:formation-db-backup \
  --target-db-snapshot-identifier formation-db-backup-us \
  --source-region eu-west-1 \
  --region us-east-1

# 13. Supprimer une instance RDS (avec snapshot final)
aws rds delete-db-instance \
  --db-instance-identifier formation-db \
  --final-db-snapshot-identifier formation-db-final-snapshot \
  --delete-automated-backups

# 14. Supprimer une instance SANS snapshot (⚠️ DANGER)
aws rds delete-db-instance \
  --db-instance-identifier formation-db-analytics \
  --skip-final-snapshot
```

---

### 6.3 Configurer Security Groups (pare-feu instance)

Les Security Groups fonctionnent en chaîne dans une architecture 3-tiers : l'ALB accepte le trafic public, les instances EC2 n'acceptent que le trafic de l'ALB, et RDS n'accepte que le trafic des instances EC2. Cette démo crée les 3 SG et configure leurs règles d'entrée dans cet ordre.

```bash
# ═══════════════════════════════════════════════════════════
# Créer Security Groups pour architecture 3-tiers
# ═══════════════════════════════════════════════════════════

# SG 1 : ALB (Application Load Balancer)
aws ec2 create-security-group \
  --group-name sg-alb \
  --description "Security Group pour Application Load Balancer" \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=SG-ALB}]'

SG_ALB="sg-alb-12345"

# SG 2 : EC2 Web (Applications)
aws ec2 create-security-group \
  --group-name sg-web \
  --description "Security Group pour serveurs web" \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=SG-Web}]'

SG_WEB="sg-web-12345"

# SG 3 : RDS Database
aws ec2 create-security-group \
  --group-name sg-db \
  --description "Security Group pour base de données RDS" \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=SG-DB}]'

SG_DB="sg-db-12345"

# ═══════════════════════════════════════════════════════════
# RÈGLES INGRESS SG-ALB : Accepter trafic Internet
# ═══════════════════════════════════════════════════════════

# HTTP (port 80) depuis Internet
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ALB \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# HTTPS (port 443) depuis Internet
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ALB \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# ═══════════════════════════════════════════════════════════
# RÈGLES INGRESS SG-Web : Accepter depuis ALB
# ═══════════════════════════════════════════════════════════

# Port 80 (HTTP) depuis SG-ALB uniquement
aws ec2 authorize-security-group-ingress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 80 \
  --source-security-group-id $SG_ALB

# Port 443 (HTTPS) depuis SG-ALB
aws ec2 authorize-security-group-ingress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 443 \
  --source-security-group-id $SG_ALB

# Port 22 (SSH) depuis IP admin (YOUR_IP remplacer)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_IP/32  # Exemple : 203.0.113.0/32

# ═══════════════════════════════════════════════════════════
# RÈGLES INGRESS SG-DB : Accepter depuis EC2 Web
# ═══════════════════════════════════════════════════════════

# Port 3306 (MySQL) depuis SG-WEB uniquement
aws ec2 authorize-security-group-ingress \
  --group-id $SG_DB \
  --protocol tcp \
  --port 3306 \
  --source-security-group-id $SG_WEB

# Port 3306 depuis autre subnet privé (si applicable)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_DB \
  --protocol tcp \
  --port 3306 \
  --cidr 10.0.0.0/8

# ═══════════════════════════════════════════════════════════
# RÈGLES EGRESS (sortante) : Vérifier et modifier si nécessaire
# ═══════════════════════════════════════════════════════════

# Par défaut, tout trafic sortant est autorisé (EGRESS)
# Lister les règles de sortie
aws ec2 describe-security-groups \
  --group-ids $SG_WEB \
  --query 'SecurityGroups[0].IpPermissionsEgress'

# Révoquer trafic EGRESS par défaut (si vous voulez contrôle strict)
aws ec2 revoke-security-group-egress \
  --group-id $SG_WEB \
  --protocol -1 \
  --cidr 0.0.0.0/0

# Autoriser seulement EGRESS vers RDS DB
aws ec2 authorize-security-group-egress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 3306 \
  --destination-security-group-id $SG_DB

# Autoriser seulement EGRESS vers Internet (DNS, HTTPS)
aws ec2 authorize-security-group-egress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-egress \
  --group-id $SG_WEB \
  --protocol udp \
  --port 53 \
  --cidr 0.0.0.0/0

# ═══════════════════════════════════════════════════════════
# OPÉRATIONS : Modifier et supprimer règles
# ═══════════════════════════════════════════════════════════

# Lister toutes les règles d'un SG
aws ec2 describe-security-groups --group-ids $SG_WEB
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "SecurityGroups": [{
>         "GroupId": "sg-web-12345",
>         "GroupName": "sg-web",
>         "Description": "Security Group pour serveurs web",
>         "IpPermissions": [
>             {"IpProtocol": "tcp", "FromPort": 80, "ToPort": 80, "UserIdGroupPairs": [{"GroupId": "sg-alb-12345"}]},
>             {"IpProtocol": "tcp", "FromPort": 443, "ToPort": 443, "UserIdGroupPairs": [{"GroupId": "sg-alb-12345"}]},
>             {"IpProtocol": "tcp", "FromPort": 22, "ToPort": 22, "IpRanges": [{"CidrIp": "203.0.113.0/32"}]}
>         ]
>     }]
> }
> ```
```bash
# Révoquer une règle ingress (exemple : SSH)
aws ec2 revoke-security-group-ingress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_IP/32

# Ajouter une règle avec description
aws ec2 authorize-security-group-ingress \
  --group-id $SG_WEB \
  --protocol tcp \
  --port 80 \
  --source-security-group-id $SG_ALB \
  --group-rule-description "HTTP from ALB"

# Supprimer un Security Group (doit d'abord être détaché)
# aws ec2 delete-security-group --group-id $SG_WEB

# ═══════════════════════════════════════════════════════════
# RÉSUMÉ : Architecture complète
# ═══════════════════════════════════════════════════════════

echo "✅ Security Groups configurés !"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "SG-ALB  : $SG_ALB"
echo "  INGRESS  : 80 (HTTP), 443 (HTTPS) from 0.0.0.0/0"
echo "  EGRESS   : All (par défaut)"
echo ""
echo "SG-Web  : $SG_WEB"
echo "  INGRESS  : 80, 443 from SG-ALB; 22 from YOUR_IP"
echo "  EGRESS   : Restreint (DB, DNS, Internet)"
echo ""
echo "SG-DB   : $SG_DB"
echo "  INGRESS  : 3306 from SG-Web"
echo "  EGRESS   : Aucun (DB ne sort pas)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

---

### 6.4 Créer une Zone Route 53 et des Enregistrements

Une Hosted Zone Route 53 est le conteneur DNS pour un domaine. On crée la zone, puis on y ajoute les enregistrements (A, CNAME, MX…) via des "change batches" JSON. Chaque modification est atomique — soit tout réussit, soit tout échoue.

```bash
# 1. Créer une zone Route 53
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s) \
  --hosted-zone-config PrivateZone=false
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>     "HostedZone": {
>         "Id": "/hostedzone/Z1234567890ABC",
>         "Name": "example.com.",
>         "Config": {"PrivateZone": false},
>         "ResourceRecordSetCount": 2
>     },
>     "DelegationSet": {
>         "NameServers": [
>             "ns-123.awsdns-15.com",
>             "ns-456.awsdns-57.net",
>             "ns-789.awsdns-31.co.uk",
>             "ns-1012.awsdns-62.org"
>         ]
>     }
> }
> ```
```bash
ZONE_ID="Z1234567890ABC"

# 2. Créer un enregistrement A (IPv4)
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "93.184.216.34"}]
      }
    }]
  }'

# 3. Créer un Health Check HTTP
aws route53 create-health-check \
  --health-check-config '{
    "Type": "HTTP",
    "IPAddress": "93.184.216.34",
    "Port": 80,
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "www.example.com",
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'

# 4. Créer un enregistrement avec failover
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Primary",
          "Failover": "PRIMARY",
          "TTL": 60,
          "ResourceRecords": [{"Value": "93.184.216.34"}],
          "HealthCheckId": "health-check-id-12345"
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Secondary",
          "Failover": "SECONDARY",
          "TTL": 60,
          "ResourceRecords": [{"Value": "93.184.216.35"}]
        }
      }
    ]
  }'
```

> [!TIP]
> **Résultat attendu :**
> ```json
> # change-resource-record-sets (enregistrement A) :
> {
>     "ChangeInfo": {
>         "Id": "/change/C1234567890ABC",
>         "Status": "PENDING",
>         "SubmittedAt": "2026-03-15T14:30:00.000Z"
>     }
> }
>
> # create-health-check :
> {
>     "HealthCheck": {
>         "Id": "health-check-id-12345",
>         "HealthCheckConfig": {
>             "IPAddress": "93.184.216.34",
>             "Port": 80,
>             "Type": "HTTP",
>             "ResourcePath": "/health",
>             "FullyQualifiedDomainName": "www.example.com",
>             "RequestInterval": 30,
>             "FailureThreshold": 3
>         },
>         "HealthCheckVersion": 1
>     }
> }
> ```
> Le statut `PENDING` est normal — Route 53 propage les changements DNS dans les 60 secondes en général. Une fois propagé, le statut passe à `INSYNC`. Les enregistrements A avec failover sont actifs : si le health check échoue sur le Primary, Route 53 bascule automatiquement vers le Secondary.
---

## Ressources

### Documentation officielle AWS
- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)
- [Amazon DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [Amazon ElastiCache Documentation](https://docs.aws.amazon.com/elasticache/)
- [AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/)

---

## Quiz interactif du chapitre

Choisissez une réponse : la correction expliquée apparaît immédiatement. Les propositions changent d’ordre à chaque nouvelle tentative.

<iframe class="quiz-frame" src="https://diablotynne.github.io/aws-initiation-approfondissement/static/quiz-aws/quiz-chapitre-4.html" title="Quiz interactif du chapitre 4" loading="lazy"></iframe>
