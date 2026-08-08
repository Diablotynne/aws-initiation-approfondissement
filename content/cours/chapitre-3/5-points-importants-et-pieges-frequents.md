---
title: "5. Points importants et pièges fréquents"
description: "\"Chapitre 3 — Amazon VPC et bases de données AWS\" - 5. Points importants et pièges fréquents"
---

<nav class="page-sequence"><a href="cours/chapitre-3/4-amazon-elasticache-mise-en-cache-distribuee">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/6-construire-une-vpc-en-cli">Suivant</a></nav>

### 5.1 Pièges RDS et Bases de données

Ce chapitre a couvert beaucoup de mécanismes RDS/Aurora en détail ; voici les confusions les plus fréquentes qui reviennent en certification comme en production :

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **RDS n'est pas auto-scalable en stockage** | Faut augmenter manuellement (RDS) ou activer auto-scaling (Aurora) | Saturation disque → downtime | Vérifier "Storage autoscaling" dans RDS config |
| **Multi-AZ RDS ≠ haute disponibilité lue** | Multi-AZ = résilience (failover), pas scalabilité lecture | Bottleneck en lecture malgré Multi-AZ | Ajouter **Read Replicas** (asynchrones) |
| **Chiffrement RDS doit être activé à la création** | On ne peut pas l'activer après coup | Recréer l'instance = downtime | Checker "Encrypt at rest" lors création |
| **Read Replica ≠ Multi-AZ** | Replica = asynchrone, pour lectures. Multi-AZ = synchrone, failover | Confondre les deux gâche design | Multi-AZ pour haute dispo, Replicas pour scalabilité lecture |
| **Snapshot RDS = backup manuel** | Snapshots manuels ne s'auto-suppriment pas | Surcoûts stockage | Supprimer manuellement ou appliquer cycle vie |

Les deux pièges les plus fréquemment testés en examen sont la confusion Multi-AZ/Read Replica et l'impossibilité de chiffrer après coup — gardez ces deux réflexes en tête avant de passer au réseau.

### 5.2 Pièges VPC et Réseau

Le réseau concentre le plus grand nombre de pièges de ce chapitre, du fait de la diversité des composants (Peering, Security Groups, NACL, Transit Gateway, ENI) :

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

Le point commun à la majorité de ces pièges réseau est une confusion entre deux mécanismes qui se ressemblent en apparence (SG/NACL, Peering/Transit Gateway, IGW/NAT) mais répondent à des besoins différents — relire leur définition respective lève la plupart de ces confusions.

### 5.3 Pièges Route 53 et DNS

Côté DNS, les pièges sont moins nombreux mais tout aussi coûteux en cas d'erreur, notamment sur le TTL :

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **DNS Route 53 a un TTL** | Les réponses sont cachées pendant le TTL | Changement DNS peut prendre 24h | Baisser TTL avant changement (300s) |
| **Health Check ≠ Failover automatique** | Health check détecte panne, failover redirection | Juste détecter ne suffit pas | Configurer failover + health check |
| **Alias Records ≠ CNAME** | Alias = pointeur AWS (gratuit, flexible), CNAME = alias DNS classique | Confondre risque problèmes CNAME root | Toujours Alias pour AWS resources (ALB, CloudFront) |

> [!warning]
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

DynamoDB fonctionne différemment d'une base relationnelle, ce qui génère des erreurs de conception typiques chez les habitués du SQL :

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **DynamoDB provisionned vs on-demand** | Mode provisionné = moins cher si prévisible | Charge imprévisible = throttling ou surcoûts | Choisir on-demand si variable, provisionné si stable |
| **Partition key ≠ Sort key** | Partition = hash (required), Sort = range (optional) | Query sans sort key = full table scan | Bien concevoir partition + sort key |
| **DynamoDB TTL n'est pas immédiat** | TTL supprime dans 24-48h après expiration | Données restent visibles brièvement | Ne pas compter sur TTL pour sécurité |
| **Global Secondary Index (GSI) coûte** | GSI = throughput supplémentaire à provisionner | Surcoûts si GSI mal utilisés | Bien planifier projections, ne créer que GSI utiles |

Le piège de la partition/sort key est le plus structurant : mal le concevoir dès le départ oblige souvent à recréer la table entière, DynamoDB ne permettant pas de modifier ces clés après coup.

### 5.5 Pièges ElastiCache

Côté cache, les pièges tiennent surtout à la confusion entre les deux moteurs et à une mauvaise gestion de la fraîcheur des données :

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **Redis vs Memcached** | Redis = persistance, structures complexes. Memcached = volatil, simple | Choisir Memcached pour données durables = perte | Redis pour sessions, Memcached pour cache éphémère |
| **Cache-Aside pattern = possibilité cache stale** | TTL peut garder données anciennes 1h | Utilisateurs voient données obsolètes | Réduire TTL ou implémenter invalidation manuelle |
| **ElastiCache dans VPC ≠ accessible depuis EC2 autre subnet** | Besoin Security Group + route table | EC2 ne peut pas accéder cache | Vérifier SG ElastiCache permet EC2, même VPC |
| **Cluster mode disabled : une seule shard** | Pas de sharding = un seul nœud max CPU | Bottleneck CPU même avec plusieurs replicas | Cluster mode enabled pour scalabilité |

Le troisième piège rappelle qu'ElastiCache reste soumis aux mêmes règles réseau qu'EC2 ou RDS : être dans le même VPC ne suffit pas, il faut aussi que les Security Groups autorisent explicitement le flux entre l'application et le cache.

### 5.6 Pièges Architecture Générale

Ces derniers pièges dépassent un service unique et concernent des choix d'architecture transverses vus dans ce chapitre :

| Piège | Réalité | Conséquence | Solution |
|-------|---------|-----------|----------|
| **Aurora Serverless = scaling pas instantané** | Scaling automatique peut durer 30-60s | Latence pics pendant scaling | Pas idéal real-time, mieux RDS provisionné |
| **VPC Endpoint S3 évite la NAT** | Mais doit configurer policies explicites | S3 accès reste privé mais règles complexes | Créer endpoint + bucket policy restrictive |
| **VPC Endpoint = interface privée (coût)** | Interface endpoint = ENI = $0.007/h | Nombreux endpoints = facture élevée | Gateway endpoint pour S3/DynamoDB (gratuit) |

Ce dernier piège illustre un principe général de ce chapitre : sur AWS, chaque mécanisme d'automatisation ou de connectivité a un modèle de coût propre, qu'il faut vérifier avant de le généraliser à grande échelle plutôt que de supposer qu'il est gratuit par défaut.

---

<nav class="page-sequence"><a href="cours/chapitre-3/4-amazon-elasticache-mise-en-cache-distribuee">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/6-construire-une-vpc-en-cli">Suivant</a></nav>
