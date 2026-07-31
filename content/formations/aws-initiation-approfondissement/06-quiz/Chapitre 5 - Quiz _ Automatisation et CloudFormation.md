# Chapitre 5 — Quiz : Automatisation, CloudFormation et Well-Architected Framework

> **Formation** : AWS Initiation + Approfondissement (Dawan)
> **Module** : Quiz d'évaluation — Chapitre 5 + Synthèse de la semaine
> **Liens** : [[README]] | [[Chapitre 5 - Formation AWS]] | [[Chapitre 5 - Travaux Pratiques]]

---

## Consignes

- Quiz individuel, sans notes ni documentation
- Cochez **une seule réponse** par question (sauf indication contraire)
- Barème : +1 point par bonne réponse, 0 point par mauvaise réponse ou absence de réponse
- Score de réussite : 7/10 pour le quiz J5, 4/5 pour le quiz synthèse

---

## Quiz Chapitre 5 — 10 questions (10 points)

---

### Q1 — Section obligatoire CloudFormation

Dans un template AWS CloudFormation YAML, quelle est la **seule section obligatoire** ?

- A) `AWSTemplateFormatVersion`
- B) `Parameters`
- C) `Resources`
- D) `Outputs`

---

### Q2 — Fonctions intrinsèques

Quelle fonction intrinsèque CloudFormation permet de **référencer un paramètre ou l'identifiant physique d'une ressource** définie dans le même template ?

- A) `!GetAtt`
- B) `!ImportValue`
- C) `!Sub`
- D) `!Ref`

---

### Q3 — Change Set CloudFormation

Que permet de faire un **Change Set** CloudFormation avant de l'exécuter ?

- A) Sauvegarder l'état actuel de la stack dans S3
- B) Prévisualiser les modifications qui seront apportées aux ressources existantes
- C) Vérifier la syntaxe YAML du template
- D) Créer automatiquement une politique IAM pour la mise à jour

---

### Q4 — Durée d'exécution Lambda

Quelle est la **durée d'exécution maximale** d'une fonction AWS Lambda ?

- A) 5 minutes
- B) 10 minutes
- C) 15 minutes
- D) 30 minutes

---

### Q5 — Piliers Well-Architected

Depuis 2021, le AWS Well-Architected Framework compte combien de piliers ?

- A) 4
- B) 5
- C) 6
- D) 7

---

### Q6 — SSM Session Manager

Quel est le **principal avantage** de AWS SSM Session Manager par rapport à un accès SSH classique ?

- A) Il permet d'accéder à des instances sans ouvrir le port 22 et sans clé SSH, tout en journalisant toutes les sessions
- B) Il est gratuit alors que SSH est payant sur AWS
- C) Il supporte uniquement Windows Server
- D) Il nécessite un Agent Bastion Host dédié dans un subnet public

---

### Q7 — CloudWatch Alarm — Actions possibles

Lorsqu'une CloudWatch Alarm passe en état ALARM, quelles actions peut-elle déclencher ? (Cochez **toutes les réponses correctes**)

- A) Envoyer une notification via Amazon SNS
- B) Déclencher une politique Auto Scaling (scale out ou scale in)
- C) Redémarrer une instance EC2
- D) Déployer automatiquement une nouvelle stack CloudFormation
- E) Invoquer directement une fonction Lambda via EventBridge

---

### Q8 — Pilier Sustainability

Quel pilier du Well-Architected Framework concerne la **durabilité environnementale** et la réduction de l'impact carbone des charges de travail ?

- A) Cost Optimization
- B) Operational Excellence
- C) Sustainability
- D) Performance Efficiency

---

### Q9 — CloudFormation Drift Detection

À quoi sert la fonctionnalité **Drift Detection** de AWS CloudFormation ?

- A) À détecter les nouvelles ressources AWS disponibles dans la région
- B) À identifier les ressources d'une stack qui ont été modifiées manuellement en dehors de CloudFormation
- C) À migrer automatiquement un template CloudFormation vers Terraform
- D) À comparer les coûts entre deux versions d'une même stack

---

### Q10 — AWS Trusted Advisor

AWS Trusted Advisor fournit des recommandations dans combien de catégories principales ?

- A) 3
- B) 4
- C) 5
- D) 6

---

## Corrigé — Quiz Chapitre 5

---

### Corrigé Q1 — Section obligatoire CloudFormation

**Réponse correcte : C) `Resources`**

