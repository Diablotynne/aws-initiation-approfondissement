# Chapitre 3 - Quiz — Formation AWS — Initiation + Approfondissement
## Stockage Amazon S3 et calcul Amazon EC2

> **Formation** : AWS Initiation + Approfondissement (Dawan)
> **Module** : Quiz — S3, EC2, EBS, Auto Scaling, Load Balancing
> **Barème** : 10 questions × 1 point = 10 points + 2 points bonus (question ouverte)
> **Liens** : [[README]] | [[Chapitre 3 - Formation AWS]] | [[Chapitre 3 - Travaux Pratiques]]

---

## Instructions

- Lisez chaque question attentivement avant de répondre.
- Pour les QCM : une seule réponse correcte par question, sauf mention contraire.
- Durée conseillée : 2 minutes par question.
- Le corrigé est en fin de document (ne pas consulter pendant le quiz).
- Score conseillé pour valider le chapitre : **7/10 minimum**.

---

## Questions

---

### Q1 — Classe de stockage S3 (1 point)

FormaTech conserve des vidéos d'archives consultées en moyenne **moins d'une fois par mois**, dans un contexte où une perte de données due à une défaillance de zone de disponibilité est **acceptable** (les vidéos peuvent être reconstituées depuis les originaux).

Quelle classe de stockage S3 offre le **meilleur rapport coût/usage** pour ce scénario ?

- A) S3 Standard
- B) S3 Standard-IA (Infrequent Access)
- C) S3 One Zone-IA
- D) S3 Glacier Flexible Retrieval

---

### Q2 — Taille maximale S3 (1 point)

Un ingénieur FormaTech tente d'uploader un fichier de **6 Gio** dans un bucket S3 en utilisant la commande `aws s3 cp`. L'upload se termine avec succès.

Quelle affirmation est vraie concernant cet upload ?

- A) L'upload a échoué car la limite par objet est 5 Gio
- B) L'upload a réussi car la CLI AWS gère automatiquement le Multipart Upload pour les fichiers > 5 Gio
- C) L'upload a réussi car la limite S3 est 5 Tio par objet, et 6 Gio est en dessous
- D) L'upload a réussi car la CLI a automatiquement compressé le fichier avant l'envoi

---

### Q3 — Famille d'instance EC2 pour mémoire (1 point)

FormaTech envisage de migrer sa base de données in-memory **Redis** sur EC2. Les métriques montrent que Redis nécessite **120 Gio de RAM** avec un usage CPU modéré.

Quelle famille d'instances EC2 est la plus adaptée ?

- A) Famille **c** (Compute Optimized)
- B) Famille **m** (General Purpose)
- C) Famille **r** (Memory Optimized)
- D) Famille **i** (Storage Optimized)

---

### Q4 — On-Demand vs Spot (1 point)

FormaTech doit encoder **500 vidéos HD** chaque nuit (tâche de 3h, automatisée, relançable en cas d'échec). L'équipe hésite entre des instances **On-Demand** et des instances **Spot**.

Parmi les affirmations suivantes, laquelle est **correcte** ?

- A) Les instances Spot sont toujours disponibles et offrent jusqu'à 90 % de réduction vs On-Demand
- B) Les instances Spot peuvent être interrompues par AWS avec un préavis de 2 minutes et offrent jusqu'à 90 % de réduction vs On-Demand ; elles sont adaptées aux workloads fault-tolerant comme l'encodage vidéo
- C) Les instances Spot ont un engagement minimum de 1 an
- D) Les instances Spot ne peuvent pas être utilisées avec les types d'instances de la famille c (Compute Optimized)

---

### Q5 — Contenu d'une AMI (1 point)

Un stagiaire FormaTech affirme qu'une AMI (Amazon Machine Image) contient uniquement l'image disque du système d'exploitation, sans les paramètres réseau.

Parmi les éléments suivants, lequel est réellement inclus dans une AMI ?

- A) Un snapshot EBS du volume racine (OS + logiciels installés) et le block device mapping associé
- B) La configuration du Security Group attaché à l'instance
- C) L'adresse IP Elastic associée à l'instance source
- D) Les identifiants IAM du rôle attaché à l'instance

---

### Q6 — Auto Scaling (1 point)

La plateforme FormaTech subit un pic de charge chaque lundi matin à 8h00 lorsque 2 000 apprenants se connectent simultanément. L'équipe ops veut anticiper ce pic automatiquement.

Quel mécanisme Auto Scaling est le plus adapté pour **anticiper** (et non simplement réagir à) ce pic prévisible ?

