# Chapitre 4 — Quiz — Formation AWS — Initiation + Approfondissement
## Réseau AWS (VPC) et bases de données

> **Formation** : AWS Initiation + Approfondissement (Dawan)
> **Module** : VPC, Sécurité réseau, Route 53, RDS, DynamoDB, Aurora, ElastiCache
> **Barème** : 10 questions × 1 point = 10 points | Seuil de validation : 7/10
> **Liens** : [[README]] | [[Chapitre 4 - Formation AWS]] | [[Chapitre 4 - Travaux Pratiques]]

---

## Instructions

- Une seule réponse correcte par question, sauf mention contraire.
- Ne pas consulter les notes pendant la première lecture.
- Répondre dans l'ordre, puis corriger en fin de quiz.

---

## Questions

---

### Q1 — VPC Peering et transitivité

FormaTech dispose de 3 VPC : `vpc-prod`, `vpc-dev` et `vpc-logs`. Un peering existe entre `vpc-prod` et `vpc-dev`, et un autre peering existe entre `vpc-dev` et `vpc-logs`.

Quelle affirmation est correcte ?

**A)** `vpc-prod` peut communiquer avec `vpc-logs` via `vpc-dev` automatiquement, car le trafic transite par `vpc-dev`.

**B)** `vpc-prod` ne peut pas communiquer avec `vpc-logs` via `vpc-dev`, car le VPC Peering n'est pas transitif. Il faudrait créer un peering direct `vpc-prod` ↔ `vpc-logs`.

**C)** `vpc-prod` peut communiquer avec `vpc-logs` si l'on ajoute une entrée de route dans `vpc-dev` pointant vers les deux autres VPC.

**D)** `vpc-prod` peut communiquer avec `vpc-logs` uniquement si les deux VPC sont dans la même région AWS.

---

### Q2 — Security Groups vs Network ACL

Un administrateur configure la sécurité du subnet privé de FormaTech. Il souhaite empêcher des connexions entrantes depuis une plage d'IP compromise (45.33.32.0/24). Quelle est la méthode correcte ?

**A)** Ajouter une règle DENY dans le Security Group `sg-ec2-app` pour bloquer cette plage CIDR.

**B)** Ajouter une règle DENY numérotée (ex: règle 50) dans la Network ACL associée au subnet privé pour bloquer cette plage CIDR en entrée.

**C)** Il est impossible de bloquer une plage d'IP avec les outils natifs AWS — il faut passer par AWS WAF.

**D)** Retirer toutes les règles ALLOW dans le Security Group pour bloquer implicitement cette plage.

---

### Q3 — Accès Internet depuis les subnets privés

Les instances EC2 du tier application de FormaTech (dans `subnet-priv-3a` et `subnet-priv-3b`) doivent pouvoir télécharger des mises à jour système depuis Internet, sans être accessibles depuis Internet.

Quel composant VPC permet cela ?

**A)** Internet Gateway (IGW) configuré avec des règles restrictives dans le Security Group.

**B)** VPC Endpoint de type Gateway pointant vers les dépôts de paquets.

**C)** NAT Gateway déployé dans un subnet public, avec une route `0.0.0.0/0 → NAT GW` dans la Route Table des subnets privés.

**D)** VPN Site-to-Site connecté au dépôt de paquets on-premises.

---

### Q4 — RDS Multi-AZ vs Read Replica

FormaTech souhaite garantir que sa base de données RDS MySQL reste disponible en cas de défaillance complète de la zone eu-west-3a. Quelle fonctionnalité RDS faut-il activer ?

**A)** Read Replica dans eu-west-3b, car les replicas permettent de basculer automatiquement la charge de lecture vers une autre AZ.

**B)** Multi-AZ, qui maintient une instance standby synchronisée dans une autre AZ et déclenche un failover automatique en cas de défaillance de l'instance primaire.

**C)** Automated Backups avec une rétention de 35 jours, pour restaurer rapidement en cas de panne.

**D)** RDS Proxy, qui redirige automatiquement les connexions vers une AZ disponible.

---

### Q5 — Durée de rétention des sauvegardes automatiques RDS

Quelle est la durée **maximale** de rétention des sauvegardes automatiques (Automated Backups) d'une instance RDS ?

**A)** 7 jours (valeur par défaut et maximum)

**B)** 14 jours

**C)** 35 jours

**D)** 90 jours

---

### Q6 — Clé primaire DynamoDB

Lors de la création de la table `FormaTech-Sessions`, quelle est la configuration minimale obligatoire de la clé primaire ?

**A)** La clé primaire doit obligatoirement être composée d'une Partition Key ET d'une Sort Key.

**B)** La clé primaire peut être constituée uniquement d'une Partition Key (la Sort Key est optionnelle).

