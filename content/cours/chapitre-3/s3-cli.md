---
title: "4. Gestion de S3 en CLI"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 4. Gestion de S3 en CLI"
---

<nav class="page-sequence"><a href="cours/chapitre-3/protection-s3">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/ec2">Suivant</a></nav>

> [!info]
> Une activité pratique permet d’approfondir la manipulation de S3 en CLI.

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

> [!tip]
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

> [!tip]
> **Résultat attendu :**
> ```
> upload: ./test.txt to s3://mon-bucket-formation/documents/test.txt
>
> # s3 ls retourne :
> 2024-05-17 14:23:05         34 test.txt
> ```

> [!warning]
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

> [!tip]
> **Résultat attendu :**
> ```
> # put-bucket-versioning : aucun output si succès
>
> # get-bucket-versioning retourne :
> {
>     "Status": "Enabled"
> }
> ```

> [!warning]
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

> [!tip]
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

> [!tip]
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

> [!tip]
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

> [!danger]
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

> [!tip]
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

> [!warning]
> **Coûts de transfert sortant (egress)** — Le téléchargement de données depuis S3 vers Internet est facturé (~0,09 $/Go pour les premiers 10 To). Les transferts entre services AWS dans la même région sont gratuits. Si vous synchronisez régulièrement des données volumineuses vers des machines hors AWS (postes développeurs, serveurs on-premise), anticipez ce coût dans votre budget.

---

---

<nav class="page-sequence"><a href="cours/chapitre-3/protection-s3">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/ec2">Suivant</a></nav>