- A) Target Tracking Scaling (CPU cible 70 %)
- B) Step Scaling (règles par paliers de CPU)
- C) Scheduled Scaling (action planifiée le lundi à 7h45)
- D) Predictive Scaling uniquement (pas d'autre option)

---

### Q7 — Type de Load Balancer (1 point)

FormaTech déploie une architecture web composée de :
- `/api/*` → backend Node.js (port 3000)
- `/static/*` → serveurs Apache statiques (port 80)
- `/admin/*` → interface Django réservée aux formateurs (port 8000)

Quel type de Load Balancer AWS permet de router les requêtes vers différents Target Groups **selon le chemin URL (path-based routing)** ?

- A) NLB (Network Load Balancer) — couche L4
- B) ALB (Application Load Balancer) — couche L7
- C) GWLB (Gateway Load Balancer) — couche L3/L4
- D) CLB (Classic Load Balancer) — ancienne génération

---

### Q8 — Pre-signed URL S3 (1 point)

FormaTech veut permettre à des apprenants **sans compte AWS** de télécharger temporairement une vidéo stockée dans un bucket **privé** S3.

Quelle affirmation sur les Pre-signed URLs S3 est **correcte** ?

- A) Seul le compte root AWS peut générer des Pre-signed URLs
- B) N'importe quel utilisateur IAM ou rôle IAM ayant les permissions `s3:GetObject` sur l'objet peut générer une Pre-signed URL pour cet objet
- C) Les Pre-signed URLs sont valables au maximum 60 minutes
- D) Une Pre-signed URL nécessite que le bucket soit public pour fonctionner

---

### Q9 — Volume EBS pour boot SSD haute performance (1 point)

L'équipe FormaTech doit choisir un type de volume EBS pour le volume racine (boot) d'un serveur de base de données MySQL nécessitant **8 000 IOPS garantis** et **300 MiB/s de débit**.

Quel type de volume EBS est le plus adapté ?

- A) st1 (Throughput Optimized HDD)
- B) sc1 (Cold HDD)
- C) gp3 (General Purpose SSD) configuré à 8 000 IOPS et 300 MiB/s
- D) gp2 (General Purpose SSD ancienne génération)

---

### Q10 — Données EBS après terminaison EC2 (1 point)

Un développeur FormaTech exécute la commande suivante en fin de journée :

```bash
aws ec2 terminate-instances --instance-ids i-0abc123def456789 --region eu-west-3
```

L'instance avait été lancée avec la configuration par défaut (pas de modification du `DeleteOnTermination`). L'instance avait **2 volumes EBS** : le volume racine (`/dev/xvda`) et un volume de données supplémentaire (`/dev/sdf`) attaché ultérieurement via la Console avec `DeleteOnTermination=false`.

Que se passe-t-il aux deux volumes EBS ?

- A) Les deux volumes sont supprimés immédiatement avec l'instance
- B) Les deux volumes sont conservés et disponibles dans EC2 → Volumes
- C) Le volume racine est supprimé (DeleteOnTermination=true par défaut), le volume de données est conservé (DeleteOnTermination=false)
- D) Les deux volumes sont convertis en snapshots automatiquement

---

## Question Ouverte Bonus — FormaTech (2 points)

**Contexte** : FormaTech a migré 500 Go de contenus pédagogiques dans S3. Le responsable technique remarque que la facture S3 augmente de mois en mois, malgré un nombre de fichiers stable.

**Question** : Citez **deux raisons techniques possibles** qui expliqueraient cette augmentation des coûts S3, et proposez **une action corrective** pour chacune.

*(Réponse libre, 5-10 lignes.)*

---

## Corrigé

### Q1 — Réponse : C — S3 One Zone-IA

**Explication** :

