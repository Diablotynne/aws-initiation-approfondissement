---
title: "3. Protéger et optimiser les données S3"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 3. Protéger et optimiser les données S3"
---

<nav class="page-sequence"><a href="cours/chapitre-4/2-amazon-s3-le-stockage-objet-scalable">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/4-gestion-de-s3-en-cli">Suivant</a></nav>

Amazon S3 propose plusieurs mécanismes pour **sécuriser vos fichiers**, **préserver leur historique**, et **réduire les coûts de stockage**. Ces options sont souvent méconnues, mais elles sont essentielles pour bien gérer vos données dans le cloud.

### 3.1 Chiffrement : protéger les fichiers contre les accès non autorisés

Quand vous stockez un fichier dans S3, vous pouvez demander à AWS de le **chiffrer automatiquement**. Cela signifie que même si quelqu'un accède physiquement au disque, il ne pourra pas lire le contenu sans la clé.

#### Types de chiffrement et gestion des clés

S3 propose trois mécanismes complémentaires, deux pour les données au repos et un pour les données en transit :

| Acronyme | Signification complète | Description pédagogique |
|---|---|---|
| **SSE-S3** | _Server-Side Encryption with Amazon S3-managed keys_ | Le chiffrement est géré **automatiquement par AWS S3**. Vous n'avez rien à configurer. |
| **SSE-KMS** | _Server-Side Encryption with AWS Key Management Service_ | Le chiffrement utilise **AWS KMS**, un service de gestion de clés. Vous définissez et contrôlez les clés. |
| **HTTPS/TLS** | _HyperText Transfer Protocol Secure / Transport Layer Security_ | Ce protocole **sécurise les échanges** entre votre navigateur ou application et AWS. |

SSE-S3 et SSE-KMS protègent la donnée stockée sur disque, tandis que HTTPS/TLS protège la donnée pendant son trajet réseau — les deux catégories sont à activer simultanément, l'une ne remplaçant pas l'autre.

### 3.2 Détails des types de chiffrement

#### SSE (Server-Side Encryption)

> Chiffrement effectué **côté serveur**, c'est-à-dire par AWS une fois que les données sont reçues.

**SSE-S3** : AWS chiffre les objets S3 avec une clé gérée par le service S3 lui-même.
- **Avantage** : aucune configuration requise.
- **Niveau de sécurité** : standard, suffisant pour de nombreux cas d'usage.

**SSE-KMS** : AWS chiffre les objets S3 avec une clé gérée par **AWS KMS**, que vous pouvez créer, activer/désactiver, auditer.
- **Avantage** : contrôle granulaire sur les clés.
- **Complexité** : nécessite configuration, permissions IAM, et gestion des quotas KMS.

C'est précisément ce service KMS, mentionné dans SSE-KMS, qui mérite d'être détaillé séparément puisqu'il ne se limite pas à S3.

#### KMS (Key Management Service)

> Service AWS permettant de **créer, stocker et gérer** des clés de chiffrement.

- Utilisé dans **SSE-KMS**, mais aussi pour chiffrer des volumes EBS, des secrets, etc.
- Permet la **rotation automatique**, l'audit via CloudTrail, et l'intégration avec IAM.

Le chiffrement au repos protège la donnée stockée, mais elle circule aussi sur le réseau avant d'arriver sur S3 — c'est là qu'intervient le troisième mécanisme.

#### HTTPS / TLS

> Protocole de **sécurisation des communications réseau**.

- **HTTPS** est HTTP + TLS.
- **TLS (Transport Layer Security)** : protocole de chiffrement qui protège les données en transit.
- Activé **par défaut** dans la console AWS et les SDK/API.

Ces trois mécanismes sont donc à considérer ensemble plutôt qu'isolément, chacun couvrant une phase différente du cycle de vie de la donnée.

### 3.3 À retenir sur le chiffrement

En résumé, le choix entre les deux mécanismes de chiffrement au repos dépend surtout de la sensibilité des données, tandis que le chiffrement en transit ne se discute pas :

