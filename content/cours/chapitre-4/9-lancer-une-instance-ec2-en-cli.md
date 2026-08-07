---
title: "9. Lancer une instance EC2 en CLI"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 9. Lancer une instance EC2 en CLI"
---

<nav class="page-sequence"><a href="cours/chapitre-4/8-options-de-tarification-aws-ec2">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/10-elastic-load-balancing-elb-repartition-du-trafic">Suivant</a></nav>

> [!info]
> Une activité pratique permet d’approfondir le lancement d’instances EC2 en CLI.

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

> [!tip]
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

> [!warning]
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

> [!tip]
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

> [!tip]
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

> [!warning]
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

> [!tip]
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

> [!tip]
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

> [!warning]
> **Coûts des snapshots EBS** — Les snapshots sont stockés dans S3 (managé par AWS) et facturés ~0,05 $/Go-mois. Un snapshot de 100 Go coûte ~5 $/mois. Les snapshots sont **incrémentiels** (seules les modifications depuis le dernier snapshot sont stockées), mais les premiers snapshots peuvent être volumineux. Planifiez une politique de rétention et supprimez les snapshots obsolètes pour maîtriser les coûts.

---

<nav class="page-sequence"><a href="cours/chapitre-4/8-options-de-tarification-aws-ec2">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/10-elastic-load-balancing-elb-repartition-du-trafic">Suivant</a></nav>
