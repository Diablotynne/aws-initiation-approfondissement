---
title: "Cheat sheet AWS"
description: "Commandes, syntaxes et reperes AWS utilises pendant la formation."
---

Référence rapide des commandes CLI et des templates types utilisés dans la formation. Pour les définitions des termes, voir le glossaire.

---

## AWS CLI dans CloudShell

Les commandes sont exécutées depuis **AWS CloudShell** dans la console temporaire du lab. Ne lancez pas `aws configure` et ne copiez aucune clé sur votre poste.

```bash
aws sts get-caller-identity            # vérifier l'identité courante
aws configure get region               # afficher la région par défaut du lab
aws ec2 describe-regions --output table # lister les régions visibles
```

---

## IAM

```bash
# Utilisateurs et groupes
aws iam create-user --user-name alice
aws iam list-users
aws iam delete-user --user-name alice
aws iam create-group --group-name Developers
aws iam add-user-to-group --user-name alice --group-name Developers

# Politiques
aws iam list-policies --scope Local
aws iam create-policy --policy-name MaPolicy --policy-document file://policy.json
aws iam attach-user-policy --user-name alice --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam attach-group-policy --group-name Developers --policy-arn <arn>

# Rôles
aws iam create-role --role-name MonRole --assume-role-policy-document file://trust.json
aws iam attach-role-policy --role-name MonRole --policy-arn <arn>
aws iam list-roles --query 'Roles[*].[RoleName,Arn]' --output table

# Clés d'accès
aws iam create-access-key --user-name alice
aws iam list-access-keys --user-name alice
aws iam delete-access-key --user-name alice --access-key-id <id>
```

**Priorité d'évaluation IAM** : Deny explicite > Allow explicite > Deny implicite (par défaut).

**Structure d'une policy JSON :**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::mon-bucket", "arn:aws:s3:::mon-bucket/*"]
    }
  ]
}
```

---

## STS

```bash
aws sts get-caller-identity

aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/mon-role \
  --role-session-name ma-session \
  --duration-seconds 3600
# Retourne : AccessKeyId, SecretAccessKey, SessionToken, Expiration
```

---

## S3

```bash
# Buckets
aws s3 mb s3://mon-bucket --region eu-west-3
aws s3 ls
aws s3 rb s3://mon-bucket --force

# Objets
aws s3 cp fichier.txt s3://mon-bucket/
aws s3 cp s3://mon-bucket/fichier.txt ./
aws s3 sync ./dossier s3://mon-bucket/dossier/ --delete
aws s3 ls s3://mon-bucket/ --recursive
aws s3 rm s3://mon-bucket/fichier.txt
aws s3 rm s3://mon-bucket/ --recursive
aws s3 presign s3://mon-bucket/rapport.pdf --expires-in 3600

# Configuration via s3api
aws s3api put-bucket-versioning --bucket mon-bucket \
  --versioning-configuration Status=Enabled
aws s3api put-bucket-policy --bucket mon-bucket --policy file://policy.json
aws s3api get-bucket-policy --bucket mon-bucket
aws s3api put-public-access-block --bucket mon-bucket \
  --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

**Classes de stockage S3** :

| Classe | Usage | Disponibilité |
|--------|-------|---------------|
| Standard | Données fréquentes | 99,99 % |
| Standard-IA | Accès rare, récup rapide | 99,9 % |
| One Zone-IA | Accès rare, 1 AZ | 99,5 % |
| Glacier Instant | Archive, récup ms | 99,9 % |
| Glacier Flexible | Archive, récup mn-h | 99,99 % |
| Glacier Deep Archive | Archive longue durée | 99,99 % |
| Intelligent-Tiering | Accès variable, auto | 99,9 % |

---

## EC2

```bash
# Instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' --output table
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name ma-cle \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx \
  --count 1
aws ec2 start-instances --instance-ids i-xxx
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx
aws ec2 create-tags --resources i-xxx --tags Key=Name,Value=mon-serveur

# Key pairs
aws ec2 create-key-pair --key-name ma-cle --query 'KeyMaterial' --output text > ma-cle.pem
chmod 400 ma-cle.pem
aws ec2 describe-key-pairs

# Security Groups
aws ec2 create-security-group --group-name mon-sg --description "Mon SG" --vpc-id vpc-xxx
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 describe-security-groups --group-ids sg-xxx

# AMIs
aws ec2 describe-images --owners self
aws ec2 create-image --instance-id i-xxx --name "mon-ami" --no-reboot
```

