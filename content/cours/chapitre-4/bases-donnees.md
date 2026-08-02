---
title: "1. Bases de données dans AWS — Du service géré à la scalabilité"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - 1. Bases de données dans AWS — Du service géré à la scalabilité"
---

<nav class="page-sequence"><a href="cours/chapitre-4/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/vpc">Suivant</a></nav>

### 1.1 Répartition des responsabilités avec une base managée

<div class="video-embed"><iframe src="https://www.youtube-nocookie.com/embed/adB--KhJ95w" title="Présentation des services de bases de données AWS" loading="lazy" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

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
```text
- Installation MySQL manuelle (4h)
- Scripts de sauvegarde réseau + test restauration (8h)
- Monitoring 24h/24 avec alertes (contrat SLA)
- Augmentation disque lors des pics → risque de downtime
- Réplication vers DataCenter secondaire (investissement)
→ Coût total 5 ans : ~100 000 €, équipe IT 2 personnes
```

**Avec RDS AWS** :
```bash
- Déploiement en 3 minutes via console AWS
- Option Multi-AZ pour automatiser le basculement, avec une interruption à tolérer côté application
- Snapshots automatiques (jusqu'à 35 jours)
- Augmentation CPU/stockage sans interruption
- Read Replicas pour diffuser les lectures (reports analytiques)
→ Coût annuel : ~1 500 $, gestion minimal (0,1 FTE)
```

---

### 1.3 Haute disponibilité et résilience dans RDS

#### Multi-AZ (Availability Zones) — Résilience automatique

Dans un déploiement RDS Multi-AZ avec une instance de secours, les modifications du principal sont répliquées de manière synchrone vers une autre zone de disponibilité. AWS peut basculer vers cette instance lors de certains incidents ou opérations de maintenance. Les autres variantes Multi-AZ peuvent utiliser plusieurs instances lisibles : il faut vérifier le comportement du moteur et du type de déploiement choisis.

<a class="schema-zoom" href="assets/schemas/rds-multiaz-read-replica.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/rds-multiaz-read-replica.svg"
     alt="Comparaison entre RDS Multi-AZ pour la disponibilité et une réplique en lecture pour décharger les lectures"
     style="display:block; margin:auto; width:95%"></a>

**Lecture du schéma.** À gauche, la réplication synchrone et le basculement visent la disponibilité. À droite, la réplication asynchrone permet d'envoyer des lectures vers un autre endpoint, avec un retard possible. Une réplique en lecture ne remplace donc pas automatiquement un déploiement Multi-AZ.

**Bénéfice** : la réplication synchrone réduit le risque de perte de données et le basculement évite une reconstruction manuelle complète.
**À savoir** : le nom de l'endpoint reste stable, mais les connexions en cours sont interrompues. L'application doit savoir se reconnecter et tolérer le délai de basculement.

> [!warning]
> **Coût Multi-AZ RDS — Attention au budget**
>
> Une configuration Multi-AZ ajoute des ressources et augmente donc la facture. Selon le moteur et le type de déploiement, l'architecture et la facturation diffèrent : instance de secours non lisible ou cluster comportant plusieurs instances. Ne déduisez pas le prix avec un coefficient générique.
>
> **Décision** : activez Multi-AZ lorsque l'objectif de disponibilité le justifie, y compris hors production si l'environnement doit tester les basculements. Comparez les options dans la console ou dans AWS Pricing Calculator.


#### Read Replicas — Répartition de la charge de lecture

Un **Read Replica** est une **copie asynchrone** de votre base, destinée à répartir les **lectures** (SELECT) sans surcharger la Primary.

Cas d'usage : Reporting, Analytics, Exports.

Une instance source peut répliquer ses modifications vers une ou plusieurs répliques en lecture, selon les capacités et quotas du moteur. L'application utilise l'endpoint propre à chaque réplique pour les requêtes de lecture. Le retard de réplication doit être surveillé : une lecture immédiate après une écriture peut ne pas encore voir la nouvelle valeur.

**Différence clé Read Replica vs Multi-AZ** :
- **Multi-AZ** : mécanisme de disponibilité et de basculement ; les possibilités de lecture dépendent du type de déploiement.
- **Read Replica** : réplication asynchrone, endpoint séparé et retard variable ; la promotion éventuelle doit être intégrée au plan de reprise.

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

#### Chiffrement

| Type | Description |
|------|---|
| **Au repos** | Les données sur disque EBS sont chiffrées via AWS KMS |
| **En transit** | TLS à configurer et, selon le moteur, à imposer aux connexions clientes |

⚠️ **Important** : on ne transforme pas directement une instance RDS non chiffrée en instance chiffrée par un simple interrupteur. Le parcours habituel consiste à créer une copie chiffrée d'un snapshot puis à restaurer une nouvelle instance, avec une stratégie de bascule adaptée.

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
  --master-user-password '<MOT_DE_PASSE_FOURNI_HORS_DU_FICHIER>' \
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
  --master-user-password '<MOT_DE_PASSE_FOURNI_HORS_DU_FICHIER>' \
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

**À retenir** : Aurora et les moteurs RDS classiques répondent à des contraintes différentes. Le choix dépend du moteur compatible, du profil d'entrées-sorties, de la disponibilité, de la capacité, des fonctions attendues et du coût total.

