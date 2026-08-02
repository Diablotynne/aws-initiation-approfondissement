---
title: "3. AWS CloudFormation — Infrastructure as Code"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 3. AWS CloudFormation — Infrastructure as Code"
---

<nav class="page-sequence"><a href="cours/chapitre-5/automatisation">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/systems-manager">Suivant</a></nav>

### 3.1 Qu'est-ce que CloudFormation ?

**AWS CloudFormation** est un service qui permet de **modéliser et déployer des ressources AWS** sous forme de code.

Plutôt que de créer manuellement une VPC, des sous-réseaux, des groupes de sécurité ou des instances EC2 dans la console, on les **décrit dans un template JSON ou YAML** et CloudFormation s'occupe du reste.

```text
Analogie : CloudFormation est comme une RECETTE DE CUISINE pour construire une infrastructure

Recette classique :
  Ingrédients : 1 VPC, 2 subnets, 1 IGW, 3 EC2
  Étapes :
    1. Créer la VPC avec CIDR 10.0.0.0/16
    2. Créer subnet public 10.0.1.0/24
    3. Créer subnet privé 10.0.2.0/24
    4. Ajouter une IGW et l'attacher à la VPC
    5. Créer 3 instances EC2 dans le subnet public
    6. Configurer les groupes de sécurité

Avantage : la recette peut être réutilisée 100 fois identiquement
          et versionnée dans Git
```

---

### 3.2 Fonctionnement simplifié de CloudFormation

1. **Rédiger un template** (JSON/YAML) décrivant les ressources à créer
2. **Créer une stack** dans CloudFormation (via console ou CLI)
3. **AWS provisionne** toutes les ressources **dans le bon ordre** (résout les dépendances automatiquement)
4. **Mettre à jour** la stack pour faire évoluer l'infrastructure (ajout, suppression, modification de ressources)
5. **Supprimer** la stack si plus besoin (CloudFormation supprime toutes les ressources associées)

<a class="schema-zoom" href="assets/schemas/cloudformation-flow.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/cloudformation-flow.svg"
     alt="Flux simplifié de CloudFormation"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Le template décrit l'état attendu. CloudFormation analyse les dépendances, appelle les API AWS et regroupe les ressources obtenues dans une stack. Une mise à jour compare la nouvelle déclaration à la stack existante avant d'appliquer les changements nécessaires.

---

### 3.3 Exemple simple de template CloudFormation (YAML)

Voici un template minimaliste créant une VPC, un subnet, et une instance EC2 :