S3 One Zone-IA est la classe la plus économique (~0,01 $/Go/mois) pour des données :
- Accédées **rarement** (moins d'une fois par mois — critère Infrequent Access)
- **Reconstituables** en cas de perte (la donnée n'existe que dans **une seule AZ**, sans redondance multi-AZ)

Pourquoi pas les autres ?
- **A (Standard)** : 0,023 $/Go/mois — trop cher pour des données rarement consultées
- **B (Standard-IA)** : 0,0125 $/Go/mois — correct pour les données rares, mais redondance 3 AZ (coût plus élevé que One Zone-IA sans bénéfice si la donnée est reproductible)
- **D (Glacier Flexible)** : adapté si la récupération peut attendre 1-12 heures. Pour des vidéos "accessibles à la demande", même si rares, la latence Glacier est trop longue

**Piège** : Si la perte de données n'est PAS acceptable, choisir Standard-IA (3 AZ) plutôt que One Zone-IA.

---

### Q2 — Réponse : B — La CLI gère automatiquement le Multipart Upload

**Explication** :

- La limite pour un **upload direct** (`PutObject` API) est **5 Gio**. Au-delà, il faut utiliser le **Multipart Upload**.
- La CLI AWS (`aws s3 cp`) gère **automatiquement** le Multipart Upload pour les fichiers dépassant un seuil (par défaut 8 Mo, configurable).
- La limite **maximale d'un objet S3** est **5 Tio** (5 120 Gio) via Multipart Upload.

Pourquoi pas les autres ?
- **A** : Faux — 6 Gio < 5 Tio, donc pas de dépassement de la limite maximale objet. Le problème serait si on utilisait l'API PutObject directement (limite 5 Gio), mais la CLI abstrait cela.
- **C** : Vrai que la limite objet est 5 Tio, mais ce n'est pas la raison principale du succès. La raison est que la CLI utilise Multipart Upload.
- **D** : Faux — La CLI ne compresse pas les données.

---

### Q3 — Réponse : C — Famille r (Memory Optimized)

**Explication** :

La famille **r** (r = RAM) est optimisée pour les workloads à haute densité mémoire :
- Ratio mémoire/vCPU élevé (8 Gio/vCPU vs 4 Gio/vCPU pour la famille m)
- Exemples : r7g.4xlarge (16 vCPU / 128 Gio RAM), r6i.8xlarge (32 vCPU / 256 Gio RAM)

Pourquoi pas les autres ?
- **A (c)** : Optimisée CPU, ratio RAM faible — inadaptée pour Redis 120 Gio
- **B (m)** : Généraliste, ratio équilibré — possible mais moins efficace en coût pour 120 Gio RAM
- **D (i)** : Optimisée stockage local NVMe — inadaptée (Redis est en RAM, pas sur disque)

**Mnémotechnique** : r = RAM, c = CPU, m = middleground, i = I/O stockage

---

### Q4 — Réponse : B

**Explication complète** :

- Les instances **Spot** offrent jusqu'à **90 %** de réduction vs On-Demand en utilisant la capacité EC2 non utilisée.
- AWS peut **interrompre** (récupérer) une instance Spot avec un préavis de **2 minutes** via l'Instance Metadata.
- Elles sont idéales pour les workloads **fault-tolerant** et **interruptibles** : encodage vidéo (relançable), batch processing, CI/CD, big data.
- L'encodage de 500 vidéos nocturnes est un cas d'usage **parfait** pour Spot : si une instance est récupérée, la tâche reprend sur une autre instance.

Pourquoi pas les autres ?
- **A** : Faux — les Spot ne sont **pas** toujours disponibles (dépend de la capacité AWS). La réduction de 90 % est correcte.
- **C** : Faux — Spot n'a **pas** d'engagement minimum. L'engagement 1/3 ans concerne les Reserved Instances.
- **D** : Faux — Spot est disponible pour toutes les familles d'instances, y compris c (Compute Optimized), particulièrement adaptée à l'encodage vidéo.

---

### Q5 — Réponse : A

**Explication** :

Une AMI contient le **snapshot EBS du volume racine** (l'OS, les logiciels pré-installés, les configurations) et le **block device mapping** associé (quels volumes EBS créer/attacher au lancement, avec taille, type et paramètre `DeleteOnTermination`) — c'est le cœur de l'AMI.

Ce que l'AMI ne contient PAS :
- **B) Security Group** : le SG est configuré au moment du **lancement** de l'instance, pas dans l'AMI
- **C) Elastic IP** : les EIP sont des ressources indépendantes associées à l'instance, pas à l'AMI
- **D) Rôle IAM** : le rôle IAM est attaché à l'instance au lancement, il ne fait pas partie de l'AMI elle-même

**Note formateur** : Cette question vise à distinguer les éléments définis dans l'AMI (template) de ceux définis au lancement (Security Group, EIP, IAM Role…).

---

### Q6 — Réponse : C — Scheduled Scaling

**Explication** :

- Le **Scheduled Scaling** permet de définir une action à un horaire précis (cron expression ou datetime). Exemple : chaque lundi à 7h45, passer Desired=4.
- Il **anticipe** la charge avant qu'elle n'arrive, sans attendre que le CPU monte (ce que fait Target Tracking).