**C)** La clé primaire doit être un attribut de type Number auto-incrémenté, comme dans SQL.

**D)** DynamoDB n'utilise pas de clé primaire — tous les attributs sont indexés automatiquement.

---

### Q7 — Réduire la latence de lecture DynamoDB

FormaTech observe que certaines pages du catalogue de cours, dont les données sont stockées dans DynamoDB, prennent 5 ms à charger en moyenne. L'équipe souhaite descendre sous la milliseconde. Quel service AWS permet d'atteindre cet objectif ?

**A)** ElastiCache Redis, en mettant en cache les résultats des requêtes DynamoDB.

**B)** DAX (DynamoDB Accelerator), qui est un cache en mémoire entièrement managé et compatible avec l'API DynamoDB, offrant des latences en microsecondes.

**C)** DynamoDB Global Tables, qui réplique les données dans plusieurs régions pour réduire la latence.

**D)** RDS Read Replica, pour décharger DynamoDB vers un moteur SQL plus rapide.

---

### Q8 — Architecture de stockage Aurora

Combien de copies des données Aurora sont maintenues, et dans combien de zones de disponibilité ?

**A)** 2 copies dans 2 AZ (primaire + standby, comme RDS Multi-AZ).

**B)** 3 copies dans 3 AZ (une par AZ).

**C)** 6 copies dans 3 AZ (2 copies par AZ).

**D)** 6 copies dans 6 AZ différentes, ce qui nécessite une région avec au moins 6 AZ.

---

### Q9 — Route 53 : politique de routage pour le failover

FormaTech souhaite configurer Route 53 pour que le trafic vers `app.formateach.com` bascule automatiquement vers une instance EC2 de secours si l'instance principale devient indisponible.

Quelle politique de routage Route 53 faut-il utiliser ?

**A)** Weighted : répartir le trafic à 90% vers l'instance principale et 10% vers l'instance de secours.

**B)** Latency : diriger le trafic vers l'instance avec la latence la plus faible, qui basculera automatiquement en cas de panne.

**C)** Failover : configurer un enregistrement PRIMARY avec health check sur l'instance principale et un enregistrement SECONDARY sur l'instance de secours.

**D)** Simple : configurer deux valeurs d'enregistrement A, Route 53 choisira aléatoirement en cas de panne.

---

### Q10 — VPC Endpoint de type Gateway

L'équipe FormaTech souhaite que les instances EC2 en subnet privé accèdent à Amazon S3 sans passer par le NAT Gateway (pour réduire les coûts de transfert). Quel type de VPC Endpoint utiliser, et pour quels services ce type est-il disponible ?

**A)** VPC Endpoint de type Interface (PrivateLink), disponible pour tous les services AWS dont S3.

**B)** VPC Endpoint de type Gateway, disponible uniquement pour S3 et DynamoDB. Il s'intègre dans la Route Table du subnet et ne génère pas de coût horaire.

**C)** VPC Endpoint de type Gateway, disponible pour tous les services AWS. Il nécessite une ENI dans le subnet privé.

**D)** AWS Direct Connect Private Virtual Interface, qui est le seul moyen d'accéder à S3 en privé.

---

## Corrigé

---

### Corrigé Q1 — VPC Peering et transitivité

**Réponse correcte : B**

Le VPC Peering AWS est **non transitif**. Même si `vpc-dev` est pairé avec `vpc-prod` ET avec `vpc-logs`, le trafic entre `vpc-prod` et `vpc-logs` ne peut pas transiter via `vpc-dev`. AWS ne route pas le trafic à travers un VPC intermédiaire.

**Pour permettre la communication entre les trois VPC** :
- Option 1 : Créer un peering direct `vpc-prod` ↔ `vpc-logs`
- Option 2 : Utiliser **AWS Transit Gateway** (conçu pour la transitivité à grande échelle)

**Pourquoi C est faux** : Ajouter des routes dans `vpc-dev` ne suffit pas — AWS bloque techniquement le trafic transitif au niveau de l'infrastructure VPC Peering.

---

### Corrigé Q2 — Security Groups vs Network ACL

**Réponse correcte : B**

Les **Security Groups ne supportent pas les règles DENY** (réponse A incorrecte). Seules les **Network ACL** permettent des règles DENY explicites.

La procédure correcte :
1. Aller dans VPC → ACL réseau → sélectionner la NACL du subnet privé
2. Ajouter une règle inbound avec un numéro inférieur aux règles ALLOW existantes (ex: règle 50)
3. Protocole : All traffic (ou TCP si spécifique), Source : `45.33.32.0/24`, Action : **DENY**

**Rappel** : les NACL évaluent les règles par ordre numérique croissant. La première règle qui correspond s'applique.

---

### Corrigé Q3 — Accès Internet depuis les subnets privés

