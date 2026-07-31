# Chapitre 2 - Quiz : Formation AWS — Initiation + Approfondissement
## Sécurité et gestion des accès avec AWS IAM

> **Liens** : [[README]] | [[Chapitre 2 - Formation AWS]] | [[Chapitre 2 - Travaux Pratiques]]

---

> **Consigne** : Une seule réponse correcte par question, sauf mention contraire. Entourez ou notez la lettre de votre réponse. Les questions sont indépendantes.

---

## Questions

---

### Q1 — Évaluation des permissions IAM

Un utilisateur IAM possède les éléments suivants :
- Une policy de groupe qui **autorise** `ec2:TerminateInstances`
- Une policy inline qui **refuse** `ec2:TerminateInstances`
- Une SCP Organizations qui **autorise** `ec2:TerminateInstances`

Que se passe-t-il lorsque cet utilisateur tente de terminer une instance EC2 ?

**A.** L'action est autorisée car la policy de groupe a la priorité sur les policies inline.

**B.** L'action est refusée car le Deny explicite (policy inline) l'emporte sur tous les Allow.

**C.** L'action est autorisée car la SCP Organizations approuve l'action.

**D.** L'action est autorisée car deux sources autorisent l'action contre une seule qui la refuse.

---

### Q2 — Méthode MFA déconseillée

Parmi les types de MFA supportés par AWS IAM, lequel AWS déconseille-t-il explicitement dans sa documentation de bonnes pratiques ?

**A.** Virtual MFA (Google Authenticator, Authy)

**B.** Hardware TOTP (token physique Gemalto)

**C.** FIDO2 Security Key (YubiKey, Titan Key)

**D.** SMS (message texte sur téléphone mobile)

---

### Q3 — Format d'un ARN IAM

Parmi les ARN suivants, lequel représente correctement un utilisateur IAM nommé `stagiaire-demo` dans le compte AWS `123456789012` ?

**A.** `arn:aws:iam:eu-west-3:123456789012:user/stagiaire-demo`

**B.** `arn:aws:iam::123456789012:user/stagiaire-demo`

**C.** `arn:aws:iam::123456789012:users/stagiaire-demo`

**D.** `arn:aws:ec2:eu-west-3:123456789012:iam/user/stagiaire-demo`

---

### Q4 — IAM Role vs IAM User

Laquelle des affirmations suivantes décrit le mieux la différence entre un IAM Role et un IAM User ?

**A.** Un IAM Role est utilisé uniquement par des services AWS internes, jamais par des humains.

**B.** Un IAM Role possède des credentials permanents (Access Key ID + Secret), tandis qu'un IAM User utilise des credentials temporaires.

**C.** Un IAM Role génère des credentials temporaires via STS et peut être assumé par des services, des comptes tiers ou des utilisateurs fédérés, sans credentials permanents attachés.

**D.** Un IAM Role est équivalent à un IAM User mais avec des droits limités dans le temps par une date d'expiration obligatoire.

---

### Q5 — Service d'audit des API calls

FormaTech souhaite savoir **qui** a supprimé une instance RDS hier à 14h37, depuis **quelle adresse IP** et avec **quel outil** (Console, CLI, SDK). Quel service AWS permet de retrouver cette information ?

**A.** Amazon CloudWatch Logs

**B.** AWS Config

**C.** AWS CloudTrail

**D.** Amazon GuardDuty

---

### Q6 — Secrets Manager vs Parameter Store

L'équipe d'exploitation hésite entre AWS Secrets Manager et AWS Systems Manager Parameter Store pour stocker le mot de passe de la base de données RDS de l'entreprise fil rouge. Quelle affirmation est exacte ?

**A.** Parameter Store supporte la rotation automatique des secrets RDS nativement, sans configuration supplémentaire.

**B.** Secrets Manager est gratuit pour les secrets standard, Parameter Store est payant pour tous les paramètres.

**C.** Secrets Manager intègre une rotation automatique via Lambda (notamment pour RDS), tandis que Parameter Store ne supporte pas la rotation automatique.

**D.** Les deux services sont strictement équivalents ; le choix est uniquement une question de préférence personnelle.

---

### Q7 — Service Control Policies (SCP)

FormaTech configure une SCP qui refuse `ec2:RunInstances` sur toutes les instances de type `*.4xlarge` et plus grand, pour l'OU `Non-Production`. Un développeur du compte `formateach-dev` (membre de l'OU Non-Production) possède `AdministratorAccess` en IAM. Que se passe-t-il s'il tente de lancer une instance `m5.4xlarge` ?

**A.** L'action est autorisée car `AdministratorAccess` surpasse toujours les SCP.

**B.** L'action est refusée car la SCP définit les limites maximales du compte, et le Deny SCP l'emporte sur l'AdministratorAccess IAM.

**C.** L'action est autorisée car les SCP ne s'appliquent qu'aux utilisateurs sans `AdministratorAccess`.

**D.** L'action est refusée seulement si le Management Account a également appliqué la SCP à la racine.

