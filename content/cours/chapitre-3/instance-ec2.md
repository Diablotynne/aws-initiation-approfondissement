---
title: "5. Choisir le bon type d'instance, AMI, stockage et sécurité"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 5. Choisir le bon type d'instance, AMI, stockage et sécurité"
---

<nav class="page-sequence"><a href="cours/chapitre-3/ec2">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/compute-optimizer">Suivant</a></nav>

### 5.1 Types d'instances EC2

AWS propose plusieurs familles d'instances, selon le type de charge à traiter :

| Famille | Usage recommandé | Exemple d'application |
|---|---|---|
| **t / t4g** | Usage général, burst occasionnel | Serveur web, environnement de test |
| **m** | Charges équilibrées CPU/Mémoire | Application métier, ERP |
| **c** | Calcul intensif | Simulation scientifique, traitement d'image |
| **r** | Mémoire importante | Base de données en mémoire, Redis |
| **g / p** | GPU (accélération graphique/IA) | Intelligence artificielle, machine learning |
| **d / h** | Stockage rapide SSD | Big Data, traitement de logs volumineux |

_Dans un environnement de formation, le type d'instance est imposé par le lab. L'éligibilité au Free Tier dépend du plan du compte, de sa date de création et des types actuellement marqués comme éligibles dans la console AWS._

> [!warning]
> **Types d'instances coûteux** — Les familles accélérées par GPU et les instances à très grande capacité peuvent avoir un coût horaire élevé. Le type autorisé pendant la formation est celui indiqué dans le lab. En entreprise, le choix doit être vérifié avec AWS Pricing Calculator et les tarifs de la région avant déploiement.


**Explication des suffixes de type** :
- `t3` : type t (général), génération 3
- `micro`, `small`, `medium` : taille croissante
- `xlarge` ou `2xlarge` : très puissants, pour les charges importantes
- le **`g`** que l'on trouve dans `t4g`, `m6g`, `c6g`, etc. signale généralement une instance équipée d'un processeur **AWS Graviton** fondé sur l'architecture ARM. Le rapport performance/prix dépend de la charge. Les binaires et images doivent être compatibles avec l'architecture choisie ; un composant compilé uniquement pour x86 doit être recompilé ou remplacé.

### 5.2 AMI (Amazon Machine Image)

L'AMI est le **système d'exploitation** de votre machine EC2.

<div class="video-embed"><iframe src="https://www.youtube-nocookie.com/embed/xjZx37dsVRw" title="Amazon Machine Images — AMI" loading="lazy" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

#### Types d'AMI

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

> Il est souvent utile en entreprise de créer ses propres AMI afin de pouvoir déployer plus rapidement des instances EC2 correspondant aux besoins spécifiques. Vous pouvez enregistrer le disque contenant cette AMI après lancement de la machine EC2 et après avoir ajouté les spécificités de l'ensemble de vos machines.

### 5.3 Stockage associé à EC2

#### EBS — Elastic Block Store

- **Disque attaché** à une instance EC2.
- **Persiste** même si l'instance est arrêtée.
- Permet les **snapshots** (sauvegardes incrémentales).
- Idéal pour le stockage de données d'application.
- Peut être attaché/détaché dynamiquement.

**Cas d'usage** : serveur web avec base de données, ERP, systèmes de fichiers importants.

#### EFS — Elastic File System

- **Système de fichiers partagé** entre plusieurs instances.
- Montable sur **plusieurs instances EC2 simultanément**.
- Idéal pour les architectures distribuées.
- Escalabilité automatique sans gestion de capacité.
- Compatible avec NFS (Network File System).

**Cas d'usage** : cluster d'applications, déploiement multi-serveurs, stockage partagé.

**Avantages** :
- Haute disponibilité multi-AZ.
- Performance prédictible et constante.
- Paiement à l'usage (pas de provisionnement anticipé).

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

### 5.4 Sécurité EC2

#### Security Groups : pare-feu virtuel

Les **Security Groups** sont des pare-feux virtuels qui :
- Autorisent ou bloquent le trafic,
- Sont **stateful** (les réponses sont automatiquement autorisées),
- Peuvent être appliqués à plusieurs instances.

##### Exemple de règles

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

### 5.5 Options de conformité EC2

Les environnements réglementés (santé, finance, RGPD) ont besoin de **garanties de conformité**. AWS fournit plusieurs mécanismes :

#### Dedicated Instances

- Instance EC2 qui s'exécute sur **matériel physique dédié**.
- Pas de partage avec d'autres clients AWS.
- Idéal pour les **exigences légales** ou de conformité.
- Coût plus élevé que le partage de matériel.

#### Dedicated Hosts

- **Serveur physique entier** réservé pour votre compte.
- Contrôle total : vous décidez quelles instances y tournent.
- Utile pour les **licences logicielles** (ex. Windows, SQL Server avec licensing par socket/processeur).
- Exigences réglementaires très strictes.

#### Instance Store (éphémère)

- Stockage **très rapide** mais **temporaire** sur l'hyperviseur physique.
- **Attention** : données perdues à l'arrêt/redémarrage de l'instance.
- Idéal pour cache, données temporaires, haute performance.
- À éviter pour données persistantes.

> [!danger]
> **Instance Store : stockage éphémère.** Les données ne persistent pas après l'arrêt ou la terminaison de l'instance, ni après certaines défaillances de l'hôte. Réservez ce stockage aux caches, espaces temporaires et données reproductibles ; placez les données persistantes sur un service adapté.


#### Encrypted EBS Volumes

- Les volumes EBS peuvent être chiffrés avec **AWS KMS**.
- Le chiffrement est **transparent** pour l'application.
- Utile pour la conformité HIPAA, PCI-DSS, ISO 27001.

**Cas d'usage conformité** :
- Données médicales → Dedicated Instance + EBS chiffré + Audit CloudTrail.
- Données financières → Dedicated Host + KMS + VPC isolé.
- Données RGPD → Région EU + Versioning S3 + Chiffrement.

---

<nav class="page-sequence"><a href="cours/chapitre-3/ec2">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/compute-optimizer">Suivant</a></nav>
