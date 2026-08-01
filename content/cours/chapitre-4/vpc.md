---
title: "2. Amazon VPC — Concevoir un réseau privé sécurisé"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - 2. Amazon VPC — Concevoir un réseau privé sécurisé"
---

<nav class="page-sequence"><a href="cours/chapitre-4/bases-donnees">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/route-53">Suivant</a></nav>

### 2.1 Du réseau on-prem au réseau virtuel

#### Définition — Qu'est-ce qu'une VPC ?

> **Amazon VPC (Virtual Private Cloud)** est un **réseau privé virtuel isolé** que vous créez dans AWS. C'est votre **datacenter logique**, entièrement contrôlé, où vous déployez vos instances EC2, vos bases RDS, vos Load Balancers, etc.

Une VPC vous offre :
- **Isolation logique** : vos ressources ne sont pas visibles aux autres comptes AWS.
- **Contrôle total** : vous définissez les adresses IP, les routes, les pare-feux.
- **Flexibilité** : ajouter ou retirer des subnets, des passerelles à volonté.

---

### 2.2 Composants clés d'une VPC

#### Schéma architectural complet d'une VPC sécurisée

<a class="schema-zoom" href="assets/schemas/vpc-architecture.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/vpc-architecture.svg"
     alt="Architecture VPC — Haute disponibilité multi-AZ"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Le VPC est découpé en sous-réseaux publics et privés répartis sur plusieurs zones de disponibilité. Les composants exposés reçoivent le trafic entrant ; les bases restent dans les sous-réseaux privés. Les routes et les groupes de sécurité contrôlent des aspects différents : chemin réseau pour les premières, autorisation des flux pour les seconds.