**Réponse correcte : C**

Le **NAT Gateway** (Network Address Translation) est le composant conçu pour permettre aux instances en subnet privé d'initier des connexions sortantes vers Internet, sans exposer leur IP privée et sans permettre de connexions entrantes.

**Fonctionnement** :
1. L'instance EC2 privée envoie une requête (ex: `apt update`)
2. La requête transite par le NAT Gateway (dans le subnet public)
3. Le NAT Gateway substitue l'IP privée par son Elastic IP publique
4. La réponse retourne vers le NAT GW qui la redirige vers l'instance privée

**Pourquoi A est faux** : L'IGW permet aux ressources avec une IP publique de communiquer directement avec Internet — il ne masque pas l'IP et n'est pas adapté aux subnets privés.

---

### Corrigé Q4 — RDS Multi-AZ vs Read Replica

**Réponse correcte : B**

Le **Multi-AZ RDS** maintient une instance standby dans une AZ différente, avec réplication **synchrone** (chaque transaction est écrite sur les deux instances avant confirmation). En cas de défaillance de l'instance primaire, le basculement est **automatique** (~60-120 secondes) et transparent pour l'application (même endpoint DNS).

**Pourquoi A est faux** : Les Read Replicas utilisent une réplication **asynchrone** et le failover est **manuel** (promotion). Elles sont conçues pour la scalabilité de lecture, pas pour la haute disponibilité automatique.

**Pourquoi C est faux** : Les sauvegardes permettent la restauration mais pas le failover automatique.

---

### Corrigé Q5 — Durée de rétention des sauvegardes automatiques RDS

**Réponse correcte : C — 35 jours**

| Paramètre | Valeur |
|-----------|--------|
| Durée minimale | 1 jour (0 = désactivé) |
| Durée par défaut | 7 jours |
| Durée **maximale** | **35 jours** |

Les sauvegardes automatiques permettent la restauration à n'importe quelle seconde dans la fenêtre de rétention (Point-In-Time Recovery — PITR).

Attention : les **Manual Snapshots** ont une durée de rétention illimitée (jusqu'à suppression manuelle).

---

### Corrigé Q6 — Clé primaire DynamoDB

**Réponse correcte : B**

Dans DynamoDB, la **Partition Key seule** est suffisante pour définir une clé primaire (clé primaire simple). La **Sort Key est optionnelle**.

| Type de clé primaire | Composition | Unicité |
|---------------------|-------------|---------|
| Simple | Partition Key uniquement | La PK doit être unique dans la table |
| Composite | Partition Key + Sort Key | La combinaison PK+SK doit être unique |

Pour `FormaTech-Sessions`, la clé composite (`apprenant_id` + `session_date`) est choisie pour permettre à un même apprenant d'avoir plusieurs sessions (différentes dates).

**Pourquoi C est faux** : DynamoDB est NoSQL et ne supporte pas l'auto-incrémentation. Il n'y a pas de séquence automatique.

---

### Corrigé Q7 — Réduire la latence de lecture DynamoDB

**Réponse correcte : B**

**DAX (DynamoDB Accelerator)** est le service conçu spécifiquement pour accélérer les lectures DynamoDB :
- Latence : de millisecondes à **microsecondes**
- Transparent : compatible avec l'API DynamoDB standard (changement minimal de code — juste remplacer l'endpoint)
- Géré par AWS, déployé dans le VPC

**Pourquoi A est partiellement faux** : ElastiCache Redis peut effectivement être utilisé en cache devant DynamoDB, mais nécessite une logique applicative spécifique (cache-aside) et n'est pas transparent comme DAX. DAX est la réponse canonique AWS pour ce cas d'usage.

**Pourquoi C est faux** : Global Tables réduit la latence pour les utilisateurs géographiquement distants, mais ne réduit pas la latence intrinsèque des requêtes.

---

### Corrigé Q8 — Architecture de stockage Aurora

**Réponse correcte : C — 6 copies dans 3 AZ**

Aurora maintient **6 copies** des données réparties sur **3 zones de disponibilité** (2 copies par AZ). Cette architecture est gérée automatiquement par le stockage distribué Aurora, indépendamment du nombre d'instances de calcul.

**Quorum Aurora** :
- Écriture : nécessite 4 copies sur 6 (quorum 4/6)
- Lecture : nécessite 3 copies sur 6 (quorum 3/6)

Cela signifie qu'Aurora peut tolérer la perte d'une AZ complète (2 copies perdues) sans interruption de service ni perte de données.

**Comparaison avec RDS Multi-AZ** : RDS Multi-AZ n'a qu'une copie standby (2 copies au total), alors qu'Aurora a 6 copies natives.

---

### Corrigé Q9 — Route 53 : politique de routage pour le failover