**Explication** : La section `Resources` est la seule section obligatoire dans un template CloudFormation. Elle contient la définition de toutes les ressources AWS à créer. Les sections `AWSTemplateFormatVersion`, `Description`, `Parameters`, `Mappings`, `Conditions` et `Outputs` sont toutes optionnelles, bien que fortement recommandées pour des templates réutilisables et maintenables.

---

### Corrigé Q2 — Fonctions intrinsèques

**Réponse correcte : D) `!Ref`**

**Explication** :
- `!Ref NomLogique` : retourne la valeur d'un paramètre ou l'identifiant physique d'une ressource (ex : ID du VPC, nom du bucket S3)
- `!GetAtt Resource.Attribute` : retourne un **attribut spécifique** d'une ressource (ex : ARN, DNS Name, URL)
- `!Sub "texte ${Var}"` : effectue une **substitution** de variables dans une chaîne de caractères
- `!ImportValue ExportName` : importe un Output exporté depuis une autre stack

---

### Corrigé Q3 — Change Set CloudFormation

**Réponse correcte : B) Prévisualiser les modifications qui seront apportées aux ressources existantes**

**Explication** : Un Change Set analyse la différence entre le template actuel d'une stack et le nouveau template proposé. Il liste pour chaque ressource l'action prévue (`Add`, `Modify`, `Remove`) et si une modification entraînera une **interruption de service** ou une **recréation complète** de la ressource (`Replacement: True`). Il s'agit de l'équivalent CloudFormation d'un `terraform plan`. Le Change Set doit ensuite être **exécuté** (`execute-change-set`) pour que les modifications soient appliquées.

---

### Corrigé Q4 — Durée d'exécution Lambda

**Réponse correcte : C) 15 minutes**

**Explication** : La durée d'exécution maximale d'une fonction Lambda est de **15 minutes** (900 secondes). C'est une limite ferme qui ne peut pas être dépassée. Pour les traitements qui nécessitent plus de temps, il faut envisager d'autres services : AWS Batch (traitements longs), Step Functions (orchestration de Lambdas), Fargate (conteneurs) ou ECS. Attention : si la Lambda est invoquée via API Gateway, le timeout maximum est de **29 secondes** côté API Gateway.

---

### Corrigé Q5 — Piliers Well-Architected

**Réponse correcte : C) 6**

**Explication** : Le Well-Architected Framework compte **6 piliers** depuis novembre 2021, date à laquelle le pilier **Sustainability** a été ajouté aux 5 piliers initiaux. Les 6 piliers sont :
1. Operational Excellence (Excellence Opérationnelle)
2. Security (Sécurité)
3. Reliability (Fiabilité)
4. Performance Efficiency (Efficacité des Performances)
5. Cost Optimization (Optimisation des Coûts)
6. **Sustainability** (Durabilité) ← ajouté en 2021

Un candidat à l'examen CLF-C02 ou SAA-C03 qui répond "5 piliers" se trompe.

---

### Corrigé Q6 — SSM Session Manager

**Réponse correcte : A) Il permet d'accéder à des instances sans ouvrir le port 22 et sans clé SSH, tout en journalisant toutes les sessions**

**Explication** : Session Manager établit un tunnel chiffré HTTPS (port 443) entre le client et l'agent SSM installé sur l'instance. Cela élimine le besoin d'ouvrir le port 22 dans les Security Groups, de gérer des paires de clés SSH, ou de déployer un bastion host. De plus, toutes les sessions sont automatiquement journalisées dans CloudTrail et optionnellement dans S3 ou CloudWatch Logs, offrant un audit complet. Session Manager fonctionne avec les instances EC2, mais aussi avec les serveurs on-premise (via Hybrid Activations).

---

### Corrigé Q7 — CloudWatch Alarm — Actions possibles

**Réponses correctes : A, B, C**

**Explication** :

| Action | Possible ? | Détails |
|--------|-----------|---------|
| A) Notification SNS | **Oui** | Action native des alarmes CloudWatch |
| B) Politique Auto Scaling | **Oui** | Scale out/in selon les métriques |
| C) Redémarrer une instance EC2 | **Oui** | Action EC2 : reboot, stop, terminate, recover |
| D) Déployer une stack CloudFormation | **Non** | CloudFormation n'est pas une action directe d'alarme |
| E) Invoquer une Lambda via EventBridge | **Oui** | Via SNS → Lambda ou EventBridge Rule (mais pas directement depuis l'Alarm) |

Note : L'alarme peut déclencher une Lambda indirectement (SNS → Lambda), mais pas invoquer Lambda directement comme action d'alarme CloudWatch.

---

### Corrigé Q8 — Pilier Sustainability

**Réponse correcte : C) Sustainability**

