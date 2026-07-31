# Glossaire — Formation AWS : Initiation + Approfondissement

Les termes sont organisés par grand thème de la formation puis par ordre alphabétique.

---

## Fondamentaux du Cloud

**AWS (Amazon Web Services)**
Plateforme de cloud computing d'Amazon lancée en 2006. Plus de 200 services couvrant le calcul, le stockage, les bases de données, la sécurité, l'IA, le DevOps.

**CAPEX (Capital Expenditures)**
Modèle économique traditionnel : investissement initial important (achat de serveurs), amorti sur plusieurs années.

**OPEX (Operational Expenditures)**
Modèle économique du Cloud : dépenses variables basées sur l'usage réel, sans investissement initial.

**Cloud Computing**
Modèle de mise à disposition de ressources informatiques (calcul, stockage, réseau) à la demande, facturées à l'usage, plutôt que possédées et exploitées en interne.

**Cloud hybride**
Modèle de déploiement combinant Cloud privé et Cloud public, avec circulation de données entre les deux environnements.

**Cloud privé**
Modèle de déploiement où l'infrastructure est dédiée à une seule organisation, non mutualisée avec d'autres clients.

**Cloud public**
Modèle de déploiement où les ressources sont hébergées dans les datacenters du fournisseur et mutualisées entre plusieurs clients, avec isolation logique.

**Conteneurisation**
Méthode d'isolation applicative où plusieurs applications partagent le même noyau OS, dans des conteneurs légers — par opposition à la virtualisation complète.

**IaaS (Infrastructure as a Service)**
Modèle de service où le fournisseur met à disposition des ressources de base (calcul, stockage, réseau) que le client configure lui-même. Exemple : EC2.

**NIST — Cinq caractéristiques du Cloud**
Référentiel définissant le Cloud Computing par cinq critères : libre-service à la demande, accès réseau étendu, mise en commun des ressources, élasticité rapide, mesurabilité du service.

**PaaS (Platform as a Service)**
Modèle de service où le fournisseur gère l'infrastructure et le runtime, le client se concentre sur son code. Exemple : Elastic Beanstalk.

**SaaS (Software as a Service)**
Modèle de service où le fournisseur gère toute la pile applicative, le client utilise le logiciel via un navigateur ou une API. Exemple : AWS WorkMail.

**Virtualisation**
Technologie permettant d'exécuter plusieurs machines virtuelles isolées sur un même serveur physique, via un hyperviseur.

**Well-Architected Framework**
Ensemble de bonnes pratiques AWS structurées en six piliers : Excellence opérationnelle, Sécurité, Fiabilité, Performance, Optimisation des coûts, Durabilité.

---

## Sécurité & IAM

**Access Key**
Paire de credentials (Access Key ID + Secret Access Key) permettant l'accès programmatique à AWS via CLI ou SDK. Ne jamais committer dans un dépôt Git.

**AssumeRole**
Action STS permettant à une identité de prendre temporairement les permissions d'un rôle IAM, via des credentials temporaires.

**ARN (Amazon Resource Name)**
Identifiant unique et universel de toute ressource AWS, au format `arn:partition:service:région:compte-id:type/nom`.

**Cognito Identity Pool**
Service échangeant un token externe (Cognito User Pool, Google, SAML) contre des credentials AWS temporaires, permettant à une application d'appeler directement S3, DynamoDB, etc.

**Cognito User Pool**
Annuaire d'utilisateurs managé pour applications web et mobiles — inscription, connexion, MFA, réinitialisation de mot de passe.

**CloudTrail**
Service d'audit qui enregistre tous les appels API effectués sur un compte AWS — qui a fait quoi, quand, depuis où.

**Explicit Deny**
Refus explicite dans une politique IAM, prioritaire sur tout Allow sans exception.

**Fédération d'identité**
Mécanisme permettant à des utilisateurs authentifiés par un système externe (Active Directory, Okta) d'accéder à AWS sans compte IAM natif.

**IAM (Identity and Access Management)**
Service centralisé de gestion des identités et des permissions AWS — définit qui peut faire quoi, sur quelle ressource, dans quelles conditions.

