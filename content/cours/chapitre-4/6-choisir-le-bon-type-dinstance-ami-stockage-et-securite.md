---
title: "6. Choisir le bon type d'instance, AMI, stockage et sécurité"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 6. Choisir le bon type d'instance, AMI, stockage et sécurité"
---

<nav class="page-sequence"><a href="cours/chapitre-4/5-amazon-ec2-la-couche-de-calcul-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/7-aws-compute-optimizer-dimensionnement-optimal">Suivant</a></nav>

### 6.1 Types d'instances EC2

AWS propose plusieurs familles d'instances, selon le type de charge à traiter :

| Famille | Usage recommandé | Exemple d'application |
|---|---|---|
| **t / t4g** | Usage général, burst occasionnel | Serveur web, environnement de test |
| **m** | Charges équilibrées CPU/Mémoire | Application métier, ERP |
| **c** | Calcul intensif | Simulation scientifique, traitement d'image |
| **r** | Mémoire importante | Base de données en mémoire, Redis |
| **g / p** | GPU (accélération graphique/IA) | Intelligence artificielle, machine learning |
| **d / h** | Stockage rapide SSD | Big Data, traitement de logs volumineux |

_Pour un TP ou un test, on utilise souvent `t3.micro` (gratuit dans le Free Tier AWS)._

> [!warning]
> **Types d'instances coûteux** — Les familles `g`, `p` (GPU) et `x` (mémoire très haute) peuvent coûter plusieurs dizaines de dollars **par heure**. Par exemple, une instance `p3.16xlarge` (GPU ML) dépasse 24 $/h. Ne lancez ces types que si votre workload le justifie, et pensez à les **arrêter immédiatement** après utilisation. En formation ou développement, restez sur des types `t3.micro` ou `t3.small`.

**Explication des suffixes de type** :
- `t3` : type t (général), génération 3
- `micro`, `small`, `medium` : taille croissante
- `xlarge` ou `2xlarge` : très puissants, pour les charges importantes
- le **`g`** que l'on trouve dans `t4g`, `m6g`, `c6g`, etc. signale que l'instance tourne sur un processeur **Graviton**, la puce ARM conçue par AWS elle-même (au lieu d'un processeur x86 classique Intel/AMD). Les instances Graviton offrent généralement un meilleur rapport performance/prix (jusqu'à 20-40 % moins cher à performance équivalente), mais nécessitent que votre application soit compilée pour l'architecture ARM — la plupart des langages interprétés (Python, Node.js, Java) et des images Docker officielles la supportent nativement, mais un vieux binaire compilé spécifiquement pour x86 ne fonctionnera pas dessus sans recompilation.

Le type d'instance ne détermine que la puissance matérielle disponible ; le logiciel qui tourne dessus au démarrage dépend d'un second choix indépendant, l'AMI.

### 6.2 AMI (Amazon Machine Image)

L'AMI est le **système d'exploitation** de votre machine EC2.