**Explication** : Le pilier **Sustainability** (Durabilité) concerne explicitement la minimisation de l'impact environnemental des charges de travail. Il inclut des pratiques comme : utiliser des instances Graviton (ARM, plus efficaces énergétiquement), choisir des régions avec un meilleur mix énergétique, réduire les ressources provisionnées en excès, utiliser des services managés plutôt que des serveurs dédiés.

Le pilier **Cost Optimization** se concentre exclusivement sur le coût financier. Ne pas confondre les deux : on peut optimiser les coûts sans réduire l'impact carbone (ex : Spot Instances, qui utilisent la même infrastructure mais facturent moins).

---

### Corrigé Q9 — CloudFormation Drift Detection

**Réponse correcte : B) Identifier les ressources d'une stack qui ont été modifiées manuellement en dehors de CloudFormation**

**Explication** : La **dérive** (drift) se produit quand une ressource gérée par CloudFormation est modifiée directement via la console, la CLI ou l'API AWS sans passer par CloudFormation. La Drift Detection compare l'état actuel de chaque ressource avec la définition dans le template. Elle retourne l'état `IN_SYNC`, `MODIFIED`, `DELETED` ou `NOT_CHECKED` pour chaque ressource. C'est une opération asynchrone : on lance la détection (`detect-stack-drift`) puis on récupère les résultats ultérieurement.

---

### Corrigé Q10 — AWS Trusted Advisor

**Réponse correcte : C) 5**

**Explication** : AWS Trusted Advisor propose des recommandations dans **5 catégories** :
1. **Cost Optimization** : instances sous-utilisées, EIPs non attachées, snapshots anciens
2. **Performance** : EC2 à fort CPU, RDS sans Read Replicas, CloudFront sans compression
3. **Security** : Security Groups permissifs, MFA non activée, clés IAM vieillissantes
4. **Fault Tolerance** : RDS sans backup, EC2 sans AZ multiples, Route 53 sans health checks
5. **Service Limits** : quotas approchant les limites de services

Les checks complets nécessitent un plan de support Business ou Enterprise. Le plan Basic donne accès uniquement aux checks de sécurité et de Service Limits essentiels.

---

## Barème et résultats — Quiz Chapitre 5

| Score | Appréciation |
|-------|-------------|
| 10/10 | Excellent — prêt pour la certification Cloud Practitioner |
| 8-9/10 | Très bien — quelques points de révision ciblés |
| 6-7/10 | Bien — réviser les points manqués avant la certification |
| 4-5/10 | Passable — relire le cours et refaire le TP |
| < 4/10 | Insuffisant — revoir l'ensemble du chapitre 5 |

---

## Question ouverte FormaTech — Synthèse finale

**Durée recommandée : 10 minutes de réflexion, 5 minutes de présentation orale**

> **Contexte** : l'équipe cloud vient de terminer sa formation AWS et doit présenter à la direction une feuille de route pour la migration de l'entreprise fil rouge vers AWS. Elle dispose d'un délai de six mois et d'une infrastructure actuellement hébergée sur site.
>
> **Question** : Selon le **AWS Well-Architected Framework**, quelle serait votre **priorité n°1** pour la migration FormaTech, et pourquoi ? Appuyez votre réponse sur au moins un pilier et donnez un exemple concret d'action à mettre en oeuvre dans les 30 premiers jours.

**Éléments attendus dans une bonne réponse** :

- Identification d'un pilier pertinent (ex : Reliability pour garantir la continuité de service pendant la migration, Security pour protéger les données des 2000 apprenants)
- Justification en lien avec le contexte FormaTech (PME, e-learning, données personnelles)
- Action concrète et réaliste sur 30 jours (ex : activation CloudTrail + MFA + Budget Alerts + définition des RTO/RPO)
- Conscience des compromis (coût vs sécurité, rapidité de migration vs stabilité)

**Exemple de bonne réponse** :

"Ma priorité n°1 serait le pilier **Security**, car FormaTech traite des données personnelles de 2000 apprenants par jour (obligation RGPD). Dans les 30 premiers jours, je mettrais en place : activation MFA obligatoire pour tous les comptes IAM, activation CloudTrail dans toutes les régions avec conservation 90 jours, configuration de GuardDuty pour la détection d'anomalies, et une revue de tous les Security Groups pour supprimer les règles 0.0.0.0/0 inutiles. Sans cette base de sécurité, tout le reste de la migration serait construit sur des fondations fragiles."

---

## Quiz de Synthèse Final de la Semaine

**5 questions cross-chapitres — 5 points**

Ce quiz évalue la capacité à relier les concepts de l'ensemble de la semaine dans un contexte architectural global.

