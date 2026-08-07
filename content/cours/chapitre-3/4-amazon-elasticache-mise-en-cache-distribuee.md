---
title: "4. Amazon ElastiCache — Mise en cache distribuée"
description: "\"Chapitre 3 — Amazon VPC et bases de données AWS\" - 4. Amazon ElastiCache — Mise en cache distribuée"
---

<nav class="page-sequence"><a href="cours/chapitre-3/3-amazon-route-53-dns-intelligent">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/5-points-importants-et-pieges-frequents">Suivant</a></nav>

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

> [!info]
> Une activité pratique permet d’approfondir le déploiement d’un cache Redis.

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

> [!tip]
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

<nav class="page-sequence"><a href="cours/chapitre-3/3-amazon-route-53-dns-intelligent">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/5-points-importants-et-pieges-frequents">Suivant</a></nav>