```yaml
# Version du format CloudFormation (toujours 2010-09-09)
AWSTemplateFormatVersion: '2010-09-09'

# Description brève du template
Description: |
  Infrastructure AWS simple pour débutants
  Crée une VPC, un subnet public, une IGW et une instance EC2

# Paramètres (permet de rendre le template réutilisable)
# Ici, on paramètre le type d'instance EC2 pour pouvoir changer facilement
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    Description: Type d'instance EC2 (t2.micro, t2.small, etc.)
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro

# Les ressources à créer
Resources:
  # Ressource 1 : Créer une VPC
  # Chaque ressource a un identifiant logique (MonVPC) et un type AWS
  MonVPC:
    # Type de ressource AWS
    Type: AWS::EC2::VPC
    # Propriétés spécifiques
    Properties:
      # CIDR block : plage d'adresses IP pour cette VPC
      CidrBlock: 10.0.0.0/16
      # Activer le hostname DNS
      EnableDnsHostnames: true
      # Tags pour identifier facilement
      Tags:
        - Key: Name
          Value: MonVPC-Formation

  # Ressource 2 : Créer un subnet public dans la VPC
  # !Ref est une fonction intrinsèque qui referéce une autre ressource
  MonSubnetPublic:
    Type: AWS::EC2::Subnet
    Properties:
      # !Ref MonVPC = ID logique de la VPC créée au-dessus
      VpcId: !Ref MonVPC
      # Plage d'adresses pour ce subnet (doit être dans le CIDR de la VPC)
      CidrBlock: 10.0.1.0/24
      # Zone de disponibilité (peut varier, AWS en assigne une par défaut)
      AvailabilityZone: eu-west-1a
      # Assigner automatiquement une IP publique aux instances
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: MonSubnet-Public

  # Ressource 3 : Créer une Internet Gateway (permet l'accès à Internet)
  MonIGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MonIGW

  # Ressource 4 : Attacher l'IGW à la VPC
  # CloudFormation comprend automatiquement que cette ressource dépend de MonVPC et MonIGW
  AttachIGW:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      # Référence à la VPC créée
      VpcId: !Ref MonVPC
      # Référence à l'IGW créé
      InternetGatewayId: !Ref MonIGW

  # Ressource 5 : Groupe de sécurité (pare-feu simplifié)
  MonSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      # Description obligatoire
      GroupDescription: Autorise SSH et HTTP
      # Associer au VPC
      VpcId: !Ref MonVPC
      # Règles de trafic entrant
      SecurityGroupIngress:
        # Permettre SSH depuis n'importe quelle IP (⚠️ À ÉVITER EN PROD)
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: SSH access
        # Permettre HTTP
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP access
      Tags:
        - Key: Name
          Value: MonSG-Formation

  # Ressource 6 : Instance EC2
  MonInstance:
    Type: AWS::EC2::Instance
    Properties:
      # AMI ID (Ubuntu 24.04 LTS en eu-west-1)
      ImageId: ami-0d71ea30463e0ff8d
      # Type d'instance (utilise le paramètre défini au-dessus)
      InstanceType: !Ref InstanceType
      # Placer dans le subnet créé
      SubnetId: !Ref MonSubnetPublic
      # Associer le security group
      SecurityGroupIds:
        - !Ref MonSecurityGroup
      # Script à exécuter au démarrage (user data)
      UserData:
        Fn::Base64: |
          #!/bin/bash
          # Mises à jour système
          apt-get update
          apt-get install -y nginx curl
          # Démarrer Nginx
          systemctl start nginx
          systemctl enable nginx
          # Créer une page de test
          echo "<h1>Bonjour du serveur créé par CloudFormation</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: MonServeur-Formation

# Outputs : affiche les résultats après déploiement
# Utile pour récupérer les IP, URLs, etc.
Outputs:
  # Affiche l'ID de la VPC créée
  VPCId:
    Description: ID de la VPC
    # !Ref récupère l'ID physique de la ressource
    Value: !Ref MonVPC
    # Export permet à d'autres stacks de référencer cette valeur
    Export:
      Name: MonVPC-Id

  # Affiche l'IP publique de l'instance
  PublicIP:
    Description: Adresse IP publique de l'instance EC2
    # !GetAtt récupère un attribut spécifique d'une ressource
    Value: !GetAtt MonInstance.PublicIp
    Export:
      Name: MonServeur-PublicIP

  # Affiche l'URL HTTP pour accéder au serveur
  WebServerURL:
    Description: URL pour accéder au serveur web
    # !Sub remplace les variables ${...} par leurs valeurs
    Value: !Sub 'http://${MonInstance.PublicIp}'
```

---
##### 3.3 (suite) — Exemple Production : VPC + Serveur Web Apache

Voici un template **production-ready** déployant une **VPC complète avec serveur web Apache** et un Security Group. C'est le template utilisé dans les ateliers Dawan.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: |
  Déploiement d'une VPC + serveur web Apache via CloudFormation
  - VPC avec CIDR 10.0.0.0/16
  - 1 subnet public (10.0.1.0/24)
  - Internet Gateway + route vers l'extérieur
  - Security Group (SSH + HTTP)
  - Instance EC2 t3.micro avec Apache2 automatiquement installé
  - Output : URL publique du serveur web