---

### QS1 — Architecture multi-couches

Une application web FormaTech est déployée selon l'architecture suivante : utilisateurs → Route 53 → CloudFront → ALB → EC2 (Auto Scaling) → RDS Multi-AZ. Un développeur veut stocker les sessions utilisateurs pour qu'elles survivent au remplacement des instances EC2 par l'Auto Scaling.

Quelle solution est la **plus adaptée** ?

- A) Stocker les sessions dans un fichier sur le volume EBS de chaque instance
- B) Utiliser Amazon ElastiCache (Redis ou Memcached) comme store de sessions partagé
- C) Augmenter la durée du cookie de session côté client à 30 jours
- D) Désactiver l'Auto Scaling pour éviter le remplacement des instances

---

### QS2 — Sécurité IAM et VPC

Une développeuse configure un script exécuté sur une instance EC2 qui doit lire des objets dans un bucket S3 du même compte. Quelle est la **méthode recommandée** pour fournir les autorisations AWS à ce script ?

- A) Créer un utilisateur IAM dédié, générer des Access Keys et les stocker dans un fichier `.aws/credentials` sur l'instance
- B) Attacher un **rôle IAM** à l'instance EC2 avec une politique S3 read-only — les credentials sont injectés automatiquement via le metadata service (IMDS)
- C) Passer les Access Keys en variables d'environnement dans le User Data de l'instance
- D) Utiliser les credentials root du compte AWS pour garantir un accès sans restriction

---

### QS3 — Stockage S3 et cycle de vie

FormaTech génère des rapports PDF chaque nuit. Ces rapports sont consultés fréquemment pendant les 30 premiers jours, puis occasionnellement entre 30 et 90 jours, et jamais après 90 jours (conservation légale obligatoire de 7 ans).

Quelle **politique de cycle de vie S3** est la **plus économique** ?

- A) Stocker tous les rapports en S3 Standard indéfiniment
- B) J0→J30 : S3 Standard → J30→J90 : S3 Standard-IA → J90→7 ans : S3 Glacier → Expiration à 7 ans
- C) Stocker directement tous les rapports en S3 Glacier pour minimiser le coût
- D) Supprimer les rapports après 90 jours pour respecter le RGPD

---

### QS4 — CloudFormation et IaC

Une équipe utilise CloudFormation pour gérer son infrastructure. Un administrateur modifie directement le Security Group `sg-web-prod` dans la console pour ajouter une règle temporaire. Deux semaines plus tard, l'équipe relance un `update-stack` avec le template original.

Que se passe-t-il ?

- A) CloudFormation détecte automatiquement la modification et garde la règle ajoutée manuellement
- B) CloudFormation écrase les modifications manuelles et ramène le Security Group à l'état défini dans le template
- C) CloudFormation refuse de mettre à jour la stack car elle est en état de dérive (drift)
- D) CloudFormation demande une confirmation manuelle avant d'écraser les règles ajoutées manuellement

---

### QS5 — Well-Architected et Architecture finale

FormaTech envisage d'économiser sur ses coûts AWS en supprimant le deuxième NAT Gateway (pour ne conserver qu'un seul NAT GW dans une seule AZ au lieu de deux). Cette modification économise environ $35/mois.

Quel pilier du Well-Architected Framework est **principalement impacté négativement** par cette décision ?

- A) Cost Optimization
- B) Performance Efficiency
- C) Reliability
- D) Sustainability

---

## Corrigé — Quiz de Synthèse

---

### Corrigé QS1 — Sessions et Auto Scaling

**Réponse correcte : B) Utiliser Amazon ElastiCache (Redis ou Memcached)**

**Explication** : Quand l'Auto Scaling remplace une instance EC2 (scale-in, mise à jour AMI, instance unhealthy), toutes les données stockées localement sur cette instance sont perdues, y compris les sessions utilisateurs. Stocker les sessions dans un cache partagé comme **ElastiCache** (Redis en production pour sa persistance et ses fonctionnalités de replication) permet à n'importe quelle instance du groupe Auto Scaling de retrouver la session d'un utilisateur, quel que soit le serveur qui répond à sa requête. Cette architecture est fondamentale pour les applications stateless à haute disponibilité.

---

### Corrigé QS2 — Credentials IAM sur EC2

**Réponse correcte : B) Rôle IAM attaché à l'instance EC2**