Pourquoi pas les autres ?
- **A (Target Tracking)** : réagit **après** que la métrique (CPU) dépasse le seuil. Il y a un délai de réaction de 5-10 minutes → trop tard pour le pic de 8h
- **B (Step Scaling)** : même problème — réactif, pas proactif
- **D (Predictive Scaling uniquement)** : Predictive Scaling peut aussi anticiper via ML, mais la question demande le mécanisme "le plus adapté" pour un pic **connu et récurrent** → Scheduled est plus simple et fiable

**Bonne pratique** : Combiner Scheduled Scaling (proactif pour les pics connus) + Target Tracking (réactif pour les variations imprévisibles).

---

### Q7 — Réponse : B — ALB (Application Load Balancer)

**Explication** :

- L'**ALB** opère au niveau **L7 (application)** du modèle OSI. Il comprend le contenu HTTP/HTTPS et peut router selon :
  - Le chemin URL (`/api/*`, `/static/*`) — **path-based routing**
  - Le nom d'hôte (`api.formateach.fr`, `www.formateach.fr`) — **host-based routing**
  - Les headers HTTP, query strings, méthodes HTTP
- Parfait pour l'architecture FormaTech décrite.

Pourquoi pas les autres ?
- **A (NLB)** : couche L4 — voit seulement les IP et ports TCP/UDP, ne comprend pas les URLs HTTP
- **C (GWLB)** : pour les appliances réseau inline (firewalls, IDS/IPS) — pas pour le routage applicatif
- **D (CLB)** : ancienne génération, déprécié. Routage basique L4+L7 limité, pas de path-based routing avancé

---

### Q8 — Réponse : B

**Explication** :

- **N'importe quel principal IAM** (utilisateur IAM, rôle IAM, etc.) ayant la permission `s3:GetObject` sur l'objet peut générer une Pre-signed URL pour cet objet.
- La Pre-signed URL "délègue" temporairement les permissions du signataire au détenteur de l'URL.

Pourquoi pas les autres ?
- **A** : Faux — le compte root peut générer des Pre-signed URLs, mais n'est pas le seul. Tout principal IAM autorisé peut le faire.
- **C** : Faux — la durée maximale est de **7 jours** (604 800 secondes) pour les credentials utilisateur IAM. Pour les credentials de rôle IAM (sessions temporaires), la durée est limitée à la durée de vie du token (max 12h par défaut pour les rôles EC2).
- **D** : Faux — le bucket peut rester **privé**. C'est justement l'intérêt des Pre-signed URLs : accorder un accès temporaire à un objet privé sans le rendre public.

---

### Q9 — Réponse : C — gp3 configuré à 8 000 IOPS et 300 MiB/s

**Explication** :

- **gp3** est la génération actuelle recommandée pour les volumes SSD à usage général. Ses capacités :
  - IOPS : de 3 000 (baseline inclus dans le prix) jusqu'à **16 000 IOPS** (provisionnables, facturés séparément)
  - Débit : de 125 MiB/s (baseline) jusqu'à **1 000 MiB/s** (provisionnable)
  - Peut être utilisé comme **volume boot**
  - 8 000 IOPS et 300 MiB/s sont dans les limites de gp3 → bon choix