---

### Q8 — Principe du moindre privilège

Une développeuse a besoin de lire uniquement les objets du bucket S3 `formation-videos`. Laquelle de ces policies respecte le mieux le principe du moindre privilège ?

**A.**
```json
{ "Effect": "Allow", "Action": "s3:*", "Resource": "*" }
```

**B.**
```json
{ "Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::formateach-videos/*" }
```

**C.**
```json
{ "Effect": "Allow", "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"], "Resource": "arn:aws:s3:::formateach-videos/*" }
```

**D.**
```json
{ "Effect": "Allow", "Action": "s3:GetObject", "Resource": "*" }
```

---

### Q9 — Coexistence Deny explicite et Allow

Un utilisateur IAM a simultanément :
- Une policy A : `Allow` sur `s3:DeleteObject` pour le bucket `formateach-backup`
- Une policy B : `Deny` sur `s3:DeleteObject` pour le bucket `formateach-backup`

Quel est le résultat de cette configuration ?

**A.** La policy la plus récente l'emporte : si B a été créée après A, c'est Deny.

**B.** Le Deny explicite (policy B) l'emporte toujours, quelle que soit la politique Allow. L'action est refusée.

**C.** Les deux policies se neutralisent et le comportement par défaut s'applique (Allow implicite).

**D.** AWS refuse de créer cette configuration et retourne une erreur de validation.

---

### Q10 — Rotation des Access Keys

Quelle est la fréquence de rotation des Access Keys IAM recommandée par AWS pour respecter les bonnes pratiques de sécurité ?

**A.** Tous les 365 jours (une fois par an)

**B.** Tous les 180 jours (tous les 6 mois)

**C.** Tous les 90 jours (tous les 3 mois) — certaines organisations imposent 30 jours

**D.** Les Access Keys n'ont pas besoin d'être rotées si elles sont utilisées régulièrement.

---

## Corrigé Complet

**Q1 — Réponse : B**

Un **Deny explicite l'emporte toujours** sur tout Allow, quelle que soit la source (policy de groupe, policy inline, SCP, resource-based policy). C'est la règle fondamentale d'évaluation IAM.

L'ordre d'évaluation AWS est :
1. Deny explicite → **REFUS IMMÉDIAT**
2. SCP Organizations
3. Resource-based Policy
4. Identity-based Policy
5. Permissions Boundary
6. Session Policy
7. Aucune règle → Deny implicite

La réponse A est fausse : il n'existe pas de hiérarchie entre types de policies pour les Allow/Deny. La réponse C est fausse : la SCP autorise mais ne prime pas sur un Deny explicite IAM. La réponse D est fausse : le vote à la majorité n'existe pas dans IAM.

---

**Q2 — Réponse : D**

AWS déconseille explicitement le **MFA par SMS**. Deux raisons principales :
1. **Vulnérabilité SS7** : le protocole de signalisation des réseaux téléphoniques peut être exploité pour intercepter les SMS.
2. **SIM swapping** : un attaquant peut convaincre un opérateur de transférer votre numéro vers une nouvelle SIM.

AWS recommande Virtual MFA (A) ou FIDO2 Security Key (C) pour la grande majorité des cas, et Hardware TOTP (B) pour les comptes très sensibles (root, administrateurs). Les trois sont des méthodes recommandées.

---

**Q3 — Réponse : B**

`arn:aws:iam::123456789012:user/stagiaire-demo`