**IAM Identity Center**
Service de Single Sign-On centralisé pour accéder à plusieurs comptes AWS et applications avec un seul identifiant.

**MFA (Multi-Factor Authentication)**
Authentification à plusieurs facteurs — mot de passe + application TOTP, clé physique FIDO2, ou biométrie.

**Permission Boundary**
Politique IAM définissant le plafond maximal de permissions qu'une identité peut avoir, même si des politiques plus permissives lui sont attachées.

**Policy IAM**
Document JSON définissant des permissions accordées ou refusées, structuré en `Version`, `Statement` (`Effect`, `Action`, `Resource`, `Condition`).

**Principal**
Entité à qui s'applique une politique IAM — utilisateur, rôle, service AWS, compte, ou tous (`*`).

**Rôle IAM**
Identité AWS sans credentials permanents, assumée temporairement par un utilisateur, un service ou un compte tiers.

**SCP (Service Control Policy)**
Politique appliquée au niveau d'une OU ou d'un compte dans AWS Organizations, limitant les permissions maximales même pour le root.

**Secrets Manager**
Service de gestion sécurisée de secrets (mots de passe, clés API), avec rotation automatique et intégration native RDS.

**STS (Security Token Service)**
Service émettant des credentials temporaires (AccessKeyId, SecretAccessKey, SessionToken), utilisé pour AssumeRole et la fédération d'identité.

**Trust Policy**
Politique attachée à un rôle IAM définissant qui a le droit de l'assumer — distincte de la policy de permissions qui définit ce que le rôle peut faire.

