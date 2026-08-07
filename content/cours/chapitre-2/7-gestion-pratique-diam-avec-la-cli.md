---
title: "7. Gestion pratique d'IAM avec la CLI"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 7. Gestion pratique d'IAM avec la CLI"
---

<nav class="page-sequence"><a href="cours/chapitre-2/6-tracabilite-et-surveillance-avec-cloudtrail">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/8-points-importants-et-pieges-frequents">Suivant</a></nav>

> [!info]
> Une activité pratique permet d’approfondir la gestion des utilisateurs, groupes et rôles IAM en CLI.

### 7.1 Créer un utilisateur IAM

La commande crée l'utilisateur `alice`, puis une seconde commande vérifie immédiatement qu'il a bien été enregistré dans IAM — confirmer la création après chaque commande est toujours une bonne pratique.

```bash
# Créer un nouvel utilisateur nommé "alice"
aws iam create-user --user-name alice

# Vérifier que l'utilisateur a été créé
aws iam get-user --user-name alice
```

> [!tip]
> **Résultat attendu — `aws iam get-user --user-name alice` :**
> ```json
> {
>     "User": {
>         "Path": "/",
>         "UserName": "alice",
>         "UserId": "AIDA4EXAMPLE7EXAMPLE",
>         "Arn": "arn:aws:iam::123456789012:user/alice",
>         "CreateDate": "2024-01-15T10:30:00+00:00"
>     }
> }
> ```
> L'utilisateur `alice` est créé. Son `UserId` commence toujours par `AIDA` pour les utilisateurs IAM. La commande `create-user` ne retourne aucun output si elle réussit.

---

### 7.2 Créer un groupe et ajouter l'utilisateur

Les groupes sont le cœur de la gestion IAM : plutôt qu'attribuer des permissions utilisateur par utilisateur, on les attache au groupe et tous les membres en héritent automatiquement.

```bash
# Créer un groupe "Developers"
aws iam create-group --group-name Developers

# Ajouter alice au groupe
aws iam add-user-to-group --group-name Developers --user-name alice

# Vérifier
aws iam get-group --group-name Developers
```