📹 **Vidéo** : [AMI — Amazon Machine Images](https://www.youtube.com/watch?v=xjZx37dsVRw)

#### Types d'AMI

On distingue trois origines possibles pour une AMI, selon qui l'a créée et publiée :

- **Public AMI** : proposées par AWS (Linux, Windows, Ubuntu, etc.)
- **Custom AMI** : créées par vous (ex. avec des logiciels préinstallés)
- **Marketplace AMI** : proposées par des éditeurs tiers (ex. WordPress, SAP)

_L'AMI détermine ce que contient votre machine au démarrage._

#### Classification des AMI

Les AMI peuvent être classifiées dans les grandes catégories suivantes :

**AMI persistantes (basées sur EBS)**
- L'ensemble du filesystem est stocké sur EBS (Elastic Block Store).
- EBS fonctionne de manière similaire à un **NAS (Network Attached Storage)** et permet le partage des données sur le réseau.
- Ces volumes ne sont associés à aucun type de matériel spécifique, ce qui les rend très pratiques pour transférer des données entre zones ou d'une région à une autre.
- Les AMI persistantes sont configurées avec un ou plusieurs volumes EBS.

**AMI volatiles (basées sur S3)**
- Contrairement aux AMI persistantes, les AMI volatiles stockent leurs données via le service AWS S3 (Simple Storage Service).
- Ces AMI **ne peuvent pas être transférées** depuis une zone de disponibilité ou région vers une autre.

![](assets/schemas/ch3-capture-02-c1a1127e.png)

> Il est souvent utile en entreprise de créer ses propres AMI afin de pouvoir déployer plus rapidement des instances EC2 correspondant aux besoins spécifiques. Vous pouvez enregistrer le disque contenant cette AMI après lancement de la machine EC2 et après avoir ajouté les spécificités de l'ensemble de vos machines.

### 6.3 Stockage associé à EC2

#### EBS — Elastic Block Store

EBS est le disque virtuel « par défaut » d'EC2, attaché à une seule instance à la fois :

- **Disque attaché** à une instance EC2.
- **Persiste** même si l'instance est arrêtée.
- Permet les **snapshots** (sauvegardes incrémentales).
- Idéal pour le stockage de données d'application.
- Peut être attaché/détaché dynamiquement.

**Cas d'usage** : serveur web avec base de données, ERP, systèmes de fichiers importants.

#### EFS — Elastic File System

Là où un volume EBS ne peut être monté que sur une seule instance à la fois, EFS lève cette limite en proposant un système de fichiers partagé :

- **Système de fichiers partagé** entre plusieurs instances.
- Montable sur **plusieurs instances EC2 simultanément**.
- Idéal pour les architectures distribuées.
- Escalabilité automatique sans gestion de capacité.
- Compatible avec NFS (Network File System).

Ce partage simultané entre plusieurs instances est précisément ce qu'EBS ne permet pas — c'est le critère de choix déterminant entre les deux services.

**Cas d'usage** : cluster d'applications, déploiement multi-serveurs, stockage partagé.

**Avantages** :
- Haute disponibilité multi-AZ.
- Performance prédictible et constante.
- Paiement à l'usage (pas de provisionnement anticipé).

EFS couvre bien les usages Linux/NFS génériques, mais certains environnements ont des besoins plus spécifiques (compatibilité Windows, calcul haute performance) auxquels répond un troisième service.

#### FSx — Managed File Systems

**Amazon FSx** propose des systèmes de fichiers managés, avec deux options principales :

**FSx for Windows File Server** :
- Compatible **Active Directory** et **SMB** (partages Windows).
- Idéal pour les environnements Windows d'entreprise.
- Partages de fichiers compatibles avec les domaines Windows.

**SMB** (Server Message Block) est le protocole de partage de fichiers natif de Windows — c'est exactement ce que vous utilisez quand vous accédez à un dossier partagé via `\\serveur\dossier` sur un réseau d'entreprise. FSx for Windows reproduit ce protocole nativement dans AWS, ce qui permet à des applications Windows existantes de continuer à fonctionner sans modification.

**FSx for Lustre** :
- Optimisé pour le **high-performance computing (HPC)** et machine learning.
- Très haute performance pour les grandes quantités de données.

**Lustre** est un système de fichiers distribué open source conçu à l'origine pour les supercalculateurs : il répartit un même fichier sur plusieurs serveurs de stockage pour permettre à des milliers de machines de le lire et l'écrire simultanément à très haut débit. C'est ce qui en fait le choix de référence pour l'entraînement de modèles de machine learning sur de gros volumes de données ou les simulations scientifiques (météo, génomique, calcul financier).

**Comparatif EFS vs FSx** :

| Critère | EFS | FSx (Windows) | FSx (Lustre) |
|---------|-----|---------------|-------------|
| **Protocole** | NFS (Linux/Unix) | SMB (Windows) | Lustre (HPC) |
| **OS Support** | Linux/Unix | Windows | Linux/HPC |
| **Active Directory** | Non | ✅ Oui | Non |
| **Performance** | Modérée, extensible | Haute | Très haute (HPC) |
| **Coût** | Bas à moyen | Moyen-élevé | Élevé |
| **Idéal pour** | Linux distribué | Partages Windows d'entreprise | Calcul scientifique/IA |

**Conseil pratique** :
- **EFS** : première option pour Linux si pas besoin de domaine.
- **FSx for Windows** : environnement Windows avec Active Directory.
- **FSx for Lustre** : uniquement si performance HPC requise.

📎 [EBS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html)
📎 [EFS Documentation](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
📎 [FSx Documentation](https://aws.amazon.com/fr/fsx/)

### 6.4 Sécurité EC2

#### Security Groups : pare-feu virtuel

Les **Security Groups** sont des pare-feux virtuels qui :
- Autorisent ou bloquent le trafic,
- Sont **stateful** (les réponses sont automatiquement autorisées),
- Peuvent être appliqués à plusieurs instances.

Ce caractère stateful est essentiel à comprendre avant de lire des règles concrètes : il signifie qu'une règle entrante suffit à autoriser la réponse correspondante, sans avoir à écrire une règle sortante symétrique.

##### Exemple de règles

Voici un jeu de règles typique pour un serveur web accessible publiquement mais administrable uniquement depuis une IP de confiance :

| Direction | Port | Source/Destination |
|---|---|---|
| Inbound | 22 (SSH) | `192.168.1.100/32` |
| Inbound | 80 (HTTP) | `0.0.0.0/0` |
| Inbound | 443 (HTTPS) | `0.0.0.0/0` |
| Outbound | Tout | par défaut |

**Règle entrante exemple** :
- Autoriser le port **22 (SSH)** uniquement depuis une IP précise.
- Autoriser les ports **80 et 443 (HTTP/HTTPS)** depuis n'importe où.

**Règle sortante (par défaut)** :
- Autoriser tout trafic sortant.

📎 [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

#### Key Pairs : accès SSH sécurisé

Les **Key Pairs** servent à sécuriser l'accès SSH :
- **Clé publique** enregistrée dans AWS et stockée dans l'instance au démarrage.
- **Clé privée** conservée localement par l'administrateur.

**Flux de connexion** :
1. Vous lancez une instance EC2 et sélectionnez une Key Pair.
2. AWS injecte la clé publique dans `~/.ssh/authorized_keys` de l'instance.
3. Depuis votre ordinateur, vous utilisez votre clé privée pour vous connecter en SSH.
4. L'authentification par clé est plus sécurisée qu'un mot de passe (impossible à craquer par brute force).

Security Groups et Key Pairs suffisent pour la grande majorité des instances ; certains secteurs réglementés exigent cependant des garanties matérielles supplémentaires.

### 6.5 Options de conformité EC2

Les environnements réglementés (santé, finance, RGPD) ont besoin de **garanties de conformité**. AWS fournit plusieurs mécanismes :

#### Dedicated Instances

Le premier niveau d'isolation matérielle, le plus simple à activer, consiste à louer du matériel non partagé sans en avoir le contrôle direct :

- Instance EC2 qui s'exécute sur **matériel physique dédié**.
- Pas de partage avec d'autres clients AWS.
- Idéal pour les **exigences légales** ou de conformité.
- Coût plus élevé que le partage de matériel.

Quand la conformité exige non seulement un matériel non partagé mais aussi un contrôle explicite du placement des instances, on passe au niveau supérieur.

#### Dedicated Hosts

Ce second niveau va plus loin que Dedicated Instances en donnant une visibilité complète sur le serveur physique sous-jacent :

- **Serveur physique entier** réservé pour votre compte.
- Contrôle total : vous décidez quelles instances y tournent.
- Utile pour les **licences logicielles** (ex. Windows, SQL Server avec licensing par socket/processeur).
- Exigences réglementaires très strictes.

Ces deux options d'isolation matérielle n'ont cependant rien à voir avec le type de stockage attaché à l'instance, sujet traité séparément ci-dessous.

#### Instance Store (éphémère)

À l'opposé de la conformité matérielle, ce type de stockage répond à un besoin de performance brute, au prix d'une contrainte forte :

- Stockage **très rapide** mais **temporaire** sur l'hyperviseur physique.
- **Attention** : données perdues à l'arrêt/redémarrage de l'instance.
- Idéal pour cache, données temporaires, haute performance.
- À éviter pour données persistantes.

> [!danger]
> **Instance Store : perte de données garantie à l'arrêt** — Contrairement à EBS, le stockage instance store **n'est pas persistant**. Toutes les données écrites dessus sont définitivement perdues si l'instance est arrêtée, terminée ou si l'hôte physique tombe en panne. Ne stockez jamais de données de production, de bases de données ou de fichiers importants sur instance store sans sauvegarde préalable vers S3 ou EBS.

#### Encrypted EBS Volumes

Dernier levier de conformité, orthogonal aux précédents puisqu'il porte sur la donnée plutôt que sur le matériel :

- Les volumes EBS peuvent être chiffrés avec **AWS KMS**.
- Le chiffrement est **transparent** pour l'application.
- Utile pour la conformité HIPAA, PCI-DSS, ISO 27001.

**Cas d'usage conformité** :
- Données médicales → Dedicated Instance + EBS chiffré + Audit CloudTrail.
- Données financières → Dedicated Host + KMS + VPC isolé.
- Données RGPD → Région EU + Versioning S3 + Chiffrement.

<div class="concept-check">
<strong>Choix de service — S3 ou EBS</strong>
<p>Où placer des images partagées par plusieurs instances EC2 et accessibles par URL ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Dans Amazon S3 : les images sont des objets indépendants des instances. EBS est un stockage bloc attaché dans une zone de disponibilité et ne constitue pas ici le bon niveau de partage.</p></details>
</div>

---

<nav class="page-sequence"><a href="cours/chapitre-4/5-amazon-ec2-la-couche-de-calcul-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/7-aws-compute-optimizer-dimensionnement-optimal">Suivant</a></nav>