Points clés :
- IAM est un service **global** → la région est **vide** (double `::` entre `aws` et l'account ID)
- Le type de ressource est `user` (singulier), pas `users` (réponse C incorrecte)
- Le service est `iam`, pas `ec2` (réponse D incorrecte)
- La réponse A est incorrecte car elle inclut `eu-west-3` — IAM n'a pas de région dans les ARNs

Structure correcte : `arn:partition:service:region:account-id:resource-type/resource-id`

---

**Q4 — Réponse : C**

Un **IAM Role** génère des credentials **temporaires** via AWS STS (Security Token Service), valides de 15 minutes à 12 heures selon la configuration. Il n'a pas de credentials permanents attachés.

La réponse A est fausse : les rôles peuvent être assumés par des humains (ex: via IAM Identity Center ou switch role console). La réponse B inverse la réalité : c'est le Role qui a des credentials temporaires, pas le User. La réponse D est fausse : un role n'a pas de date d'expiration propre, c'est la session STS qui a une durée limitée.

---

**Q5 — Réponse : C**

**AWS CloudTrail** est le service d'audit des API calls. Il enregistre chaque appel API avec : qui (ARN de l'utilisateur), quoi (nom de l'action), quand (timestamp UTC), depuis où (adresse IP source), avec quel outil (User-Agent).

- CloudWatch Logs (A) est pour les logs applicatifs et système, pas pour l'audit des appels API AWS.
- AWS Config (B) enregistre l'état des ressources et leurs changements de configuration, pas les appels API.
- GuardDuty (D) détecte les menaces intelligemment mais se base sur les logs CloudTrail — ce n'est pas la source primaire d'audit.

---

**Q6 — Réponse : C**

**Secrets Manager** intègre nativement la rotation automatique via des fonctions Lambda gérées par AWS pour RDS (MySQL, PostgreSQL, Oracle, SQL Server), Redshift et d'autres services. Parameter Store ne propose pas de rotation automatique native.

La réponse A est fausse : Parameter Store ne fait pas de rotation automatique. La réponse B est fausse : Secrets Manager est payant ($0.40/secret/mois), Parameter Store Standard est gratuit. La réponse D est fausse : les deux services ont des cas d'usage distincts — Secrets Manager pour les secrets critiques avec rotation, Parameter Store pour la configuration applicative.

---

**Q7 — Réponse : B**

La **SCP définit les limites maximales** (permission boundary au niveau du compte). Aucune policy IAM, même `AdministratorAccess`, ne peut dépasser ce que la SCP autorise. Le Deny SCP est irrévocable depuis l'intérieur du compte.

La réponse A est fausse : `AdministratorAccess` ne surpasse pas les SCP. La réponse C est fausse : les SCP s'appliquent à tous les principals du compte sans exception. La réponse D est fausse : une SCP sur une OU s'applique à tous les comptes membres sans condition supplémentaire.

> **Note formateur** : insister sur ce point — c'est une incompréhension très fréquente qui mène à des configurations dangereuses.

---

**Q8 — Réponse : B**

```json
{ "Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::formateach-videos/*" }
```

Cette policy respecte le **principe du moindre privilège** : action minimale (`s3:GetObject` seulement), sur la ressource précise (objets du bucket `formateach-videos` uniquement).

- Réponse A : `s3:*` sur `*` = AdministratorAccess S3 complet — jamais acceptable
- Réponse C : `PutObject` et `DeleteObject` ne sont pas nécessaires pour une lecture seule
- Réponse D : `s3:GetObject` sur `*` = peut lire n'importe quel bucket du compte — trop large

---

**Q9 — Réponse : B**

Le **Deny explicite l'emporte toujours**, sans exception. C'est la règle cardinale d'IAM. Peu importe combien de policies Allow existent, un seul Deny explicite suffit à bloquer l'action.

La réponse A est fausse : il n'y a pas de notion de "politique la plus récente" dans l'évaluation IAM. La réponse C est fausse : il n'existe pas d'"Allow implicite" dans AWS — l'absence de règle = Deny implicite. La réponse D est fausse : AWS accepte parfaitement cette configuration ; c'est un pattern courant pour les exceptions.

---

**Q10 — Réponse : C**

AWS recommande une rotation des Access Keys tous les **90 jours**. Certaines organisations (secteur financier, défense) imposent 30 jours. La fréquence peut être appliquée via une policy IAM avec condition sur `aws:username` et une règle AWS Config `access-keys-rotated`.

La réponse A (365 jours) est trop longue — une clé compromise pendant un an représente une fenêtre d'exposition inacceptable. La réponse B (180 jours) est trop longue. La réponse D est incorrecte : une utilisation régulière ne compense pas la nécessité de rotation — une clé ancienne peut avoir été compromise silencieusement.

> **Bonne pratique complémentaire** : préférer les IAM Roles aux Access Keys chaque fois que c'est possible. Les Access Keys ne devraient exister que pour les cas sans alternative (scripts locaux, pipelines CI/CD hors AWS).

---

## Question Ouverte — FormaTech (bonus, 5 minutes)

> Cette question n'entre pas dans le barème des 10/10. Elle sert de discussion en groupe.

**Énoncé** :

La responsable RH souhaite accéder à un tableau de bord de statistiques sur l'utilisation de la plateforme e-learning (nombre de connexions par stagiaire, temps passé par module). Ces statistiques sont stockées dans une base de données RDS dans le compte AWS `formation-prod`.

Proposez une architecture de sécurité IAM permettant à la responsable RH d'accéder à ces statistiques **sans lui donner de compte IAM AWS**, tout en respectant le principe du moindre privilège.

**Pistes de réflexion** :
- Peut-on utiliser Amazon QuickSight ou une API Gateway ?
- Quel service AWS permet une authentification par identité externe (Active Directory, Google) ?
- Comment s'assurer que la responsable RH ne voit que les données agrégées correspondant à son périmètre ?

**Éléments de réponse attendus** :
- IAM Identity Center (AWS SSO) avec fédération Active Directory ou Google Workspace
- Amazon QuickSight avec Row-Level Security (RLS) pour filtrer les données par profil
- Amazon API Gateway + Lambda + IAM Authorizer comme couche d'abstraction devant RDS
- Principe : la responsable RH s'authentifie avec son identité d'entreprise → fédération → accès temporaire limité au tableau de bord uniquement

---

*Document formateur — Dawan — Formation AWS Initiation + Approfondissement — Chapitre 2*
*Formateur Dawan — formation@dawan.fr — Version mars 2026*
