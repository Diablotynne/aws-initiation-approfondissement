---
title: "3. Protéger et optimiser les données S3"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 3. Protéger et optimiser les données S3"
---

<nav class="page-sequence"><a href="cours/chapitre-3/s3">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/s3-cli">Suivant</a></nav>

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

> [!info]
> **SSE-KMS et coûts KMS** — L'utilisation de clés KMS peut générer des appels KMS facturables en plus des opérations S3. Pour un bucket très sollicité, évaluez **S3 Bucket Keys**, qui réduisent le trafic de requêtes de S3 vers KMS. La réduction réelle et les conditions d'éligibilité doivent être vérifiées dans la documentation et la tarification courantes.


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

<a class="schema-zoom" href="assets/schemas/cycle-vie-s3.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/cycle-vie-s3.svg"
     alt="Cycle de vie d'un objet S3 depuis sa création jusqu'aux transitions, à l'archivage et à l'expiration"
     style="display:block; margin:auto; width:95%"></a>

**Lecture du schéma.** Une règle sélectionne des objets par préfixe ou par tags, puis applique les actions configurées. Les transitions disponibles, les durées minimales de stockage et les délais de restauration dépendent de la classe choisie. Une expiration est une suppression : elle doit être alignée sur la politique de conservation de l'organisation.

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
- Des frais de surveillance et d'automatisation s'ajoutent pour chaque objet éligible ; leur impact dépend donc fortement du nombre et de la taille des objets.
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
```bash
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
> ```text
> # put-bucket-accelerate-configuration : aucun output si succès
>
> # s3 cp retourne la progression :
> upload: ./mon-fichier-gros.zip to s3://mon-bucket/uploads/mon-fichier-gros.zip
> ```
> Transfer Acceleration est activé sur le bucket. Les uploads utilisent désormais les Edge Locations CloudFront pour rejoindre le bucket S3, ce qui réduit la latence depuis les clients distants.


**Coûts** :
- Des frais d'accélération s'ajoutent au transfert standard et varient selon le trajet des données.
- Le gain doit être mesuré avec l'outil de comparaison de vitesse AWS avant activation.

#### S3 comme origine CloudFront — distribuer du contenu statique à grande échelle

Transfer Acceleration optimise l'**upload** vers S3. Le cas d'usage inverse — beaucoup plus fréquent en production — est de distribuer efficacement du contenu **depuis** S3 vers des millions de visiteurs : c'est le rôle de **CloudFront** utilisé comme CDN devant un bucket S3.

```text
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
> **Coûts Transfer Acceleration** : cette fonctionnalité ajoute un coût au volume transféré. Ne l'activez qu'après avoir mesuré un gain utile depuis les emplacements réels des clients. Un test de performance et une estimation sur la page tarifaire S3 sont plus fiables qu'un seuil de taille générique.


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
> ```json
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

<nav class="page-sequence"><a href="cours/chapitre-3/s3">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/s3-cli">Suivant</a></nav>