#### Comparer le coût de RDS et d'Aurora

| Poste à mesurer | RDS | Aurora |
|---|---|---|
| Calcul | Classe et nombre d'instances, durée de fonctionnement | Instances provisionnées ou capacité Aurora Serverless v2 |
| Stockage | Volume provisionné, type de stockage et performances configurées | Volume consommé et configuration choisie |
| Haute disponibilité | Déploiement mono-AZ ou Multi-AZ | Stockage distribué ; nombre d'instances à définir pour la disponibilité du calcul |
| Entrées-sorties | Dépend du type de stockage et de sa configuration | Dépend de l'édition de tarification Aurora choisie |
| Sauvegarde et transfert | Rétention, snapshots, copies et transferts | Rétention, snapshots, copies et transferts |

**Méthode** : mesurer la charge, construire les deux variantes dans AWS Pricing Calculator, puis tester les performances et le basculement. La mention « serverless » ne signifie ni arrêt total automatique dans toutes les configurations, ni coût nul au repos.

📎 [AWS RDS Pricing](https://aws.amazon.com/rds/pricing/)
📎 [AWS Aurora Pricing](https://aws.amazon.com/rds/aurora/pricing/)


---

### 1.6 Amazon DynamoDB — NoSQL à ultra-haute scalabilité

**DynamoDB** est une **base NoSQL managée** de type clé-valeur et document. Les performances dépendent notamment de la conception des clés, de la taille des items, du mode de capacité et de la distribution de la charge. AWS la conçoit pour fournir une latence de l'ordre de la milliseconde à grande échelle, sous réserve d'un modèle de données adapté.

#### Structure d'une table DynamoDB

```text
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
| **À la demande** | Capacité ajustée par le service, soumise aux quotas et aux partitions | Selon les requêtes traitées | Charge variable |

- **RCU** (Read Capacity Unit) : 1 RCU = une lecture fortement cohérente d'un item ≤ 4 Ko
- **WCU** (Write Capacity Unit) : 1 WCU = une écriture d'un item ≤ 1 Ko

**Méthode d'estimation** : comptez les lectures et écritures, leur taille, le niveau de cohérence, le stockage, les sauvegardes et les index secondaires. Le mode à la demande suit l'activité ; le mode provisionné réserve une capacité et peut utiliser l'auto-scaling. Le meilleur choix dépend du profil réel et doit être recalculé avec le tarif régional courant.

📎 [Amazon DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)

#### DynamoDB vs RDS

| Aspect | RDS (SQL) | DynamoDB (NoSQL) |
|--------|-----------|---|
| **Schéma** | Structuré, relationnel | Flexible, semi-structuré |
| **Requêtes** | Complexes (jointures) | Simples (accès par clé) |
| **Scalabilité** | Verticale surtout | Horizontale, ultra-massive |
| **Latence** | ms-s | ms |

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

**AWS DMS (Database Migration Service)** est un service managé de déplacement et de réplication de données entre des moteurs pris en charge. Il peut effectuer une copie initiale, répliquer ensuite les changements ou combiner les deux. DMS réduit la durée d'indisponibilité potentielle, mais ne garantit pas une migration sans interruption : la bascule applicative, la validation et le retour arrière restent à concevoir.

#### Architecture de migration DMS

Une migration s'appuie sur un endpoint source, un endpoint cible et une configuration de réplication ou une instance de réplication selon le mode DMS choisi. La connectivité réseau, les autorisations, les types de données compatibles et la capacité doivent être validés avant le transfert.

#### Processus de migration par étapes

**Phase 1 — Full Load (copie complète)** : DMS copie les tables sélectionnées. Les objets de schéma, index, contraintes, procédures et fonctions ne sont pas tous recréés de la même manière ; une migration hétérogène peut nécessiter AWS Schema Conversion Tool ou une conversion manuelle.

**Phase 2 — CDC (Change Data Capture)** : pour les moteurs compatibles, DMS lit les journaux de transactions de la source et applique les changements à la cible. La latence varie avec la charge, le réseau, la capacité de réplication et la cible ; elle doit être surveillée.

**Phase 3 — Cutover (basculement applicatif)** :
1. Arrêter ou geler les écritures selon la stratégie retenue.
2. Vérifier que le retard de réplication est compatible avec le RPO.
3. Rediriger les connexions vers la cible.
4. Vérifier les logs applicatifs.
5. Garder un plan de rollback armé.

#### Types de migrations DMS

| Type | Exemple | Point d'attention |
|------|---------|---|
| **Homogène** | MySQL → RDS for MySQL | Versions, paramètres, extensions et temps de bascule |
| **Hétérogène** | Oracle → Aurora PostgreSQL | Conversion du schéma, du code SQL et des types de données |
| **Schéma complexe** | Moteur propriétaire → PostgreSQL | Compatibilité fonctionnelle, tests et réécriture éventuelle |

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
  --password '<SECRET_SOURCE_NON_VERSIONNE>' \
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
  --password '<SECRET_CIBLE_NON_VERSIONNE>' \
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

<nav class="page-sequence"><a href="cours/chapitre-4/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/vpc">Suivant</a></nav>