> [!tip]
> **Résultat attendu — `aws iam get-group --group-name Developers` :**
> ```json
> {
>     "Group": {
>         "Path": "/",
>         "GroupName": "Developers",
>         "GroupId": "AGPA4EXAMPLEGROUP",
>         "Arn": "arn:aws:iam::123456789012:group/Developers",
>         "CreateDate": "2024-01-15T10:31:00+00:00"
>     },
>     "Users": [
>         {
>             "UserName": "alice",
>             "UserId": "AIDA4EXAMPLE7EXAMPLE",
>             "Arn": "arn:aws:iam::123456789012:user/alice"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> Le groupe `Developers` est créé et `alice` apparaît bien dans le tableau `Users`.

---

### 7.3 Attacher une policy au groupe

On attache une policy AWS gérée au groupe `Developers`. Tous les membres actuels et futurs du groupe hériteront automatiquement de ces permissions, sans aucune action supplémentaire.

```bash
# Attacher la policy AWS gérée "AmazonS3ReadOnlyAccess" au groupe
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Vérifier
aws iam list-attached-group-policies --group-name Developers
```

> [!tip]
> **Résultat attendu — `aws iam list-attached-group-policies --group-name Developers` :**
> ```json
> {
>     "AttachedPolicies": [
>         {
>             "PolicyName": "AmazonS3ReadOnlyAccess",
>             "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La policy `AmazonS3ReadOnlyAccess` est bien attachée au groupe `Developers`. Tous les membres actuels et futurs du groupe en héritent automatiquement.

---

### 7.4 Créer des clés d'accès pour un utilisateur

Les clés d'accès (`AccessKeyId` + `SecretAccessKey`) permettent d'appeler l'API AWS depuis la CLI ou un script. Elles sont générées **une seule fois** et le `SecretAccessKey` ne peut jamais être récupéré ensuite — AWS ne le stocke pas.

```bash
# Générer une paire de clés d'accès pour alice
aws iam create-access-key --user-name alice

# Le résultat contient AccessKeyId et SecretAccessKey
# Attention : SAUVEGARDEZ CES CLÉS EN LIEU SÛR
```

> [!tip]
> **Résultat attendu — `aws iam create-access-key --user-name alice` :**
> ```json
> {
>     "AccessKey": {
>         "UserName": "alice",
>         "AccessKeyId": "AKIA4EXAMPLEKEYID12",
>         "Status": "Active",
>         "SecretAccessKey": "<SECRET_TEMPORAIRE_MASQUE>",
>         "CreateDate": "2024-01-15T10:35:00+00:00"
>     }
> }
> ```
> Un `AccessKeyId` permanent utilise généralement le préfixe `AKIA`, tandis qu'un identifiant temporaire STS utilise `ASIA`. Le `SecretAccessKey` d'une nouvelle clé permanente ne s'affiche qu'une seule fois : ne le copiez jamais dans un support, un ticket ou un dépôt. Stockez-le dans un gestionnaire de secrets, puis préférez les rôles et identifiants temporaires dès que le cas d'usage le permet.

> [!danger]
> **Ne jamais stocker ces clés en clair dans le code source, les fichiers `.env` versionnés ou les dépôts Git.** Utilisez AWS Secrets Manager ou un coffre-fort (Vault, 1Password) pour les conserver. En cas de fuite, révoquez immédiatement la clé dans IAM et générez-en une nouvelle.

---

### 7.5 Créer une policy JSON personnalisée

Quand aucune policy AWS gérée ne correspond exactement à vos besoins, vous créez une policy inline. Ici, on génère d'abord le fichier JSON localement, puis on l'attache à l'utilisateur.

```bash
# Créer une policy pour accès limité à S3
cat > s3-readonly-policy.json << 'EOF'
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
EOF

# Attacher cette policy à alice
aws iam put-user-policy \
  --user-name alice \
  --policy-name S3-Readonly \
  --policy-document file://s3-readonly-policy.json
```

> [!tip]
> **Résultat attendu — `aws iam list-user-policies --user-name alice` (commande de vérification) :**
> ```json
> {
>     "PolicyNames": [
>         "S3-Readonly"
>     ],
>     "IsTruncated": false
> }
> ```
> La policy inline `S3-Readonly` est bien attachée directement à l'utilisateur `alice`. Une policy inline est stockée directement sur l'entité (user/group/role) et ne peut pas être réutilisée ailleurs.

---

### 7.6 Vérifier les permissions d'un utilisateur

Avant toute intervention sur un compte, il est utile de dresser l'inventaire complet des permissions effectives d'un utilisateur : policies directes ET celles héritées via les groupes.

```bash
# Lister les policies attachées à alice
aws iam list-user-policies --user-name alice

# Lister les policies de groupe pour alice
aws iam list-groups-for-user --user-name alice
```

> [!tip]
> **Résultat attendu — `aws iam list-groups-for-user --user-name alice` :**
> ```json
> {
>     "Groups": [
>         {
>             "Path": "/",
>             "GroupName": "Developers",
>             "GroupId": "AGPA4EXAMPLEGROUP",
>             "Arn": "arn:aws:iam::123456789012:group/Developers",
>             "CreateDate": "2024-01-15T10:31:00+00:00"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> `alice` appartient au groupe `Developers`. Ses permissions effectives = ses policies directes + les policies de tous ses groupes d'appartenance.

---

### 7.7 Créer un rôle IAM

Un rôle n'a pas de credentials permanents : il est **assumé temporairement** par un service ou un utilisateur. Le fichier `trust-policy.json` (appelé "trust policy" ou "politique de confiance") définit **qui** a le droit d'assumer ce rôle — ici, le service EC2.

```bash
# Créer un rôle pour EC2
aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document file://trust-policy.json

# Contenu de trust-policy.json :
# {
#   "Version": "2012-10-17",
#   "Statement": [{
#     "Effect": "Allow",
#     "Principal": {"Service": "ec2.amazonaws.com"},
#     "Action": "sts:AssumeRole"
#   }]
# }
```

> [!tip]
> **Résultat attendu — `aws iam create-role --role-name EC2-S3-Access ...` :**
> ```json
> {
>     "Role": {
>         "Path": "/",
>         "RoleName": "EC2-S3-Access",
>         "RoleId": "AROA4EXAMPLEROLEID1",
>         "Arn": "arn:aws:iam::123456789012:role/EC2-S3-Access",
>         "CreateDate": "2024-01-15T10:40:00+00:00",
>         "AssumeRolePolicyDocument": {
>             "Version": "2012-10-17",
>             "Statement": [{
>                 "Effect": "Allow",
>                 "Principal": {"Service": "ec2.amazonaws.com"},
>                 "Action": "sts:AssumeRole"
>             }]
>         },
>         "MaxSessionDuration": 3600
>     }
> }
> ```
> Le rôle `EC2-S3-Access` est créé. Son `RoleId` commence par `AROA`. La trust policy indique que seul le service EC2 (`ec2.amazonaws.com`) peut assumer ce rôle.

---

### 7.8 Attacher une policy à un rôle

Une fois le rôle créé avec sa trust policy (qui définit qui peut l'assumer), on lui attache des permissions (ce qu'il peut faire). Toute instance EC2 qui assumera ce rôle pourra lire S3 **sans credentials statiques**.

```bash
# Attacher AmazonS3ReadOnlyAccess au rôle
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

> [!tip]
> **Résultat attendu — `aws iam list-attached-role-policies --role-name EC2-S3-Access` :**
> ```json
> {
>     "AttachedPolicies": [
>         {
>             "PolicyName": "AmazonS3ReadOnlyAccess",
>             "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `attach-role-policy` ne retourne aucun output si elle réussit. La vérification confirme que `AmazonS3ReadOnlyAccess` est bien attachée au rôle `EC2-S3-Access`.

---

### 7.9 Activer MFA pour un utilisateur

L'activation du MFA lie un appareil virtuel (application TOTP type Google Authenticator ou Authy) à l'utilisateur. Les deux codes consécutifs (`authentication-code1` et `authentication-code2`) sont nécessaires pour synchroniser l'horloge de l'appareil avec AWS.

```bash
# Créer un appareil MFA virtuel
aws iam enable-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice-mfa \
  --authentication-code1 123456 \
  --authentication-code2 654321
```

> [!tip]
> **Résultat attendu — `aws iam list-mfa-devices --user-name alice` :**
> ```json
> {
>     "MFADevices": [
>         {
>             "UserName": "alice",
>             "SerialNumber": "arn:aws:iam::123456789012:mfa/alice-mfa",
>             "EnableDate": "2024-01-15T10:45:00+00:00"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `enable-mfa-device` ne retourne aucun output si elle réussit. La vérification confirme que le périphérique MFA virtuel est bien enregistré pour `alice`.

---

### 7.10 Politique conditionnelle : exiger MFA

Cette démo crée une policy qui bloque **absolument toutes les actions AWS** si l'utilisateur n'a pas activé MFA pour sa session. C'est une protection forte recommandée pour les comptes administrateurs — sans MFA active, même `aws s3 ls` sera refusé.

```bash
# Créer une policy refusant tout si MFA n'est pas présente
cat > mfa-required.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {
        "aws:MultiFactorAuthPresent": "false"
      }
    }
  }]
}
EOF

# Attacher au groupe Admins
aws iam put-group-policy \
  --group-name Admins \
  --policy-name MFA-Required \
  --policy-document file://mfa-required.json
```

> [!tip]
> **Résultat attendu — `aws iam list-group-policies --group-name Admins` :**
> ```json
> {
>     "PolicyNames": [
>         "MFA-Required"
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `put-group-policy` ne retourne aucun output si elle réussit. Pour tester : connectez-vous à la console AWS sans MFA — toutes les actions retourneront `AccessDenied`. Activez MFA, reconnectez-vous, et les accès sont rétablis.

---

<nav class="page-sequence"><a href="cours/chapitre-2/6-tracabilite-et-surveillance-avec-cloudtrail">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/8-points-importants-et-pieges-frequents">Suivant</a></nav>
