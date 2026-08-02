---
title: "3. Construire une VPC en CLI"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - 3. Construire une VPC en CLI"
---

<nav class="page-sequence"><a href="cours/chapitre-4/vpc">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/route-53">Suivant</a></nav>

> [!info]
> Une activité pratique permet d’approfondir la construction d’un VPC complet.

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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

---

<nav class="page-sequence"><a href="cours/chapitre-4/vpc">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/route-53">Suivant</a></nav>