**Types d'instances courants** :

| Famille | Usage | Exemple |
|---------|-------|---------|
| t3/t4g | Usage général (burstable) | t3.micro, t3.small |
| m6i | Usage général équilibré | m6i.large |
| c6i | Calcul intensif | c6i.xlarge |
| r6i | Mémoire optimisée | r6i.large |
| p3/g4 | GPU | p3.2xlarge |

---

## VPC & Réseau

```bash
# VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 describe-vpcs
aws ec2 modify-vpc-attribute --vpc-id vpc-xxx --enable-dns-hostnames

# Subnets
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24 --availability-zone eu-west-3a
aws ec2 modify-subnet-attribute --subnet-id subnet-xxx --map-public-ip-on-launch

# Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxx --vpc-id vpc-xxx

# NAT Gateway
aws ec2 allocate-address --domain vpc
aws ec2 create-nat-gateway --subnet-id subnet-xxx --allocation-id eipalloc-xxx

# Route Tables
aws ec2 create-route-table --vpc-id vpc-xxx
aws ec2 create-route --route-table-id rtb-xxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxx
aws ec2 associate-route-table --route-table-id rtb-xxx --subnet-id subnet-xxx
```

**SG vs NACL** : Security Group = stateful, niveau instance, allow only. NACL = stateless, niveau subnet, allow + deny.

---

## RDS

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name mon-subnet-group \
  --db-subnet-group-description "Subnet group formation" \
  --subnet-ids subnet-aaa subnet-bbb

aws rds create-db-instance \
  --db-instance-identifier mon-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MonMotDePasse! \
  --allocated-storage 20 \
  --db-subnet-group-name mon-subnet-group \
  --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier mon-db
aws rds describe-db-instances --query 'DBInstances[0].Endpoint.Address'
aws rds stop-db-instance --db-instance-identifier mon-db
aws rds start-db-instance --db-instance-identifier mon-db

# Snapshots
aws rds create-db-snapshot --db-instance-identifier mon-db --db-snapshot-identifier mon-snapshot
aws rds describe-db-snapshots
aws rds delete-db-instance --db-instance-identifier mon-db --skip-final-snapshot
```

---

## Lambda

```bash
zip function.zip lambda_function.py

aws lambda create-function \
  --function-name ma-fonction \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/LambdaRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 128

aws lambda invoke \
  --function-name ma-fonction \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  response.json

aws lambda update-function-code --function-name ma-fonction --zip-file fileb://function-v2.zip
aws lambda update-function-configuration --function-name ma-fonction --timeout 60 --memory-size 256
aws lambda list-functions --query 'Functions[*].[FunctionName,Runtime,LastModified]' --output table
aws logs tail /aws/lambda/ma-fonction --follow
```

---

## CloudFormation

```bash
aws cloudformation validate-template --template-body file://template.yaml

aws cloudformation create-stack \
  --stack-name mon-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,ParameterValue=prod \
  --capabilities CAPABILITY_IAM

aws cloudformation wait stack-create-complete --stack-name mon-stack
aws cloudformation describe-stacks --stack-name mon-stack --query 'Stacks[0].Outputs'
aws cloudformation describe-stack-events --stack-name mon-stack
aws cloudformation update-stack --stack-name mon-stack --template-body file://template.yaml
aws cloudformation delete-stack --stack-name mon-stack
aws cloudformation detect-stack-drift --stack-name mon-stack
```

**Fonctions intrinsèques** :

| Fonction | Usage |
|----------|-------|
| `!Ref LogicalId` | Référencer une ressource ou un paramètre |
| `!GetAtt Resource.Attr` | Attribut d'une ressource |
| `!Sub 'texte ${Var}'` | Substitution de variable |
| `!Join [sep, [a,b]]` | Concaténer des valeurs |
| `!Select [idx, liste]` | Sélectionner dans une liste |
| `!If [cond, vrai, faux]` | Condition |
| `!ImportValue nom` | Importer un Output d'un autre stack |

**Template de référence — VPC + EC2 + Security Group :**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  SubnetPublic:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  IGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  RouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  RouteDefault:
    Type: AWS::EC2::Route
    DependsOn: IGWAttachment
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref SubnetPublic
      RouteTableId: !Ref RouteTable
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SG formation — HTTP/HTTPS
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - {IpProtocol: tcp, FromPort: 80, ToPort: 80, CidrIp: 0.0.0.0/0}
        - {IpProtocol: tcp, FromPort: 443, ToPort: 443, CidrIp: 0.0.0.0/0}
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !Ref InstanceType
      SubnetId: !Ref SubnetPublic
      SecurityGroupIds: [!Ref SecurityGroup]
Outputs:
  PublicIP:
    Value: !GetAtt EC2Instance.PublicIp
```

