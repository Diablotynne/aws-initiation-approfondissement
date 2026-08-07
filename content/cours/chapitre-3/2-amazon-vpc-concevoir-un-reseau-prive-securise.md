---
title: "2. Amazon VPC — Concevoir un réseau privé sécurisé"
description: "\"Chapitre 3 — Amazon VPC et bases de données AWS\" - 2. Amazon VPC — Concevoir un réseau privé sécurisé"
---

<nav class="page-sequence"><a href="cours/chapitre-3/1-bases-de-donnees-dans-aws-du-service-gere-a-la-scalabilite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/3-amazon-route-53-dns-intelligent">Suivant</a></nav>

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

<a class="schema-zoom" href="assets/schemas/vpc-architecture.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/vpc-architecture.svg"
     alt="Architecture VPC — Haute disponibilité multi-AZ"
     style="display:block; margin:auto; width:90%"></a>

**Flux de trafic :**
1. Internet → ALB (IGW ouvre l'accès)
2. ALB → EC2 Web (Security Group + règles subnet)
3. EC2 → RDS (Security Group DB ouvre port 3306)
4. EC2 → Internet (via NAT Gateway, pour updates)

---

#### 1. CIDR Block (Classless Inter-Domain Routing)

Une VPC commence par une **plage d'adresses IP privées**. Par exemple, `10.0.0.0/16` signifie :

```
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

**Subnet public** : route vers Internet Gateway → instances accessibles depuis Internet.
**Subnet privé** : route vers NAT Gateway → instances qui accèdent Internet, mais non accessibles de l'extérieur.

#### 3. Internet Gateway (IGW)

L'**Internet Gateway** est la **passerelle de sortie vers Internet**.

**Coût** : 0 € (sauf si vous utilisez des adresses Elastic IP).

#### 4. NAT Gateway

Permet aux instances **privées** d'accéder à Internet **de manière sécurisée** sans être exposées.

**Coût** : $0.032/h + $0.032 par Go transféré.

##### Coût des adresses IPv4 publiques — Point important depuis 2024

Depuis le **1er février 2024**, AWS facture **toutes les adresses IPv4 publiques**, y compris celles attachées à une instance EC2 en cours d'exécution.

**Le tarif :** `$0,005 / heure` par adresse IPv4 publique, soit **~3,65 $ / mois** par IP.

```
Avant février 2024 :
  ✅ IP publique attachée à une instance → GRATUIT
  ❌ Elastic IP non attachée             → 0,005 $/h

Depuis février 2024 :
  ❌ IP publique attachée à EC2          → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur ELB                 → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur RDS                 → 0,005 $/h  ← NOUVEAU
  ❌ IP publique sur NAT Gateway         → 0,005 $/h  ← NOUVEAU
  ❌ Elastic IP non attachée            → 0,005 $/h  (inchangé)
```

**Pourquoi cette décision ?**

L'épuisement des adresses IPv4 est un problème mondial. Il ne reste plus d'adresses disponibles dans le registre IANA depuis 2011. AWS détient environ **130 millions d'adresses IPv4**, qu'il doit acheter sur le marché secondaire.

```
Prix de marché d'une adresse IPv4 en 2024 : environ 55 $ l'adresse
AWS facture 0,005 $/h × 8 760 h/an = 43,80 $/an par IP
→ AWS récupère son investissement en ~15 mois par adresse
```

**Impact concret :**

| Scénario | Nombre d'IPs | Coût mensuel |
|----------|-------------|-------------|
| 1 instance EC2 avec IP publique | 1 | ~3,65 $ |
| 10 instances EC2 + 1 ELB | ~12 | ~43,80 $ |
| Architecture 3-tiers standard | ~5–8 | ~18–29 $ |

> 💡 **Bonne pratique** : utiliser IPv6 là où c'est possible (gratuit), réduire le nombre de ressources avec IP publique, et regrouper les accès via un seul Load Balancer ou une NAT Gateway.

📎 [AWS — Annonce facturation IPv4 (2023)](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)

##### AWS = plateforme 100 % API — À quoi servent vraiment les Elastic IPs ?

**Vous n'avez jamais besoin d'une IP publique pour piloter AWS.** Créer une instance EC2, configurer un VPC, déployer une Lambda — tout cela se fait via l'API AWS, que vous passiez par la console web, la CLI ou un SDK.

```
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

> 💡 Une instance EC2 sans IP publique est tout à fait fonctionnelle si elle n'a pas besoin d'être atteinte depuis Internet. Elle peut appeler des services AWS (S3, DynamoDB, SSM…) via des VPC Endpoints, sans jamais exposer la moindre adresse publique.

**Les cas d'usage légitimes d'une Elastic IP :**

| Cas d'usage | Pourquoi une EIP est nécessaire |
|------------|--------------------------------|
| **Whitelist IP chez un partenaire** | Le firewall du client autorise uniquement votre IP fixe. Si l'instance redémarre avec une nouvelle IP, la connexion est bloquée. |
| **Failover manuel rapide** | En cas de panne, vous réassignez l'EIP de l'instance morte vers une instance de remplacement en quelques secondes — sans changer la DNS ni attendre la propagation. |
| **NAT Gateway** | Obligatoire : le NAT Gateway a toujours besoin d'une EIP pour sortir sur Internet au nom des instances privées. |
| **Serveur mail (SMTP)** | Les serveurs de messagerie tiers filtrent par IP source. Une IP fixe est indispensable pour la réputation mail. |
| **VPN ou tunnel IPsec** | L'extrémité du tunnel doit être connue à l'avance et ne pas changer. |

**Les cas où une EIP n'est PAS la bonne réponse :**

| Mauvais réflexe | Meilleure alternative |
|----------------|----------------------|
| "Je veux accéder à mon serveur web depuis Internet" | → **Load Balancer** avec DNS (pas d'IP fixe nécessaire) |
| "Je veux une URL stable pour mon API" | → **Route 53** + nom de domaine (DNS, pas IP) |
| "Je veux accéder à mon instance en SSH" | → **Session Manager** (SSM) : SSH sans IP publique, sans port 22 ouvert |
| "Je veux que mes Lambda puissent appeler une API externe" | → **NAT Gateway** (une seule EIP pour tout le subnet) |

> ⚠️ **Le réflexe à éviter** : assigner une Elastic IP à chaque instance "au cas où". C'est la première source de surcoût inutile et de surface d'attaque élargie. Préférez toujours un accès via Load Balancer, Route 53, ou SSM Session Manager.

```
Architecture naïve (coûteuse et risquée) :
  EC2-web    ←── EIP  ($3,65/mois, port 80/443 ouvert sur toute l'IP)
  EC2-api    ←── EIP  ($3,65/mois, port 8080 ouvert)
  EC2-admin  ←── EIP  ($3,65/mois, port 22 ouvert)
  Total IPs  : 3 × 3,65 = 10,95 $/mois + surface d'attaque maximale

Architecture correcte (économique et sécurisée) :
  ALB        ←── 1 EIP  ($3,65/mois, ports 80/443 uniquement)
    - EC2-web   (privé, pas d'IP publique)
    - EC2-api   (privé, pas d'IP publique)
  EC2-admin  (privé) ←── SSM Session Manager (0 IP publique, 0 port ouvert)
  Total IPs  : 1 × 3,65 = 3,65 $/mois + sécurité maximale
```

📎 [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
📎 [Elastic IP Addresses — Documentation AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)


#### 5. Route Tables (Tables de routage)

Chaque subnet est associé à une **table de routage** qui définit comment le trafic circule.

<a class="schema-zoom" href="assets/schemas/vpc-route-tables.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/vpc-route-tables.svg"
     alt="Tables de routage VPC — subnet public vs privé, association subnet/route table"
     style="display:block; margin:auto; width:90%"></a>

---

### 2.3 Sécurité réseau — Security Groups et Network ACLs

La sécurité dans une VPC repose sur **deux couches** complémentaires.

#### Security Groups — Pare-feu au niveau instance

Un **Security Group** est un ensemble de **règles de filtrage** appliquées à une ou plusieurs instances EC2.

**Caractéristiques** :
- **Stateful** : si vous autorisez les demandes entrantes, les réponses sortantes sont automatiquement autorisées.
- Changements appliqués **immédiatement**.
- Peut être modifié sur une instance en cours d'exécution.

![](assets/schemas/ch4-capture-05-73664d94.png)

📹 [Groupes de sécurité : pourquoi faire ? Comment ?](https://www.youtube.com/watch?v=QwhexkU2ya4)

#### Network ACLs — Pare-feu au niveau subnet

Un **Network ACL** est un ensemble de règles appliquées à un **subnet entier**.

**Caractéristiques** :
- **Stateless** : vous devez définir EXPLICITEMENT les règles entrantes ET sortantes.
- Numérotées (ordre d'évaluation).
- Application par subnet.

![](assets/schemas/ch4-capture-06-27e106c5.png)

![](assets/schemas/ch4-capture-05-73664d94.png)

#### Différences clés

| Aspect | Security Group | Network ACL |
|--------|---|---|
| **Portée** | Instance | Subnet |
| **État** | Stateful | Stateless |
| **Défaut** | Tout refusé sauf règles | Tout refusé sauf règles |

**Bonne pratique** : utilisez les **Security Groups** pour les **règles fines** (par instance), et les **ACLs** pour les **règles larges** (par subnet).

#### Comment tout s'imbrique — architecture 3-tiers dans une VPC

Ces briques (subnets, Security Groups, NAT Gateway) ne prennent tout leur sens qu'assemblées avec les ressources RDS/EC2 vues plus haut dans ce chapitre. Voici l'architecture la plus enseignée en SAA-C03 : un serveur web accessible depuis Internet, une base de données qui ne l'est jamais.

```
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
| **Coût** | $0.01 par million de requêtes |
| **Inter-comptes** | Deux VPC dans deux comptes AWS différents peuvent être peered |
| **Inter-régions** | Deux VPC dans deux régions différentes peuvent être peered |

> [!warning]
> **VPC Peering est non-transitif — Piège architectural classique**
>
> Si vous avez 3 VPCs : **Prod ↔ Shared** et **Dev ↔ Shared**, cela ne signifie PAS que Prod peut parler à Dev via Shared. Le trafic ne transite JAMAIS par un VPC intermédiaire.
>
> Pour interconnecter N VPCs avec transitivité, utilisez **AWS Transit Gateway** (hub centralisé). Avec 4 VPCs, VPC Peering crée 6 connexions à gérer — avec 10 VPCs, c'est 45 connexions. Transit Gateway réduit cela à 1 attachement par VPC.

![](assets/schemas/ch4-capture-07-5d8ca32f.png)

📎 [Documentation VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

---

### 2.5 Modèle Multi-VPC / Multi-Comptes et AWS Transit Gateway

#### Limitation du VPC Peering

Quand vos infrastructures deviennent **complexes** (10+ VPCs, plusieurs comptes AWS), le peering classique crée un problème :

```
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

En architecture hub-and-spoke, l'AWS Transit Gateway devient le hub centralisé (policies et routing uniques) auquel se connectent, chacun via un seul attachement : le réseau On-Premise, VPC-Prod (`10.0.0.0/16`), VPC-Dev (`10.1.0.0/16`), VPC-Staging (`10.2.0.0/16`) et VPC-Shared (`10.3.0.0/16`) — une connexion unique par ressource, à l'échelle illimitée.

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
| **Multi-comptes** | Non (même compte seulement) | Oui (via Resource Access Manager) |
| **On-premise** | Non | Oui (VPN + Direct Connect) |
| **Policies centralisées** | Impossible | Oui (Network Policy) |
| **Coût** | $0.01 par million requêtes | $0.05 par attachement/h + data |

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
| **Coût : $0.05/attachement/h** | Budget pour 20 VPCs = ~$72/mois (0.05 × 20 × 720 h) |
| **Association subnet obligatoire** | Au moins 1 subnet par AZ pour la résilience |
| **CIDR overlap interdit** | VPCs partagés doivent avoir CIDRs différents |

---

### 2.6 VPC Endpoints — Accès privé aux services AWS

#### Définition

> Un **VPC Endpoint** est une **passerelle privée** qui permet à vos ressources d'accéder à des **services AWS sans passer par Internet**.

#### Deux types

| Type | Services | Fonctionnement | Coût |
|------|----------|---|---|
| **Gateway Endpoint** | S3, DynamoDB | Interface simple (ajoute une route) | Gratuit |
| **Interface Endpoint** | EC2, Secrets Manager, Lambda, etc. | Interface élastique privée (ENI) | $0.007/h + transfert |

![](assets/schemas/ch4-capture-08-302703e2.png)

📎 [Documentation VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)

---

### 2.7 ENI (Elastic Network Interfaces) — Les cartes réseau d'AWS

#### Qu'est-ce qu'une ENI ?

> Une **ENI** est une **carte réseau virtuelle** attachée à une instance EC2. Elle gère vos **adresses IP**, vos **adresses MAC**, vos **Security Groups** et vos **routes réseau**.

Dans une infrastructure **on-prem**, vous aviez des **cartes réseau physiques** (NIC) dans vos serveurs. Sur AWS, c'est exactement la même chose, mais **virtuelle et reconfigurable**.

#### Anatomie d'une ENI

```
Instance EC2 (t3.medium)

- Primary ENI (eth0)  [obligatoire]

   - Primary IP privée : 10.0.1.42 (CIDR subnet)

   - Secondary IP privées : 10.0.1.43, 10.0.1.44 (optionnel)

   - Elastic IP publique : 203.0.113.12 (optionnel)

   - MAC Address : 02:c1:1f:a0:2b:4d (auto-générée)

   - Security Group : sg-12345678

   - Source/Dest Check : ✅ activée (drop trafic non-destiné)
```

📹 [Comment conserver son IP sur AWS ?](https://www.youtube.com/watch?v=oSMEQlQDohM)

#### Cas d'usage : Multiple ENIs sur une même instance

Certains scénarios nécessitent **plusieurs ENIs** sur une instance :

```
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
> ```
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

<div class="concept-check">
<strong>Diagnostic réseau — routage et Security Groups</strong>
<p>Une instance possède une IP publique et un Security Group autorisant HTTPS, mais reste inaccessible depuis Internet. Que faut-il encore vérifier ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>La table de routage du subnet doit contenir une route vers une Internet Gateway attachée au VPC. Une adresse et une règle de sécurité ne créent pas à elles seules le chemin réseau.</p></details>
</div>

---

<nav class="page-sequence"><a href="cours/chapitre-3/1-bases-de-donnees-dans-aws-du-service-gere-a-la-scalabilite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/3-amazon-route-53-dns-intelligent">Suivant</a></nav>