**Flux de trafic :**
1. Internet → ALB (IGW ouvre l'accès)
2. ALB → EC2 Web (Security Group + règles subnet)
3. EC2 → RDS (Security Group DB ouvre port 3306)
4. EC2 → Internet (via NAT Gateway, pour updates)

---

#### 1. CIDR Block (Classless Inter-Domain Routing)

Une VPC commence par une **plage d'adresses IP privées**. Par exemple, `10.0.0.0/16` signifie :

```bash
10.0.0.0/16 = 65 536 adresses disponibles (10.0.0.0 à 10.0.255.255)

Adresses réservées AWS :
10.0.0.0     = Adresse réseau (réservée)
10.0.0.1     = Gateway AWS (réservée)
10.0.0.2     = DNS AWS (réservé)
10.0.0.3     = Réservé pour l'avenir
10.0.0.4–... = EC2, RDS, ALB, etc.
```

**Plages privées standards (RFC 1918)** :
- `10.0.0.0/8` (la plus courante, 16 M d'adresses)
- `172.16.0.0/12` (1 M d'adresses)
- `192.168.0.0/16` (65 536 adresses)

#### 2. Subnets (Sous-réseaux)

Un subnet est une **subdivision logique** d'une VPC, liée à une **zone de disponibilité (AZ)** spécifique.

**Subnet public** : sa table de routage contient une route vers une Internet Gateway. Une ressource n'est toutefois joignable depuis Internet que si elle possède aussi une adresse publique et si ses contrôles de sécurité autorisent le trafic.
**Subnet privé** : sa table de routage ne contient pas de route directe vers une Internet Gateway. Une route vers une NAT Gateway est une option pour les connexions sortantes IPv4, pas une propriété obligatoire du subnet privé.

#### 3. Internet Gateway (IGW)

L'**Internet Gateway** est la **passerelle de sortie vers Internet**.

Une Internet Gateway n'est pas facturée à l'heure. Les adresses publiques et les transferts de données associés aux ressources restent des postes de coût distincts.

#### 4. NAT Gateway

Permet aux instances **privées** d'accéder à Internet **de manière sécurisée** sans être exposées.

Une NAT Gateway cumule une facturation horaire et une facturation au volume traité. Le transfert de données et l'adresse IPv4 publique peuvent également intervenir. Le tarif varie avec la région.

##### Coût des adresses IPv4 publiques — Point important depuis 2024

Depuis le **1er février 2024**, AWS facture **toutes les adresses IPv4 publiques**, y compris celles attachées à une instance EC2 en cours d'exécution.

Les adresses IPv4 publiques utilisées dans AWS sont facturées. Le prix exact et les éventuelles exceptions se vérifient sur la page officielle de tarification VPC.

**Conséquence d'architecture** : inventorier les adresses avec Public IP Insights, retirer celles qui ne sont pas nécessaires et privilégier les accès privés, les points de terminaison VPC, Systems Manager ou IPv6 lorsque le besoin le permet. Réduire les IPv4 ne doit pas conduire à centraliser tous les flux sur une ressource unique non résiliente.

📎 [AWS — Annonce facturation IPv4 (2023)](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)

##### AWS = plateforme 100 % API — À quoi servent vraiment les Elastic IPs ?

**Vous n'avez jamais besoin d'une IP publique pour piloter AWS.** Créer une instance EC2, configurer un VPC, déployer une Lambda — tout cela se fait via l'API AWS, que vous passiez par la console web, la CLI ou un SDK.

```bash
Vous (navigateur / terminal / code)

           HTTPS → api.aws.amazon.com
           (pas besoin d'IP publique sur vos ressources)
        ▼
   AWS Control Plane  (infrastructure interne AWS)

        ▼
   Votre ressource (EC2, RDS, Lambda...)
   dans votre VPC privé
```

Le **Control Plane** est entièrement géré par AWS et accessible depuis Internet sur ses propres IPs. Vos ressources peuvent très bien vivre dans un subnet privé sans aucune IP publique.

Une instance EC2 sans IP publique reste pleinement fonctionnelle dans le VPC. Elle peut joindre certains services AWS au moyen de points de terminaison VPC compatibles, sous réserve des routes, du DNS, des politiques d'endpoint et des groupes de sécurité requis.

**Les cas d'usage légitimes d'une Elastic IP :**

| Cas d'usage | Pourquoi une EIP est nécessaire |
|------------|--------------------------------|
| **Whitelist IP chez un partenaire** | Le firewall du client autorise uniquement votre IP fixe. Si l'instance redémarre avec une nouvelle IP, la connexion est bloquée. |
| **Adresse source fixe attendue par un tiers** | Un partenaire peut filtrer les connexions sortantes sur une adresse connue ; l'architecture doit alors fournir cette adresse de manière résiliente. |
| **NAT Gateway** | Obligatoire : le NAT Gateway a toujours besoin d'une EIP pour sortir sur Internet au nom des instances privées. |
| **Équipement ou service exigeant une adresse fixe** | Certains protocoles ou systèmes hérités ne savent pas utiliser un nom DNS comme point de terminaison. |

**Les cas où une EIP n'est PAS la bonne réponse :**

| Mauvais réflexe | Meilleure alternative |
|----------------|----------------------|
| "Je veux publier plusieurs serveurs web" | → **Load Balancer** et nom DNS, avec des cibles privées |
| "Je veux une URL stable pour mon API" | → **Route 53** + nom de domaine (DNS, pas IP) |
| "Je veux accéder à mon instance en SSH" | → **Session Manager** (SSM) : SSH sans IP publique, sans port 22 ouvert |
| "Je veux que mes Lambda puissent appeler une API externe" | → **NAT Gateway** (une seule EIP pour tout le subnet) |

> ⚠️ **Le réflexe à éviter** : assigner une Elastic IP à chaque instance « au cas où ». Chaque exposition doit correspondre à un flux documenté. Pour un service web réparti, l'Application Load Balancer fournit un nom DNS ; il ne reçoit pas directement une Elastic IP. Pour l'administration, Session Manager peut éviter une adresse publique et l'ouverture du port SSH.

📎 [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
📎 [Elastic IP Addresses — Documentation AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)


#### 5. Route Tables (Tables de routage)

Chaque subnet est associé à une **table de routage** qui définit comment le trafic circule.

<a class="schema-zoom" href="assets/schemas/vpc-route-tables.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/vpc-route-tables.svg"
     alt="Tables de routage VPC — subnet public vs privé, association subnet/route table"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Le sous-réseau public possède une route vers l'Internet Gateway. Le sous-réseau privé n'en possède pas ; lorsqu'une sortie Internet est nécessaire, sa route pointe vers une NAT Gateway située dans un sous-réseau public. Le trafic retour suit l'état de la traduction NAT.

---

### 2.3 Sécurité réseau — Security Groups et Network ACLs

La sécurité dans une VPC repose sur **deux couches** complémentaires.

#### Security Groups — Pare-feu au niveau instance

Un **Security Group** est un ensemble de **règles de filtrage** appliquées à une ou plusieurs instances EC2.

**Caractéristiques** :
- **Stateful** : si vous autorisez les demandes entrantes, les réponses sortantes sont automatiquement autorisées.
- Changements appliqués **immédiatement**.
- Peut être modifié sur une instance en cours d'exécution.

<div class="video-embed"><iframe src="https://www.youtube-nocookie.com/embed/QwhexkU2ya4" title="Comprendre les groupes de sécurité AWS" loading="lazy" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

#### Network ACLs — Pare-feu au niveau subnet

Un **Network ACL** est un ensemble de règles appliquées à un **subnet entier**.

**Caractéristiques** :
- **Stateless** : vous devez définir EXPLICITEMENT les règles entrantes ET sortantes.
- Numérotées (ordre d'évaluation).
- Application par subnet.

#### Différences clés

| Aspect | Security Group | Network ACL |
|--------|---|---|
| **Portée** | Instance | Subnet |
| **État** | Stateful | Stateless |
| **Défaut** | Tout refusé sauf règles | Tout refusé sauf règles |

**Bonne pratique** : utilisez les **Security Groups** pour les **règles fines** (par instance), et les **ACLs** pour les **règles larges** (par subnet).

#### Comment tout s'imbrique — architecture 3-tiers dans une VPC

Ces briques prennent leur sens lorsqu'elles forment un chemin réseau cohérent. Le modèle suivant expose uniquement le point d'entrée web et maintient la base de données dans des subnets privés.

```text
VPC 10.0.0.0/16
│
├── Subnet PUBLIC (10.0.1.0/24, AZ 1a)
│    ├── Route Table → 0.0.0.0/0 via Internet Gateway
│    ├── Application Load Balancer (ALB)
│    │    Security Group ALB : inbound 443 depuis 0.0.0.0/0
│    └── NAT Gateway (sort vers Internet pour le subnet privé)
│
├── Subnet PRIVÉ — App (10.0.10.0/24, AZ 1a)
│    ├── Route Table → 0.0.0.0/0 via NAT Gateway (pas d'IGW direct)
│    └── Instance EC2 (serveur web)
│         Security Group EC2 : inbound 80 UNIQUEMENT depuis Security Group ALB
│
└── Subnet PRIVÉ — Data (10.0.20.0/24, AZ 1a)
     ├── Route Table → pas de route vers Internet (isolé)
     └── Instance RDS (déployée en section 1 de ce chapitre)
          Security Group RDS : inbound 3306 UNIQUEMENT depuis Security Group EC2
```

**Ce que cette architecture garantit :**
- Seul l'ALB est exposé sur Internet (subnet public, port 443 ouvert à tous).
- L'EC2 n'accepte du trafic que depuis l'ALB — jamais directement depuis Internet, même si son IP était devinée.
- Le RDS n'accepte du trafic que depuis l'EC2 — la base de données n'a **aucune route vers Internet**, donc même une erreur de configuration Security Group ne peut pas l'exposer.
- Chaque Security Group référence un *autre Security Group* comme source (pas une plage d'IP) — c'est la bonne pratique : si l'EC2 change d'adresse IP (redémarrage, remplacement), la règle reste valide.

---

### 2.4 VPC Peering — Connecter plusieurs VPC

> **VPC Peering** établit une **connexion réseau privée** entre deux VPC, permettant aux instances de communiquer comme si elles étaient dans le même réseau.

#### Caractéristiques

| Aspect | Détail |
|--------|--------|
| **Non-transitif** | A ↔ B fonctionne. Mais A ↔ C ne passera pas par B : il faut une peering A-C explicite. |
| **Coût** | La connexion de peering n'est pas facturée à l'heure ; le transfert de données applicable reste facturable |
| **Inter-comptes** | Deux VPC dans deux comptes AWS différents peuvent être peered |
| **Inter-régions** | Deux VPC dans deux régions différentes peuvent être peered |

> [!warning]
> **VPC Peering est non-transitif — Piège architectural classique**
>
> Si vous avez 3 VPCs : **Prod ↔ Shared** et **Dev ↔ Shared**, cela ne signifie PAS que Prod peut parler à Dev via Shared. Le trafic ne transite JAMAIS par un VPC intermédiaire.
>
> Pour interconnecter N VPCs avec transitivité, utilisez **AWS Transit Gateway** (hub centralisé). Avec 4 VPCs, VPC Peering crée 6 connexions à gérer — avec 10 VPCs, c'est 45 connexions. Transit Gateway réduit cela à 1 attachement par VPC.


📎 [Documentation VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

---

### 2.5 Modèle Multi-VPC / Multi-Comptes et AWS Transit Gateway

#### Limitation du VPC Peering

Quand vos infrastructures deviennent **complexes** (10+ VPCs, plusieurs comptes AWS), le peering classique crée un problème :

```text
AVEC PEERING CLASSIQUE (non-transitif) :
Pour N VPCs : N × (N-1) / 2 peerings = EXPLOSION combinatoire

Exemple 4 VPCs :
VPC-Prod ↔ VPC-Dev        (1)
VPC-Prod ↔ VPC-Staging    (2)
VPC-Prod ↔ VPC-Shared     (3)
VPC-Dev ↔ VPC-Staging     (4)
VPC-Dev ↔ VPC-Shared      (5)
VPC-Staging ↔ VPC-Shared  (6)
             ↓
        6 peerings manuels ! 😰

Problèmes :
- Non-transitif : Prod ↔ Dev ↔ Shared = Prod ne voit pas Shared
- Gestion complexe : chaque nouveau VPC = 3 peerings à créer
- Pas de contrôle centralisé : règles partagées impossibles
```

#### AWS Transit Gateway — La solution

> **AWS Transit Gateway** est un **hub réseau centralisé** qui connecte **toutes vos VPCs, comptes AWS et réseaux on-prem** via une seule interface.

En architecture *hub-and-spoke*, l'AWS Transit Gateway devient un hub de routage auquel les VPC et certaines connexions réseau s'attachent. Cette centralisation simplifie les relations nombreuses, mais reste soumise aux quotas, aux routes, aux autorisations et au coût du service.

Attachements possibles :
- ✅ VPC (plusieurs)
- ✅ Comptes AWS (via RAM — Resource Access Manager)
- ✅ On-premise (via VPN ou Direct Connect)
- ✅ Transit Gateway externe (inter-régions)

#### Architecture complète : Multi-Comptes avec Transit Gateway

L'organisation AWS regroupe plusieurs comptes, tous rattachés au même Transit Gateway (`tgw-xxx`, possédé par le Compte Prod) :

| Compte | Ressource | Rattachement |
|---|---|---|
| PROD (123456789012) | VPC-Prod (`10.0.0.0/16`) | Attachement TGW |
| DEV (210987654321) | VPC-Dev (`10.1.0.0/16`) | Attachement TGW |
| SHARED (shared-000) | VPC-Shared (`10.3.0.0/16`) | Attachement TGW |
| ON-PREMISE | Network (`192.168.0.0/16`) | Attachement VPN/Direct Connect |
| MGMT | CloudTrail logs (audit) | — |

Le Transit Gateway maintient des route tables distinctes (Production, Dev, Shared) pour contrôler quel trafic peut transiter entre quels VPC.

#### Avantages Transit Gateway

| Aspect | VPC Peering | Transit Gateway |
|--------|---|---|
| **Connexions N VPCs** | N(N-1)/2 peerings 😱 | 1 attachement par VPC ✅ |
| **Transitif** | Non (A↔B, B↔C ≠ A↔C) | Oui (contrôlé par routing) |
| **Multi-comptes** | Oui, avec acceptation de la connexion | Oui, notamment via Resource Access Manager |
| **On-premise** | Non | Oui (VPN + Direct Connect) |
| **Policies centralisées** | Impossible | Oui (Network Policy) |
| **Coût** | Transfert de données applicable | Attachements et traitement de données, selon la région |

#### Configuration AWS CLI — Transit Gateway, principe

Le Transit Gateway est créé une seule fois, puis chaque VPC s'y attache via une **attachment**. La table de routage du TGW détermine quels VPCs peuvent se parler.

```bash
# Créer le Transit Gateway
aws ec2 create-transit-gateway \
  --description "Formation Transit Gateway Hub" \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=FormationTGW}]'
# Résultat : TransitGatewayId = tgw-0123456789abcdef

# Attacher un VPC au Transit Gateway
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-0123456789abcdef \
  --vpc-id vpc-prod-12345 \
  --subnet-ids subnet-prod-1a subnet-prod-1b
```

> [!tip]
> **Résultat attendu :**
> ```json
> {"TransitGatewayVpcAttachment": {"State": "pending", "TransitGatewayId": "tgw-0123456789abcdef", "VpcId": "vpc-prod-12345"}}
> ```


> [!info]
> **Pour aller plus loin — hors périmètre de cette formation** : partager un Transit Gateway entre plusieurs comptes AWS (via Resource Access Manager) et l'étendre à un réseau on-premise (VPN Site-to-Site) relèvent du niveau Advanced Networking Specialty. Le principe reste le même — un attachement par ressource, une table de routage centralisée — mais la mise en œuvre cross-account est un sujet à part entière.


#### Pièges Transit Gateway

| Piège | Solution |
|-------|----------|
| **TGW par défaut permet tout** | Créer des route tables TGW restrictives par environnement |
| **Chaque attachement et volume traité contribue au coût** | Compter les attachements, les heures et le trafic dans AWS Pricing Calculator |
| **Association subnet obligatoire** | Au moins 1 subnet par AZ pour la résilience |
| **CIDR overlap interdit** | VPCs partagés doivent avoir CIDRs différents |

---

### 2.6 VPC Endpoints — Accès privé aux services AWS

#### Définition

> Un **VPC Endpoint** est une **passerelle privée** qui permet à vos ressources d'accéder à des **services AWS sans passer par Internet**.

#### Deux types

| Type | Services | Fonctionnement | Modèle de coût |
|------|----------|---|---|
| **Gateway Endpoint** | S3, DynamoDB | Cible ajoutée aux tables de routage sélectionnées | Pas de facturation horaire propre à l'endpoint |
| **Interface Endpoint** | Services compatibles avec AWS PrivateLink | Interfaces réseau privées dans les subnets choisis | Heures d'endpoint et volume traité, selon la région |

📎 [Documentation VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)

---

### 2.7 ENI (Elastic Network Interfaces) — Les cartes réseau d'AWS

#### Qu'est-ce qu'une ENI ?

> Une **ENI** est une **carte réseau virtuelle** attachée à une instance EC2. Elle gère vos **adresses IP**, vos **adresses MAC**, vos **Security Groups** et vos **routes réseau**.

Dans une infrastructure **on-prem**, vous aviez des **cartes réseau physiques** (NIC) dans vos serveurs. Sur AWS, c'est exactement la même chose, mais **virtuelle et reconfigurable**.

#### Anatomie d'une ENI

```text
Instance EC2 (t3.medium)

- Primary ENI (eth0)  [obligatoire]

   - Primary IP privée : 10.0.1.42 (CIDR subnet)

   - Secondary IP privées : 10.0.1.43, 10.0.1.44 (optionnel)

   - Elastic IP publique : 203.0.113.12 (optionnel)

   - MAC Address : 02:c1:1f:a0:2b:4d (auto-générée)

   - Security Group : sg-12345678

   - Source/Dest Check : ✅ activée (drop trafic non-destiné)
```

<div class="video-embed"><iframe src="https://www.youtube-nocookie.com/embed/oSMEQlQDohM" title="Conserver une adresse IP avec Elastic IP" loading="lazy" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

#### Cas d'usage : Multiple ENIs sur une même instance

Certains scénarios nécessitent **plusieurs ENIs** sur une instance :

```text
SCÉNARIO : Serveur pare-feu / VPN / Load Balancer

Instance (m5.xlarge) : 4 ENIs possibles

- eth0 (Primary) : connectée à VPC public  → Internet
    - 10.0.1.10

- eth1 (Secondary) : connectée à VPC privée → Apps internes
    - 10.1.1.10

- eth2 (Secondary) : connectée à VPC client (peering) → Client network
    - 10.2.1.10

- eth3 (Secondary) : Management/monitoring
    - 10.0.2.10 (subnet privé)

Résultat : machine routeur/pare-feu multi-réseaux ! 🔥
```

#### Configuration ENI via AWS CLI

On crée ici une ENI indépendante puis on l'attache à une instance existante. Cela permet d'ajouter une interface réseau secondaire sans recréer l'instance — utile pour un serveur bastion ou un routeur multi-réseaux.

```bash
# ═══════════════════════════════════════════════════════════
# 1. Créer une ENI seule (détachée)
# ═══════════════════════════════════════════════════════════

aws ec2 create-network-interface \
  --subnet-id subnet-12345678 \
  --description "Secondary management interface" \
  --private-ip-addresses PrivateIpAddress=10.0.2.100,Primary=true \
  --groups sg-mgmt-12345678 \
  --tag-specifications 'ResourceType=network-interface,Tags=[{Key=Name,Value=ENI-Management}]'

# Résultat : NetworkInterfaceId = eni-0a1b2c3d4e5f6g7h8

# ═══════════════════════════════════════════════════════════
# 2. Attacher une ENI existante à une instance
# ═══════════════════════════════════════════════════════════

aws ec2 attach-network-interface \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --instance-id i-0123456789abcdef0 \
  --device-index 1  # eth1 (0=primary/eth0, 1=eth1, etc.)

# ═══════════════════════════════════════════════════════════
# 3. Allouer une Elastic IP et l'attacher à une ENI
# ═══════════════════════════════════════════════════════════

# Allouer Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --private-ip-address 10.0.2.100

# Résultat : PublicIp = 203.0.113.50

# ═══════════════════════════════════════════════════════════
# 4. Ajouter une IP privée secondaire à une ENI
# ═══════════════════════════════════════════════════════════

aws ec2 assign-private-ip-addresses \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --private-ip-addresses 10.0.2.101 10.0.2.102

# ═══════════════════════════════════════════════════════════
# 5. Changer de Security Group sur une ENI
# ═══════════════════════════════════════════════════════════

aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-0a1b2c3d4e5f6g7h8 \
  --groups sg-new-12345678 sg-management-67890

# ═══════════════════════════════════════════════════════════
# 6. Détacher une ENI (reste dans la VPC, peut être réattachée)
# ═══════════════════════════════════════════════════════════

aws ec2 detach-network-interface \
  --attachment-id eni-attach-12345678

# ═══════════════════════════════════════════════════════════
# 7. Voir toutes les ENI d'une instance
# ═══════════════════════════════════════════════════════════

aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --query 'Reservations[0].Instances[0].NetworkInterfaces[*].[NetworkInterfaceId,Attachment.DeviceIndex,PrivateIpAddresses[0].PrivateIpAddress,PrivateIpAddresses[0].Association.PublicIp]'

# Résultat (exemple) :
# [
#   ["eni-primary", 0, "10.0.1.42", "203.0.113.50"],    ← eth0 + Elastic IP
#   ["eni-secondary", 1, "10.0.2.100", null]             ← eth1 (privée)
# ]
```

> [!tip]
> **Résultat attendu :**
> ```json
> [
>     ["eni-0a1b2c3d4e5f00001", 0, "10.0.1.42", "203.0.113.50"],
>     ["eni-0a1b2c3d4e5f00002", 1, "10.0.2.100", null]
> ]
> ```


#### Cas d'usage réels : ENI multiples

| Scénario | Interfaces | Bénéfice |
|----------|---|---|
| **Routeur/Pare-feu** | 3-4 ENIs | Connexion à plusieurs VPCs/subnets sans NAT |
| **Haute disponibilité** | Primary ENI + failover | IP privée = même, instance change (failover transparent) |
| **Serveur DNS interne** | Primary + management | Trafic DNS sur une interface, logs/monitoring sur autre |
| **Load Balancer maison** | Multiple NICs | Distribution load par interface réseau |
| **Serveur VPN/bastion** | 2+ interfaces | Accès de plusieurs subnets via une machine unique |

#### Piège : Source/Destination Check

Par défaut, AWS bloque tout paquet dont l'IP source ou destination ne correspond pas à l'interface. C'est intentionnel pour la sécurité, mais cela empêche un routeur ou un pare-feu de forwarder le trafic. Il faut donc **désactiver cette vérification** sur les instances qui jouent un rôle de routage.

```bash
# ⚠️ Par défaut, une ENI REFUSE le trafic non-destiné à elle-même.
# Exemple : routeur firewall doit FOWARDER le trafic.

# Désactiver Source/Destination check
aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-12345678 \
  --no-source-dest-check

# Résultat : routeur peut maintenant forwarder (forwarding activé)

# Réactiver (sécurité)
aws ec2 modify-network-interface-attribute \
  --network-interface-id eni-12345678 \
  --source-dest-check
```

> [!tip]
> **Résultat attendu :**
> ```json
> # modify-network-interface-attribute : aucun output si succès
>
> # Vérification : aws ec2 describe-network-interface-attribute --network-interface-id eni-12345678 --attribute sourceDestCheck
> {
>     "NetworkInterfaceId": "eni-12345678",
>     "SourceDestCheck": {
>         "Value": false
>     }
> }
> ```
> La vérification Source/Destination est désactivée (`Value: false`). L'interface peut désormais forwarder du trafic dont elle n'est pas la destination finale — comportement requis pour une instance jouant le rôle de routeur ou de pare-feu.


---

<nav class="page-sequence"><a href="cours/chapitre-4/bases-donnees">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/route-53">Suivant</a></nav>