Pourquoi pas les autres ?
- **A (st1)** : HDD Throughput Optimized — **ne peut pas être un volume boot**. Max 500 IOPS, adapté aux lectures séquentielles (Big Data), pas aux bases de données
- **B (sc1)** : HDD Cold — **ne peut pas être un volume boot**. Max 250 IOPS, pour les données froides uniquement
- **D (gp2)** : ancienne génération. Les IOPS sont liées à la taille (3 IOPS/Go, max 16 000). Pour 8 000 IOPS, il faudrait ~2 700 Go de volume gp2 (versus n'importe quelle taille avec gp3). gp3 est moins cher et plus flexible.

**Quand choisir io2 ?** Si les IOPS requises dépassent 16 000 (jusqu'à 256 000 avec io2 Block Express) ou si la cohérence des IOPS est critique (SLA garanti).

---

### Q10 — Réponse : C — Volume racine supprimé, volume de données conservé

**Explication** :

La valeur par défaut de `DeleteOnTermination` dépend de comment le volume a été associé :

| Volume | Comment attaché | DeleteOnTermination par défaut | Résultat à la terminaison |
|---|---|---|---|
| Volume racine (`/dev/xvda`) | Au lancement (défini dans le Launch Template / Console) | **true** | **Supprimé** |
| Volume de données (`/dev/sdf`) | Attaché ultérieurement via Console/CLI | **false** | **Conservé** |

Dans notre scénario :
- Volume racine : `DeleteOnTermination=true` par défaut → **supprimé**
- Volume de données attaché manuellement : `DeleteOnTermination=false` par défaut → **conservé** (reste disponible dans EC2 → Volumes, état "available")

Pourquoi pas les autres ?
- **A** : Faux — le volume de données est conservé (DeleteOnTermination=false)
- **B** : Faux — le volume racine est supprimé (DeleteOnTermination=true par défaut)
- **D** : Faux — AWS ne crée **pas** de snapshots automatiquement à la terminaison. Vous devez le faire manuellement ou via Data Lifecycle Manager.

**Conseil** : Vérifier toujours la valeur de `DeleteOnTermination` avant de terminer une instance, surtout si des données importantes se trouvent sur le volume racine.

---

## Barème et Corrigé Synthétique

| Question | Bonne réponse | Points | Thème |
|---|---|---|---|
| Q1 | C | /1 | Classes de stockage S3 |
| Q2 | B | /1 | Limites S3 et Multipart Upload |
| Q3 | C | /1 | Familles d'instances EC2 |
| Q4 | B | /1 | Modèles de tarification EC2 |
| Q5 | A | /1 | Contenu d'une AMI |
| Q6 | C | /1 | Auto Scaling — Scheduled |
| Q7 | B | /1 | Types de Load Balancer |
| Q8 | B | /1 | Pre-signed URLs S3 |
| Q9 | C | /1 | Types de volumes EBS |
| Q10 | C | /1 | Cycle de vie EBS / Terminaison EC2 |
| **Bonus** | Voir ci-dessous | /2 | Analyse de coût S3 |
| **TOTAL** | | **/12** | |

**Seuil de validation** : 7/10 (sans le bonus) ou 9/12 (avec le bonus).

---

## Corrigé de la Question Ouverte Bonus

**Deux raisons possibles d'augmentation des coûts S3 avec un volume de fichiers stable :**

**Raison 1 — Le versioning accumule des versions non nettoyées**

Si le versioning S3 est activé (recommandé pour la protection des données), chaque modification d'un fichier (re-upload, modification de métadonnées) crée une nouvelle version. Sans Lifecycle Rule sur les anciennes versions (`NoncurrentVersionExpiration`), toutes les versions historiques s'accumulent et sont facturées.

Action corrective : Ajouter une règle Lifecycle `NoncurrentVersionExpiration` → supprimer les versions non-courantes après 90 jours.

```json
{
  "NoncurrentVersionExpiration": { "NoncurrentDays": 90 }
}
```

**Raison 2 — Des classes de stockage inadaptées (Standard pour des contenus archivés)**

Si des vidéos créées il y a 6 mois restent en S3 Standard alors qu'elles ne sont plus consultées, elles sont facturées au tarif Standard (0,023 $/Go/mois) au lieu de Glacier (0,001-0,004 $/Go/mois). Sans Lifecycle Rule de transition, les coûts croissent avec le volume total.

Action corrective : Mettre en place des Lifecycle Rules de transition automatique vers Intelligent-Tiering (30 jours) puis Glacier (90 jours) pour les préfixes `videos/` et `pdf/`.

**Raison 3 (bonus)** — Les Delete Markers s'accumulent (versioning sans expiration des markers)

Chaque suppression d'objet versionné crée un Delete Marker (non facturé lui-même mais qui empêche le nettoyage complet). Si des règles d'expiration sont configurées mais pas les règles de suppression des Delete Markers, des coûts indirects apparaissent.

Action corrective : Ajouter `ExpiredObjectDeleteMarker: true` dans la règle Lifecycle.

---

## Récapitulatif Pédagogique

Ce quiz couvre les connaissances essentielles du Chapitre 3. Si vous avez manqué :

- **Q1, Q2** : Revoir la section "Classes de stockage S3" et "Limites et quotas S3" du cours
- **Q3, Q4, Q10** : Revoir "Familles d'instances EC2", "Modèles de tarification" et "Etats d'instance"
- **Q5** : Revoir "AMI — Amazon Machine Image"
- **Q6** : Revoir "Auto Scaling et Load Balancing"
- **Q7** : Revoir "Types de Load Balancers AWS"
- **Q8** : Revoir "Pre-signed URLs" dans la section S3
- **Q9** : Revoir "Types de volumes EBS"

**Chapitres suivants** : VPC, RDS et architectures réseau (Chapitre 4).

---

*Chapitre 3 — Quiz — Formation AWS Initiation + Approfondissement — Dawan — Formateur Dawan*
*Dernière mise à jour : mars 2026*