**Explication** : C'est le pattern recommandé par AWS pour les applications tournant sur EC2. Le rôle IAM est attaché au Instance Profile de l'instance. Le SDK AWS (boto3, aws-cli, etc.) récupère automatiquement des credentials temporaires depuis le **metadata service** (IMDS, `http://169.254.169.254/latest/meta-data/iam/security-credentials/`). Ces credentials sont automatiquement renouvelés, ne sont jamais stockés dans un fichier, et respectent le principe du moindre privilège. Stocker des Access Keys dans un fichier ou en variable d'environnement crée un risque de fuite de credentials si l'instance est compromise.

---

### Corrigé QS3 — Politique de cycle de vie S3

**Réponse correcte : B) S3 Standard → Standard-IA → Glacier → Expiration**

**Explication** : Cette politique correspond exactement aux patterns d'accès décrits :
- **J0 à J30** : accès fréquent → S3 Standard (coût de stockage le plus élevé, mais accès instantané et peu coûteux)
- **J30 à J90** : accès occasionnel → S3 Standard-IA (coût de stockage réduit, coût de récupération plus élevé — acceptable pour un accès peu fréquent)
- **J90 à 7 ans** : jamais consulté mais conservation légale obligatoire → S3 Glacier (coût de stockage très bas, délai de récupération de quelques heures — acceptable)
- **Expiration à 7 ans** : suppression automatique après la période de rétention légale

Mettre en Glacier dès le début serait problématique car les rapports sont consultés fréquemment les 30 premiers jours.

---

### Corrigé QS4 — CloudFormation et modifications manuelles

**Réponse correcte : B) CloudFormation écrase les modifications manuelles**

**Explication** : CloudFormation est un outil **déclaratif et autoritaire**. Lors d'un `update-stack`, il compare l'état souhaité (template) avec l'état actuel des ressources et applique les différences nécessaires pour ramener les ressources à l'état décrit dans le template. La règle ajoutée manuellement sera donc **supprimée**. C'est précisément pourquoi il faut utiliser la Drift Detection régulièrement et ne jamais modifier manuellement des ressources gérées par CloudFormation. Cette situation illustre parfaitement l'importance de tout coder dans l'IaC et d'utiliser les Change Sets pour les modifications.

---

### Corrigé QS5 — NAT Gateway unique et Reliability

**Réponse correcte : C) Reliability**

**Explication** : Le pilier **Reliability** prône l'élimination des Single Points of Failure (SPOF). Un NAT Gateway est un service régional mais est déployé dans **une seule AZ**. Si cette AZ subit une panne (peu fréquent mais possible), toutes les instances en subnet privé de cette AZ perdent leur accès internet (mises à jour, appels API AWS, téléchargements). Avec un NAT GW par AZ, chaque subnet privé utilise le NAT GW de sa propre AZ — en cas de panne d'une AZ, l'autre AZ continue de fonctionner normalement. L'économie de $35/mois fragilise la haute disponibilité de l'architecture pour un risque de panne certes rare mais aux conséquences potentiellement importantes.

---

## Barème — Quiz de Synthèse

| Score | Appréciation |
|-------|-------------|
| 5/5 | Excellent — vision architecturale globale maîtrisée |
| 4/5 | Très bien — un seul angle aveugle à corriger |
| 3/5 | Bien — quelques révisions cross-chapitres nécessaires |
| 2/5 | Passable — revoir les liens entre les services |
| < 2/5 | Insuffisant — reprendre l'ensemble de la semaine |

---

## Résultat global de la semaine

| Quiz | Points possibles | Votre score |
|------|-----------------|-------------|
| Quiz Chapitre 1 — Fondamentaux | /10 | |
| Quiz Chapitre 2 — IAM et Sécurité | /10 | |
| Quiz Chapitre 3 — S3 et EC2 | /10 | |
| Quiz Chapitre 4 — VPC et RDS | /10 | |
| Quiz Chapitre 5 — CloudFormation et Well-Architected | /10 | |
| Quiz Synthèse cross-chapitres | /5 | |
| **TOTAL** | **/55** | **/55** |

| Score global | Appréciation |
|-------------|-------------|
| 50-55/55 | Excellent — niveau Solutions Architect Associate à portée |
| 42-49/55 | Très bien — prêt pour Cloud Practitioner, continuer sur SAA-C03 |
| 33-41/55 | Bien — consolider les chapitres faibles avant la certification |
| 25-32/55 | Passable — formation supplémentaire recommandée |
| < 25/55 | Insuffisant — revoir les fondamentaux AWS |

---

*Quiz rédigé par Formateur Dawan — Dawan — Mars 2026*
*Contenu aligné avec l'examen AWS Certified Cloud Practitioner (CLF-C02) et AWS Certified Solutions Architect Associate (SAA-C03)*