**Template de référence — Lambda + API Gateway HTTP :**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - {Effect: Allow, Principal: {Service: lambda.amazonaws.com}, Action: sts:AssumeRole}
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
  HelloFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Role: !GetAtt LambdaRole.Arn
      Handler: index.handler
      Code:
        ZipFile: |
          def handler(event, context):
              return {'statusCode': 200, 'body': 'Bonjour depuis Lambda !'}
  HttpApi:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      ProtocolType: HTTP
      Target: !GetAtt HelloFunction.Arn
  LambdaPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref HelloFunction
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${HttpApi}/*
Outputs:
  ApiUrl:
    Value: !Sub https://${HttpApi}.execute-api.${AWS::Region}.amazonaws.com/
```

---

## Auto Scaling & Load Balancing

```bash
aws elbv2 create-load-balancer --name mon-alb --subnets subnet-111 subnet-222 --security-groups sg-xxx --type application
aws elbv2 create-target-group --name mon-tg --protocol HTTP --port 80 --vpc-id vpc-xxx

aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name mon-asg \
  --launch-template "LaunchTemplateName=mon-template" \
  --min-size 1 --max-size 5 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-111,subnet-222"
```

---

## Elastic Beanstalk

```bash
eb init -p node.js-18 monapp --region eu-west-3
eb create monapp-env --instance-type t3.small --min-instances 1 --max-instances 3
eb status
eb open
eb logs
eb deploy
eb scale 2
eb terminate monapp-env
```

---

## Systems Manager

```bash
# Run Command
aws ssm send-command \
  --instance-ids i-xxx \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["apt-get update","apt-get install -y apache2"]'
aws ssm get-command-invocation --command-id <id> --instance-id i-xxx

# Parameter Store
aws ssm put-parameter --name /prod/database/password --value "..." --type "SecureString"
aws ssm get-parameter --name /prod/database/password --with-decryption

# Session Manager (SSH sans port 22 ouvert)
aws ssm start-session --target i-xxx
```

---

## ElastiCache

```bash
aws elasticache create-cache-cluster \
  --cache-cluster-id formation-redis \
  --cache-node-type cache.t3.micro \
  --engine redis --engine-version 7.0 \
  --num-cache-nodes 1 \
  --vpc-security-group-ids sg-xxx \
  --cache-subnet-group-name my-subnet-group

aws elasticache wait cache-cluster-available --cache-cluster-id formation-redis
aws elasticache describe-cache-clusters --cache-cluster-id formation-redis
```

---

## Route 53

```bash
aws route53 create-hosted-zone --name example.com --caller-reference $(date +%s)

aws route53 change-resource-record-sets --hosted-zone-id ZONE_ID --change-batch '{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {"Name": "www.example.com", "Type": "A", "TTL": 300, "ResourceRecords": [{"Value": "93.184.216.34"}]}
  }]
}'

aws route53 create-health-check --health-check-config '{
  "Type": "HTTP", "IPAddress": "93.184.216.34", "Port": 80,
  "ResourcePath": "/health", "RequestInterval": 30, "FailureThreshold": 3
}'
```

---

## SQS & SNS

```bash
# SQS
aws sqs create-queue --queue-name MonQueue
aws sqs send-message --queue-url <url> --message-body "Bonjour"
aws sqs receive-message --queue-url <url> --max-number-of-messages 1
aws sqs delete-message --queue-url <url> --receipt-handle <receipt-handle>

# SNS
aws sns create-topic --name MonTopic
aws sns subscribe --topic-arn <arn> --protocol email --notification-endpoint admin@example.com
aws sns publish --topic-arn <arn> --message "Alerte"
```

---

## CloudWatch

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "CPU-Élevé" \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-xxx \
  --alarm-actions arn:aws:sns:eu-west-3:123456789012:MonTopic

aws cloudwatch describe-alarms
aws cloudwatch delete-alarms --alarm-names "CPU-Élevé"
aws cloudwatch put-metric-data --namespace "MonApp" --metric-name "Ventes" --value 42 --unit Count
```

---

## AWS Backup

```bash
aws backup create-backup-vault --backup-vault-name mon-vault
aws backup create-backup-plan --backup-plan file://backup-plan.json
aws backup create-backup-selection --backup-plan-id <id> --backup-selection file://selection.json
aws backup list-recovery-points-by-resource --resource-arn <arn>
aws backup start-restore-job --recovery-point-arn <arn> --iam-role-arn <role-arn>
```

---

## Well-Architected Framework — 6 piliers

| Pilier | Principe clé |
|--------|-------------|
| Excellence opérationnelle | Automatiser, observer, améliorer |
| Sécurité | Moindre privilège, chiffrement, traçabilité |
| Fiabilité | Redondance, reprise, tests de panne |
| Performance | Choisir le bon service, élasticité |
| Optimisation des coûts | Pay-as-you-go, rightsizing, Reserved Instances |
| Durabilité | Réduire l'empreinte carbone |

---

## Régions et zones de disponibilité

```bash
aws ec2 describe-regions
aws ec2 describe-availability-zones --region eu-west-3
```

| Région | Nom | AZ |
|--------|-----|----|
| Paris | eu-west-3 | eu-west-3a/b/c |
| Irlande | eu-west-1 | eu-west-1a/b/c |
| Frankfurt | eu-central-1 | eu-central-1a/b/c |

---

## Filtres et sortie CLI

```bash
# JMESPath (--query)
aws ec2 describe-instances --query 'Reservations[0].Instances[0].InstanceId'
aws ec2 describe-instances --query 'Reservations[*].Instances[?State.Name==`running`].[InstanceId,PublicIpAddress]' --output table
aws s3api list-buckets --query 'length(Buckets)'

# Formats de sortie
aws ec2 describe-instances --output json    # défaut, idéal pour scripts
aws ec2 describe-instances --output table   # idéal pour lecture humaine
aws ec2 describe-instances --output text    # idéal pour scripts bash
aws ec2 describe-instances --output yaml    # AWS CLI v2

# Filtres serveur
aws ec2 describe-instances --filters Name=tag:Name,Values=mon-serveur
aws ec2 describe-instances --filters Name=instance-state-name,Values=running
```

---

## Tags et diagnostic

```bash
# Tagger et rechercher par tag
aws ec2 create-tags --resources i-xxx --tags Key=Environment,Value=Production
aws ec2 describe-instances --filters "Name=tag:Environment,Values=Production"

# Coûts
aws ce get-cost-and-usage \
  --time-period Start=2026-01-01,End=2026-01-31 \
  --granularity MONTHLY --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=Project

# Limites de compte
aws service-quotas list-service-quotas --service-code ec2
```

**Nommage des ressources — convention** :

| Ressource | Convention | Exemple |
|-----------|-----------|---------|
| VPC | `vpc-[env]-[projet]` | `vpc-prod-webapp` |
| Subnet | `sn-[public/private]-[az]` | `sn-public-a` |
| EC2 | `[role]-[env]-[numéro]` | `web-prod-01` |
| Security Group | `sg-[role]-[env]` | `sg-web-prod` |
| S3 bucket | `[org]-[projet]-[env]-[région]` | `acme-backup-prod-eu` |
| IAM Role | `role-[service]-[permissions]` | `role-ec2-s3read` |

---

## Services clés — tableau de référence

| Service | Description | Console |
|---------|-------------|---------|
| **IAM** | Gestion identités et permissions | iam.amazonaws.com |
| **EC2** | Machines virtuelles | console.aws.amazon.com/ec2 |
| **S3** | Stockage objet | console.aws.amazon.com/s3 |
| **VPC** | Réseau privé virtuel | console.aws.amazon.com/vpc |
| **RDS** | Bases de données managées | console.aws.amazon.com/rds |
| **Lambda** | Fonctions serverless | console.aws.amazon.com/lambda |
| **CloudFormation** | IaC AWS | console.aws.amazon.com/cloudformation |
| **CloudWatch** | Monitoring et logs | console.aws.amazon.com/cloudwatch |
| **Route 53** | DNS managé | console.aws.amazon.com/route53 |
| **ELB** | Load balancing | console.aws.amazon.com/ec2/v2#LoadBalancers |
| **Auto Scaling** | Scalabilité automatique | console.aws.amazon.com/ec2/autoscaling |
| **ECS** | Conteneurs Docker managés | console.aws.amazon.com/ecs |
| **EKS** | Kubernetes managé | console.aws.amazon.com/eks |
| **Secrets Manager** | Gestion des secrets | console.aws.amazon.com/secretsmanager |
| **KMS** | Gestion des clés de chiffrement | console.aws.amazon.com/kms |