**Réponse correcte : C**

La politique **Failover** Route 53 est conçue exactement pour ce cas d'usage :
1. Créer un enregistrement `app.formateach.com` de type **PRIMARY** pointant vers l'instance principale, avec un **health check** associé
2. Créer un enregistrement `app.formateach.com` de type **SECONDARY** pointant vers l'instance de secours
3. Si le health check détecte une défaillance du PRIMARY, Route 53 résout automatiquement vers le SECONDARY

**Pourquoi A est faux** : Weighted distribue le trafic en permanence selon les poids — ce n'est pas un mécanisme de failover.

**Pourquoi B est faux** : Latency dirige vers la région la plus rapide mais ne gère pas le failover (pas de health check obligatoire).

**Pourquoi D est faux** : Simple avec plusieurs valeurs effectue une rotation aléatoire (round-robin DNS) sans logique de disponibilité.

---

### Corrigé Q10 — VPC Endpoint de type Gateway

**Réponse correcte : B**

Le **VPC Endpoint de type Gateway** est disponible uniquement pour **S3 et DynamoDB**. Il fonctionne en ajoutant une entrée dans la Route Table du subnet (préfixe de service → endpoint gateway) et ne nécessite pas d'ENI ni de coût horaire.

| Caractéristique | Gateway Endpoint | Interface Endpoint (PrivateLink) |
|----------------|-----------------|--------------------------------|
| Services | S3 et DynamoDB uniquement | 100+ services AWS |
| Mécanisme | Entrée dans Route Table | ENI avec IP privée dans le subnet |
| Coût | **Gratuit** | ~7-10 $/mois par AZ |
| Bande passante | Illimitée | Selon l'ENI |

**Avantage concret FormaTech** : les EC2 privés accèdent à S3 (pour les vidéos de cours) sans traverser le NAT Gateway, économisant les frais de traitement de données NAT (0,048 $/Go).

---

## Barème et interprétation

| Score | Interprétation |
|-------|----------------|
| 10/10 | Excellent — maîtrise complète des concepts VPC et BDD AWS |
| 8-9/10 | Très bien — quelques points à consolider |
| 7/10 | Validé — réviser les points manqués avant la certification |
| 5-6/10 | Insuffisant — relire le cours sections concernées et refaire le quiz |
| < 5/10 | Révision complète du chapitre nécessaire |

**Questions les plus souvent ratées** :
- Q1 (transitivité VPC Peering) — piège fondamental
- Q2 (DENY dans SG impossible) — confusion fréquente avec les NACL
- Q4 (Multi-AZ vs Read Replica) — sujet d'examen récurrent
- Q10 (Gateway endpoint = S3 + DynamoDB uniquement)

---

## Question ouverte — Cas FormaTech

**Contexte** : La DSI de FormaTech envisage de migrer sa base de données MySQL on-premises (80 Go, 500 connexions simultanées aux heures de pointe) vers AWS. Le DBA hésite entre RDS MySQL Multi-AZ, Aurora MySQL Serverless v2, et RDS MySQL avec Read Replicas.

**Question** : Rédigez en 10-15 lignes une recommandation argumentée pour FormaTech. Votre réponse doit couvrir :
1. Le service recommandé et les raisons techniques
2. La configuration Multi-AZ et/ou Read Replica envisagée
3. L'impact sur les coûts (comparaison qualitative)
4. La gestion des 500 connexions simultanées (quel service annexe ?)

*Cette question sera corrigée et discutée collectivement avec le formateur.*

---

## Récapitulatif des pièges d'examen — Chapitre 4

| Sujet | Piège | Bonne réponse |
|-------|-------|---------------|
| VPC Peering | Transitif par défaut | **Non transitif** — Transit Gateway pour la transitivité |
| Security Group | Possède des règles DENY | **Non** — ALLOW uniquement |
| NACL | Stateful comme les SG | **Non** — stateless, ouvrir les ports éphémères |
| RDS Multi-AZ | Améliore les perfs de lecture | **Non** — standby passif, pour la HA uniquement |
| RDS Read Replica | Failover automatique | **Non** — promotion manuelle |
| Automated Backups RDS | Durée max = 14 jours | **35 jours** |
| DynamoDB Sort Key | Obligatoire | **Optionnelle** |
| Gateway Endpoint | Pour tous les services | **S3 et DynamoDB uniquement** |
| Aurora stockage | 3 copies (comme Multi-AZ) | **6 copies sur 3 AZ** |
| Route 53 CNAME | Utilisable sur apex domaine | **Non** — utiliser Alias |
| NAT Gateway | Bidirectionnel comme IGW | **Sortant uniquement** |
| Subnet public | Défini par AWS | **Défini par la Route Table** (route vers IGW) |