- **SSE-S3** : simple, automatique, suffisant pour les données non sensibles.
- **SSE-KMS** : recommandé pour les données sensibles ou les environnements réglementés.
- **HTTPS/TLS** : toujours activé pour sécuriser les échanges réseau.

> [!info]
> **SSE-KMS et coûts KMS** — Chaque requête de chiffrement/déchiffrement via KMS est facturée (environ 0,03 $ pour 10 000 requêtes). Pour un bucket avec de nombreuses petites opérations, cela peut s'accumuler. Activez le **Bucket Key** (option KMS) pour réduire le nombre d'appels KMS jusqu'à 99% en utilisant une clé de données par bucket plutôt que par objet.

### 3.4 Versioning : garder l'historique des fichiers

Le **versioning** permet de conserver toutes les versions d'un fichier, même si vous le modifiez ou le supprimez par erreur.

#### Exemple :

Concrètement, le versioning fonctionne ainsi :

- Vous téléversez `rapport.pdf`
- Vous le modifiez et téléversez une nouvelle version
- Vous pouvez toujours revenir à la version précédente

Chaque nouvelle version occupe un espace de stockage supplémentaire — S3 ne remplace jamais un fichier, il en ajoute une nouvelle version, ce qui a un impact direct sur la facturation à surveiller. Au-delà de ce point d'attention, le versioning répond à plusieurs besoins concrets :
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

Ces optimisations de classe de stockage concernent la donnée déjà présente dans S3 ; un autre levier d'optimisation, indépendant, concerne la vitesse à laquelle cette donnée y arrive.

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

> [!tip]
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

Transfer Acceleration ne concerne que le sens montant (upload) ; le scénario bien plus fréquent en production consiste à distribuer efficacement du contenu déjà présent dans S3 vers un grand nombre de visiteurs.

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

> [!tip]
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

> [!warning]
> **Coûts Transfer Acceleration** : Cette fonctionnalité engendre des frais supplémentaires (~0,04 $/Go pour les transferts vers des Edge Locations). N'activez Transfer Acceleration que si vous uploadez régulièrement des fichiers volumineux (> 100 Mo) depuis des clients géographiquement éloignés de la région S3. Pour les petits fichiers ou les accès locaux, le gain est négligeable et le surcoût inutile.

**Cas d'usage** :
- Uploads de fichiers vidéo ou binaires depuis un client distant.
- Synchronisations multi-sites hautes performances.
- Distributions de fichiers volumineux vers plusieurs régions AWS.

Chiffrement, classes de stockage et distribution du contenu reposent tous sur un même socle : la capacité à définir précisément qui peut faire quoi sur un bucket, ce que permettent les bucket policies.

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

Les trois cas d'usage suivants montrent comment combiner ces éléments pour répondre à des besoins concrets, du plus permissif au plus restrictif.

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

> [!danger]
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

> [!tip]
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

Pour conclure cette section, voici une synthèse des réflexes à adopter systématiquement lors de la création d'un bucket en production, en reprenant chaque mécanisme vu plus haut :

- Activez le **versioning** dès que vous stockez des fichiers importants.
- Utilisez **SSE-S3** pour un chiffrement simple et automatique.
- Créez des **règles de cycle de vie** pour archiver ou supprimer les fichiers inutilisés.
- Choisissez la **classe de stockage** adaptée à chaque type de données.
- Appliquez des **bucket policies** pour restreindre l'accès selon le principe du moindre privilège.
- Utilisez **Intelligent-Tiering** si l'accès est imprévisible.
- Activez **Transfer Acceleration** pour les uploads volumineux critiques.

Passons maintenant à la pratique : la section suivante applique concrètement ces bonnes pratiques via l'AWS CLI.

---

<nav class="page-sequence"><a href="cours/chapitre-4/2-amazon-s3-le-stockage-objet-scalable">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/4-gestion-de-s3-en-cli">Suivant</a></nav>
