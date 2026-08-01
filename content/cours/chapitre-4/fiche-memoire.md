---
title: "Cheat sheet — Réseau et données"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - Cheat sheet — Réseau et données"
---

<nav class="page-sequence"><a href="cours/chapitre-4/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/travaux-pratiques">Suivant</a></nav>

### Chemin d'un paquet dans un VPC

```text
client → route → passerelle éventuelle → NACL → security group → interface réseau → service
```

Une route indique une destination et une cible ; elle n'autorise pas à elle seule le trafic. Le chemin retour, la résolution DNS et les contrôles de sécurité doivent aussi être valides.

### Composants réseau

| Besoin | Composant | Repère |
|---|---|---|
| Isoler un réseau logique | Amazon VPC | Plage CIDR et paramètres DNS |
| Segmenter le VPC | Sous-réseau | Une seule zone de disponibilité par sous-réseau |
| Entrée/sortie Internet pour ressource publique | Internet Gateway + route | Une adresse publique et les règles de sécurité restent nécessaires |
| Sortie Internet IPv4 d'un sous-réseau privé | NAT Gateway dans un sous-réseau public | Ne fournit pas d'entrée initiée depuis Internet |
| Filtrer une interface | Security Group | Avec état, règles d'autorisation uniquement |
| Filtrer un sous-réseau | Network ACL | Sans état, règles d'autorisation et de refus, trafic retour à prévoir |
| Relier deux VPC | VPC Peering ou Transit Gateway selon l'échelle | Le peering n'est pas transitif |
| Résoudre et orienter le DNS | Amazon Route 53 | Zone hébergée, enregistrement, TTL et politique de routage |

### Calcul CIDR rapide

| CIDR IPv4 | Nombre total d'adresses | Exemple |
|---|---:|---|
| `/16` | 65 536 | `10.0.0.0/16` |
| `/20` | 4 096 | `10.0.16.0/20` |
| `/24` | 256 | `10.0.1.0/24` |
| `/28` | 16 | `10.0.1.0/28` |

AWS réserve cinq adresses IPv4 dans chaque sous-réseau. Vérifier aussi l'absence de chevauchement avant toute interconnexion.

```bash
# Cartographier VPC, sous-réseaux et tables de routage
aws ec2 describe-vpcs --query 'Vpcs[].[VpcId,CidrBlock,IsDefault]' --output table
aws ec2 describe-subnets \
  --query 'Subnets[].[SubnetId,VpcId,AvailabilityZone,CidrBlock,MapPublicIpOnLaunch]' \
  --output table
aws ec2 describe-route-tables --output table

# Examiner les contrôles réseau
aws ec2 describe-security-groups --output table
aws ec2 describe-network-acls --output table
```

### Choisir un service de données

| Besoin | Service à examiner | Décision structurante |
|---|---|---|
| SQL, relations et transactions | Amazon RDS | Moteur, classe, stockage, sauvegardes et déploiement Multi-AZ |
| Compatibilité MySQL/PostgreSQL optimisée AWS | Amazon Aurora | Capacité, réplication et mode de déploiement |
| Clé-valeur ou document à grande échelle | Amazon DynamoDB | Clé de partition, clé de tri et mode de capacité |
| Cache en mémoire | Amazon ElastiCache | Redis/Valkey ou Memcached, stratégie d'invalidation et tolérance à la perte |

### Disponibilité, lecture et reprise

- **RDS Multi-AZ** vise la disponibilité et le basculement ; l'instance de secours n'est pas une cible de lecture applicative ordinaire.
- Une **réplique en lecture** sert principalement à répartir les lectures et utilise une réplication asynchrone.
- Un **snapshot** permet une restauration à un nouvel état de base ; il ne constitue pas un basculement instantané.
- Dans DynamoDB, une mauvaise clé de partition peut concentrer les accès et dégrader la répartition de charge.

```bash
# Inventaire des bases relationnelles
aws rds describe-db-instances \
  --query 'DBInstances[].[DBInstanceIdentifier,Engine,DBInstanceStatus,MultiAZ,PubliclyAccessible]' \
  --output table

# Inventaire DynamoDB
aws dynamodb list-tables --output table
```

### Diagnostic réseau

1. Vérifier l'adresse source et la destination réellement utilisées.
2. Contrôler la résolution DNS.
3. Lire la table de routage associée au sous-réseau.
4. Vérifier security groups des deux côtés, puis NACL aller et retour.
5. Vérifier passerelle, NAT, peering ou Transit Gateway selon le chemin attendu.
6. Tester le port applicatif, puis consulter journaux et métriques.

<nav class="page-sequence"><a href="cours/chapitre-4/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/travaux-pratiques">Suivant</a></nav>
