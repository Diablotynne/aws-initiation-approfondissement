# Chapitre 3 — Stockage et Calcul — Amazon S3 & Amazon EC2

---

> [!NOTE]
> Après avoir sécurisé les accès avec IAM au chapitre précédent, les stagiaires disposent des bases nécessaires pour créer et protéger de vraies ressources AWS : ce chapitre aborde les deux briques les plus utilisées du Cloud AWS, le stockage objet S3 et le calcul EC2.
>
> **Objectifs du chapitre**
>
> À l'issue de ce chapitre, les stagiaires seront capables de :
>
> - **Créer** et configurer un bucket Amazon S3 (chiffrement, versioning, politiques d'accès)
> - **Mettre en œuvre** des règles de lifecycle S3 pour optimiser le coût du stockage
> - **Manipuler** S3 via la CLI (upload, download, synchronisation, gestion des permissions)
> - **Choisir** un type d'instance EC2, une AMI et un mode de stockage adaptés à un besoin donné
> - **Configurer** des Security Groups pour contrôler le trafic réseau d'une instance EC2
> - **Utiliser** AWS Compute Optimizer pour dimensionner correctement une instance
> - **Comparer** les modèles de tarification EC2 (On-Demand, Reserved, Spot, Savings Plans)
> - **Lancer et administrer** une instance EC2 via la CLI
> - **Déployer** un Elastic Load Balancer pour répartir le trafic entre plusieurs instances
> - **Configurer** un groupe Auto Scaling pour adapter dynamiquement la capacité aux besoins
> - **Concevoir** une architecture haute disponibilité combinant S3, EC2, ELB et Auto Scaling
![Architecture résiliente combinant S3, équilibrage de charge, EC2, EBS et Auto Scaling](formations/aws-initiation-approfondissement/11-images/ch3-carte-stockage-calcul.svg)

<div class="concept-check">
<strong>Choix de service — avant de poursuivre</strong>
<p>Où placer des images partagées par plusieurs instances EC2 et accessibles par URL ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Dans Amazon S3 : les images sont des objets indépendants des instances. EBS est un stockage bloc attaché dans une zone de disponibilité et ne constitue pas ici le bon niveau de partage.</p></details>
</div>

---

## 1. Introduction aux services de stockage AWS

📹 **Vidéo** : [Introduction to Amazon S3](https://www.youtube.com/watch?v=4RI3pDKpx38)

### 1.1 Pourquoi plusieurs services de stockage ?

Le stockage est au cœur de toute infrastructure cloud. AWS propose plusieurs types de stockage, mais **Amazon Simple Storage Service (S3)** est le service le plus emblématique : fiable, scalable et économique.

Créé en 2006, S3 a révolutionné la façon dont les entreprises stockent leurs données en passant d'un modèle de serveur à un modèle **d'espace de stockage à la demande**.

Avant de plonger dans les services techniques, il est essentiel de comprendre **pourquoi AWS propose plusieurs modèles de stockage** et dans quels contextes les utiliser.

Dans une entreprise traditionnelle, le stockage repose sur :
- des **disques durs internes** (pour les postes ou serveurs locaux),
- des **baies NAS/SAN** (pour le stockage partagé),
- parfois des sauvegardes sur bande ou sur site distant.

AWS transpose ces modèles dans le Cloud et les **rend flexibles, évolutifs et disponibles à la demande**.

### 1.2 Les trois modèles de stockage AWS

| Type de stockage | Service AWS        | Cas d'usage typique                              | Analogie utilisateur                 |
|-------------------|--------------------|--------------------------------------------------|---------------------------------------|
| **Objet**         | **Amazon S3**      | Sauvegarde, site statique, logs, Data Lake       | Dropbox / Google Drive               |
| **Bloc**          | **Amazon EBS**     | Disque de VM, base de données, stockage persistant | Disque dur local                     |
| **Fichier**       | **Amazon EFS/FSx** | Partage réseau, systèmes distribués              | NAS / Partage Windows                |

**À retenir** : Chaque type de stockage a ses propres performances, coûts et scénarios d'usage. Nous allons détailler S3 et EBS en priorité, car ce sont les services les plus utilisés par les administrateurs AWS en début de carrière.

📎 [Documentation Amazon S3](https://docs.aws.amazon.com/s3/)

---

## 2. Amazon S3 : Le stockage objet scalable

### 2.1 Qu'est-ce qu'Amazon S3 ?

**Amazon S3 (Simple Storage Service)** est le service de stockage le plus utilisé sur AWS. Il permet de stocker des fichiers (appelés **objets**) dans le cloud, avec une capacité quasiment illimitée, une très haute disponibilité (accès garanti) et une durabilité exceptionnelle (les données ne sont pas perdues).

Mais attention : **S3 ne fonctionne pas comme un disque dur classique**. C'est un système de **stockage objet**, ce qui signifie que chaque fichier est stocké avec des informations supplémentaires (appelées **métadonnées**) dans un conteneur appelé **bucket**.

### 2.2 L'armoire de rangement : analogie avec S3

Imaginez une **armoire de rangement** :

<img src="formations/aws-initiation-approfondissement/11-images/s3-bucket-structure.svg"
     alt="Structure d'un bucket Amazon S3"
     style="display:block; margin:auto; width:90%">

- L'armoire, c'est le **bucket** : un conteneur dans lequel vous rangez vos fichiers.
- Chaque fichier est un **objet** : il contient le contenu (ex. une image) + des étiquettes (métadonnées) comme son nom, sa date, ses droits d'accès.
- Les "dossiers" que vous voyez dans S3 ne sont pas réels : ce sont juste des **préfixes logiques** dans le nom du fichier (ex. `images/logo.png`).

### 2.3 Exemple concret

Vous créez un bucket nommé `site-web-entreprise`. Vous y déposez :

- `images/logo.png`
- `css/style.css`
- `js/app.js`

Ces fichiers peuvent ensuite être **consultés via Internet**, sans serveur web, si vous configurez le bucket en mode **site statique**.

📎 [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)

### 2.4 Structure interne de S3

| Élément | Description |
|--------|-------------|
| **Bucket** | Conteneur global (nom unique dans AWS), lié à une région |
| **Objet** | Fichier + métadonnées (nom, taille, type, permissions) |
| **Préfixe** | Partie du nom qui simule un dossier (ex. `images/`) |
| **Key** | Identifiant unique de l'objet au sein du bucket |
| **Métadonnées** | Informations sur l'objet (format, date, ACL, tags) |

### 2.5 Caractéristiques clés de S3

| Caractéristique | Description |
|------------------|-------------|
| **Durabilité** | 99,999999999% (11 neuf) grâce à la réplication automatique sur plusieurs zones. |
| **Disponibilité** | Haute disponibilité (jusqu'à 99,99% selon la classe). |
| **Évolutivité** | Pas de limite pratique en nombre d'objets. |
| **Sécurité** | Contrôle fin via IAM, ACL, policies de bucket et chiffrement. |
| **Coût à l'usage** | Payez uniquement pour le stockage et les requêtes. |

📎 [S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)

### 2.6 Console S3 — Création d'un bucket

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-01-c832001d.png)

---

## 3. Protéger et optimiser les données S3

Amazon S3 propose plusieurs mécanismes pour **sécuriser vos fichiers**, **préserver leur historique**, et **réduire les coûts de stockage**. Ces options sont souvent méconnues, mais elles sont essentielles pour bien gérer vos données dans le cloud.

### 3.1 Chiffrement : protéger les fichiers contre les accès non autorisés

Quand vous stockez un fichier dans S3, vous pouvez demander à AWS de le **chiffrer automatiquement**. Cela signifie que même si quelqu'un accède physiquement au disque, il ne pourra pas lire le contenu sans la clé.

#### Types de chiffrement et gestion des clés

| Acronyme | Signification complète | Description pédagogique |
|---|---|---|
| **SSE-S3** | _Server-Side Encryption with Amazon S3-managed keys_ | Le chiffrement est géré **automatiquement par AWS S3**. Vous n'avez rien à configurer. |
| **SSE-KMS** | _Server-Side Encryption with AWS Key Management Service_ | Le chiffrement utilise **AWS KMS**, un service de gestion de clés. Vous définissez et contrôlez les clés. |
| **HTTPS/TLS** | _HyperText Transfer Protocol Secure / Transport Layer Security_ | Ce protocole **sécurise les échanges** entre votre navigateur ou application et AWS. |

### 3.2 Détails des types de chiffrement

#### SSE (Server-Side Encryption)

> Chiffrement effectué **côté serveur**, c'est-à-dire par AWS une fois que les données sont reçues.

**SSE-S3** : AWS chiffre les objets S3 avec une clé gérée par le service S3 lui-même.
- **Avantage** : aucune configuration requise.
- **Niveau de sécurité** : standard, suffisant pour de nombreux cas d'usage.

**SSE-KMS** : AWS chiffre les objets S3 avec une clé gérée par **AWS KMS**, que vous pouvez créer, activer/désactiver, auditer.
- **Avantage** : contrôle granulaire sur les clés.
- **Complexité** : nécessite configuration, permissions IAM, et gestion des quotas KMS.

#### KMS (Key Management Service)

> Service AWS permettant de **créer, stocker et gérer** des clés de chiffrement.

- Utilisé dans **SSE-KMS**, mais aussi pour chiffrer des volumes EBS, des secrets, etc.
- Permet la **rotation automatique**, l'audit via CloudTrail, et l'intégration avec IAM.

#### HTTPS / TLS

> Protocole de **sécurisation des communications réseau**.

- **HTTPS** est HTTP + TLS.
- **TLS (Transport Layer Security)** : protocole de chiffrement qui protège les données en transit.
- Activé **par défaut** dans la console AWS et les SDK/API.

### 3.3 À retenir sur le chiffrement

- **SSE-S3** : simple, automatique, suffisant pour les données non sensibles.
- **SSE-KMS** : recommandé pour les données sensibles ou les environnements réglementés.
- **HTTPS/TLS** : toujours activé pour sécuriser les échanges réseau.

> [!NOTE]
> **SSE-KMS et coûts KMS** — Chaque requête de chiffrement/déchiffrement via KMS est facturée (environ 0,03 $ pour 10 000 requêtes). Pour un bucket avec de nombreuses petites opérations, cela peut s'accumuler. Activez le **Bucket Key** (option KMS) pour réduire le nombre d'appels KMS jusqu'à 99% en utilisant une clé de données par bucket plutôt que par objet.
### 3.4 Versioning : garder l'historique des fichiers

Le **versioning** permet de conserver toutes les versions d'un fichier, même si vous le modifiez ou le supprimez par erreur.

#### Exemple :
- Vous téléversez `rapport.pdf`
- Vous le modifiez et téléversez une nouvelle version
- Vous pouvez toujours revenir à la version précédente

C'est utile pour :
- Éviter les pertes accidentelles
- Respecter des exigences réglementaires
- Tracer les modifications

📎 [S3 Versioning Guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html)

### 3.5 Lifecycle Policies : automatiser le nettoyage et l'archivage

Les **politiques de cycle de vie** permettent de définir des règles pour :

- Supprimer automatiquement les fichiers après X jours
- Déplacer les fichiers vers une classe de stockage moins coûteuse
- Archiver dans Glacier pour la conformité

>  Cela vous aide à **réduire les coûts** sans perdre vos données.

**Exemple concret** : Un fichier log commence en **Standard** (accès rapide), est déplacé en **Standard-IA** après 30 jours (moins accédé, moins cher), puis archivé en **Glacier** après 90 jours (rarement consulté).

📎 [S3 Lifecycle Rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html)

### 3.6 Classes de stockage : choisir le bon niveau selon l'usage

Amazon S3 propose plusieurs **classes de stockage**, selon la fréquence d'accès et le niveau de disponibilité souhaité.

| Classe | Disponibilité | Coût | Temps d'accès | Cas d'usage |
|--------|---------------|------|---------------|-------------|
| **Standard** | Multi-AZ | 💰💰 | Millisecondes | Fichiers actifs, souvent consultés |
| **Standard-IA** | Multi-AZ | 💰 | Millisecondes | Sauvegardes, fichiers rarement lus |
| **One Zone-IA** | Mono-AZ | 💰 | Millisecondes | Données non critiques |
| **Intelligent-Tiering** | Automatique | 💰 | Variable | Accès imprévisible |
| **Glacier / Deep Archive** | Multi-AZ | 💰 très bas | Minutes à heures | Archivage long terme, conformité |

> Moins une donnée est accessible rapidement, moins elle coûte. C'est un **levier puissant pour optimiser votre budget**.

#### S3 Intelligent-Tiering : Optimisation automatique

**S3 Intelligent-Tiering** est une classe de stockage qui déplace automatiquement les objets entre quatre niveaux d'accès selon votre modèle de consultation réel.

| Niveau | Délai avant bascule | Économie |
|---|---|---|
| Frequent Access | 0-30 jours | coût = Standard |
| Infrequent Access | 30-90 jours | ~40% |
| Archive Instant | 90-180 jours | ~70% |
| Deep Archive | 180+ jours | ~95% |

**Avantages** :
- Aucune configuration manuelle requise.
- Frais de gestion faibles (0,0025 $ par objet par mois).
- Idéal pour les données dont la fréquence d'accès est **imprévisible**.

**Cas d'usage** :
- Logs d'applications avec accès sporadique.
- Archives de données sans schéma d'accès défini.
- Données de machine learning exploratoires.

**Différence avec Lifecycle** :
- Lifecycle : vous définissez les règles (ex. "après 90 jours, archiver en Glacier").
- Intelligent-Tiering : AWS observe votre accès réel et adapte automatiquement.

#### S3 Transfer Acceleration : Optimisation des uploads volumineux

**S3 Transfer Acceleration** améliore les **vitesses d'upload** vers S3 en utilisant le réseau CloudFront d'AWS.

**Fonctionnement** :
```
Upload standard (lent)
Votre ordinateur ──────────────────► AWS S3 Région distant
                      ❌ Slow, high latency

Avec Transfer Acceleration
Votre ordinateur ──► CloudFront Edge Location (près de vous)
                         
                          Optimisé (route accélérée AWS)
                        ▼
                     AWS S3 Région distant
                      ✅ Rapide
```

**Activation** :
```bash
# Activer Transfer Acceleration
aws s3api put-bucket-accelerate-configuration \
    --bucket mon-bucket \
    --accelerate-configuration Status=Enabled

# Upload avec accélération
aws s3 cp mon-fichier-gros.zip \
    s3://mon-bucket/uploads/ \
    --region eu-west-1
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-bucket-accelerate-configuration : aucun output si succès
>
> # s3 cp retourne la progression :
> upload: ./mon-fichier-gros.zip to s3://mon-bucket/uploads/mon-fichier-gros.zip
> ```
> Transfer Acceleration est activé sur le bucket. Les uploads utilisent désormais les Edge Locations CloudFront pour rejoindre le bucket S3, ce qui réduit la latence depuis les clients distants.
**Coûts** :
- Frais supplémentaires par Go transféré (environ $0.04/Go).
- À justifier uniquement pour uploads volumineux ou latence critique.

#### S3 comme origine CloudFront — distribuer du contenu statique à grande échelle

Transfer Acceleration optimise l'**upload** vers S3. Le cas d'usage inverse — beaucoup plus fréquent en production — est de distribuer efficacement du contenu **depuis** S3 vers des millions de visiteurs : c'est le rôle de **CloudFront** utilisé comme CDN devant un bucket S3.

```
Sans CloudFront (chaque visiteur télécharge depuis S3 directement) :
  Visiteur Tokyo    ──► S3 eu-west-3 (Paris)   ❌ Latence élevée, coût egress à chaque fois
  Visiteur New York ──► S3 eu-west-3 (Paris)   ❌ Latence élevée, coût egress à chaque fois
  Visiteur Paris     ──► S3 eu-west-3 (Paris)   ✅ Rapide (mais N requêtes = N factures egress)

Avec CloudFront devant S3 :
  Visiteur Tokyo    ──► Edge Location Tokyo    (cache HIT après 1er accès, < 50 ms)
  Visiteur New York ──► Edge Location New York (cache HIT après 1er accès, < 50 ms)
  Visiteur Paris     ──► Edge Location Paris    (cache HIT après 1er accès, < 20 ms)
                              │
                              ▼ (uniquement au premier accès, cache MISS)
                         S3 eu-west-3 (Paris) — origine unique
```

**Pourquoi c'est la bonne pratique, pas juste une option :**
- **Coût réduit** : le trafic sortant de S3 vers CloudFront est gratuit (les deux services AWS communiquent via le réseau interne). Seul le trafic CloudFront → visiteur final est facturé, à un tarif généralement inférieur à l'egress S3 direct.
- **Bucket privé possible** : avec une **Origin Access Control (OAC)**, le bucket S3 n'a besoin d'aucun accès public — seul CloudFront peut le lire. Les visiteurs ne touchent jamais directement S3.
- **Cache réduit la charge S3** : un fichier consulté 100 000 fois par jour ne génère qu'une poignée de requêtes S3 réelles (une par Edge Location, tant que le cache est valide), le reste est servi depuis le cache CloudFront.

**Configuration minimale (CLI) :**
```bash
# Créer la distribution CloudFront avec S3 comme origine et OAC
aws cloudfront create-distribution \
  --origin-domain-name mon-bucket.s3.eu-west-3.amazonaws.com \
  --default-root-object index.html
```

> [!TIP]
> **Résultat attendu (extrait) :**
> ```json
> {
>   "Distribution": {
>     "Id": "E1A2B3C4D5E6F7",
>     "DomainName": "d111111abcdef8.cloudfront.net",
>     "Status": "InProgress"
>   }
> }
> ```
> La distribution devient `Deployed` après quelques minutes de propagation sur le réseau mondial CloudFront. Le bucket S3 reste privé — seule cette distribution CloudFront (via son OAC) est autorisée à le lire, configuré automatiquement dans la bucket policy.
📎 [Amazon CloudFront — Restricting access to S3](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)

> [!WARNING]
> **Coûts Transfer Acceleration** : Cette fonctionnalité engendre des frais supplémentaires (~0,04 $/Go pour les transferts vers des Edge Locations). N'activez Transfer Acceleration que si vous uploadez régulièrement des fichiers volumineux (> 100 Mo) depuis des clients géographiquement éloignés de la région S3. Pour les petits fichiers ou les accès locaux, le gain est négligeable et le surcoût inutile.
**Cas d'usage** :
- Uploads de fichiers vidéo ou binaires depuis un client distant.
- Synchronisations multi-sites hautes performances.
- Distributions de fichiers volumineux vers plusieurs régions AWS.

### 3.7 Stratégies de compartiment (Bucket Policies) : Contrôle d'accès granulaire

Les **bucket policies** sont des documents JSON qui définissent **qui** peut accéder **à quoi** dans un bucket S3.

#### Structure d'une bucket policy

Une bucket policy est un document JSON attaché directement au bucket (et non à un utilisateur IAM). Elle contrôle qui peut accéder à quoi, y compris des accès publics ou inter-comptes AWS.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mon-bucket/*"
    },
    {
      "Sid": "DenyEncryptedUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mon-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

**Éléments clés** :
- **Sid** : identifiant lisible de la règle (ex. "AllowPublicRead")
- **Effect** : `Allow` ou `Deny`
- **Principal** : qui a l'accès (`*` = tout le monde, ou un ARN spécifique)
- **Action** : quelle opération S3 (`s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`, etc.)
- **Resource** : sur quel objet (ARN format)
- **Condition** : contextes additionnels (IP, SSL, chiffrement, etc.)

#### Cas d'usage 1 : Site web statique public

Pour héberger un site web HTML/CSS sur S3, le bucket doit autoriser la lecture publique. Voici la policy à appliquer — notez `"Principal": "*"` qui signifie "tout le monde sans authentification" :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mon-site-web/*"
    }
  ]
}
```

**Effet** : Tous les utilisateurs peuvent **lire** les fichiers du bucket (parfait pour un site statique).

> [!IMPORTANT]
> **Bucket public : risque de fuite de données** — L'utilisation de `"Principal": "*"` rend l'ensemble des objets du bucket accessibles sur Internet **sans authentification**. Ne l'appliquez **jamais** à un bucket contenant des données sensibles (fichiers clients, logs internes, clés, backups). Depuis 2023, AWS bloque par défaut les accès publics sur les nouveaux buckets — cette policy nécessite de désactiver explicitement ce blocage.
#### Cas d'usage 2 : Restreindre à une adresse IP spécifique

Pour un bucket contenant des données sensibles, on limite l'accès au réseau de l'entreprise via la condition `aws:SourceIp`. Tout accès depuis une IP extérieure sera refusé, même avec des credentials IAM valides.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RestrictToCompanyIP",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::mon-bucket-prive/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Effet** : Seules les IPs du réseau 203.0.113.0/24 peuvent accéder au bucket.

#### Cas d'usage 3 : Forcer le chiffrement pour tous les uploads

Cette policy est une protection anti-erreur : elle refuse tout upload qui ne spécifie pas le chiffrement côté serveur (SSE-S3). Même si un développeur oublie de configurer le chiffrement dans son code, AWS rejette la requête.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mon-bucket-sensible/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

**Effet** : Toute tentative d'upload sans chiffrement SSE-S3 sera **rejetée**.

#### Appliquer une bucket policy via CLI

Une fois le fichier JSON rédigé, voici comment l'appliquer, le vérifier, et le supprimer si nécessaire :

```bash
# Créer un fichier policy.json (voir ci-dessus)
# Appliquer la policy
aws s3api put-bucket-policy \
    --bucket mon-bucket \
    --policy file://policy.json

# Vérifier la policy
aws s3api get-bucket-policy --bucket mon-bucket

# Supprimer la policy
aws s3api delete-bucket-policy --bucket mon-bucket
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-bucket-policy : aucun output si succès
>
> # get-bucket-policy retourne :
> {
>     "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"AllowPublicRead\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::mon-bucket/*\"}]}"
> }
>
> # delete-bucket-policy : aucun output si succès
> ```
**Point important** : Les bucket policies s'ajoutent aux **ACL (Access Control Lists)**, il faut les deux pour une sécurité complète.

### 3.8 Bonnes pratiques S3

- Activez le **versioning** dès que vous stockez des fichiers importants.
- Utilisez **SSE-S3** pour un chiffrement simple et automatique.
- Créez des **règles de cycle de vie** pour archiver ou supprimer les fichiers inutilisés.
- Choisissez la **classe de stockage** adaptée à chaque type de données.
- Appliquez des **bucket policies** pour restreindre l'accès selon le principe du moindre privilège.
- Utilisez **Intelligent-Tiering** si l'accès est imprévisible.
- Activez **Transfer Acceleration** pour les uploads volumineux critiques.

---

## 4. Gestion de S3 en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Storage) vous permettra d'approfondir la manipulation de S3 en CLI.
### 4.1 Créer un bucket S3

Le nom d'un bucket S3 doit être **unique à l'échelle mondiale** (personne d'autre dans le monde ne peut avoir le même nom). La syntaxe utilise `$(date +%s)` pour ajouter un timestamp et éviter les conflits.

```bash
# Créer un bucket (le nom doit être unique à l'échelle mondiale AWS)
aws s3api create-bucket \
    --bucket mon-bucket-formation-$(date +%s) \
    --region eu-west-1

# Vérifier la création
aws s3api list-buckets \
    --query 'Buckets[].Name' \
    --output table
```

**Explications** :
- `create-bucket` : commande pour créer un nouveau bucket
- `--bucket` : nom du bucket (doit respecter les règles : minuscules, pas d'espaces)
- `--region` : région AWS cible
- `--query` : filtre la réponse pour n'afficher que les noms
- `--output table` : affiche le résultat sous forme de tableau

> [!TIP]
> **Résultat attendu :**
> ```
> # create-bucket retourne :
> {
>     "Location": "/mon-bucket-formation-1715936400"
> }
>
> # list-buckets --output table affiche :
> -----------------------------------------
> |              ListBuckets              |
> +---------------------------------------+
> |  mon-bucket-formation-1715936400      |
> |  autre-bucket-existant                |
> +---------------------------------------+
> ```
### 4.2 Télécharger un fichier dans S3

On crée d'abord un fichier local de test, puis on le pousse dans S3. La commande `s3 cp` fonctionne dans les deux sens : local → S3 ou S3 → local.

```bash
# Créer un fichier d'exemple
echo "Ceci est un contenu test pour S3" > test.txt

# Télécharger le fichier
aws s3 cp test.txt s3://mon-bucket-formation/documents/test.txt

# Vérifier le téléchargement
aws s3 ls s3://mon-bucket-formation/documents/ --recursive
```

**Explications** :
- `s3 cp` : copie un fichier local vers S3
- `s3://` : préfixe pour les chemins S3
- `s3 ls` : liste les contenus d'un bucket S3
- `--recursive` : affiche les fichiers dans les sous-dossiers aussi

> [!TIP]
> **Résultat attendu :**
> ```
> upload: ./test.txt to s3://mon-bucket-formation/documents/test.txt
>
> # s3 ls retourne :
> 2024-05-17 14:23:05         34 test.txt
> ```
> [!WARNING]
> **Coûts S3 : requêtes et transfert** — Chaque opération `s3 cp` ou `s3 ls` génère des requêtes facturées (PUT, GET, LIST). Le tarif standard est ~0,005 $ pour 1 000 requêtes PUT et ~0,004 $ pour 1 000 requêtes GET. Les transferts de données **sortants d'AWS vers Internet** sont facturés (~0,09 $/Go). Préférez regrouper les petits fichiers et minimisez les téléchargements hors AWS pour maîtriser les coûts.
### 4.3 Activer le versioning sur un bucket

Le versioning conserve toutes les versions d'un fichier. Si on écrase `test.txt`, l'ancienne version reste accessible. Une fois activé, le versioning **ne peut pas être désactivé** (seulement suspendu).

```bash
# Activer le versioning
aws s3api put-bucket-versioning \
    --bucket mon-bucket-formation \
    --versioning-configuration Status=Enabled

# Vérifier le versioning
aws s3api get-bucket-versioning \
    --bucket mon-bucket-formation
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-bucket-versioning : aucun output si succès
>
> # get-bucket-versioning retourne :
> {
>     "Status": "Enabled"
> }
> ```
> [!WARNING]
> **Versioning et coûts de stockage** — Une fois activé, le versioning conserve **toutes les versions** de chaque objet — y compris celles écrasées ou supprimées. Si vous modifiez fréquemment des fichiers volumineux, le stockage total peut rapidement multiplier les coûts. Combinez toujours le versioning avec une **lifecycle policy** qui expire les anciennes versions (ex. : supprimer les versions antérieures à 90 jours).
### 4.4 Créer une politique de cycle de vie

Automatiser le déplacement des données vers des classes de stockage moins chères est une bonne pratique FinOps. Ici, on définit deux règles : les logs passent en STANDARD_IA à 30 jours, puis en GLACIER à 90 jours ; les fichiers temporaires sont supprimés après 1 an.

```bash
# Créer un fichier JSON pour la politique
cat > lifecycle-policy.json << 'EOF'
{
  "Rules": [
    {
      "Id": "ArchiveAfter30Days",
      "Status": "Enabled",
      "Prefix": "logs/",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    },
    {
      "Id": "DeleteAfter365Days",
      "Status": "Enabled",
      "Prefix": "temp/",
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
EOF

# Appliquer la politique
aws s3api put-bucket-lifecycle-configuration \
    --bucket mon-bucket-formation \
    --lifecycle-configuration file://lifecycle-policy.json
```

**Explications** :
- `put-bucket-lifecycle-configuration` : applique une politique de cycle de vie
- `Transitions` : changement automatique de classe de stockage selon l'âge
- `Expiration` : suppression automatique après X jours
- `Prefix` : la règle s'applique uniquement aux fichiers commençant par ce préfixe

> [!TIP]
> **Résultat attendu :**
> ```
> # put-bucket-lifecycle-configuration : aucun output si succès
>
> # get-bucket-lifecycle-configuration retourne :
> {
>     "Rules": [
>         {
>             "ID": "ArchiveAfter30Days",
>             "Status": "Enabled",
>             "Prefix": "logs/",
>             "Transitions": [
>                 { "Days": 30, "StorageClass": "STANDARD_IA" },
>                 { "Days": 90, "StorageClass": "GLACIER" }
>             ]
>         },
>         {
>             "ID": "DeleteAfter365Days",
>             "Status": "Enabled",
>             "Prefix": "temp/",
>             "Expiration": { "Days": 365 }
>         }
>     ]
> }
> ```
### 4.5 Activer le chiffrement par défaut

Le chiffrement par défaut garantit que **tout objet** ajouté au bucket est automatiquement chiffré, sans que le développeur ait à y penser. C'est une exigence fréquente en entreprise pour la conformité.

```bash
# Créer une configuration de chiffrement
cat > encryption-config.json << 'EOF'
{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }
  ]
}
EOF

# Appliquer le chiffrement
aws s3api put-bucket-encryption \
    --bucket mon-bucket-formation \
    --server-side-encryption-configuration file://encryption-config.json
```

**Explications** :
- `put-bucket-encryption` : active le chiffrement automatique
- `AES256` : chiffrement SSE-S3 (clés gérées par AWS)
- Toute nouvelle donnée sera automatiquement chiffrée

> [!TIP]
> **Résultat attendu :**
> ```
> # put-bucket-encryption : aucun output si succès
>
> # get-bucket-encryption retourne :
> {
>     "ServerSideEncryptionConfiguration": {
>         "Rules": [
>             {
>                 "ApplyServerSideEncryptionByDefault": {
>                     "SSEAlgorithm": "AES256"
>                 },
>                 "BucketKeyEnabled": false
>             }
>         ]
>     }
> }
> ```
### 4.6 Bloquer l'accès public au bucket

Par défaut depuis 2023, AWS bloque l'accès public sur les nouveaux buckets. Cette commande permet de le confirmer explicitement — et de le vérifier sur des buckets existants qui pourraient être mal configurés.

```bash
# Bloquer tout accès public (meilleure pratique pour les données sensibles)
aws s3api put-public-access-block \
    --bucket mon-bucket-formation \
    --public-access-block-configuration \
        "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Vérifier la configuration
aws s3api get-public-access-block \
    --bucket mon-bucket-formation
```

**Explications** :
- `put-public-access-block` : active les protections contre l'accès accidentellement public
- `BlockPublicAcls=true` : empêche d'ajouter des ACL publiques
- `IgnorePublicAcls=true` : ignore les ACL publiques existantes
- `BlockPublicPolicy=true` : empêche les policies publiques
- `RestrictPublicBuckets=true` : restreint tous les accès publics

> [!TIP]
> **Résultat attendu :**
> ```
> # put-public-access-block : aucun output si succès
>
> # get-public-access-block retourne :
> {
>     "PublicAccessBlockConfiguration": {
>         "BlockPublicAcls": true,
>         "IgnorePublicAcls": true,
>         "BlockPublicPolicy": true,
>         "RestrictPublicBuckets": true
>     }
> }
> ```
> [!IMPORTANT]
> **Bucket accessible publiquement** — Si l'un des quatre paramètres est à `false`, le bucket peut être exposé publiquement. Des milliers de fuites de données AWS ont été causées par des buckets S3 mal configurés. Vérifiez systématiquement cette configuration sur vos buckets de production, en particulier ceux créés avant 2023 ou migrés depuis un autre compte AWS.
### 4.7 Télécharger un fichier depuis S3

La commande `s3 sync` est particulièrement utile pour les sauvegardes : elle compare le contenu local et S3, et ne transfère que les fichiers qui ont changé (plus rapide et économique qu'un `cp` complet).

```bash
# Télécharger un fichier spécifique
aws s3 cp s3://mon-bucket-formation/documents/test.txt ./test-local.txt

# Télécharger un dossier entier en récursif
aws s3 cp s3://mon-bucket-formation/documents/ ./documents/ --recursive

# Synchroniser un bucket avec un dossier local (bidirectionnel)
aws s3 sync s3://mon-bucket-formation/ ./backup-local/

# Afficher la taille d'un objet
aws s3api head-object \
    --bucket mon-bucket-formation \
    --key documents/test.txt \
    --query 'ContentLength' \
    --output text
```

**Explications** :
- `s3 cp` : copie des fichiers depuis S3 vers local
- `--recursive` : copie tous les fichiers d'un "dossier"
- `s3 sync` : synchronise des répertoires (idéal pour sauvegarde)
- `head-object` : récupère les métadonnées sans télécharger le fichier

> [!TIP]
> **Résultat attendu :**
> ```
> # s3 cp retourne :
> download: s3://mon-bucket-formation/documents/test.txt to ./test-local.txt
>
> # s3 sync retourne :
> download: s3://mon-bucket-formation/images/logo.png to ./backup-local/images/logo.png
> download: s3://mon-bucket-formation/css/style.css to ./backup-local/css/style.css
> download: s3://mon-bucket-formation/documents/test.txt to ./backup-local/documents/test.txt
>
> # head-object retourne (taille en octets) :
> 34
> ```
> [!WARNING]
> **Coûts de transfert sortant (egress)** — Le téléchargement de données depuis S3 vers Internet est facturé (~0,09 $/Go pour les premiers 10 To). Les transferts entre services AWS dans la même région sont gratuits. Si vous synchronisez régulièrement des données volumineuses vers des machines hors AWS (postes développeurs, serveurs on-premise), anticipez ce coût dans votre budget.
---

## 5. Amazon EC2 : La couche de calcul AWS

### 5.1 Introduction à EC2

📹 **Vidéo** : [Lancer sa première machine virtuelle Windows avec EC2](https://www.youtube.com/watch?v=aARcLxcGJaU)

Après avoir stocké nos données avec Amazon S3, nous allons voir comment **les traiter, les héberger ou les exécuter** grâce à **Amazon Elastic Compute Cloud (EC2)**.

EC2 est l'un des premiers services historiques d'AWS (2006). Il permet de **louer de la puissance de calcul à la demande**, avec une flexibilité inégalée par rapport aux serveurs physiques traditionnels.

📎 [Documentation officielle Amazon EC2](https://docs.aws.amazon.com/ec2/)

**Amazon EC2 (Elastic Compute Cloud)** est le service AWS qui permet de créer des **machines virtuelles** dans le cloud, appelées **instances EC2**.

### 5.2 Pourquoi utiliser EC2 ?

EC2 reprend le principe familier d'un serveur physique — un système d'exploitation, du CPU, de la RAM, du stockage, une carte réseau — mais en supprime toutes les contraintes matérielles, ce qui explique son adoption massive comme brique de calcul de base sur AWS.

Le **lancement est rapide** : là où commander, recevoir et configurer un serveur physique prenait des semaines, une instance EC2 est prête à l'emploi en quelques clics ou quelques lignes de CLI, avec un système d'exploitation déjà installé. Le service est aussi **flexible** : vous choisissez la puissance de calcul, le système d'exploitation, le type de stockage attaché et la configuration réseau, et vous pouvez faire évoluer ces choix a posteriori si les besoins changent — un projet peut commencer sur une petite instance et migrer vers une plus puissante sans réinstallation. Le modèle est **économique** parce que la facturation suit la consommation réelle plutôt qu'un investissement matériel figé : vous payez à l'heure ou à la seconde pour ce qui tourne, et vous pouvez arrêter une instance dès qu'elle n'est plus utile pour cesser d'être facturé. Enfin, EC2 est nativement **connecté** au reste de l'écosystème AWS : une instance peut lire et écrire dans un bucket S3, s'authentifier via un rôle IAM sans stocker de clé d'accès, et vivre dans un VPC dont vous contrôlez entièrement le découpage réseau — cette intégration native évite d'avoir à recoller manuellement des briques hétérogènes comme sur une infrastructure on-premise.

### 5.3 Les composants essentiels d'une instance EC2

| Composant | Rôle dans l'architecture EC2 |
|---|---|
| **Instance EC2** | Machine virtuelle hébergée chez AWS |
| **AMI** | Image système (Linux, Windows, etc.) utilisée comme modèle |
| **Type d'instance** | Détermine la puissance (CPU, RAM, réseau) |
| **Security Group** | Pare-feu virtuel qui contrôle les accès réseau |
| **EBS** | Disque dur virtuel attaché à l'instance |
| **EFS** | Système de fichiers partagé entre plusieurs instances |
| **Key Pair** | Clé SSH utilisée pour se connecter à l'instance en toute sécurité |

_Lors du lancement d'une instance, vous devez choisir chacun de ces éléments._

---

## 6. Choisir le bon type d'instance, AMI, stockage et sécurité

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

> [!WARNING]
> **Types d'instances coûteux** — Les familles `g`, `p` (GPU) et `x` (mémoire très haute) peuvent coûter plusieurs dizaines de dollars **par heure**. Par exemple, une instance `p3.16xlarge` (GPU ML) dépasse 24 $/h. Ne lancez ces types que si votre workload le justifie, et pensez à les **arrêter immédiatement** après utilisation. En formation ou développement, restez sur des types `t3.micro` ou `t3.small`.
**Explication des suffixes de type** :
- `t3` : type t (général), génération 3
- `micro`, `small`, `medium` : taille croissante
- `xlarge` ou `2xlarge` : très puissants, pour les charges importantes
- le **`g`** que l'on trouve dans `t4g`, `m6g`, `c6g`, etc. signale que l'instance tourne sur un processeur **Graviton**, la puce ARM conçue par AWS elle-même (au lieu d'un processeur x86 classique Intel/AMD). Les instances Graviton offrent généralement un meilleur rapport performance/prix (jusqu'à 20-40 % moins cher à performance équivalente), mais nécessitent que votre application soit compilée pour l'architecture ARM — la plupart des langages interprétés (Python, Node.js, Java) et des images Docker officielles la supportent nativement, mais un vieux binaire compilé spécifiquement pour x86 ne fonctionnera pas dessus sans recompilation.

### 6.2 AMI (Amazon Machine Image)

L'AMI est le **système d'exploitation** de votre machine EC2.

📹 **Vidéo** : [AMI — Amazon Machine Images](https://www.youtube.com/watch?v=xjZx37dsVRw)

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

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-02-c1a1127e.png)

> Il est souvent utile en entreprise de créer ses propres AMI afin de pouvoir déployer plus rapidement des instances EC2 correspondant aux besoins spécifiques. Vous pouvez enregistrer le disque contenant cette AMI après lancement de la machine EC2 et après avoir ajouté les spécificités de l'ensemble de vos machines.

### 6.3 Stockage associé à EC2

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

### 6.4 Sécurité EC2

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

### 6.5 Options de conformité EC2

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

> [!IMPORTANT]
> **Instance Store : perte de données garantie à l'arrêt** — Contrairement à EBS, le stockage instance store **n'est pas persistant**. Toutes les données écrites dessus sont définitivement perdues si l'instance est arrêtée, terminée ou si l'hôte physique tombe en panne. Ne stockez jamais de données de production, de bases de données ou de fichiers importants sur instance store sans sauvegarde préalable vers S3 ou EBS.
#### Encrypted EBS Volumes

- Les volumes EBS peuvent être chiffrés avec **AWS KMS**.
- Le chiffrement est **transparent** pour l'application.
- Utile pour la conformité HIPAA, PCI-DSS, ISO 27001.

**Cas d'usage conformité** :
- Données médicales → Dedicated Instance + EBS chiffré + Audit CloudTrail.
- Données financières → Dedicated Host + KMS + VPC isolé.
- Données RGPD → Région EU + Versioning S3 + Chiffrement.

---

## 7. AWS Compute Optimizer — Dimensionnement optimal

### 7.1 Qu'est-ce que AWS Compute Optimizer ?

**AWS Compute Optimizer** est un service qui **analyse vos patterns d'utilisation** des instances EC2 et recommande des types plus optimisés en coût et performance.

| | Instance actuelle | Recommandation |
|---|---|---|
| Type | t3.large (trop puissante) | t3.small (plus économique) |
| CPU utilisation | 5% | — |
| Mémoire | 12% | — |
| Coût mensuel | 80 $ | ~48 $ (économie 60%) |
| Performance | — | identique |
| Risque | — | très faible (marges CPU) |

Compute Optimizer applique cette analyse automatiquement à partir des métriques CloudWatch collectées.

### 7.2 Fonctionnement

1. **Collecte** : Compute Optimizer récupère les métriques CloudWatch (CPU, mémoire, réseau) sur **14 jours minimum**.
2. **Analyse** : Machine Learning compare votre utilisation réelle avec les capabilities des autres types.
3. **Recommandation** : Propose des types économiquement viables.
4. **Confiance** : Indique un score de confiance (low, medium, high).

### 7.3 Types de recommandations

| Recommandation | Bénéfice | Risque | Exemple |
|---|---|---|---|
| **Downsizer** | Économies importantes | Risque d'augmenter le CPU > 100% | t3.large → t3.small |
| **Upgrade** | Meilleure performance | Légère augmentation de coût | m5.large → m5.xlarge |
| **Switch Family** | Meilleure performance/$ | Changement d'architecture | t3.large → m6i.large |
| **Aucune recommandation** | Instance bien dimensionnée | N/A | ✅ Garder tel quel |

### 7.4 Activation et utilisation

Compute Optimizer analyse l'utilisation réelle de vos instances (CPU, mémoire, réseau) sur 14 jours et suggère le type le mieux adapté. La commande suivante affiche ces recommandations sous forme de tableau comparatif.

```bash
# Vérifier les recommandations Compute Optimizer
aws compute-optimizer get-ec2-instance-recommendations \
    --region eu-west-1 \
    --query 'instanceRecommendations[].{
        Instance:instanceArn,
        Current:currentInstanceType,
        Recommended:recommendationOptions[0].instanceType,
        Savings:recommendationOptions[0].savingsOpportunity.estimatedMonthlySavings.value,
        ConfidenceLevel:currentInstanceType
    }' \
    --output table
```

> [!TIP]
> **Résultat attendu :**
> ```
> -------------------------------------------------------------------------------------------
> |                         GetEc2InstanceRecommendations                                   |
> +-------------------------------------+----------+------------+---------+------------------+
> |              Instance               | Current  | Recommended| Savings | ConfidenceLevel  |
> +-------------------------------------+----------+------------+---------+------------------+
> |  arn:aws:ec2:eu-west-1:123:instance | t3.large | t3.small   |  47.82  |  t3.large        |
> |  arn:aws:ec2:eu-west-1:123:instance | m5.xlarge| m5.large   |  62.40  |  m5.xlarge       |
> +-------------------------------------+----------+------------+---------+------------------+
> ```
> Si aucune recommandation n'apparaît, Compute Optimizer manque encore de données (il lui faut au minimum 30h d'activité sur les instances).
### 7.5 Cas d'usage

- **Optimisation de coûts** : identifier toutes les instances surdimensionnées.
- **Gouvernance cloud** : politiques de rightsizing automatisées.
- **Migration** : recommandations pour basculer vers une architecture nouvelle.
- **Audit FinOps** : justification des dépenses EC2.

**Avantage clé** : Compute Optimizer s'appuie sur **12-14 jours de données réelles**, pas sur des hypothèses théoriques.

---

## 8. Options de tarification AWS EC2

AWS propose plusieurs modèles de tarification pour s'adapter aux besoins techniques et budgétaires des entreprises. Le choix dépend du niveau de prévisibilité des workloads, du budget disponible, et de la tolérance aux interruptions.

### 8.1 On-Demand (À la demande)

- **Paiement à l'heure** ou à la seconde pour la capacité de calcul utilisée, sans engagement à long terme.
- **Idéal pour** les charges de travail à court terme, les tests et le développement.
- **Pas de paiement anticipé** ni d'engagement minimum.
- **Prix plus élevé** que les autres options mais offre une flexibilité maximale.
- **Recommandé pour** les applications ne pouvant pas être interrompues et ayant des charges de travail imprévisibles.

**Exemple** : Vous avez un pic de trafic imprévu. Vous lancez des instances On-Demand pour répondre à la demande, puis les arrêtez après le pic.

### 8.2 Savings Plans

- **Engagement de consommation horaire en dollars** ($/heure) sur une période de 1 ou 3 ans.
- **Réductions** pouvant atteindre **72%** par rapport au tarif à la demande.
- **Deux types principaux** :
  - **Compute Savings Plans** : Flexibilité maximale couvrant EC2, Fargate et Lambda, avec support multi-familles d'instances, tailles et régions.
  - **EC2 Instance Savings Plans** : Réductions plus importantes mais limité à une famille d'instances dans une région spécifique.
- **Options de paiement** flexibles impactant le taux de réduction :
  - **No Upfront** : Aucun paiement initial.
  - **Partial Upfront** : Paiement partiel initial.
  - **Full Upfront** : Paiement total initial offrant les meilleures réductions.

### 8.3 Instances Spot (À prix réduit)

- **Utilisation de la capacité EC2 inutilisée** d'AWS.
- **Réductions jusqu'à 90%** par rapport au prix à la demande.
- **Les instances peuvent être interrompues** avec un préavis de 2 minutes si AWS a besoin de la capacité.
- **Idéal pour** :
  - Les charges de travail tolérantes aux interruptions.
  - Le calcul haute performance (HPC).
  - Les jobs batch (traitement par lots).
  - Les workloads flexibles en termes de début et de fin.

**Best practices pour la résilience** :
- Utiliser les **groupes d'auto-scaling** pour gérer les interruptions automatiquement.
- Implémenter via **EC2 Spot Fleet** ou **EC2 Spot Instances Requests**.
- Concevoir l'application pour tolérer les interruptions.

> [!WARNING]
> **Instances Spot : interruption en 2 minutes** — AWS peut récupérer vos instances Spot avec seulement **2 minutes de préavis** lorsque la capacité est nécessaire. Ne jamais utiliser des instances Spot pour des workloads critiques sans tolérance aux interruptions (bases de données de production, serveurs web sans état de session externalisé). Toujours prévoir un mécanisme de sauvegarde ou de checkpoint des données en cours de traitement.
### 8.4 Reserved Instances (RI)

- **Engagement** sur une instance spécifique pour **1 ou 3 ans**.
- **Réductions jusqu'à 75%** par rapport au prix à la demande.
- **Différences principales** avec les Savings Plans :
  - Les RI sont liées à une instance spécifique (type, taille, région, zone).
  - Les Savings Plans sont basés sur un engagement de consommation en dollars (plus flexibles).
  - Les RI peuvent être vendues sur le **AWS RI Marketplace**, pas les Savings Plans.

### 8.5 Comparatif synthétique

| Critère | On-Demand | Reserved | Spot | Savings Plans |
|---------|-----------|----------|------|----------------|
| **Engagement** | Aucun | 1 ou 3 ans | Aucun | 1 ou 3 ans |
| **Réduction potentielle** | ❌ | ✅ (75%) | ✅✅✅ (90%) | ✅✅ (72%) |
| **Flexibilité** | ✅✅✅ | ❌ | ✅✅ | ✅✅ |
| **Risque d'interruption** | ❌ | ❌ | ✅✅✅ | ❌ |
| **Idéal pour** | Dev/Test | Prod stable | Batch/CI | Prod optimisée |

**Recommandation** : Pour la plupart des entreprises, **Savings Plans** offre le meilleur compromis entre réduction (72%) et flexibilité.

### 8.6 Cas métier : Choisir la meilleure option tarifaire

#### Cas 1 : Site e-commerce avec trafic prévisible

- **Charge** : trafic stable, pics prévisibles en fin d'année.
- **Infrastructure** : 10 instances t3.large en continu, +20 during soldes.
- **Recommandation** : **Savings Plans (Compute)** pour les 10 instances permanentes + **Spot** pour les 20 supplémentaires pendant les soldes.
- **Économie** : ~72% sur la base, ~90% sur les renforts = **78% global**.

```
Coût mensuel sans optimisation (tout On-Demand) :
  10 × 730h × 0,096$ (t3.large) = 699 $

Coût optimisé (Savings Plans + Spot) :
  10 × (0,096$ × 0,28) + 20 × (0,096$ × 0,10) = 26,88 $ + 19,2 $ = 46,08 $

Économie : 652,92 $ / mois = 7,835 $ / an
```

#### Cas 2 : Environnement de développement/test

- **Charge** : variable, utilisation heures de travail uniquement.
- **Infrastructure** : 2-4 instances selon le sprint en cours.
- **Recommandation** : **On-Demand** uniquement (pas d'engagement, flexibilité totale).
- **Économie** : aucune, mais coûts minimaux et liberté maximale.

#### Cas 3 : Job batch nocturne de traitement

- **Charge** : lance chaque nuit des instances pour 4h, puis arrêt.
- **Infrastructure** : 50 instances c5.2xlarge pour le parallélisme.
- **Recommandation** : **Spot instances** avec **Spot Fleet** (demande auto-scaling de remplacement).
- **Économie** : ~90% vs On-Demand = **81$/j au lieu de 810$**, soit 8,100$/mois.

```bash
# Configuration Spot Fleet pour job batch
aws ec2 request-spot-fleet \
    --spot-fleet-request-config '{
        "IamFleetRole": "arn:aws:iam::xxxxx:role/fleet",
        "SpotPrice": "0.10",
        "TargetCapacity": 50,
        "LaunchSpecifications": [{
            "ImageId": "ami-xxxxx",
            "InstanceType": "c5.2xlarge",
            "KeyName": "ma-cle"
        }]
    }'
```

> [!TIP]
> **Résultat attendu :**
> ```
> {
>     "SpotFleetRequestId": "sfr-0a1b2c3d4e5f6789a",
>     "SpotFleetRequestState": "submitted"
> }
> ```
#### Cas 4 : Application critiques 24/7 avec charge non prévisible

- **Charge** : pas de pattern clair, augmentations soudaines.
- **Infrastructure** : 4-30 instances selon la demande.
- **Recommandation** : **Savings Plans (mélange)** pour la charge de base + **On-Demand** pour les pics.
- **Avantage** : si charge explose au-delà des prévisions, On-Demand absorbe sans coupure.

### 8.7 Outil : AWS Pricing Calculator

```
URL : https://calculator.aws/
1. Sélectionner la région
2. Ajouter EC2 : type, nombre, durée
3. Sélectionner option (On-Demand, Reserved, Spot)
4. Voir l'estimation mensuelle/annuelle
5. Exporter en PDF pour justifier budgets
```

📎 [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

## 9. Lancer une instance EC2 en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Compute) vous permettra d'approfondir le lancement d'instances EC2 en CLI.
### 9.1 Créer une Key Pair

La Key Pair est l'équivalent d'une clé SSH. AWS génère la paire (privée + publique), conserve la clé publique, et vous remet la clé privée **une seule fois**. Le `chmod 600` est obligatoire — SSH refuse de se connecter si la clé est trop permissive.

```bash
# Créer une nouvelle paire de clés
aws ec2 create-key-pair \
    --key-name ma-cle-formation \
    --query 'KeyMaterial' \
    --output text > ~/.ssh/ma-cle-formation.pem

# Définir les permissions correctes (très important pour SSH)
chmod 600 ~/.ssh/ma-cle-formation.pem

# Vérifier les clés existantes
aws ec2 describe-key-pairs \
    --query 'KeyPairs[].KeyName' \
    --output table
```

**Explications** :
- `create-key-pair` : génère une nouvelle paire de clés
- `--key-name` : identifiant de la clé
- `--output text` : affiche juste le contenu privé
- `chmod 600` : permissions restrictives (propriétaire seul peut lire)

> [!TIP]
> **Résultat attendu :**
> ```
> # La clé privée est écrite dans ~/.ssh/ma-cle-formation.pem (aucun output CLI)
>
> # describe-key-pairs --output table affiche :
> ------------------------------------
> |        DescribeKeyPairs          |
> +----------------------------------+
> |       ma-cle-formation           |
> +----------------------------------+
> ```
> [!WARNING]
> **Clé privée : une seule chance de la télécharger** — AWS ne conserve jamais la clé privée. Si vous perdez le fichier `.pem`, vous devrez créer une nouvelle Key Pair et relancer une instance — il est impossible de récupérer l'accès SSH autrement. Sauvegardez systématiquement votre clé dans un gestionnaire de secrets (AWS Secrets Manager, HashiCorp Vault) ou dans un endroit sécurisé.
### 9.2 Lancer une instance EC2 simple

On récupère d'abord l'ID de la dernière AMI Ubuntu 20.04 disponible, puis on lance une instance `t3.micro` (éligible au Free Tier). La variable `$AMI_ID` évite de copier-coller un ID d'image qui change régulièrement.

```bash
# Récupérer l'AMI Ubuntu la plus récente
AMI_ID=$(aws ec2 describe-images \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*" \
    --query 'sort_by(Images, &CreationDate)[-1].[ImageId]' \
    --output text)

# Lancer l'instance
aws ec2 run-instances \
    --image-id $AMI_ID \
    --instance-type t3.micro \
    --key-name ma-cle-formation \
    --security-groups sg-formation \
    --region eu-west-1 \
    --query 'Instances[0].[InstanceId,PublicIpAddress,State.Name]' \
    --output table
```

**Explications** :
- `describe-images` : recherche d'une AMI Ubuntu récente
- `--image-id` : l'AMI à utiliser (système d'exploitation)
- `--instance-type t3.micro` : type d'instance (gratuit dans Free Tier)
- `--key-name` : la clé SSH pour se connecter
- `--security-groups` : pare-feu assigné à l'instance

> [!TIP]
> **Résultat attendu :**
> ```
> ---------------------------------------------------
> |               RunInstances                      |
> +---------------------+---------------+-----------+
> |     InstanceId      | PublicIpAddr  |   State   |
> +---------------------+---------------+-----------+
> |  i-0abc123def45678  |  None         |  pending  |
> +---------------------+---------------+-----------+
>
> # Après ~30 secondes, l'état passe à "running" :
> +---------------------+---------------+-----------+
> |  i-0abc123def45678  |  54.171.23.45 |  running  |
> +---------------------+---------------+-----------+
> ```
### 9.3 Se connecter à l'instance

La première commande récupère l'IP publique de l'instance via CLI (plutôt que de la copier depuis la console), la seconde s'y connecte en SSH avec la clé créée précédemment.

```bash
# Obtenir l'adresse IP publique
INSTANCE_ID="i-xxxxx"
IP=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' \
    --output text)

# Se connecter en SSH
ssh -i ~/.ssh/ma-cle-formation.pem ubuntu@$IP

# Une fois connecté, les commandes Linux habituelles s'exécutent normalement
```

**Explications** :
- `describe-instances` : récupère les infos de l'instance
- `PublicIpAddress` : adresse IP pour accès depuis Internet
- `-i ~/.ssh/ma-cle-formation.pem` : utilise votre clé privée
- `ubuntu` : utilisateur par défaut dans les AMI Ubuntu

> [!TIP]
> **Résultat attendu :**
> ```
> # describe-instances retourne l'IP :
> 54.171.23.45
>
> # Connexion SSH réussie :
> Warning: Permanently added '54.171.23.45' (ED25519) to the list of known hosts.
> Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1053-aws x86_64)
>
>  * Documentation:  https://help.ubuntu.com
>  * Management:     https://landscape.canonical.com
>  * Support:        https://ubuntu.com/advantage
>
> ubuntu@ip-10-0-1-45:~$
> ```
> [!WARNING]
> **Sécurité SSH : limitez l'accès au port 22** — Ouvrir le port 22 à `0.0.0.0/0` (toutes les IPs) expose votre instance aux scanners automatisés et tentatives de brute force. Dans votre Security Group, restreignez toujours l'accès SSH à votre adresse IP publique uniquement (`monip/32`). Utilisez `curl ifconfig.me` pour connaître votre IP courante.
### 9.4 Créer une image (AMI) à partir d'une instance

Créer une AMI depuis une instance configurée permet de la **cloner** rapidement : toutes vos installations et configurations sont préservées. C'est la base du déploiement automatisé avec Auto Scaling.

```bash
# Créer une AMI personnalisée
aws ec2 create-image \
    --instance-id i-xxxxx \
    --name "Ma-Formation-AMI" \
    --description "AMI avec mes configurations de formation" \
    --no-reboot \
    --output table

# Vérifier la création
aws ec2 describe-images \
    --owners self \
    --query 'Images[].[ImageId,Name,State]' \
    --output table
```

**Explications** :
- `create-image` : crée une nouvelle AMI à partir d'une instance
- `--no-reboot` : ne redémarre pas l'instance (accélère le processus)
- `--owners self` : affiche uniquement vos AMI personnalisées

> [!TIP]
> **Résultat attendu :**
> ```
> # create-image retourne :
> {
>     "ImageId": "ami-0a1b2c3d4e5f67890"
> }
>
> # describe-images --output table affiche :
> ---------------------------------------------------------
> |                   DescribeImages                      |
> +---------------------+------------------+--------------+
> |       ImageId       |       Name       |    State     |
> +---------------------+------------------+--------------+
> |  ami-0a1b2c3d4e5f6  |  Ma-Formation-AMI|   pending    |
> +---------------------+------------------+--------------+
>
> # Après quelques minutes :
> |  ami-0a1b2c3d4e5f6  |  Ma-Formation-AMI|  available   |
> ```
### 9.5 Créer un snapshot EBS

Un snapshot EBS est une sauvegarde du disque d'une instance. Contrairement à une AMI (qui inclut le système complet), le snapshot est juste le contenu du volume — utile pour restaurer des données sans recréer toute une instance.

```bash
# Lister les volumes EBS attachés
aws ec2 describe-volumes \
    --filters "Name=attachment.instance-id,Values=i-xxxxx" \
    --query 'Volumes[].[VolumeId,Size,State]' \
    --output table

# Créer un snapshot d'un volume
aws ec2 create-snapshot \
    --volume-id vol-xxxxx \
    --description "Snapshot de sauvegarde formation" \
    --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=backup-formation}]' \
    --output table

# Vérifier l'état du snapshot
aws ec2 describe-snapshots \
    --owner-ids self \
    --query 'Snapshots[].[SnapshotId,VolumeSize,State,Progress]' \
    --output table
```

**Explications** :
- `describe-volumes` : liste les disques EBS attachés à une instance
- `create-snapshot` : crée une sauvegarde du disque
- `--tag-specifications` : ajoute des étiquettes pour l'organiser
- Le snapshot est **incrémental** : seuls les changements sont sauvegardés

> [!TIP]
> **Résultat attendu :**
> ```
> # describe-volumes --output table :
> -------------------------------------------
> |          DescribeVolumes                |
> +------------------+------+---------------+
> |    VolumeId      | Size |     State     |
> +------------------+------+---------------+
> |  vol-0a1b2c3d4e  |  20  |   in-use      |
> +------------------+------+---------------+
>
> # create-snapshot retourne :
> {
>     "SnapshotId": "snap-0a1b2c3d4e5f67890",
>     "VolumeId": "vol-0a1b2c3d4e",
>     "State": "pending",
>     "Progress": ""
> }
>
> # describe-snapshots après quelques minutes :
> --------------------------------------------------------------
> |                   DescribeSnapshots                        |
> +---------------------+------+-----------+---------+--------+
> |     SnapshotId      | Size |   State   |Progress |        |
> +---------------------+------+-----------+---------+--------+
> |  snap-0a1b2c3d4e5  |  20  |  completed|  100%   |        |
> +---------------------+------+-----------+---------+--------+
> ```
> [!WARNING]
> **Coûts des snapshots EBS** — Les snapshots sont stockés dans S3 (managé par AWS) et facturés ~0,05 $/Go-mois. Un snapshot de 100 Go coûte ~5 $/mois. Les snapshots sont **incrémentiels** (seules les modifications depuis le dernier snapshot sont stockées), mais les premiers snapshots peuvent être volumineux. Planifiez une politique de rétention et supprimez les snapshots obsolètes pour maîtriser les coûts.
---

## 10. Elastic Load Balancing (ELB) — Répartition du trafic

### 10.1 Pourquoi un Load Balancer ?

Un **Load Balancer** agit comme un répartiteur de trafic. Il reçoit les requêtes des clients et les distribue vers les instances EC2 disponibles, selon des règles de routage et de santé (**health checks**).

#### Architecture simple

<img src="formations/aws-initiation-approfondissement/11-images/elb-architecture.svg"
     alt="Elastic Load Balancing — Architecture"
     style="display:block; margin:auto; width:90%">

### 10.2 Types de Load Balancer AWS

| Type | Cas d'usage typique | Protocole | Niveau OSI |
|---|---|---|---|
| **ALB (Application Load Balancer)** | Applications web, microservices | HTTP/HTTPS | Couche 7 (Application) |
| **NLB (Network Load Balancer)** | Faible latence, TCP | TCP/UDP | Couche 4 (Transport) |
| **GLB (Gateway Load Balancer)** | Appliances réseau (firewall, inspection) | IP | Couche 3 (Réseau) |

**ALB (Application Load Balancer)** : conçu pour les applications web. Il fonctionne au niveau **HTTP/HTTPS** (couche 7 du modèle OSI) et permet un **routage avancé** (par URL, en-tête, hostname, etc.).

**NLB (Network Load Balancer)** : adapté aux applications nécessitant une **faible latence**. Il fonctionne au niveau **TCP** (couche 4), idéal pour les bases de données ou les services temps réel.

**GLB (Gateway Load Balancer)** : utilisé pour intégrer des **appliances réseau** comme des pare-feu ou des outils d'inspection. Il fonctionne au niveau **IP**.

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-03-43719ff5.png)

### 10.3 Fonctionnement du Load Balancer

- Le Load Balancer **vérifie l'état** des instances via des **health checks** (tests de disponibilité).
- Il **répartit les requêtes** vers les instances **saines** uniquement.
- Il s'**adapte automatiquement** à l'ajout ou la suppression d'instances via **Auto Scaling**.

**Health Check - Exemple** :
```
Chaque 30 secondes :
1. LB envoie requête GET http://instance:80/health
2. Instance répond HTTP 200 OK
3. Instance est marquée "saine"

Si pas de réponse ou erreur 5xx :
1. Instance marquée "défaillante"
2. Pas plus de trafic envoyé vers elle
```

📎 [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)

---

## 11. Auto Scaling — Adaptation dynamique des ressources

### 11.1 Qu'est-ce qu'Auto Scaling ?

Un **Auto Scaling Group (ASG)** est un groupe d'instances EC2 géré automatiquement. Il peut être associé à un Load Balancer pour garantir que :

- Les nouvelles instances sont automatiquement **enregistrées** auprès du Load Balancer.
- Les instances défaillantes sont **retirées** du pool.
- Le trafic est toujours dirigé vers les **ressources disponibles**.

### 11.2 Politiques de scaling — Fondamentaux

Les politiques définissent **quand et comment** ajouter ou retirer des instances.

#### Scale-out (Agrandissement)

```
Charge CPU dépasse 70% pendant 5 min
                 ▼
Ajouter 2 instances supplémentaires
                 ▼
Attendre que les instances démarrent
                 ▼
Health check OK : instances intégrées au LB
```

#### Scale-in (Réduction)

```
Charge CPU chute à 30% pendant 10 min
                 ▼
Retirer 1 instance
                 ▼
Attendre que les requêtes actuelles finissent
                 ▼
Fermer l'instance, libérer les ressources
```

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-04-88735023.png)

### 11.3 Configuration d'Auto Scaling

Un ASG typique comporte :

Min Size : 2 instances · Max Size : 10 instances · Desired Capacity : 4 instances
Launch Template : my-ami-config · Load Balancer : my-alb

Scaling Policies : Target CPU 70% · Scale out +2 instances/5 min · Scale in -1 instance/10 min

**Paramètres clés** :
- **Min Size** : minimum d'instances (au moins 2 pour la haute disponibilité)
- **Max Size** : limite supérieure pour éviter les coûts explosifs
- **Desired Capacity** : nombre d'instances cible en ce moment
- **Launch Template** : modèle (AMI, type, security group, etc.) pour les nouvelles instances

### 11.4 Métriques CloudWatch et politiques de scaling avancées

Auto Scaling peut se baser sur **plusieurs métriques CloudWatch**, pas seulement CPU.

#### Métriques disponibles

| Métrique | Source | Cas d'usage typique |
|----------|--------|-------------------|
| **CPU Utilization** | CloudWatch | Charge générale, serverless |
| **NetworkIn / NetworkOut** | CloudWatch | Applications réseau intensives |
| **ALB Target Count** | CloudWatch + ELB | Nombre de requêtes traitées |
| **Request Count Per Target** | CloudWatch + ELB | Répartition de charge par instance |
| **Target Response Time** | CloudWatch + ELB | Dégradation de performance |
| **Memory Utilization** | CloudWatch Agent | Applications mémoire-intensives |
| **Queue Depth** (SQS) | SQS | Traitement asynchrone |

#### Exemple de politique de scaling multi-métriques

On crée ici deux politiques indépendantes sur le même ASG : l'une réagit au CPU, l'autre au débit réseau. AWS les évalue en parallèle et déclenche le scaling dès que l'une d'elles est satisfaite.

```bash
# Politique 1 : Scale out si CPU > 75% pendant 2 minutes
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name mon-asg \
    --policy-name cpu-scale-out \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 75.0,
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "ScaleOutCooldown": 120,
        "ScaleInCooldown": 300
    }'

# Politique 2 : Scale out si Network > 1 Gbps
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name mon-asg \
    --policy-name network-scale-out \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 70.0,
        "CustomizedMetricSpecification": {
            "MetricName": "NetworkOut",
            "Namespace": "AWS/EC2",
            "Statistic": "Average"
        }
    }'
```

> [!TIP]
> **Résultat attendu :**
> ```
> # put-scaling-policy (cpu-scale-out) retourne :
> {
>     "PolicyARN": "arn:aws:autoscaling:eu-west-1:123456789012:scalingPolicy:a1b2c3d4:autoScalingGroupName/mon-asg:policyName/cpu-scale-out",
>     "Alarms": [
>         {
>             "AlarmName": "TargetTracking-mon-asg-AlarmHigh-cpu-scale-out",
>             "AlarmARN": "arn:aws:cloudwatch:eu-west-1:123456789012:alarm:TargetTracking-mon-asg-AlarmHigh"
>         }
>     ]
> }
>
> # put-scaling-policy (network-scale-out) retourne de même avec un ARN différent
> ```
#### Cooldown Periods (délais entre actions)

- **ScaleOutCooldown** (120-300s) : attend avant la prochaine augmentation.
  - Évite les oscillations rapides (scaling de "ping-pong").
  - Laisse le temps aux instances de démarrer.

- **ScaleInCooldown** (300-900s) : plus long que scale-out.
  - Garantit l'équilibre avant réduction.
  - Préserve la performance en cas de pics rapides.

**Exemple réaliste** :
```
T=0s    CPU = 80% → Déclenche scale-out (+2 instances)
T=120s  Instances démarrent (cool-down scale-out)
T=180s  CPU = 60% → Pourrait déclencher scale-in MAIS...
T=240s  Attendre le cooldown scale-in
T=540s  CPU toujours < 30% → Scale-in (-1 instance)
```

**Conseil** : Définir des cooldowns asymétriques (court pour scale-out, long pour scale-in) pour favorer la disponibilité.

📎 [Auto Scaling EC2](https://docs.aws.amazon.com/autoscaling/ec2/)

### 11.5 Avantages combinés Load Balancer + Auto Scaling

- **Résilience** : les instances défaillantes sont automatiquement **remplacées**.
- **Scalabilité** : le nombre d'instances s'adapte à la **charge** en temps réel.
- **Performance** : le trafic est réparti de manière **optimale** entre les ressources disponibles.
- **Économie** : vous payez uniquement pour les ressources utilisées.
- **Sécurité** : le Load Balancer peut gérer les **certificats SSL/TLS** pour sécuriser les communications.

---

## 12. AWS Lambda — Le calcul sans serveur

### 12.1 Pourquoi Lambda, quand on a déjà EC2 et Auto Scaling ?

Vous venez de voir comment EC2 et Auto Scaling permettent d'adapter dynamiquement une flotte de serveurs à la charge. Mais même avec Auto Scaling, une instance EC2 minimale **tourne en permanence** — vous la payez même quand elle ne traite aucune requête.

**AWS Lambda** pousse le modèle serverless plus loin : au lieu de faire tourner un serveur en continu, vous déployez une **fonction** — un bloc de code — qu'AWS exécute uniquement quand un événement le déclenche (requête HTTP, fichier déposé sur S3, message dans une file, tâche planifiée...). Entre deux exécutions, **aucune ressource ne tourne, donc rien n'est facturé**.

| | EC2 (même avec Auto Scaling) | Lambda |
|---|---|---|
| **Ce que vous gérez** | OS, runtime, mises à jour, capacité | Uniquement votre code |
| **Facturation** | À l'heure/seconde tant que l'instance tourne | À l'exécution (durée × mémoire allouée) |
| **Charge nulle** | Coût minimal non nul (au moins 1 instance) | **0 $** — aucune exécution, aucun coût |
| **Démarrage** | Minutes (boot instance) ou secondes (déjà démarrée) | Millisecondes à quelques secondes (cold start) |
| **Durée d'exécution max** | Illimitée | **15 minutes** par exécution |

### 12.2 Fonctionnement d'une fonction Lambda

Une fonction Lambda est un paquet de code (Python, Node.js, Java, Go, etc.) associé à une configuration : mémoire allouée (128 Mo à 10 Go), timeout maximal, et un ou plusieurs **triggers** — les événements qui la déclenchent.

```
Événement déclencheur                    Fonction Lambda                Résultat
─────────────────────                    ────────────────                ────────
Requête HTTP (API Gateway)      ──►    Exécute le code       ──►    Réponse HTTP
Fichier déposé sur S3            ──►    (runtime + mémoire     ──►    Traitement du fichier
Message dans une file SQS        ──►     alloués à la demande) ──►    Traitement du message
Planification (EventBridge)      ──►                            ──►    Tâche exécutée
```

Le CPU alloué est proportionnel à la mémoire configurée — une fonction à 1 769 Mo de RAM obtient l'équivalent d'un vCPU complet. AWS gère entièrement l'infrastructure sous-jacente : vous ne choisissez ni AMI, ni type d'instance, ni Security Group pour la fonction elle-même.

### 12.3 Créer et invoquer une fonction Lambda en CLI

> [!NOTE]
> La pratique dans **AWS Academy** (module Serverless / Lambda) vous permettra d'approfondir la création et l'invocation de fonctions Lambda.
Cette séquence crée une fonction Lambda Python minimale, l'invoque manuellement, puis vérifie les logs d'exécution dans CloudWatch.

```bash
# 1. Écrire le code de la fonction (fichier local)
cat > lambda_function.py << 'EOF'
def lambda_handler(event, context):
    nom = event.get('nom', 'monde')
    return {
        'statusCode': 200,
        'body': f'Bonjour, {nom} ! Fonction exécutée avec succès.'
    }
EOF

# 2. Empaqueter le code en ZIP (format attendu par Lambda)
zip function.zip lambda_function.py

# 3. Créer le rôle IAM que la fonction va assumer (permissions d'exécution)
aws iam create-role \
  --role-name formation-lambda-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# 4. Attacher la policy minimale pour écrire les logs CloudWatch
aws iam attach-role-policy \
  --role-name formation-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# 5. Créer la fonction Lambda
aws lambda create-function \
  --function-name formation-bonjour \
  --runtime python3.12 \
  --role arn:aws:iam::123456789012:role/formation-lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --memory-size 128 \
  --timeout 10

# 6. Invoquer la fonction avec un événement de test
aws lambda invoke \
  --function-name formation-bonjour \
  --payload '{"nom": "Formation AWS"}' \
  --cli-binary-format raw-in-base64-out \
  reponse.json

cat reponse.json
```

> [!TIP]
> **Résultat attendu :**
> ```json
> {
>   "StatusCode": 200,
>   "ExecutedVersion": "$LATEST"
> }
> ```
> Contenu de `reponse.json` :
> ```json
> {"statusCode": 200, "body": "Bonjour, Formation AWS ! Fonction exécutée avec succès."}
> ```
> La fonction s'est exécutée en quelques centaines de millisecondes. Aucune instance EC2 n'a été provisionnée — Lambda a alloué l'environnement d'exécution le temps de traiter cette seule invocation, puis l'a libéré.
```bash
# 7. Consulter les logs d'exécution (CloudWatch Logs, créés automatiquement)
aws logs tail /aws/lambda/formation-bonjour --follow
```

> [!NOTE]
> **Le rôle IAM est la seule "sécurité réseau" de Lambda par défaut.** Contrairement à EC2, une fonction Lambda n'a pas de Security Group tant qu'elle n'est pas explicitement rattachée à un VPC (`--vpc-config`). Une Lambda simple qui n'a besoin que d'appeler d'autres services AWS (S3, DynamoDB) via leurs API n'a généralement pas besoin d'être dans un VPC — le rôle IAM suffit à contrôler ce qu'elle a le droit de faire.
### 12.4 Combien coûte Lambda ?

Lambda facture deux choses : le **nombre d'invocations** et la **durée × mémoire** consommée.

| Élément | Tarif (région Paris) | Free Tier mensuel |
|---|---|---|
| Invocations | 0,20 $ par million | 1 million gratuit |
| Durée de calcul | 0,0000166667 $ par Go-seconde | 400 000 Go-secondes gratuites |

**Exemple concret** : une fonction à 512 Mo (0,5 Go) qui s'exécute 200 ms, appelée 2 millions de fois par mois :
```
Invocations : 2 000 000 × 0,20 $ / 1 000 000        = 0,40 $
Durée       : 2 000 000 × 0,2 s × 0,5 Go × 0,0000166667 $ = 3,33 $
                                                    TOTAL ≈ 3,73 $/mois
```

À comparer à une instance EC2 t3.micro tournant en continu pour traiter le même trafic (~12 $/mois de base, même si elle est idle 80 % du temps) : Lambda devient très avantageux dès que la charge est **intermittente** plutôt que constante. À l'inverse, pour un service qui reçoit du trafic 24h/24 à haut volume, une instance EC2 dimensionnée correctement (ou plusieurs derrière un ELB) redevient souvent moins chère que des millions d'invocations Lambda.

📎 [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
📎 [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)

---

## 13. Architecture complète : Illustration e-commerce

Scénario réaliste — montée en charge pendant les soldes, puis retour à la normale :

### 13.1 Avant les soldes (charge normale)

Les clients normaux passent par l'Application Load Balancer, qui répartit le trafic sur trois instances EC2 (EC2-1, EC2-2, EC2-3). L'Auto Scaling Group est configuré avec Min=2, Max=10, Desired=3.

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-05-28953c2e.png)

### 13.2 Pendant les soldes (pic de trafic)

Le trafic est multiplié par 5, le CPU moyen passe à 85% — cela déclenche le scale-out : 4 instances supplémentaires sont ajoutées derrière l'Application Load Balancer, portant le total à 7 instances actives (Desired Capacity = 7).

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-06-da6c3cb3.png)

### 13.3 Après les soldes (retour à la normale)

```
Trafic revient à la normale
          
    CPU chute à 40%
          
    Déclenchement du scale-in
          
    Retirer 4 instances
          
    Desired Capacity = 3
    Coûts réduits
```

![](formations/aws-initiation-approfondissement/11-images/ch3-capture-07-1dd6e3f9.png)

---

## 14. Bonnes pratiques — Architecture hautement disponible

### 14.1 Architecture résiliente S3

**Multi-région** :
- S3 est déjà multi-AZ au sein d'une région.
- Pour une résilience maximale, activer la **réplication cross-région** (CRR).
- Utile pour respecter des exigences de conformité (RGPD, etc.).

**Versioning + Lifecycle** :
- Toujours activer le versioning sur les buckets de production.
- Combiner avec des règles de cycle de vie pour éviter les coûts exponentiels.
- Exemple : garder 30 versions actives, archiver le reste en Glacier.

**Monitoring et alertes** :
- CloudWatch pour surveiller les métriques S3.
- CloudTrail pour auditer les accès et modifications.
- S3 Access Analyzer pour vérifier les politiques d'accès.

### 14.2 Architecture résiliente EC2

**Load Balancer + Auto Scaling minimum** :
- Toujours au minimum 2 instances (haute disponibilité).
- Répartir sur **plusieurs zones de disponibilité (AZ)**.
- Configurer les health checks correctement.

**Sécurité en couches** :
- Security Groups : bloquer les ports inutiles.
- IAM Roles : donner uniquement les permissions nécessaires.
- Subnets privés pour les instances sans accès Internet.

**Sauvegardes** :
- Snapshots EBS réguliers (quotidiens ou hebdomadaires).
- Externaliser les sauvegardes sur S3.
- Tester la restauration régulièrement !

### 14.3 Optimisation des coûts

**S3 Coûts** :
- Utiliser **Intelligent-Tiering** si l'accès est imprévisible.
- Activer les **lifecycle policies** agressives pour archiver.
- Monitorer la bande passante (les téléchargements hors AWS coûtent cher).

> [!WARNING]
> **Attention aux coûts cachés S3** — Le prix du stockage S3 Standard (~0,023 $/Go-mois) est souvent bien inférieur aux coûts de **transfert sortant** (~0,09 $/Go) et aux **frais de requêtes** (PUT, COPY, LIST, GET). Pour un Data Lake avec des millions d'objets, les requêtes LIST peuvent représenter une part significative de la facture. Activez **Cost Explorer** avec les tags S3 pour identifier les sources de dépenses.
**EC2 Coûts** :
- Utiliser **Savings Plans** pour les workloads stables (72% d'économie).
- **Spot** pour les job batch ou CI/CD tolérants aux interruptions.
- AWS Compute Optimizer pour dimensionner correctement.
- Éteindre les ressources de dev/test en fin de journée.

**Estimateur AWS** :
- Utiliser le **Pricing Calculator** pour estimer les coûts futurs.
- Vérifier les coûts inattendus via la **Cost Explorer**.

### 14.4 Performance et scalabilité

**S3 Performance** :
- Utiliser des **préfixes intelligents** pour éviter les goulots d'étranglement (ex. `2024/03/24/log-xxxxx`).
- Activer **S3 Transfer Acceleration** pour les uploads volumineux (MultiPart Upload).
- CloudFront comme CDN pour la distribution worldwide.

**EC2 Performance** :
- **Monitoring continu** : CPU, mémoire, réseau, disque.
- **Auto Scaling** sur **multiple métriques** : CPU, mémoire, débit réseau.
- Usar **Read Replicas** pour les bases de données.
- **Connexion pooling** pour les applications critiques.

---

## 15. Conformité et sécurité pour les données sensibles

Les environnements soumis à des réglementations (RGPD, HIPAA, PCI-DSS) nécessitent des garanties strictes.

### 15.1 Frameworks de conformité AWS

| Framework | Objectif | Services AWS applicables |
|-----------|----------|-------------------------|
| **RGPD** | Protection des données personnelles UE | Encryption, Data Residency, CloudTrail |
| **HIPAA** | Confidentialité des données santé | Dedicated Instance, Encrypted EBS, Audit logs |
| **PCI-DSS** | Sécurité des données cartes bancaires | VPC isolé, Encryption, Firewall |
| **ISO 27001** | Gestion de la sécurité informatique | IAM, KMS, CloudTrail, Monitoring |

### 15.2 Bonnes pratiques de conformité pour S3

- ✅ Chiffrement : SSE-KMS (clés maîtrisées)
- ✅ Versioning : actif (trace des modifications)
- ✅ Bucket Policy : restreint à IP/domaine
- ✅ Logging : S3 Access Logs dans un bucket séparé
- ✅ CloudTrail : audit API dans le compte AWS
- ✅ MFA Delete : protection contre la suppression
- ✅ Block Public : tous les accès publics bloqués
- ✅ Lifecycle : archivage des données obsolètes
- ✅ Réplication CRR : backup multi-région

### 15.3 Bonnes pratiques de conformité pour EC2

- ✅ Dedicated Instance : pas de partage d'hôte physique
- ✅ EBS chiffré : SSE-KMS pour tous les volumes
- ✅ Security Group : minimaliste (moindre privilège)
- ✅ IAM Role : permissions spécifiques au rôle
- ✅ CloudWatch Agent : logs applicatifs
- ✅ VPC privé : pas d'accès Internet direct
- ✅ Snapshots EBS : conservés X années
- ✅ Patch Management : système à jour
- ✅ Monitoring : alertes sur anomalies

### 15.4 Exemple : Architecture RGPD multi-région

```
Région EU (Ireland)
- VPC Privé
   - Subnets privés (applications)
   - Subnet public (NAT Gateway seulement)
   - Security Group très restrictif
- S3 Bucket
   - Versioning activé
   - SSE-KMS (clé EU managée)
   - Bucket Policy : IP/IAM restrictifs
   - CloudTrail logging
- EC2 Instances
   - Dedicated Instance
   - EBS chiffré KMS
   - Snapshots quotidiens → S3
   - CloudWatch + CloudTrail

Région EU (Frankfurt) — Backup
- S3 Réplication CRR du bucket EU-Ireland
   - Préservé 7 ans (conformité)
```

### 15.5 Audit et certification

**AWS Artifact** : plateforme d'AWS pour les certifications de conformité. Les rapports téléchargés (SOC 2, ISO 27001) servent à prouver à vos clients ou auditeurs qu'AWS respecte les normes de sécurité.

```bash
# Dans la console AWS → Security, Identity & Compliance → Artifact
# Télécharger :
# - AWS Compliance Summary
# - SOC 2 Type II reports
# - ISO 27001 certificates
# Utile pour les audits externes
```

**CloudTrail** pour l'audit — ces commandes permettent de lister les trails actifs et de rechercher les actions effectuées sur une ressource précise (ici, un bucket S3) :
```bash
aws cloudtrail describe-trails --region eu-west-1
aws cloudtrail list-events \
    --region eu-west-1 \
    --max-results 50 \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=mon-bucket
```

> [!TIP]
> **Résultat attendu :**
> ```
> # describe-trails retourne :
> {
>     "trailList": [
>         {
>             "Name": "management-events-trail",
>             "S3BucketName": "my-cloudtrail-logs-bucket",
>             "IncludeGlobalServiceEvents": true,
>             "IsMultiRegionTrail": true,
>             "HomeRegion": "eu-west-1",
>             "TrailARN": "arn:aws:cloudtrail:eu-west-1:123456789012:trail/management-events-trail",
>             "LogFileValidationEnabled": true
>         }
>     ]
> }
>
> # list-events retourne des événements du type :
> {
>     "Events": [
>         {
>             "EventId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
>             "EventName": "PutObject",
>             "ReadOnly": "false",
>             "EventTime": "2024-05-17T14:23:05+00:00",
>             "Username": "stagiaire-demo",
>             "Resources": [
>                 {
>                     "ResourceType": "AWS::S3::Object",
>                     "ResourceName": "mon-bucket/documents/rapport.pdf"
>                 }
>             ]
>         }
>     ]
> }
> ```
---

## 16. Points importants et pièges fréquents

| Piège | Réalité | Conséquence |
|-------|---------|------------|
| **S3 a une structure de dossiers** | Non ! C'est du stockage objet, les "dossiers" sont juste des préfixes dans les noms | Impossible de renommer les dossiers, penser en clés, pas en hiérarchies |
| **Versioning S3 ne prend pas de place supplémentaire** | Faux ! Chaque version est stockée complètement | Les coûts explosent vite si vous versionnez des fichiers volumineux |
| **Les instances EC2 garderont leurs données après arrêt** | Seulement si vous utilisez EBS persistant | Les données en instance store (stockage éphémère) sont perdues à l'arrêt |
| **On-Demand est la meilleure option tarifaire** | Non, c'est la plus chère | Reserved / Spot / Savings Plans peuvent économiser 70-90% |
| **Auto Scaling remplace les instances défaillantes instantanément** | Non, il faut le temps de démarrage (2-5 min) | Configurer les health checks correctement et accepter un délai |
| **Toute IP EC2 est durable** | Non, les IPs publiques changent à l'arrêt/redémarrage | Utiliser Elastic IP pour les IPs stables ou les DNS |
| **Un Security Group "ouvert" (0.0.0.0/0) sur tous les ports est OK si la machine n'a rien à cacher** | Non ! C'est une faille de sécurité | Les scanners de ports peuvent découvrir la machine, minimiser l'exposition |
| **EBS et S3 sont interchangeables** | Non ! EBS est un disque (bloc), S3 est du stockage objet | Choisir le bon service selon le cas d'usage |

---

## 17. Ressources

### Documentation officielle AWS
- [AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/)
- [CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
- [AWS Compliance](https://aws.amazon.com/compliance/)
- [Artifact Console](https://console.aws.amazon.com/artifact)
- [S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering-overview.html)
- [S3 Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html)
- [Bucket Policies Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)


---

## Quiz interactif du chapitre

Choisissez une réponse : la correction expliquée apparaît immédiatement. Les propositions changent d’ordre à chaque nouvelle tentative.

<iframe class="quiz-frame" src="https://diablotynne.github.io/aws-initiation-approfondissement/static/quiz-aws/quiz-chapitre-3.html" title="Quiz interactif du chapitre 3" loading="lazy"></iframe>