**Utilisateur IAM**
Identité permanente associée à des identifiants (login/mot de passe ou clés d'accès), représentant une personne ou une application.

---

## Stockage S3

**ACL (Access Control List)**
Mécanisme legacy de permissions sur un bucket ou un objet S3 — préférer les Bucket Policies.

**Bucket S3**
Conteneur de stockage d'objets, au nom unique mondial, rattaché à une région.

**Bucket Policy**
Document JSON attaché directement à un bucket S3, définissant qui peut accéder à quoi.

**Classe de stockage S3**
Niveau de tarification et de disponibilité d'un objet S3 : Standard, Standard-IA, One Zone-IA, Glacier (Instant/Flexible/Deep Archive), Intelligent-Tiering.

**Durabilité 11×9**
Propriété de S3 Standard : 99,999999999 % de durabilité des données stockées.

**Lifecycle Policy (S3)**
Règle automatisant la transition d'objets entre classes de stockage ou leur suppression après un délai défini.

**Object (S3)**
Unité de base de S3 — une clé, des données (jusqu'à 5 To), des métadonnées et un ETag.

**Presigned URL**
URL temporaire signée cryptographiquement, permettant un accès direct à un objet S3 privé sans credentials AWS.

**S3 (Simple Storage Service)**
Service de stockage d'objets à haute durabilité et disponibilité, accessible via HTTPS.

**S3 Transfer Acceleration**
Fonctionnalité accélérant les uploads vers S3 en passant par le réseau Edge de CloudFront.

**SSE (Server-Side Encryption)**
Chiffrement côté serveur pour S3 — SSE-S3 (clé gérée par AWS), SSE-KMS (clé gérée dans KMS, avec audit), SSE-C (clé fournie par le client).

**Versioning (S3)**
Fonctionnalité conservant toutes les versions d'un objet S3, protégeant contre la suppression ou l'écrasement accidentels.

---

## Calcul EC2 & services managés

**AMI (Amazon Machine Image)**
Image disque prête à l'emploi pour lancer une instance EC2 — OS, logiciels préinstallés, configuration.

**Auto Scaling Group**
Groupe gérant automatiquement le nombre d'instances EC2 selon la charge, entre un minimum, un maximum et une capacité désirée.

**Availability Zone (AZ)**
Centre de données isolé physiquement au sein d'une région AWS — déployer sur plusieurs AZ garantit la haute disponibilité.

**Elastic Beanstalk**
Service PaaS gérant automatiquement l'infrastructure (EC2, Load Balancer, Auto Scaling) autour du code applicatif fourni.

**EBS (Elastic Block Store)**
Volume de stockage bloc attaché à une instance EC2, persistant après l'arrêt de l'instance.

**EFS (Elastic File System)**
Système de fichiers NFS managé, partageable entre plusieurs instances EC2 simultanément.

**ECS (Elastic Container Service)**
Service d'orchestration de conteneurs Docker managé par AWS, sur EC2 ou en mode serverless Fargate.

**EKS (Elastic Kubernetes Service)**
Service Kubernetes managé — AWS gère le control plane, le client gère (ou délègue à Fargate) les workers.

**Fargate**
Mode d'exécution serverless pour ECS/EKS — aucune instance à gérer, facturation à la ressource consommée.

**Instance Store**
Stockage temporaire attaché physiquement à l'hôte EC2, perdu à l'arrêt ou au redémarrage de l'instance.

**Key Pair**
Paire de clés SSH (publique/privée) utilisée pour l'authentification sécurisée à une instance EC2.

**Lambda**
Service de calcul sans serveur (FaaS) exécutant du code en réponse à un événement, facturé à l'invocation et à la durée d'exécution.

**Launch Template**
Modèle de configuration (AMI, type, Security Group, user data) utilisé pour lancer des instances EC2, notamment via un Auto Scaling Group.

**Reserved Instance**
Engagement d'utilisation EC2 sur 1 ou 3 ans, avec réduction tarifaire pouvant atteindre 75 % par rapport au tarif à la demande.

**Savings Plans**
Engagement de dépense horaire ($/h) sur 1 ou 3 ans, plus flexible que les Reserved Instances, couvrant EC2, Lambda et Fargate.

**Security Group**
Pare-feu stateful appliqué au niveau de l'instance EC2 — règles entrantes et sortantes, allow uniquement.

**Spot Instance**
Capacité EC2 inutilisée louée à prix réduit (jusqu'à 90 %), interruptible par AWS avec 2 minutes de préavis.

**Type d'instance EC2**
Combinaison de vCPU, RAM, réseau et stockage — familles t (burstable), m (équilibrée), c (calcul), r (mémoire), g/p (GPU).

**User Data**
Script exécuté au premier démarrage d'une instance EC2, pour l'installation ou la configuration initiale.

---

## Réseau VPC

**CIDR (Classless Inter-Domain Routing)**
Notation définissant une plage d'adresses IP — par exemple `10.0.0.0/16` pour 65 536 adresses.

**ENI (Elastic Network Interface)**
Carte réseau virtuelle attachée à une instance EC2, portant IP privée/publique, adresse MAC et Security Groups.

**Internet Gateway (IGW)**
Passerelle permettant la communication entre un subnet public et Internet. Gratuite, une seule par VPC.

**NACL (Network Access Control List)**
Pare-feu stateless au niveau du subnet — règles numérotées, entrantes et sortantes évaluées séparément.

**NAT Gateway**
Service permettant à des instances en subnet privé d'initier des connexions vers Internet sans être accessibles depuis l'extérieur.

**Peering VPC**
Connexion réseau privée entre deux VPC, non transitive — A↔B et B↔C ne créent pas A↔C.

**Route Table**
Table de routage associée à un subnet, déterminant où envoyer le trafic selon sa destination.

**Subnet**
Subdivision d'un VPC rattachée à une zone de disponibilité — public (route vers IGW) ou privé (route vers NAT Gateway ou aucune route Internet).

**Transit Gateway**
Hub réseau centralisé interconnectant plusieurs VPC, comptes AWS et réseaux on-premise via un seul attachement par ressource.

**VPC (Virtual Private Cloud)**
Réseau privé virtuel isolé dans AWS, avec sa propre plage d'adresses IP, ses subnets, tables de routage et règles de sécurité.

**VPC Endpoint**
Point de connexion privé permettant d'accéder à un service AWS depuis une VPC sans transiter par Internet.

**VPN Site-to-Site**
Connexion chiffrée entre un réseau on-premise et un VPC AWS, via deux tunnels IPsec redondants.

---

## Bases de données

**Aurora**
Moteur de base de données relationnel propriétaire AWS, compatible MySQL et PostgreSQL, avec réplication native sur 6 copies dans 3 zones de disponibilité.

**Aurora Serverless v2**
Version auto-scalante d'Aurora, facturée en ACU-heures, capable de s'ajuster en quelques secondes selon la charge.

**DynamoDB**
Base de données NoSQL serverless stockant des paires clé-valeur ou documents JSON, avec latence inférieure à la milliseconde.

**ElastiCache**
Service de cache en mémoire managé — moteur Redis (structures avancées, persistance) ou Memcached (cache simple).

**Hot Partition**
Situation où une Partition Key DynamoDB à faible cardinalité concentre toutes les requêtes sur une seule partition, provoquant un throttling.

**RCU / WCU**
Unités de capacité de lecture et d'écriture DynamoDB en mode provisionné — 1 RCU = une lecture fortement cohérente d'un item ≤ 4 Ko, 1 WCU = une écriture d'un item ≤ 1 Ko.

**RDS (Relational Database Service)**
Service de bases de données relationnelles managé — MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.

**RDS Multi-AZ**
Déploiement RDS en haute disponibilité — instance principale et standby synchrone dans une autre zone, failover automatique en ~30 secondes.

**Read Replica**
Copie en lecture seule d'une base RDS ou Aurora, en réplication asynchrone, utilisée pour décharger les lectures de l'instance principale.

**AWS DMS (Database Migration Service)**
Service managé de migration de bases de données, combinant une copie complète initiale (Full Load) et une réplication continue des changements (CDC).

---

## DNS & découplage

**Alias Record (Route 53)**
Type d'enregistrement DNS propre à AWS pointant vers une ressource AWS (ALB, CloudFront, S3), sans les limitations d'un CNAME classique.

**Dead Letter Queue (DLQ)**
File SQS ou topic SNS recevant les messages non traités avec succès après plusieurs tentatives.

**Health Check (Route 53)**
Vérification périodique de la disponibilité d'une ressource, utilisée pour déclencher un routage de failover.

**Route 53**
Service DNS managé d'AWS — enregistrement de domaines, résolution DNS, health checks, politiques de routage avancées.

**SNS (Simple Notification Service)**
Service de messagerie pub/sub — un message publié sur un topic est distribué à tous les abonnés (email, SMS, SQS, Lambda).

**SQS (Simple Queue Service)**
Service de file d'attente de messages managée, découplant un producteur d'un consommateur.

---

## Automatisation & Infrastructure as Code

**API Gateway**
Service exposant un point d'entrée HTTP unique, routant chaque requête vers le service backend adapté (Lambda, conteneur, EC2).

**CloudFormation**
Service d'Infrastructure as Code AWS — décrit une infrastructure en YAML/JSON (template) et la déploie sous forme de stack.

**CloudFormation Drift**
Divergence entre l'état réel d'une ressource et son template CloudFormation, survenant après une modification manuelle.

**CloudFormation Stack**
Ensemble de ressources AWS créées et gérées ensemble par un même template, avec création/mise à jour/suppression atomiques.

**CloudWatch**
Service de supervision AWS — métriques, logs, alarmes, tableaux de bord.

**EventBridge**
Service d'événements serverless routant des événements entre services AWS, applications SaaS et fonctions Lambda selon des règles.

**IaC (Infrastructure as Code)**
Paradigme consistant à décrire et gérer l'infrastructure via du code versionnable plutôt que par des actions manuelles.

**Quick Start**
Template CloudFormation préconfiguré et validé par AWS, déployant une architecture complète éprouvée.

**RPO (Recovery Point Objective)**
Quantité maximale de données qu'une organisation accepte de perdre en cas de sinistre, exprimée en durée.

**RTO (Recovery Time Objective)**
Durée maximale d'indisponibilité acceptable d'un service avant que l'impact métier devienne intolérable.

**AWS Systems Manager (SSM)**
Suite d'outils d'administration à distance sans SSH — Run Command, Session Manager, Patch Manager, Parameter Store.

**Step Functions**
Service d'orchestration de workflows serverless sous forme de machine à états, coordonnant Lambda et d'autres services AWS.

---

*Ce glossaire couvre les cinq chapitres de la formation. Pour la syntaxe des commandes CLI correspondantes, voir le cheat-sheet.*