# Paramètres pour rendre le template réutilisable
Parameters:
  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nom de la paire de clés EC2 existante (pour SSH)

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t2.micro
    Description: Type d'instance EC2

# Les ressources à créer
Resources:
  # VPC
  FormationVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Formation-VPC-WebLab

  # Subnet public
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref FormationVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: eu-west-3a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: Formation-Subnet-Public

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Formation-IGW

  # Attacher IGW à la VPC
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref FormationVPC
      InternetGatewayId: !Ref InternetGateway

  # Table de routage publique
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref FormationVPC
      Tags:
        - Key: Name
          Value: Formation-PublicRouteTable

  # Route vers l'IGW (tout ce qui va en dehors du VPC)
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # Associer la route table au subnet
  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # Security Group (pare-feu)
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permettre HTTP et SSH
      VpcId: !Ref FormationVPC
      SecurityGroupIngress:
        # SSH (port 22) - accès depuis n'importe où (⚠️ en production : limiter à votre IP)
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: SSH access
        # HTTP (port 80) - serveur web public
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP web server
      Tags:
        - Key: Name
          Value: Formation-WebServerSG

  # Instance EC2
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-04a92520784b94538  # Amazon Linux 2023 (eu-west-3)
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      # Script pour configurer Apache2
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          # Mise à jour du système
          yum update -y

          # Installer Apache HTTP Server
          yum install -y httpd

          # Activer et démarrer Apache
          systemctl enable httpd
          systemctl start httpd

          # Créer une page HTML de test
          cat > /var/www/html/index.html << 'ENDHTML'
          <!DOCTYPE html>
          <html>
          <head>
              <title>CloudFormation - Formation AWS</title>
              <style>
                  body { font-family: Arial, sans-serif; margin: 40px; }
                  h1 { color: #FF9900; }
              </style>
          </head>
          <body>
              <h1>Bienvenue sur votre serveur web CloudFormation!</h1>
              <p>Cette instance EC2 a été déployée automatiquement via CloudFormation.</p>
              <p><strong>Informations serveur :</strong></p>
              <ul>
                  <li>Instance ID : ${AWS::StackId}</li>
                  <li>Région : ${AWS::Region}</li>
                  <li>Instance Type : ${InstanceType}</li>
              </ul>
          </body>
          </html>
          ENDHTML

      Tags:
        - Key: Name
          Value: Formation-WebServer

# Outputs - affiche les résultats utiles après déploiement
Outputs:
  WebServerURL:
    Description: URL publique du serveur web
    Value: !Sub 'http://${WebServerInstance.PublicDnsName}'
    Export:
      Name: !Sub '${AWS::StackName}-WebServerURL'

  WebServerPublicIP:
    Description: Adresse IP publique de l'instance
    Value: !Sub '${WebServerInstance.PublicIp}'

  SSHCommand:
    Description: Commande SSH pour se connecter au serveur
    Value: !Sub 'ssh -i /chemin/vers/cle.pem ec2-user@${WebServerInstance.PublicDnsName}'

  SecurityGroupId:
    Description: ID du Security Group
    Value: !Ref WebServerSecurityGroup
```

##### Déployer ce template

> [!info]
> Une activité pratique permet d’approfondir le déploiement de ce template CloudFormation.


```bash
# 1. Créer la stack depuis le fichier YAML local
aws cloudformation create-stack \
  --stack-name formation-webserver-lab \
  --template-body file://vpc-webserver.yaml \
  --parameters \
    ParameterKey=KeyPairName,ParameterValue=ma-clé-ssh \
    ParameterKey=InstanceType,ParameterValue=t3.micro \
  --region eu-west-3

# Attendre que la stack soit créée (statut CREATE_COMPLETE)
aws cloudformation wait stack-create-complete \
  --stack-name formation-webserver-lab \
  --region eu-west-3

# 2. Récupérer l'URL publique du serveur
aws cloudformation describe-stacks \
  --stack-name formation-webserver-lab \
  --query 'Stacks[0].Outputs[?OutputKey==`WebServerURL`].OutputValue' \
  --output text \
  --region eu-west-3

# Output : http://ec2-12-34-56-78.eu-west-3.compute.amazonaws.com
# → Ouvrir cette URL dans un navigateur pour voir le serveur Apache

# 3. Supprimer toute l'infrastructure quand vous avez terminé
aws cloudformation delete-stack \
  --stack-name formation-webserver-lab \
  --region eu-west-3

# Attendre la suppression complète
aws cloudformation wait stack-delete-complete \
  --stack-name formation-webserver-lab \
  --region eu-west-3
```

> [!tip]
> **Résultat attendu :**
> ```json
> # create-stack :
> {
>     "StackId": "arn:aws:cloudformation:eu-west-3:123456789012:stack/formation-webserver-lab/b2c3d4e5-f6a7-8901-bcde-f12345678901"
> }
>
> # (après wait stack-create-complete — ~3 à 5 minutes)
> # describe-stacks → WebServerURL :
> http://ec2-15-236-78-42.eu-west-3.compute.amazonaws.com
>
> # Le navigateur affiche la page HTML avec "Bienvenue sur votre serveur web CloudFormation!"
> ```


> [!warning]
> **IAM requis pour CloudFormation :** Pour déployer ce template, l'utilisateur (ou le rôle IAM) doit avoir les permissions de créer des ressources EC2, VPC, Security Groups. En entreprise, créer un rôle IAM dédié `CloudFormationDeployRole` avec les permissions nécessaires, plutôt que d'utiliser un compte admin.


> [!danger]
> **`delete-stack` supprime toutes les ressources :** La commande `delete-stack` détruit la VPC, le subnet, l'IGW, le Security Group ET l'instance EC2 de manière irréversible. Toutes les données stockées sur l'instance EBS seront perdues. Toujours vérifier `--stack-name` avant d'exécuter.


##### Points clés de ce template production

| Élément | Explication | Bonne pratique |
|---------|------------|----------------|
| **VPC CIDR 10.0.0.0/16** | Classe privée standard pour les VPC | Utiliser RFC 1918 (10.x, 172.16.x, 192.168.x) |
| **Subnet 10.0.1.0/24** | Plage de 251 adresses disponibles | Laisser de l'espace pour futurs subnets |
| **IGW + Route 0.0.0.0/0** | Rend le subnet public et accessible d'Internet | Nécessaire pour un serveur web public |
| **Security Group restrictif** | HTTP (80) + SSH (22) explicites | En prod : limiter SSH à des IPs spécifiques |
| **UserData script** | Configure Apache automatiquement au lancement | Évite la configuration manuelle post-lancement |
| **Outputs** | Affiche l'URL et l'IP après déploiement | Indispensable pour que l'utilisateur sache comment accéder |
| **Tags** | Identification et traçabilité | Permet le suivi des ressources pour la facturation |

---


### 3.4 Déployer le template avec AWS CLI

Une fois le template rédigé, on peut le déployer depuis la ligne de commande :

> [!info]
> **Valider le template avant déploiement :** Avant de créer une stack, il est conseillé de valider la syntaxe YAML avec `aws cloudformation validate-template --template-body file://mon-fichier.yaml`. Cette commande vérifie la syntaxe mais pas la validité des valeurs (AMI ID, type d'instance, etc.).


```bash
# 1. Créer la stack (remplacer mon-fichier.yaml par le chemin réel)
aws cloudformation create-stack \
  --stack-name ma-premiere-stack \
  --template-body file://mon-fichier.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
  --region eu-west-1

# Résultat attendu : StackId
# Output : arn:aws:cloudformation:eu-west-1:123456789:stack/ma-premiere-stack/guid

# 2. Suivre la progression du déploiement
aws cloudformation describe-stack-events \
  --stack-name ma-premiere-stack \
  --region eu-west-1

# 3. Une fois CREATE_COMPLETE, afficher les outputs
aws cloudformation describe-stacks \
  --stack-name ma-premiere-stack \
  --query 'Stacks[0].Outputs' \
  --region eu-west-1

# 4. Pour mettre à jour la stack (ex. changer le type d'instance)
aws cloudformation update-stack \
  --stack-name ma-premiere-stack \
  --template-body file://mon-fichier.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small \
  --region eu-west-1

# 5. Supprimer la stack (attention : cela supprime TOUTES les ressources)
aws cloudformation delete-stack \
  --stack-name ma-premiere-stack \
  --region eu-west-1
```

> [!tip]
> **Résultat attendu :**
> ```json
> # create-stack :
> {
>     "StackId": "arn:aws:cloudformation:eu-west-1:123456789012:stack/ma-premiere-stack/a1b2c3d4-e5f6-7890-abcd-ef1234567890"
> }
>
> # describe-stack-events (extrait) :
> {
>     "StackEvents": [
>         {
>             "StackId": "arn:aws:cloudformation:eu-west-1:...",
>             "EventId": "...",
>             "ResourceStatus": "CREATE_COMPLETE",
>             "ResourceType": "AWS::EC2::Instance",
>             "LogicalResourceId": "MonInstance",
>             "Timestamp": "2026-03-24T10:32:15.000Z"
>         },
>         {
>             "ResourceStatus": "CREATE_COMPLETE",
>             "ResourceType": "AWS::CloudFormation::Stack",
>             "LogicalResourceId": "ma-premiere-stack"
>         }
>     ]
> }
>
> # describe-stacks (Outputs) :
> [
>     {
>         "OutputKey": "PublicIP",
>         "OutputValue": "54.12.34.56",
>         "Description": "Adresse IP publique de l'instance EC2"
>     },
>     {
>         "OutputKey": "WebServerURL",
>         "OutputValue": "http://54.12.34.56",
>         "Description": "URL pour accéder au serveur web"
>     }
> ]
> ```


> [!danger]
> **Attention — `delete-stack` supprime TOUTES les ressources !** La commande `aws cloudformation delete-stack` détruit définitivement toutes les ressources créées par la stack (instances EC2, VPC, RDS, S3…). Il n'y a pas de corbeille. Assurez-vous d'avoir des sauvegardes et de cibler la bonne stack avant d'exécuter cette commande.


---

### 3.5 Avantages de CloudFormation

| Avantage | Bénéfice pédagogique |
|----------|----------------------|
| **Automation complète** | Créer/détruire une infrastructure complexe en quelques minutes |
| **Gestion des dépendances** | CloudFormation sait que l'IGW doit être créée AVANT d'être attachée à la VPC |
| **Versionning** | Stocker les templates dans Git, tracer tous les changements |
| **Reproductibilité** | Déployer exactement la même infrastructure en 10 environnements différents |
| **Rollback** | Si une erreur survient, CloudFormation annule les changements automatiquement |
| **Coût** | CloudFormation est gratuit (on paie seulement les ressources créées) |

> [!warning]
> **CloudFormation Drift — Divergence de configuration :** Si vous modifiez manuellement des ressources gérées par CloudFormation (via la console ou la CLI), elles entrent en état de **drift** — elles ne correspondent plus au template. CloudFormation ne détecte pas ces écarts automatiquement. Utilisez `aws cloudformation detect-stack-drift --stack-name <nom>` pour identifier les ressources divergentes. Règle d'or : **ne jamais modifier manuellement une ressource gérée par CloudFormation**.


> **Référence** : [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)

---

<nav class="page-sequence"><a href="cours/chapitre-5/automatisation">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/systems-manager">Suivant</a></nav>
