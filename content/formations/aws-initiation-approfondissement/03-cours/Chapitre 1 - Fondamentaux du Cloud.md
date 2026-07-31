# Chapitre 1 — Fondamentaux du Cloud & Présentation AWS


---

> [!NOTE]
> Cette formation débute par les fondations théoriques indispensables avant de manipuler la console AWS : sans comprendre ce qu'est réellement le Cloud Computing, ses modèles économiques et ses responsabilités, il est impossible de faire des choix d'architecture pertinents par la suite.
>
> **Objectifs du chapitre**
>
> À l'issue de ce chapitre, les stagiaires seront capables de :
>
> - **Expliquer** ce qu'est AWS, son historique et son positionnement sur le marché du Cloud
> - **Identifier** les cinq caractéristiques fondamentales du Cloud Computing selon le NIST
> - **Distinguer** le modèle économique CAPEX du modèle OPEX et en expliquer l'impact sur les entreprises
> - **Situer** les responsabilités respectives d'AWS et du client dans le modèle de responsabilité partagée
> - **Différencier** les modèles de service IaaS, PaaS et SaaS
> - **Comparer** les modèles de déploiement Cloud public, privé et hybride
> - **Expliquer** les principes de la virtualisation, de la conteneurisation et des microservices
> - **Décrire** l'infrastructure mondiale AWS (régions, zones de disponibilité, edge locations)
> - **Appliquer** les six piliers du AWS Well-Architected Framework à un cas simple
> - **Naviguer** dans l'AWS Management Console et identifier ses principales fonctionnalités
> - **Utiliser** les outils de suivi budgétaire AWS (Billing Dashboard, Budgets, Cost Explorer) pour maîtriser les coûts
![Carte du chapitre : des besoins métier aux services AWS et aux principes d'architecture](formations/aws-initiation-approfondissement/11-images/ch1-carte-concepts.svg)

<div class="concept-check">
<strong>Décision d'architecture — avant de poursuivre</strong>
<p>Une application française contient des données sensibles et doit rester disponible lors de la panne d'un datacenter. Quel premier choix faut-il formuler ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Choisir une région conforme aux contraintes de localisation, puis répartir les composants sur plusieurs zones de disponibilité. Une région et une AZ ne répondent pas au même niveau de panne.</p></details>
</div>

---

## 1. Introduction

### 1.1 Qu'est-ce qu'AWS et pourquoi le découvrir ?

**Amazon Web Services (AWS)** est une plateforme de cloud computing lancée en 2006. Elle propose plus de 200 services pour le calcul, le stockage, les bases de données, la sécurité, l'IA, le DevOps, et bien plus encore.

Derrière vous, il y a des millions de stagiaires, d'administrateurs et de développeurs qui utilisent AWS chaque jour pour faire fonctionner les sites web, les applications mobiles, les bases de données, les analyses de données et l'intelligence artificielle du monde entier.

**Pourquoi apprendre AWS ?**
- C'est le **leader mondial** du Cloud public (environ 32 % du marché).
- Les compétences AWS sont très recherchées sur le marché de l'emploi.
- C'est une **porte d'entrée** vers l'architecture cloud moderne.
- AWS ne cesse de croître et innover : chaque jour, de nouveaux services apparaissent.

Vous trouverez AWS dans **quasiment tous les secteurs** : startups qui testent une idée en quelques jours, grandes banques qui hébergent leurs systèmes critiques, hôpitaux qui gèrent les données médicales, gouvernements, universités, agences spatiales…

### 1.2 Quelques dates clés

| Année | Événement | Explication pédagogique |
|-------|-----------|------------------------|
| **2006** | Lancement d'AWS avec **S3** et **EC2** | AWS débute avec deux services fondamentaux : **S3** pour le stockage objet (sauvegardes, fichiers, images) et **EC2** pour le calcul (machines virtuelles). Ces deux services incarnent les piliers du cloud : stockage et puissance de calcul à la demande. |
| **2009** | Introduction de **VPC** et **RDS** | AWS ajoute **VPC** (Virtual Private Cloud) pour créer des réseaux privés isolés, et **RDS** (Relational Database Service) pour gérer des bases de données relationnelles sans administrer les serveurs. Cela marque l'arrivée des services réseau et des bases gérées. |
| **2012** | Lancement de **DynamoDB** | AWS introduit **DynamoDB**, une base NoSQL scalable et sans schéma, adaptée aux applications modernes (web, mobile, IoT). C'est le tournant vers les architectures serverless et les microservices. |
| **2014** | Lancement de **AWS Lambda** | AWS révolutionne le cloud avec **Lambda**, qui permet d'exécuter du code sans serveur. C'est le début du **serverless computing**, où l'infrastructure disparaît derrière la logique métier. |
| **2025** | Plus de **200 services**, leader mondial | AWS devient le fournisseur cloud le plus complet, couvrant tous les domaines : calcul, stockage, IA, sécurité, DevOps, IoT, bases de données, etc. Il est utilisé par des millions d'entreprises dans le monde entier. |

Ces dates montrent comment AWS a évolué d'un simple fournisseur de serveurs et de stockage vers une **plateforme cloud complète**, capable de répondre à tous les besoins informatiques : hébergement, sécurité, automatisation, intelligence artificielle, etc.

### 1.3 Certifications AWS

AWS structure son parcours de certification en quatre niveaux, pensés pour accompagner une montée en compétence progressive plutôt que pour classer les candidats par mérite. Chaque niveau valide un périmètre de responsabilités différent en entreprise, et il est tout à fait normal — recommandé, même — de les passer dans l'ordre.

Le niveau **Fondamental** (Cloud Practitioner) s'adresse à toute personne qui doit comprendre le vocabulaire et les grands principes du cloud AWS sans nécessairement configurer quoi que ce soit elle-même : commerciaux, chefs de projet, décideurs, ou stagiaires qui découvrent AWS pour la première fois. L'examen porte sur les concepts (modèles de tarification, responsabilité partagée, services principaux) plutôt que sur la mise en œuvre technique.

Le niveau **Associate** cible les professionnels qui déploient et opèrent concrètement des infrastructures AWS au quotidien. Il se décline selon le métier visé : **Solutions Architect – Associate** pour la conception d'architectures, **CloudOps Engineer – Associate (SOA-C03)** pour l'exploitation et la supervision — ce nom remplace SysOps Administrator depuis 2025 — et **Developer – Associate** pour l'intégration d'applications avec les API et services AWS. Le catalogue comprend également des certifications Associate orientées données et machine learning.

Le niveau **Professional** approfondit les deux certifications Architecte et DevOps du niveau Associate avec des scénarios de complexité réelle : migrations à grande échelle, architectures multi-comptes, optimisation fine des coûts et de la résilience. Ces examens supposent une expérience pratique significative — AWS les recommande après plusieurs années d'usage professionnel du cloud.

Le niveau **Spécialité** (Specialty) valide une expertise pointue plutôt qu'une vision généraliste. Le catalogue évolue régulièrement : Database Specialty et plusieurs autres examens ont été retirés en 2024, puis Machine Learning Specialty le 31 mars 2026. Il faut donc consulter le catalogue officiel avant de présenter une liste d'examens comme actuelle. Security Specialty et Advanced Networking Specialty restent des repères pertinents pour les domaines abordés ici.

Pour cette formation, le niveau visé est le **Solutions Architect Associate (SAA-C03)** : c'est la certification la plus demandée sur le marché de l'emploi et celle qui couvre le plus largement les services abordés dans les chapitres suivants (EC2, S3, VPC, RDS, IAM, CloudFormation). Le Chapitre 5 détaillera la pondération exacte des domaines de l'examen une fois tous les services vus.

![](formations/aws-initiation-approfondissement/11-images/ch1-capture-01-6cc3f203.png)

📎 [Certifications AWS](https://aws.amazon.com/certification/)

---

## 2. Fondamentaux du Cloud Computing

### 2.1 Qu'est-ce que le Cloud Computing ?

Le **Cloud Computing** est une évolution majeure dans la manière dont les systèmes d'information sont conçus, déployés et exploités.

Pendant des décennies, les entreprises ont investi massivement dans leurs propres infrastructures : datacenters internes, salles machines, serveurs physiques, systèmes de climatisation, sécurité, licences logicielles, équipes d'exploitation dédiées.

Ce modèle repose sur des investissements lourds (**CAPEX**) et des cycles de décision longs.

Le Cloud vient bouleverser cette logique en offrant un modèle **flexible**, **scalable** et **orienté services** : les ressources sont **louées à la demande** et **payées à l'usage**.

![](formations/aws-initiation-approfondissement/11-images/ch1-capture-02-17ee61f5.png)

📎 [Documentation officielle AWS — What is Cloud Computing](https://aws.amazon.com/what-is-cloud-computing/)

### 2.2 Les cinq caractéristiques fondamentales du Cloud (NIST)

Le **NIST (National Institute of Standards and Technology)** a défini cinq caractéristiques fondamentales du Cloud Computing. Ces critères permettent de distinguer une véritable solution Cloud d'une simple externalisation :

#### 1. Libre-service à la demande (On-demand self-service)

Un utilisateur peut provisionner des ressources informatiques (par exemple, une machine virtuelle) sans intervention humaine du fournisseur.

**En pratique avec AWS** : Vous cliquez sur « Lancer une instance EC2 », configurez les paramètres, et en quelques secondes, votre serveur est actif et prêt à l'emploi.

#### 2. Mise en commun des ressources (Resource pooling)

Les ressources physiques (serveurs, stockage, réseau) sont partagées entre plusieurs clients mais restent isolées logiquement.

**En pratique avec AWS** : Vos instances EC2 exécutent sur du matériel physique partagé avec d'autres clients, mais AWS garantit une isolation complète — vos données restent vos données.

#### 3. Élasticité et scalabilité (Rapid elasticity)

Les ressources peuvent être augmentées ou réduites dynamiquement en fonction de la demande.

**En pratique avec AWS** : Votre application reçoit soudain 10 fois plus de visiteurs. AWS augmente automatiquement la capacité de calcul, puis la réduit quand le trafic baisse.

#### 4. Mesurabilité et facturation à l'usage (Measured service)

Chaque ressource est mesurée et facturée selon la consommation réelle : CPU, mémoire, bande passante, stockage, etc.

**En pratique avec AWS** : Vous payez par heure pour une instance EC2, par Go stocké sur S3, par requête sur Lambda. Pas de frais fixes, pas de ressources inutilisées payées « pour rien ».

#### 5. Accès étendu au réseau (Broad network access)

Les utilisateurs gèrent leurs ressources via une console Web, des API ou des outils en ligne de commande (CLI).

**En pratique avec AWS** : Vous ne contactez pas AWS pour dire « besoin d'un serveur ». Vous vous connectez à la console, créez ce que vous voulez, et c'est instantané.

📎 [NIST — The NIST Definition of Cloud Computing](https://csrc.nist.gov/publications/detail/sp/800-145/final)

### 2.3 CAPEX vs OPEX — Comprendre le changement de modèle économique

Historiquement, les entreprises ont fonctionné sur un modèle **CAPEX** (*Capital Expenditures*) : elles achetaient leurs équipements, les installaient dans leurs propres locaux et les exploitaient pendant plusieurs années.

Avec le Cloud, elles passent à un modèle **OPEX** (*Operational Expenditures*) : elles louent des ressources à la demande et paient uniquement pour leur consommation réelle.

| Modèle | Définition | Exemple typique | Gestion |
|--------|-----------|-----------------|---------|
| **CAPEX** | Investissement initial important amorti sur plusieurs années | Achat de serveurs physiques | Budget fixe |
| **OPEX** | Dépenses variables basées sur l'usage | Paiement horaire d'une instance EC2 | Budget flexible |

**Exemple concret :**

**CAPEX (Avant le Cloud)** : Une entreprise achète 10 serveurs physiques pour héberger une application interne. Coût initial : 50 000 €. Ils fonctionnent en permanence, même en période creuse, avec des coûts d'entretien et de maintenance élevés (climatisation, électricité, renouvellement matériel tous les 5-7 ans).

**OPEX (Avec AWS)** : La même entreprise utilise **Amazon EC2 (Elastic Compute Cloud)**, un service AWS permettant de lancer et gérer des **machines virtuelles**. Elle ne paie que les heures réellement consommées (ex. 100 € par mois en moyenne) et peut arrêter les instances lorsqu'elles ne sont pas nécessaires.

La différence ? Flexibilité, prévisibilité des coûts, et liberté d'adapter l'infrastructure en temps réel.

> [!WARNING]
> **Attention aux coûts AWS — le modèle OPEX peut surprendre :**
> Une instance EC2 laissée tournante 24h/7j sans utilisation reste facturée. Contrairement à un investissement CAPEX amorti sur plusieurs années, les coûts OPEX s'accumulent en temps réel. Activez toujours des **alertes de budget (AWS Budgets)** dès le premier jour.
📎 [AWS Economics Center](https://aws.amazon.com/economics/)

### 2.4 La rupture culturelle du libre-service

Dans les environnements informatiques traditionnels, **les équipes métiers et techniques** devaient **passer par des processus centralisés** pour obtenir des ressources :

* Demande de serveurs à l'équipe IT
* Validation budgétaire et technique
* Commande physique, installation, configuration…
  👉 Résultat : **des délais de plusieurs semaines** avant de pouvoir simplement démarrer un projet.

Avec **le Cloud AWS**, ce modèle est **complètement bouleversé**.

#### 1. Libre-service immédiat

* Les équipes peuvent **provisionner elles-mêmes** des ressources (VM, bases de données, stockage…) **en quelques clics ou via des API**.
* Plus besoin d'attendre une autre équipe pour « débloquer » les moyens techniques.
* Cela change profondément la manière de travailler : les développeurs deviennent **autonomes**.

#### 2. Responsabilisation et gouvernance

* Le libre-service ne signifie pas « anarchie » : cela s'accompagne de **règles**, **quotas**, **politiques IAM**, **budgets**, etc.
* Les organisations doivent mettre en place une **gouvernance cloud** qui encadre cette autonomie.

#### 3. Accélération de l'innovation

* Cette autonomie permet de tester une idée en quelques heures plutôt qu'en plusieurs semaines.
* Les équipes peuvent **échouer vite et pas cher**, puis pivoter ou étendre si le projet fonctionne.

#### 4. Une vraie rupture culturelle

* Le passage au cloud AWS **n'est pas qu'une question de technologie**.
* Il faut **changer les mentalités** :
  * Déléguer le pouvoir de créer des ressources.
  * Accepter la rapidité et parfois le désordre initial.
  * Former et responsabiliser les équipes pour qu'elles deviennent **consommatrices actives** du cloud.

« Avant, il fallait une demande, une validation, une livraison. Aujourd'hui, on clique — ou on lance une commande API — et on a une infrastructure prête en quelques secondes. Cette autonomie est la clé du Cloud, mais elle exige une nouvelle culture : celle de la confiance, de la responsabilisation et de la gouvernance bien pensée. »

### 2.5 Les limites et contraintes du Cloud

Aucun modèle n'est parfait. Le Cloud introduit aussi plusieurs défis importants :

* **Dépendance au fournisseur (vendor lock-in)** : migrer entre AWS, Azure ou GCP peut nécessiter une réarchitecture importante.
* **Connexion Internet critique** : sans réseau, pas d'accès aux ressources hébergées dans le cloud.
* **Sécurité partagée** : certaines responsabilités ne sont **pas déléguées** au fournisseur.
* **Maîtrise des coûts** : une mauvaise configuration peut générer des dépenses importantes et rapides.

Nous allons détailler **la contrainte liée à la sécurité partagée**, car elle est souvent **mal comprise** par les entreprises débutant dans le cloud.

### 2.6 Le modèle de responsabilité partagée AWS

Avant d'étudier le tableau, utilisez le [simulateur interactif de responsabilité partagée](../07-annexes/simulateur-responsabilite-partagee.html). Comparez EC2, RDS et Lambda : plus le service est managé, plus AWS prend en charge de couches techniques, sans jamais devenir responsable de la classification des données ni des autorisations décidées par le client.

Le modèle de responsabilité partagée AWS répartit la sécurité entre deux parties, et confondre les deux moitiés est l'une des erreurs les plus fréquentes chez les entreprises qui découvrent le cloud.

AWS porte la responsabilité de la **sécurité du cloud**, c'est-à-dire de tout ce qui constitue l'infrastructure sous-jacente sur laquelle les clients construisent leurs services. Cela couvre la sécurité physique des data centers — accès contrôlé, vidéosurveillance, alimentation redondante — que le client n'a jamais à gérer lui-même. Cela couvre aussi le matériel et la couche de virtualisation qui isole chaque client des autres sur un même serveur physique, ainsi que la disponibilité globale de la plateforme, avec ses multiples régions et zones de disponibilité conçues pour survivre à la panne d'un data center entier.

Le **client**, de son côté, porte la responsabilité de la **sécurité dans le cloud** : tout ce qu'il configure et déploie au-dessus de cette infrastructure reste de son ressort. Il doit gérer les accès et les identités via IAM — qui a le droit de faire quoi sur son compte — et sécuriser ses données par le chiffrement et la rotation régulière des clés. Il doit configurer correctement son réseau (VPC, Security Groups, NACL) pour éviter d'exposer par erreur des ressources sensibles sur Internet. Et lorsqu'il utilise un service de type IaaS comme EC2, où AWS fournit la machine virtuelle mais pas ce qui tourne dessus, il reste responsable des sauvegardes, des correctifs de sécurité et des mises à jour du système d'exploitation.

Cela signifie que même si AWS est hautement sécurisé, **une mauvaise configuration côté client peut compromettre la sécurité** (par ex. un bucket S3 public par erreur).

> [!IMPORTANT]
> **Responsabilité client — erreurs fréquentes en production :**
> - Un bucket S3 configuré en **accès public par erreur** expose toutes vos données sur Internet.
> - L'utilisation du **compte root** pour les opérations quotidiennes est une faille de sécurité majeure.
> - Un **Security Group ouvert sur 0.0.0.0/0 port 22** expose vos instances SSH au monde entier.
> - L'absence de **MFA** sur le compte root est la première cause de compromission de compte AWS.
>
> AWS ne peut pas vous protéger de vos propres erreurs de configuration — c'est votre responsabilité.
![](formations/aws-initiation-approfondissement/11-images/ch1-capture-03-37dd1c7c.png)

📎 [Shared Responsibility Model – AWS](https://aws.amazon.com/fr/compliance/shared-responsibility-model/)

### 2.7 Sécurité de l'information dans le Cloud

La **sécurité de l'information** repose sur trois piliers fondamentaux (**CID**) :

| Pilier | Définition | Exemple AWS |
|--------|-----------|-------------|
| **Confidentialité** | Seules les personnes autorisées doivent accéder aux données. | IAM, KMS, Bucket Policies |
| **Intégrité** | Les données ne doivent pas être altérées ou corrompues. | Checksums, versioning S3 |
| **Disponibilité** | Les systèmes doivent être accessibles à tout moment. | Multi-AZ, Load Balancer, snapshots |

Ce tableau cite trois exemples AWS dont vous ne verrez le détail que plus tard dans la formation — c'est normal à ce stade : **IAM** (gestion des identités et des accès) est le sujet entier du Chapitre 2, **KMS** (Key Management Service, le service de gestion des clés de chiffrement) et les **Bucket Policies** (règles d'accès attachées à un bucket S3) seront présentés respectivement en section 6 de ce chapitre et au Chapitre 3. Retenez pour l'instant seulement le principe : chaque pilier CID s'appuie sur des outils AWS concrets que vous manipulerez au fil de la formation.

AWS fournit des outils et des certifications (ISO 27001, SOC 2, PCI-DSS…), mais **l'entreprise reste responsable de son usage**.

**Exemples concrets de responsabilités du client :**

* Un bucket S3 mal configuré (public par erreur) = **fuite de données** → responsabilité **du client**.
* Un mot de passe faible ou MFA non activée = **intrusion possible** → responsabilité **du client**.
* Absence de sauvegarde / réplication = **perte de disponibilité** → responsabilité **du client**.
* Firewall mal ouvert (0.0.0.0/0 sur SSH) = exposition à Internet → responsabilité **du client**.

C'est pourquoi **les bonnes pratiques de configuration** sont **aussi importantes que les services AWS eux-mêmes**.

**En résumé :**

| Sécurité du cloud (AWS) | Sécurité dans le cloud (Client) |
|-------------------------|--------------------------------|
| Data centers, matériel | Gestion des accès et des rôles (IAM) |
| Virtualisation et réseau global | Sécurisation des données et des flux |
| Résilience physique | Mise à jour, chiffrement, surveillance |
| Conformité et certifications | Respect des politiques internes et réglementations sectorielles |

### 2.8 Externalisation du Système d'Information (SI)

#### Pourquoi externaliser ?

Externaliser le SI vers le Cloud permet de :
- **Réduire les coûts** d'infrastructure et de maintenance.
- **Accéder à une expertise** et à des technologies de pointe.
- **Améliorer la disponibilité et la continuité d'activité.**

Mais cela implique aussi une **réévaluation des risques** :
- Dépendance contractuelle envers le fournisseur.
- Nécessité de revoir les politiques de sauvegarde et d'accès.
- Conformité légale (RGPD, hébergement des données, localisation géographique).

**Conseil :**
Avant toute migration, réalisez une **analyse d'impact** (juridique, technique et organisationnelle).
AWS met à disposition un outil appelé **AWS Migration Readiness Assessment (MRA)**.

📎 [AWS Migration Hub](https://aws.amazon.com/fr/migration-hub/)

---

## 3. Modèles de services : IaaS, PaaS et SaaS

Les modèles de services sont essentiels pour comprendre **comment les entreprises consomment le Cloud**.
Ils structurent les **couches techniques** (infrastructure, plateforme, application) et déterminent qui — du client ou du fournisseur — est responsable de chaque couche.

📎 [AWS Cloud Service Models](https://aws.amazon.com/types-of-cloud-computing/)

Ces modèles sont une clé de lecture indispensable pour architecturer correctement une solution Cloud.

### 3.1 Infrastructure as a Service (IaaS)

**Infrastructure as a Service (IaaS)** désigne un modèle dans lequel le fournisseur met à disposition des ressources informatiques de base : **puissance de calcul**, **stockage**, **réseau** et **sécurité**.
Le client gère et configure ces ressources selon ses besoins.

#### Caractéristiques clés :

- Accès à des **machines virtuelles** configurables.
- Gestion des disques, du réseau et de la sécurité.
- Contrôle total sur l'OS et les applications.
- Évolutivité à la demande.

#### Exemples AWS :

- **Amazon EC2 (Elastic Compute Cloud)** : service qui permet de créer et gérer des machines virtuelles dans le Cloud.
- **Amazon EBS (Elastic Block Store)** : stockage en mode bloc attaché aux instances EC2.
- **Amazon VPC (Virtual Private Cloud)** : réseau virtuel isolé et configurable.
- **Elastic Load Balancing (ELB)** : répartition automatique du trafic entre plusieurs ressources.

**Analogie** : L'IaaS est comparable à la location d'un terrain pour construire une maison — le client choisit tout, mais doit aussi tout gérer.

### 3.2 Platform as a Service (PaaS)

**Platform as a Service (PaaS)** décharge le client de la gestion de l'infrastructure et du système d'exploitation.
Le fournisseur gère les couches basses, le client se concentre sur le développement et la mise en production.

#### Caractéristiques clés :

- Pas de gestion des serveurs ni de patching système.
- Environnements prêts à l'emploi pour le déploiement applicatif.
- Scalabilité intégrée.

#### Exemples AWS :

- **AWS Elastic Beanstalk** : service de déploiement et d'orchestration d'applications Web.
- **AWS Lambda** : service serverless qui exécute du code sans gérer de serveur.
- **Amazon RDS (Relational Database Service)** : base de données relationnelle gérée (patchs, sauvegardes, monitoring automatisés).

📎 [Documentation AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk)

Le PaaS permet de **réduire la charge opérationnelle** et d'accélérer les cycles de développement.

### 3.3 Software as a Service (SaaS)

**Software as a Service (SaaS)** est un modèle dans lequel le fournisseur gère toute la pile technique — infrastructure, plateforme et application — et livre un service clé en main.

#### Caractéristiques clés :

- Aucune gestion technique côté client.
- Accès via un navigateur ou une API.
- Maintenance et sécurité assurées par le fournisseur.

#### Exemples AWS :

- **AWS WorkMail** : service de messagerie hébergée.
- **Salesforce** : CRM en ligne.
- **Microsoft 365**, **Google Workspace** : solutions collaboratives.

📎 [AWS SaaS Factory](https://aws.amazon.com/saas-factory/)

Retenons les cas d'usage typiques du SaaS — messagerie, CRM, outils de collaboration, ERP.

### 3.4 Comparatif synthétique des modèles

![](formations/aws-initiation-approfondissement/11-images/ch1-capture-04-764ffd51.jpg)

| Élément | IaaS | PaaS | SaaS |
|--------|------|------|------|
| **Flexibilité** | Très élevée | Moyenne | Faible |
| **Temps de déploiement** | Plus long | Rapide | Instantané |
| **Cas d'usage typique** | Migration, environnements complexes | Déploiements rapides, automatisation | Solutions métiers clés en main |

---

## 4. Modèles de déploiement : Public, Privé et Hybride

Les **modèles de déploiement** définissent *où* sont hébergées les ressources Cloud et *qui* les gère.
Ce choix a un impact direct sur la **gouvernance**, la **sécurité**, la **performance** et le **coût global** d'une solution Cloud.

Il existe trois grands modèles de déploiement reconnus par le NIST et adoptés par la majorité des entreprises : **Cloud Public**, **Cloud Privé** et **Cloud Hybride**.

Le modèle de déploiement choisi dépend souvent d'un compromis entre agilité, sécurité, conformité et maîtrise de l'environnement technique.

### 4.1 Le Cloud Public

Le **Cloud Public** est un modèle dans lequel les ressources sont hébergées dans les **datacenters du fournisseur Cloud** et partagées entre plusieurs clients.
Les infrastructures sont **mutualisées**, mais chaque client bénéficie d'un environnement isolé logiquement.

#### Caractéristiques :

- Hébergement sur l'infrastructure du fournisseur.
- Mutualisation des ressources physiques.
- Facturation à l'usage.
- Évolutivité quasi illimitée.
- Accès global via Internet sécurisé.

#### Avantages :

- Pas de gestion matérielle.
- Déploiement rapide.
- Coût réduit grâce à la mutualisation.
- Accès à un large éventail de services.

#### Inconvénients :

- Moins de contrôle sur l'infrastructure physique.
- Dépendance au fournisseur.
- Besoins accrus en gouvernance et sécurité.

#### Exemple AWS :

**AWS Public Cloud** repose sur l'infrastructure mondiale d'**Amazon Web Services (AWS)**, composée de **régions** (Regions), **zones de disponibilité** (Availability Zones, AZ) et **points de présence** (Edge Locations).

- Une *région* est une zone géographique (par exemple *eu-west-3* pour Paris).
- Une *zone de disponibilité (AZ)* est un datacenter isolé dans cette région, assurant redondance et tolérance aux pannes.
- Les *points de présence* servent principalement aux services de diffusion de contenu comme Amazon CloudFront.

**Services AWS typiques dans le Cloud public :**

- **Amazon EC2** : instances de calcul à la demande
- **Amazon S3** : stockage objet scalable
- **AWS Lambda** : exécution de code sans serveur
- **Amazon RDS** : bases de données relationnelles gérées
- **Amazon CloudFront** : CDN mondial pour accélérer les contenus

📎 [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

### 4.2 Le Cloud Privé

Le **Cloud Privé** est un modèle dans lequel l'entreprise déploie ses propres ressources Cloud sur une **infrastructure dédiée**, souvent hébergée dans ses propres locaux ou dans un datacenter tiers, mais **non mutualisée** avec d'autres clients.

#### Caractéristiques :

- Infrastructure dédiée à une seule organisation.
- Contrôle complet sur l'environnement.
- Peut utiliser des technologies de virtualisation similaires à celles du Cloud public.
- Peut être automatisé et orchestré comme un Cloud public.

#### Avantages :

- Meilleure maîtrise de la sécurité et de la conformité.
- Contrôle total sur les configurations.
- Plus de personnalisation possible.

#### Inconvénients :

- Coût d'investissement plus élevé (proche d'un modèle CAPEX).
- Moins de flexibilité et de scalabilité.
- Nécessite des équipes internes pour la gestion.

#### Pourquoi le Cloud Privé existe-t-il encore ?

Avec la transformation digitale — certaines entreprises conservent des charges sensibles en interne pour des raisons réglementaires.

**Exemple : le RGPD (Règlement Général sur la Protection des Données)**

Le **RGPD** impose aux entreprises européennes une **maîtrise stricte** de la localisation, de la protection et de la gouvernance des données personnelles. De ce fait, certaines entreprises — notamment dans les secteurs **banque**, **santé** ou **secteur public** — choisissent de **conserver certaines données ou systèmes sensibles en interne** ou sur des infrastructures "souveraines" pour :

- Garantir que les données ne quittent pas l'Espace Économique Européen,
- Mieux contrôler les accès et traitements,
- Répondre aux obligations de **traçabilité** et de **confidentialité**.

**Exemples concrets :**

- **Système bancaire français** hébergeant les données de ses clients sur une infrastructure locale ou un cloud "de confiance" pour respecter :
    - L'article 44 du RGPD (transferts hors UE),
    - Les obligations de minimisation et de limitation de traitement (articles 5 et 6),
    - Les recommandations de la CNIL sur l'hébergement de données sensibles.

- **Code de la santé publique** → données de santé hébergées uniquement chez un **Hébergeur de Données de Santé (HDS)** agréé en France. Les hôpitaux et laboratoires doivent conserver certaines données sur site ou dans un cloud certifié HDS.

- **Secteur de la Défense** → obligations de **souveraineté et de sécurité** imposées par l'ANSSI (ex. SecNumCloud), qui limitent l'externalisation à des fournisseurs labellisés.

**En résumé :**

| Législation / Réglementation | Secteur impacté | Conséquence sur l'hébergement |
|---|---|---|
| RGPD | Tous secteurs manipulant des données personnelles | Contrôle de localisation, transferts restreints |
| HDS | Santé | Hébergement uniquement chez des prestataires certifiés |
| ANSSI / SecNumCloud | Secteurs sensibles / OIV | Cloud souverain ou on-prem obligatoire pour certaines charges |

Ce type de réglementation **explique pourquoi certaines entreprises gardent encore des charges sensibles "on-premise"** ou sur des **infrastructures dédiées** plutôt que dans le cloud public pur.

> [!WARNING]
> **RGPD et localisation des données AWS :**
> Par défaut, AWS peut stocker vos données dans n'importe quelle zone de disponibilité de la région choisie. Pour les données personnelles de citoyens européens, vous devez impérativement choisir une **région européenne** (ex. `eu-west-3` Paris, `eu-central-1` Francfort) et vérifier que les services utilisés ne transfèrent pas les données hors UE sans votre accord explicite.
#### AWS s'est adapté :

- Un groupe bancaire peut déployer un **Cloud privé** sur sa propre infrastructure virtualisée avec VMware ou OpenStack, tout en automatisant les déploiements comme dans AWS.

📎 [VMware Cloud Foundation](https://www.vmware.com/products/cloud-foundation.html)
📎 [OpenStack](https://www.openstack.org/)

**Services AWS typiques pour le Cloud privé :**

- **AWS Outposts** : infrastructure AWS déployée dans le datacenter du client
- **VMware Cloud on AWS** : environnement VMware géré dans AWS
- **Amazon VPC** : réseau virtuel isolé
- **AWS Direct Connect** : liaison réseau privée entre le client et AWS

![](formations/aws-initiation-approfondissement/11-images/ch1-capture-05-ca65c3fb.png)

### 4.3 Le Cloud Hybride

Le **Cloud Hybride** combine le **Cloud Privé** et le **Cloud Public**, en permettant aux applications et données de circuler entre les deux environnements.
C'est aujourd'hui le modèle **le plus répandu** dans les grandes organisations.

#### Caractéristiques :

- Combinaison de ressources internes et publiques.
- Interopérabilité entre les environnements.
- Optimisation des coûts et de la sécurité.
- Support de scénarios de migration progressive.

#### Avantages :

- Flexibilité maximale.
- Répartition intelligente des charges de travail.
- Sécurité renforcée pour les données sensibles.
- Possibilité d'exploiter le meilleur des deux mondes.

#### Inconvénients :

- Gestion plus complexe.
- Nécessite des compétences solides en architecture et interconnexion.
- Risque de dépendances multiples.

#### Exemple AWS :

**AWS Direct Connect** est un service AWS qui permet de **créer une liaison réseau privée dédiée** entre l'infrastructure interne d'une entreprise et AWS.
Cela permet de combiner la sécurité d'un Cloud privé avec la puissance d'un Cloud public.

**Services AWS typiques pour le Cloud hybride :**

- **AWS Storage Gateway** : intégration du stockage local avec S3
- **AWS Transit Gateway** : interconnexion de VPC et réseaux sur site
- **AWS Systems Manager** : gestion centralisée des ressources hybrides
- **AWS Backup** : sauvegarde unifiée pour ressources cloud et sur site

📎 [AWS Direct Connect](https://aws.amazon.com/directconnect/)

Le Cloud hybride est souvent une **étape transitoire** vers une adoption plus large du Cloud public.

![](formations/aws-initiation-approfondissement/11-images/ch1-capture-06-717544e3.png)

### 4.4 Comparatif synthétique

| Élément | Public Cloud | Private Cloud | Hybrid Cloud |
|--------|--------------|---------------|--------------|
| **Propriété de l'infrastructure** | Fournisseur (ex. AWS) | Entreprise | Mixte |
| **Isolation** | Mutualisée, logique | Dédiée, physique | Mixte |
| **Coûts** | OPEX, à la demande | CAPEX, investissement initial | Mixte |
| **Sécurité** | Standardisée, contrôlée par fournisseur | Complète, contrôlée par l'entreprise | Partagée |
| **Scalabilité** | Très élevée | Limitée | Élevée |
| **Gouvernance** | Fournisseur | Entreprise | Mixte |

### 4.5 Recommandations par contexte

| **Modèle** | **Recommandé pour…** | **Explication simplifiée** |
|-----------|-------|---|
| **Cloud public** | Startups, PME, environnements Dev/Test, charges variables | AWS gère toute l'infrastructure. Vous utilisez les services à la demande, sans rien installer ni maintenir. Idéal pour démarrer vite, tester, ou adapter les ressources selon le trafic. |
| **Cloud privé** | Organisations réglementées, données sensibles, applications critiques | L'environnement est dédié à une seule entreprise. Plus de contrôle, plus de sécurité, mais aussi plus de configuration. Utilisé quand les données ne doivent pas sortir d'un périmètre défini (ex : santé, finance). |
| **Cloud hybride** | Entreprises en transition, contraintes réglementaires, intégration avec systèmes existants | Combine les deux : une partie des ressources reste sur site ou en cloud privé, l'autre est dans le cloud public. Permet de migrer progressivement, tout en respectant les contraintes métier ou techniques. |

AWS permet aux entreprises de choisir le **modèle le plus adapté à leur contexte** :

- Le **cloud public** est parfait pour démarrer rapidement, tester des idées, ou gérer des pics de trafic.
- Le **cloud privé** offre un **contrôle maximal** pour les secteurs sensibles ou réglementés.
- Le **cloud hybride** est une **solution intermédiaire** qui permet de **connecter l'ancien et le nouveau**, en douceur.

Cette flexibilité est l'un des grands atouts d'AWS : vous pouvez commencer petit, évoluer progressivement, et adapter votre architecture à vos besoins métier, techniques et réglementaires.

---

## 5. Fondamentaux techniques : Virtualisation et Conteneurs

Derrière le Cloud se cachent des **technologies fondamentales** qui le rendent possible.
Deux piliers en particulier ont permis l'essor massif des infrastructures à la demande :

- La **virtualisation**, qui permet de découper une machine physique en plusieurs machines virtuelles indépendantes.
- La **conteneurisation**, qui permet d'isoler et d'exécuter des applications de manière légère et flexible.

Comprendre ces concepts est essentiel pour appréhender le fonctionnement d'**Amazon Web Services (AWS)** et de nombreux autres fournisseurs Cloud.

📎 [AWS Compute Documentation](https://docs.aws.amazon.com/compute/)
📎 [Introduction to Virtualization — VMware](https://www.vmware.com/topics/glossary/content/virtualization.html)
📎 [Docker Documentation](https://docs.docker.com/get-started/)

Insistons ici sur la **différence entre virtualisation et conteneurisation**, car cette distinction conditionne la compréhension des services comme Amazon EC2, ECS ou EKS.

### 5.1 Virtualisation — Principe et rôle dans le Cloud

#### Définition

La **virtualisation** est une technologie qui permet d'exécuter plusieurs **machines virtuelles (VM)** sur un seul serveur physique.
Chaque VM fonctionne comme une machine indépendante avec son propre système d'exploitation et ses ressources allouées (CPU, mémoire, stockage, réseau).

La virtualisation est gérée par une couche logicielle appelée **hyperviseur**.

#### Types d'hyperviseurs

- **Type 1 (bare-metal)** : installé directement sur le matériel physique. Exemples : VMware ESXi, Microsoft Hyper-V, KVM.
- **Type 2 (hosted)** : installé au-dessus d'un système d'exploitation. Exemples : VirtualBox, VMware Workstation.

#### Avantages de la virtualisation :

- Meilleure utilisation des ressources matérielles.
- Isolation entre les environnements.
- Déploiement rapide.
- Maintenance simplifiée.

#### Virtualisation et AWS

Sur AWS, la virtualisation est au cœur de nombreux services, en particulier :

- **Amazon EC2 (Elastic Compute Cloud)** : service permettant de créer et gérer des machines virtuelles dans le Cloud AWS.
- **Amazon EBS (Elastic Block Store)** : service de stockage en mode bloc utilisé par les instances EC2.
- **Amazon VPC (Virtual Private Cloud)** : service qui fournit un réseau virtuel isolé dans AWS pour héberger les ressources.

### 5.2 Paravirtualisation et Émulation

Deux modes techniques sont utilisés dans la virtualisation Cloud :

- **Émulation** : le matériel est simulé intégralement par logiciel. Plus flexible mais moins performant.
- **Paravirtualisation** : le système invité est conscient qu'il est virtualisé et interagit directement avec l'hyperviseur, ce qui offre de meilleures performances.

AWS utilise principalement des technologies de **paravirtualisation** et de **virtualisation matérielle assistée** pour offrir des performances quasi natives.

📎 [AWS Nitro System](https://aws.amazon.com/ec2/nitro/) — La plateforme AWS Nitro est une combinaison de matériel dédié et de logiciels légers qui améliore la sécurité et les performances de la virtualisation EC2.

### 5.3 Conteneurisation — Une approche plus légère

#### Définition

La **conteneurisation** est une méthode d'isolation applicative qui permet d'exécuter plusieurs applications sur le même système d'exploitation, dans des environnements isolés appelés **conteneurs**.

Contrairement à une machine virtuelle, un conteneur ne contient pas son propre OS complet — il partage le noyau de l'hôte, ce qui le rend **beaucoup plus léger**.

#### Avantages des conteneurs :

- Démarrage quasi instantané.
- Consommation réduite de ressources.
- Facilité de portabilité et de déploiement.
- Standardisation des environnements.

#### Outils et services associés

- **Docker** : technologie de conteneurisation la plus répandue, permettant de créer, packager et exécuter des conteneurs.
- **Amazon ECS (Elastic Container Service)** : service AWS permettant de déployer et gérer des conteneurs Docker à grande échelle.
- **Amazon EKS (Elastic Kubernetes Service)** : service AWS qui fournit une plateforme Kubernetes managée pour orchestrer des conteneurs.

📎 [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)
📎 [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)

Il est important de comprendre la différence clé — VM = OS complet, Conteneur = application isolée. C'est une **rupture technologique** qui a permis l'essor de l'architecture microservices.

### 5.3 Microservices — Architecture distribuée et scalable

Les **microservices** sont une approche architecturale qui consiste à **découper une application monolithique en petits services indépendants**, chacun responsable d'une fonction métier spécifique.

**Exemple :** Une application de e-commerce ne sera plus UNE grosse application, mais plusieurs petits services :
- Service d'authentification
- Service de gestion de panier
- Service de paiement
- Service de recommandations
- Service de livraison

#### Avantages des microservices

Le monolithe (une application unique de 500k lignes, une seule technologie, un déploiement lent et une scalabilité limitée) s'oppose aux microservices (Auth, Cart, Payment, Reco découpés en services indépendants), qui autorisent des technologies différentes par service, un déploiement rapide et une scalabilité flexible par équipe.

- **Agilité** : chaque équipe développe son service indépendamment.
- **Scalabilité** : seul le service de paiement reçoit beaucoup de trafic ? Scalez juste ce service.
- **Résilience** : si le service de recommandations est down, le reste de l'app fonctionne.
- **Technologie** : chaque service peut utiliser Node.js, Python, Java… selon le besoin.

#### Microservices et conteneurs

Les microservices **sans conteneurs** seraient très compliqués à gérer. C'est pourquoi **Docker + Kubernetes** (ou **AWS ECS**) sont devenus essentiels.

Chaque microservice est empaqueté dans son propre **conteneur Docker**, puis orchestré par un outil qui gère les déploiements, la scalabilité et la résilience.

**Services AWS pour microservices :**

- **Amazon ECS (Elastic Container Service)** : orchestration native AWS, plus simple que Kubernetes.
- **Amazon EKS (Elastic Kubernetes Service)** : Kubernetes managé, si vous avez une flotte massive.
- **AWS Fargate** : exécution serverless de conteneurs (sans gérer les instances EC2).
- **AWS Lambda** : exécution de fonctions sans serveur (cas de microservices très légers).

#### Un exemple en image

<img src="formations/aws-initiation-approfondissement/11-images/aws-microservices-ecs-fargate.svg"
     alt="Architecture microservices : CloudFront vers ALB, distribué vers Auth/Cart/Payment/Recomend puis vers RDS/DynamoDB/S3"
     style="display:block; margin:auto; width:90%">

📎 [Microservices Documentation AWS](https://aws.amazon.com/microservices/)

### 5.4 Virtualisation vs conteneurisation — tableau comparatif

| Élément | Virtualisation (VM) | Conteneurisation |
|--------|--------|---------|
| **Système** | OS complet par machine virtuelle | Noyau partagé, application isolée |
| **Démarrage** | Lent (minutes) | Rapide (secondes) |
| **Consommation de ressources** | Élevée | Faible |
| **Portabilité** | Moins flexible | Très flexible |
| **Cas d'usage typiques** | Migration d'applications legacy, serveurs complets | Microservices, CI/CD, déploiements rapides |

Ce tableau permet de bien fixer les différences entre VM et conteneurs.

---

## 6. Présentation d'AWS et de son écosystème

### 6.1 Vue d'ensemble

**Amazon Web Services (AWS)** est aujourd'hui le leader mondial du Cloud public.
Lancé en 2006, AWS est une filiale d'Amazon qui propose une **plateforme Cloud complète** couvrant :

- L'infrastructure (serveurs, stockage, réseau),
- Les plateformes de développement et déploiement applicatif,
- Les services managés et serverless,
- L'intelligence artificielle, la data et l'IoT.

AWS est présent dans la quasi-totalité des secteurs d'activité : industrie, banque, administration publique, éducation, santé, etc.

📎 [Site officiel AWS](https://aws.amazon.com/)
📎 [AWS Overview Documentation](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html)

### 6.2 Les applications dans les offres Cloud

AWS n'est pas seulement une plateforme d'infrastructure : c'est aussi un **écosystème d'applications et de solutions métiers**.

Quelques exemples typiques :
- **Hébergement web et API** : via Amazon EC2, Elastic Beanstalk ou Lightsail.
- **Stockage et sauvegarde** : avec S3, EBS et Glacier.
- **Analyse de données** : grâce à Redshift et Athena.
- **Sécurité et conformité** : via IAM, CloudTrail et Security Hub.

Ces briques sont interconnectées.
Une **application complète AWS** s'appuie généralement sur plusieurs de ces services, orchestrés ensemble pour offrir disponibilité, sécurité et performance.

### 6.3 Les principaux acteurs du marché du Cloud

Le Cloud Computing est un marché concurrentiel dominé par **trois géants**, avec quelques acteurs spécialisés ou souverains :

#### Panorama comparatif simplifié

Avant de décortiquer AWS, il est utile de comprendre son positionnement face à ses concurrents principaux.

| Fournisseur | Origine | Points forts | Contexte d'utilisation | Position sur le marché |
|-------------|---------|-------------|------------------------|----------------------|
| **AWS** | Amazon (2006) | Large catalogue (200+ services), maturité, écosystème, documentation, communauté | Startups, PME, grandes entreprises, multinationales | Leader (~32 % du marché) |
| **Microsoft Azure** | Microsoft (2010) | Intégration native Windows/Office 365/AD, modèle hybride fort, support entreprise | Organisations Microsoft existantes, conformité européenne | 2ᵉ position (~23 %) |
| **Google Cloud Platform (GCP)** | Google (2008) | Outils data/ML excellents, BigQuery, Vertex AI, simplicité d'usage | Projets data-driven, IA/ML, startups tech | 3ᵉ position (~10 %) |
| **OpenStack** (Open Source) | Communauté | Infrastructure privée, totale maîtrise, coût réduit | Cloud privé, institutions académiques, hébergeurs | Marché de niche |
| **OVHcloud** (Européen) | OVH (France) | Souveraineté données EU, prix compétitifs | PME EU, données sensibles, RGPD stricte | Marché de niche EU |
| **IBM Cloud, Oracle Cloud** | IBM / Oracle | Spécialisés (IA, bases données massives) | Grandes entreprises, solutions legacy | Marché de niche |

#### Comparatif détaillé : AWS vs Azure vs GCP

| Aspect | AWS | Azure | GCP |
|--------|-----|-------|-----|
| **Nombre de services** | 200+ | 200+ | 100+ |
| **Services de calcul** | EC2, Lambda, Fargate | VMs, App Services, AKS | Compute Engine, Cloud Functions |
| **Bases de données** | RDS, DynamoDB, Aurora | SQL Database, Cosmos DB | Cloud SQL, Firestore |
| **Données et IA** | SageMaker, Glue | Azure ML, Synapse | BigQuery, Vertex AI ⭐ |
| **Points de présence** | 30+ régions | 60+ régions | 40+ régions |
| **Matériel propriétaire** | AWS Nitro | Optimisé | Processeurs TPU (IA) |
| **Force communautaire** | Très forte | Forte (corporations) | Croissante (startups) |
| **Apprentissage** | Longue courbe | Intégré MS, plus facile | Pythoniste-friendly |
| **Certifications** | Cloud Practitioner, CloudOps Engineer, Solutions Architect | AZ-900, AZ-104 | Cloud Digital Leader, Associate Cloud Engineer, Professional |

**Ce que signifient les lignes moins évidentes de ce tableau :**

- **Matériel propriétaire** : chaque fournisseur a développé du matériel sur mesure pour ses data centers plutôt que d'utiliser des composants standards du marché. **AWS Nitro** (déjà présenté en section 5.2 de ce chapitre) est un système matériel et logiciel conçu par AWS qui décharge la virtualisation, le réseau et la sécurité du CPU principal du serveur physique — cela signifie que quasiment 100 % de la puissance de la machine physique est disponible pour vos instances EC2, au lieu qu'une partie soit consommée par l'hyperviseur lui-même. Chez GCP, les **TPU (Tensor Processing Units)** sont des puces conçues spécifiquement pour accélérer les calculs de machine learning (entraînement et inférence de modèles d'IA) — un TPU va beaucoup plus vite qu'un CPU classique sur ce type de calcul précis, mais ne sert à rien pour un usage généraliste.
- **Force communautaire** : désigne le volume et l'activité de la communauté d'utilisateurs autour du fournisseur — tutoriels, forums (Stack Overflow, Reddit), projets open source, meetups. Plus cette communauté est active, plus il est facile de trouver de l'aide en cas de blocage. AWS bénéficie ici de son avance historique (premier arrivé en 2006) : davantage de contenus accumulés sur près de 20 ans.
- **Apprentissage** : la "courbe d'apprentissage" désigne la difficulté et le temps nécessaires pour devenir opérationnel sur la plateforme. AWS a la réputation d'avoir une courbe plus longue du fait de son catalogue de 200+ services parfois redondants (plusieurs façons de faire la même chose). Azure est souvent perçue comme plus accessible pour des équipes déjà habituées à l'écosystème Microsoft (Windows Server, Active Directory, Office 365) — les concepts et l'interface leur sont familiers. "**Pythoniste-friendly**" signifie que GCP a orienté beaucoup de ses outils et de sa documentation autour de Python, le langage dominant en data science — un développeur Python s'y retrouve plus vite, notamment pour les services data/IA (BigQuery, Vertex AI).
- **Certifications** : chaque fournisseur a son propre parcours de certification, avec des noms et des codes différents mais un principe similaire (un premier niveau généraliste, puis des spécialisations). Côté Azure, **AZ-900** est la certification fondamentale (équivalent du AWS Cloud Practitioner) et **AZ-104** la certification Administrator Associate (gestion quotidienne des ressources Azure, équivalent du AWS SysOps). Les certifications AWS sont détaillées à la section 1.3 de ce chapitre.

#### AWS, pourquoi c'est le leader ?

**Les trois raisons principales :**

1. **First-mover advantage** : AWS a lancé la révolution du cloud en 2006. Les équipes AWS ont maturé progressivement, corrigé les bugs, collecté les retours.
2. **Catalogue massif** : Plus de 200 services interconnectés permettent de construire à peu près n'importe quelle architecture.
3. **Écosystème** : Partenaires, formations, documentation, communauté, marketplace. Plus vous apprenez AWS, moins vous avez intérêt à partir.

C'est pourquoi AWS représente environ **32 % du marché cloud**, loin devant Azure (23 %) et GCP (10 %).

#### Solutions open source

Enfin, une mention importante : **OpenStack**.

OpenStack est une plateforme **open source complète** qui permet à toute organisation de **construire son propre cloud privé**, sans dépendre d'un fournisseur unique. Elle est utilisée par :
- Universités et instituts de recherche
- Gouvernements (souveraineté)
- Hébergeurs et fournisseurs de services cloud alternatifs
- Organisations ayant besoin de total contrôle

OpenStack ne joue pas sur le même terrain qu'AWS/Azure/GCP (qui sont des services managés) — c'est une brique technologique que vous installez et maintenez vous-même.

📎 [Gartner Magic Quadrant 2024 – Cloud Infrastructure and Platform Services](https://www.gartner.com/en/documents)

### 6.4 L'infrastructure mondiale d'AWS

Le schéma suivant part d'une requête utilisateur et montre pourquoi une région ne suffit pas, à elle seule, à garantir la disponibilité. Le trafic doit être distribué vers plusieurs zones indépendantes et les ressources zonales doivent réellement être dupliquées.

![Distribution animée d'une charge entre les trois zones de disponibilité de la région AWS Paris](formations/aws-initiation-approfondissement/11-images/aws-global-infrastructure-animated.svg)

Les trois AZ appartiennent à la même région et communiquent par le réseau régional AWS, mais constituent des domaines de défaillance distincts. Déployer une instance dans `eu-west-3a` et une base uniquement dans cette même AZ reste une architecture mono-AZ, même si la région en propose trois.

L'un des piliers majeurs d'AWS est son **infrastructure mondiale** extrêmement étendue et redondée.
Elle est structurée en trois grandes composantes :

#### 1. Régions AWS (Regions)

- Une *région* est une zone géographique physique (exemple : `eu-west-3` pour Paris).
- Chaque région contient plusieurs datacenters appelés Zones de disponibilité (AZ).
- Les services AWS sont déployés par région et choisis par l'utilisateur.

#### 2. Zones de disponibilité (Availability Zones - AZ)

- Une AZ correspond à un ou plusieurs datacenters indépendants reliés entre eux par des liens à très faible latence.
- Les AZ sont conçues pour **garantir la haute disponibilité** et **la tolérance aux pannes**.

#### 3. Points de présence (Edge Locations)

- Répartis dans le monde entier, ils permettent d'accélérer la diffusion de contenu et d'optimiser les performances réseau via des services comme **Amazon CloudFront** (réseau de diffusion de contenu — CDN).

#### Vidéo explicative : infrastructure AWS

<iframe width="560" height="315" src="https://www.youtube.com/embed/TFxSjn8AHi8?si=jCQPMlT4-X9HT7Ip" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Retenons que le concept de *région* et de *localisation géographique des données* est souvent déterminant pour des raisons légales et de performance.

### 6.5 Principes de tarification AWS

L'un des aspects stratégiques de l'adoption du Cloud est la **tarification à l'usage**.
AWS applique un modèle de facturation souple et transparent basé sur la consommation réelle.

#### Principes clés :

- **Pay-as-you-go** : paiement en fonction de l'utilisation réelle des ressources.
- **Sans engagement initial** : pas de coût d'entrée, contrairement au modèle CAPEX.
- **Économies d'échelle** : les tarifs peuvent diminuer avec l'usage.
- **Facturation par service** : chaque service (EC2, S3, RDS…) est facturé séparément selon ses métriques propres.

Selon le service et l'engagement pris, plusieurs modes de tarification existent — à la demande, instances réservées, Savings Plans, Spot Instances. Le Chapitre 3 détaille chacun de ces modes avec des cas métier chiffrés, au moment où vous configurerez vos premières instances EC2.

📎 [AWS Pricing Overview](https://aws.amazon.com/pricing/)
📎 [AWS Pricing Calculator](https://calculator.aws/)

---

## 7. AWS Well-Architected Framework

Le **Well-Architected Framework** est une méthodologie officielle AWS qui permet de **concevoir des architectures Cloud robustes**, sécurisées, performantes, optimisées en coûts et **durables**.

C'est une **philosophie de design** qui s'applique à chaque projet AWS, du plus petit au plus grand.

Ce framework repose sur **six piliers**, souvent représentés par le sigle **SOPREC** :

- 🏛️ **Security** → Sécurité, IAM, chiffrement
- ⚙️ **Operational Excellence** → Monitoring, automatisation, processus
- 💪 **Reliability** → Résilience, haute disponibilité
- ⚡ **Performance** → Efficacité des ressources
- 💰 **Cost Optimization** → Optimisation des dépenses
- 🌱 **Sustainability** → Impact environnemental, efficacité

> [!TIP]
> **Vérification AWS CLI — liste des piliers WAF disponibles pour une review :**
> ```bash
> aws wellarchitected list-lenses --lens-type AWS_OFFICIAL
> ```
> ```json
> {
>     "LensSummaries": [
>         { "LensAlias": "wellarchitected", "LensName": "AWS Well-Architected Framework",
>           "LensVersion": "2023-04-10", "Description": "Six pillars review" },
>         { "LensAlias": "serverless", "LensName": "Serverless Lens", "LensVersion": "3.0" },
>         { "LensAlias": "saas", "LensName": "SaaS Lens", "LensVersion": "1.0" }
>     ]
> }
> ```
> Le Framework Well-Architected est disponible depuis la console et via API. Vous pouvez lancer une revue avec **AWS Well-Architected Tool** : les questions sont organisées selon les six piliers. Ce service ne doit pas être confondu avec **AWS WAF**, le pare-feu applicatif web.
#### 1. Security (Sécurité)

Mettre en œuvre des mesures de protection fortes à tous les niveaux — **IAM**, **chiffrement**, **sécurité réseau**, **MFA**, **audit**.

**En pratique :**
- Créer des rôles IAM spécifiques plutôt que d'utiliser le compte root.
- Chiffrer les données au repos et en transit.
- Utiliser des security groups et NACL pour isoler les ressources.
- Activer CloudTrail pour auditer les accès.

#### 2. Operational Excellence (Excellence opérationnelle)

Surveiller et améliorer continuellement les opérations — **mise en place de processus**, **d'automatisation** et de **monitoring**.

**En pratique :**
- Utiliser CloudWatch pour surveiller les ressources.
- Automatiser les déploiements avec CloudFormation ou Terraform.
- Documenter les procédures d'incident.
- Faire des reviews régulières de l'architecture.

#### 3. Reliability (Fiabilité / Résilience)

Assurer la **résilience** et la **tolérance aux pannes** — **redondance**, **sauvegarde**, **failover automatique**.

**En pratique :**
- Déployer les applications dans plusieurs **Availability Zones** pour la haute disponibilité.
- Mettre en place des sauvegardes automatiques (EBS snapshots, RDS backups).
- Utiliser des auto-scaling groups pour ajuster le nombre d'instances en cas de charge.
- Tester régulièrement les scénarios de récupération d'urgence (RTO/RPO).

#### 4. Performance Efficiency (Efficacité des performances)

Utiliser les ressources de manière **efficace et évolutive** — **choix des bonnes instances**, **optimisation des requêtes**.

**En pratique :**
- Choisir le bon type d'instance EC2 selon le workload (t3 pour web, m5 pour applications, c5 pour calcul).
- Utiliser des caches (ElastiCache, CloudFront) pour réduire la latence.
- Optimiser les requêtes de bases de données.
- Utiliser AWS Compute Optimizer pour recommander les bonnes tailles.

#### 5. Cost Optimization (Optimisation des coûts)

Éviter les dépenses inutiles et **ajuster la capacité à la demande** — **monitoring des coûts**, **réservation d'instances**, **suppression des ressources non utilisées**.

**En pratique :**
- Utiliser des **Reserved Instances** ou **Savings Plans** pour réduire les coûts.
- Mettre en place des **budgets d'alerte** dans AWS Billing.
- Arrêter les ressources non utilisées (instances EC2, RDS développement).
- Analyser les logs de coûts avec Cost Explorer ou AWS Cost Anomaly Detection.

#### 6. Sustainability (Durabilité) — **Le pilier le plus récent** ✨

Minimiser l'**impact environnemental** de votre infrastructure Cloud — **efficacité énergétique**, **utilisation optimale des ressources**, **choix des régions**.

**En pratique :**
- Utiliser les régions AWS qui utilisent des énergies renouvelables (ex : `eu-west-1` en Irlande, alimentée à >80% par des renouvelables).
- Éteindre les ressources inutilisées plutôt que de les laisser tourner.
- Utiliser le **Compute Optimizer** pour ajuster les tailles d'instances et réduire la consommation.
- Préférer les architectures **serverless** (Lambda, Fargate) qui consomment moins que les EC2 permanentes.
- Monitorer l'empreinte carbone de vos déploiements via le **AWS Customer Carbon Footprint Tool**.

C'est un enjeu de plus en plus important pour les entreprises souhaitant atteindre leurs **objectifs de développement durable (ODD)** et de **net-zéro** (carbon-neutral).

📎 [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
📎 [AWS Sustainability — Carbon Footprint Tool](https://aws.amazon.com/about-aws/sustainability/)
#### Appliquer le Framework au quotidien

Lors de **chaque conception d'architecture AWS**, on doit se poser ces questions :

- **🔒 Sécurité** : Les données sont-elles protégées ? IAM bien configuré ?
- **⚙️ Opérations** : Comment monitorer et alerter ? Comment déployer sans risque ?
- **💪 Résilience** : Que se passe-t-il si un service tombe ? Avons-nous des sauvegardes ?
- **⚡ Performance** : Les ressources sont-elles bien dimensionnées ? Y a-t-il des goulots ?
- **💰 Coûts** : Payons-nous ce que nous utilisons réellement ? Pouvons-nous optimiser ?
- **🌱 Durabilité** : Quel est notre impact environnemental ? Pouvons-nous le réduire ?

Cette approche sera **reprise au fil des jours**, notamment lors des ateliers EC2, S3, RDS et VPC.

---

## 8. AWS Management Console

La **console AWS** est une interface Web centralisée qui permet de gérer tous les services AWS depuis un seul endroit.

### 8.1 Principales fonctionnalités

La console web regroupe en un seul endroit tout ce dont un administrateur a besoin pour piloter son compte AWS au quotidien, et c'est justement ce qui la rend accessible aux débutants malgré la richesse du catalogue de services.

Elle donne d'abord accès à l'ensemble des services AWS disponibles dans la région sélectionnée : EC2, S3, RDS, Lambda et les centaines d'autres services se retrouvent derrière une barre de recherche unique, sans avoir à mémoriser des noms de commandes ou des endpoints d'API. Elle permet ensuite de visualiser en temps réel l'état des ressources déjà créées — combien d'instances EC2 tournent, quels buckets S3 existent, comment est configuré le VPC — ce qui en fait le point d'entrée naturel pour diagnostiquer un problème avant de creuser en ligne de commande. Elle sert aussi de porte d'entrée vers **AWS IAM (Identity and Access Management)**, où se gèrent les comptes, les utilisateurs et leurs permissions : c'est depuis la console que l'on crée les premiers utilisateurs IAM et qu'on leur attribue des droits, avant même de savoir écrire une policy JSON. Enfin, elle donne un accès direct aux métriques d'usage et à la facturation, ce qui permet de surveiller les coûts sans quitter l'interface — un réflexe indispensable pour éviter les mauvaises surprises en fin de mois.

### 8.2 Structure de la console

| Zone | Rôle |
|------|------|
| **Barre de recherche** | Trouver rapidement un service (ex : EC2, S3, IAM) |
| **Panneau de navigation** | Accéder aux sections : Services, Ressources, Facturation |
| **Région** | Indique la localisation actuelle (ex : Paris `eu-west-3`) |
| **Compte utilisateur** | Informations, factures, connexions IAM |
| **Notifications** | Alertes et mises à jour |
| **Support** | Documentation, centre d'aide, tickets AWS |

📎 [AWS Console](https://aws.amazon.com/console/)
📎 [Guide de l'interface AWS Console](https://docs.aws.amazon.com/awsconsole)

> [!TIP]
> **Vérification AWS CLI — lister les régions disponibles :**
> ```bash
> aws ec2 describe-regions --output table
> ```
> ```
> -------------------------------------------------------
> |                    DescribeRegions                  |
> +-------------------+---------------------------------+
> |   RegionName      |   Endpoint                      |
> +-------------------+---------------------------------+
> |  eu-west-3        |  ec2.eu-west-3.amazonaws.com   |  ← Paris
> |  eu-west-1        |  ec2.eu-west-1.amazonaws.com   |  ← Irlande
> |  eu-central-1     |  ec2.eu-central-1.amazonaws.com|  ← Francfort
> |  us-east-1        |  ec2.us-east-1.amazonaws.com   |  ← Virginie du Nord
> |  ap-southeast-1   |  ec2.ap-southeast-1.amazonaws.com| ← Singapour
> +-------------------+---------------------------------+
> ```
> La région `eu-west-3` (Paris) est votre région de travail par défaut pour cette formation. Vérifiez toujours que vous êtes bien dans la bonne région avant de créer une ressource — une instance EC2 créée dans `us-east-1` par erreur sera difficile à retrouver.
Rappelons que la console est un moyen parmi d'autres d'interagir avec AWS — il existe également des API, des SDK et la **CLI AWS** pour les automatisations.

### 8.3 Bonnes pratiques de base

- Toujours vérifier la **région active** avant de créer des ressources.
- Fermer les ressources non utilisées pour éviter des coûts inutiles.
- Utiliser **IAM** pour créer des comptes utilisateurs distincts plutôt que d'utiliser le compte root — rappel : voir l'encart sur le compte root en section 2.6 de ce chapitre.
- Activer la **MFA (Multi-Factor Authentication)** pour sécuriser les accès.
- Surveiller les coûts dans **Billing & Cost Management**.

📎 [Best Practices AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

### 8.4 Gestion du budget et des coûts AWS

L'un des **risques majeurs** du cloud est la **perte de contrôle des coûts**. Un bucket S3 public par erreur, une instance EC2 oubliée, un transfert de données non optimisé — et la facture peut exploser en quelques jours.

AWS fournit des outils puissants pour **monitorer, budgéter et optimiser** les coûts.

### Principes clés de la tarification AWS

1. **Pay-as-you-go** : payer ce qu'on utilise réellement.
2. **Granularité** : chaque service a ses propres métriques — EC2 facturé par heure (ou seconde), S3 par Go stocké + requêtes, RDS par heure + stockage + transfert.
3. **Réductions possibles** : Reserved Instances (engagement 1 ou 3 ans), Savings Plans (engagement flexible), Spot Instances (capacité excédentaire à -70%).
4. **AWS Free Tier** : pour les comptes créés depuis le 15 juillet 2025, AWS propose un système de crédits — 100 USD à l'inscription et jusqu'à 100 USD supplémentaires via des activités — ainsi qu'un plan gratuit limité à six mois. Les comptes plus anciens restent soumis au régime historique, qui comportait notamment certaines offres gratuites pendant douze mois. Dans tous les cas, un service hors périmètre, un dépassement de crédit ou un dépassement de quota est facturé au tarif normal.

> [!TIP]
> **Exemple de commande AWS CLI pour vérifier votre consommation du Free Tier :**
> ```bash
> aws ce get-cost-and-usage \
>   --time-period Start=2024-01-01,End=2024-01-31 \
>   --granularity MONTHLY \
>   --metrics BlendedCost \
>   --group-by Type=DIMENSION,Key=SERVICE
> ```
> ```json
> {
>     "ResultsByTime": [{
>         "TimePeriod": { "Start": "2024-01-01", "End": "2024-01-31" },
>         "Groups": [
>             { "Keys": ["Amazon EC2"], "Metrics": { "BlendedCost": { "Amount": "0.00", "Unit": "USD" } } },
>             { "Keys": ["Amazon S3"], "Metrics": { "BlendedCost": { "Amount": "0.23", "Unit": "USD" } } },
>             { "Keys": ["Amazon RDS"], "Metrics": { "BlendedCost": { "Amount": "0.00", "Unit": "USD" } } }
>         ]
>     }]
> }
> ```
> Dans cet exemple fictif, les crédits promotionnels absorbent encore la consommation EC2 et RDS. S3 affiche un coût car l'usage concerné n'est plus entièrement couvert. Le résultat réel dépend de la date de création du compte, du plan choisi, du solde de crédits, de la région et des services utilisés : il faut toujours vérifier la page **Free Tier** et la facturation du compte actif.
### Outils de monitoring des coûts

#### AWS Billing Dashboard

Accessible depuis la console AWS, le **Billing Dashboard** affiche :
- Facture du mois en cours
- Prévisions de coûts (estimé) jusqu'à fin du mois
- Services coûtant le plus

C'est un premier coup d'œil rapide, mais peu détaillé.

#### AWS Cost Explorer

**Cost Explorer** permet une **analyse fine des dépenses** :
- Filtrer par service (EC2, S3, RDS…)
- Filtrer par période (jour, mois, année)
- Filtrer par région
- Voir l'historique et les tendances

**Cas d'usage :** « Pourquoi ma facture a-t-elle explosé en janvier ? Quel service en est responsable ? »

#### AWS Budgets

**AWS Budgets** permet de **définir des seuils d'alerte** :
- Budget mensuel maximum (ex : 500 €/mois)
- Alertes si dépassement imminent
- Alertes si usage atypique détecté

**Cas d'usage :** Recevoir une notification avant d'exploser le budget plutôt qu'une mauvaise surprise à la fin du mois.

#### AWS Cost Anomaly Detection

Utilise l'**apprentissage automatique** pour détecter les **anomalies de dépenses** :
- Si vous dépensez soudain 10x plus que normal dans une région, alerter.
- Si un service coûte étrangement cher, signaler.

C'est très utile pour détecter des ressources oubliées ou des fuites de données.

#### AWS Pricing Calculator

**AWS Pricing Calculator** permet d'**estimer les coûts** avant de déployer :
- Configurez votre architecture (1 instance EC2 t3.micro, 100 Go S3…)
- Voir le coût mensuel/annuel
- Comparer différentes configurations

C'est idéal pour les **POC (Proof of Concept)** et les **RFQ** (appels d'offre).

### Bonnes pratiques pour maîtriser les coûts

| Pratique | Description | Impact |
|----------|-------------|--------|
| **Utiliser des Reserved Instances** | Engagement 1 ou 3 ans pour réduction tarifaire (~30-60%) | Très utile pour charges stables 24/7 |
| **Utiliser Spot Instances** | Capacité excédentaire à -70% (mais peut être interrompue) | Idéal pour tâches batch, tests |
| **Éteindre les ressources non utilisées** | Arrêter EC2 dev/test en fin de journée, supprimer RDS inutilisés | Économies rapides et simples |
| **Auto-scaling** | Adapter le nombre d'instances au trafic | Évite surprovisionnement |
| **Optimiser le stockage** | S3 Standard pour accès fréquent, S3 Glacier pour archive | Différences tarifaires importantes |
| **Utiliser des réductions volantes** | Savings Plans, commitments, consolidated billing | ~30-40% de réduction possible |
| **Monitorer régulièrement** | Cost Explorer, Budgets, anomaly detection | Détection précoce des dérives |

### Exemple concret : budget mal maîtrisé

Imaginons une startup qui lance une application web sur AWS.

**Scénario optimiste :**
- Petite instance EC2 dont la consommation est couverte par le solde de crédits du compte de démonstration
- 50 Go S3 (~1 € / mois)
- **Coût mensuel : ~1 €**

**Scénario problématique (sans monitoring) :**
- Instance EC2 mal fermée le week-end : +50 €
- Bucket S3 avec version history excessive : +100 €
- Transfert de données cross-région oublié : +200 €
- **Coût mensuel : ~350 € !**

**Avec les outils AWS Budgets :**
- Budget défini : 50 €/mois
- Alerte à 40 € : Action rapide
- Découverte de l'instance oubliée → suppression
- Découverte des versions excessives → nettoyage
- **Coût final : 5 €/mois**

> [!TIP]
> **Résultat attendu — alerte AWS Budgets reçue par email :**
> ```
> De : no-reply@notifications.aws
> Objet : [AWS Budgets] Alerte — Mon Budget Mensuel (80% atteint)
>
> Bonjour,
>
> Votre budget "Mon Budget Mensuel" a atteint 80% de votre seuil d'alerte.
>
> Budget : 50,00 USD
> Dépensé : 40,23 USD
> Prévu ce mois : 52,31 USD
>
> Service le plus coûteux : Amazon EC2 (28,50 USD)
> Recommandation : vérifiez les instances EC2 actives dans toutes les régions.
>
> → Accéder au Cost Explorer : https://console.aws.amazon.com/cost-management/
> ```
> Grâce à cette alerte précoce, vous pouvez arrêter l'instance oubliée avant que la facture n'explose. Sans ce budget configuré, vous n'auriez découvert le problème qu'à la réception de la facture mensuelle.
C'est pourquoi **mettre en place un monitoring budgétaire dès le départ est essentiel**.

📎 [AWS Billing Documentation](https://docs.aws.amazon.com/billing/)
📎 [AWS Cost Explorer Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
📎 [AWS Cost Optimization Best Practices](https://aws.amazon.com/aws-cost-management/cost-optimization/)

---

## 9. Les services AWS les plus utilisés

AWS (Amazon Web Services) propose **plus de 200 services complets** couvrant le calcul, le stockage, le réseau, les bases de données, l'IA, la sécurité et bien plus encore.
Sa philosophie est simple : offrir à chaque entreprise, quelle que soit sa taille, **la même puissance technologique** qu'Amazon utilise pour ses propres opérations mondiales.

| Domaine | Service | Explication | Cas d'usage | Analogie |
|---------|---------|-------------|-----------|----------|
| **Calcul** | EC2 | Machines virtuelles configurables (CPU, RAM, OS) | Hébergement web, backend, batch | Louer un serveur dans le cloud |
| | Lambda | Exécution de code sans serveur, déclenchée par événements | Microservices, automatisation, API | Interrupteur intelligent : déclenche une action sans infrastructure |
| **Stockage** | S3 | Stockage d'objets scalable et durable | Sauvegardes, fichiers, hébergement statique | Entrepôt numérique : chaque fichier est une boîte |
| | EBS | Stockage bloc attaché à EC2, persistant | Bases de données, stockage système | Disque dur virtuel |
| | EFS | Système de fichiers partagé pour EC2 | Dossiers partagés, applications multi-instances | Dossier réseau partagé |
| **Bases de données** | RDS | Base relationnelle gérée (MySQL, PostgreSQL…) | ERP, CRM, applications transactionnelles | Serveur SQL géré par AWS |
| | DynamoDB | Base NoSQL scalable sans schéma | IoT, mobile, jeux, logs | Carnet de notes sans structure fixe |
| **Réseau** | VPC | Réseau privé isolé avec sous-réseaux et sécurité | Isolation d'environnements, segmentation | Immeuble privé avec étages et portiers |
| | CloudFront | CDN mondial pour accélérer les contenus | Sites web, vidéos, fichiers statiques | Réseau de relais pour livrer plus vite |
| **Sécurité** | IAM | Gestion des identités et des accès | Contrôle des permissions, MFA, rôles | Badge d'accès pour chaque utilisateur |
| | KMS | Gestion des clés de chiffrement | Sécurité des données, RGPD, PCI-DSS | Coffre-fort numérique pour les clés |
| **DevOps** | CloudFormation | Déploiement d'infrastructure via des fichiers YAML/JSON | Automatisation, standardisation | Plan d'architecte pour construire automatiquement |
| | CodePipeline | Orchestration CI/CD pour le déploiement | Intégration continue, tests, livraison | Chaîne de montage automatisée |
| **IA/ML** | SageMaker | Plateforme de machine learning gérée | Modèles prédictifs, classification, NLP | Laboratoire de data science automatisé |
| | Rekognition | Analyse d'images et vidéos par IA | Détection faciale, modération, OCR | Caméra intelligente qui comprend les images |
| **Analyse** | Athena | Requêtes SQL sur des fichiers S3 | Exploration de données, logs, BI | Loupe SQL sur vos fichiers |
| | Redshift | Entrepôt de données pour BI et reporting | Tableaux de bord, KPIs, analytique | Base de données XXL pour l'analyse |

📎 [AWS Products](https://aws.amazon.com/products/)

Nous n'entrons pas ici dans les détails techniques de chaque service — cela sera traité dans les jours suivants (IAM, S3, EC2, RDS, VPC…).

### 9.1 Automatisation et orchestration : Chef, Puppet et OpsWorks

Au-delà des services de base, AWS propose des outils pour **automatiser le déploiement et la gestion de l'infrastructure**.
Ces outils sont essentiels dans une approche **DevOps** et **Infrastructure as Code (IaC)**.

#### AWS OpsWorks — Orchestration managée par AWS

**AWS OpsWorks** était une famille de services AWS destinée au déploiement et à la gestion automatisés d'applications et d'infrastructures.

Il supporte deux moteurs d'automation populaires :
- **Chef** (propriétaire de OpsWorks Chef)
- **Puppet** (partenariat AWS)

> [!WARNING]
> **Toute la famille OpsWorks est désormais en fin de vie et désactivée.** OpsWorks for Chef Automate et OpsWorks for Puppet Enterprise ont été arrêtés le 5 mai 2024 ; OpsWorks Stacks l'a été le 26 mai 2024. Cette section sert uniquement à reconnaître une infrastructure historique et à comprendre les principes de Chef et Puppet. Pour une nouvelle architecture AWS, privilégiez notamment AWS Systems Manager, les images automatisées, les services de conteneurs ou une solution de gestion de configuration encore maintenue.
**Cas d'usage typiques :**
- Provisionner automatiquement des serveurs EC2 avec une stack applicative complète.
- Gérer les configurations à grande échelle sans intervention manuelle.
- Assurer la conformité des serveurs (tous les serveurs web ont la même configuration).
- Automatiser les déploiements continus (CI/CD).

#### Chef — Automation as Code

**Chef** est un outil d'**automatisation de configuration** très populaire. Il fonctionne sur un modèle de **recettes (recipes)** écrites en Ruby qui décrivent **l'état souhaité** d'une machine.

```ruby
# Exemple simplifié d'une recipe Chef
package 'apache2' do
  action :install
end

service 'apache2' do
  action [:enable, :start]
end

template '/var/www/html/index.html' do
  source 'index.html.erb'
end
```

> [!TIP]
> **Résultat attendu — exécution de `chef-client` :**
> ```
> [2024-01-15T09:12:34+00:00] INFO: Starting Chef Infra Client Run
> [2024-01-15T09:12:35+00:00] INFO: Installing package apache2
> [2024-01-15T09:12:42+00:00] INFO: package[apache2] installed version 2.4.57
> [2024-01-15T09:12:43+00:00] INFO: service[apache2] enabled and started
> [2024-01-15T09:12:43+00:00] INFO: template[/var/www/html/index.html] created file
> [2024-01-15T09:12:43+00:00] INFO: Chef Infra Client Run complete in 9.123 seconds.
> ```
> Apache est installé, démarré et la page d'accueil est en place. Si vous relancez `chef-client`, rien ne change — c'est l'idempotence en action.
**Intérêt :** Au lieu de cliquer dans une UI ou d'écrire des scripts bash, vous décrivez le **résultat attendu** (Apache installé, service actif, fichiers à jour).
Chef **idempotent** — si vous exécutez la recipe 10 fois, le résultat sera toujours le même.

📎 [Chef Documentation](https://docs.chef.io/)

#### Puppet — Configuration Management

**Puppet** est un outil comparable à Chef, mais avec une approche **déclarative** différente.
Au lieu de recettes, Puppet utilise un **langage déclaratif** qui décrit l'état souhaité en termes de ressources. Voici le même objectif (Apache installé et actif) exprimé en syntaxe Puppet :

```puppet
# Exemple simplifié de Puppet
package { 'apache2':
  ensure => present,
}

service { 'apache2':
  ensure => running,
  enable => true,
}
```

> [!TIP]
> **Résultat attendu — exécution de `puppet agent --test` :**
> ```
> Info: Caching catalog for node01.example.com
> Info: Applying configuration version '1705312800'
> Notice: /Stage[main]/Main/Package[apache2]/ensure: created
> Notice: /Stage[main]/Main/Service[apache2]/ensure: ensure changed 'stopped' to 'running'
> Notice: Applied catalog in 8.54 seconds
> ```
> Puppet a appliqué l'état souhaité : `apache2` installé et le service actif. Tout est tracé dans les logs Puppet Master.
**Intérêt :** Comme Chef, il permet d'automatiser des déploiements complexes à grande échelle.

📎 [Puppet Documentation](https://puppet.com/docs/)

#### Ansible — Automatisation agentless et modules AWS

**Ansible** est aujourd'hui l'un des outils d'automatisation les plus utilisés dans les environnements Cloud, notamment en **complément de Terraform** pour la gestion de configuration.

Contrairement à Chef ou Puppet, Ansible est **agentless** : il n'installe rien sur les machines cibles — il se connecte via SSH (Linux) ou WinRM (Windows) et exécute les tâches définies dans des **playbooks** YAML. Voici un playbook typique pour configurer un serveur Apache sur une instance EC2 :

```yaml
# Exemple de playbook Ansible — installation Apache sur EC2
---
- name: Configurer un serveur web Apache sur EC2
  hosts: ec2_instances
  become: yes

  tasks:
    - name: Installer Apache
      apt:
        name: apache2
        state: present

    - name: Démarrer et activer le service
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Copier la page d'accueil
      copy:
        src: index.html
        dest: /var/www/html/index.html
```

> [!TIP]
> **Résultat attendu — `ansible-playbook site.yml -i inventory.ini` :**
> ```
> PLAY [Configurer un serveur web Apache sur EC2] ****************************
>
> TASK [Gathering Facts] *****************************************************
> ok: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Installer Apache] ****************************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Démarrer et activer le service] **************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Copier la page d'accueil] ********************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> PLAY RECAP *****************************************************************
> ec2-18-234-56-78.eu-west-3.compute.amazonaws.com : ok=4  changed=3  unreachable=0  failed=0
> ```
> Apache est installé et opérationnel sur l'instance EC2. La ligne `changed=3` confirme que les trois tâches ont apporté des modifications. Si vous relancez le playbook, vous verrez `changed=0` — c'est l'idempotence Ansible.
Ansible propose également une **collection dédiée AWS** (`amazon.aws`) permettant de piloter directement les ressources AWS depuis un playbook :

```yaml
# Exemple : créer une instance EC2 avec le module Ansible AWS
- name: Lancer une instance EC2
  amazon.aws.ec2_instance:
    name: "mon-serveur-web"
    instance_type: t3.micro
    image_id: ami-0c55b159cbfafe1f0
    region: eu-west-3
    key_name: ma-cle-ssh
    security_groups:
      - sg-xxxxxxxxxx
    tags:
      Env: production
      Owner: formation
```

> [!TIP]
> **Résultat attendu — `ansible-playbook create-ec2.yml` :**
> ```
> TASK [Lancer une instance EC2] *********************************************
> changed: [localhost]
>
> PLAY RECAP *****************************************************************
> localhost : ok=1  changed=1  unreachable=0  failed=0
>
> Instance créée : i-0a1b2c3d4e5f67890
> IP publique    : 15.236.142.87
> Région         : eu-west-3
> Statut         : running
> ```
> L'instance EC2 `mon-serveur-web` est lancée en `eu-west-3`. Elle est tagguée `Env: production` et visible dans la console AWS sous EC2 > Instances. Vous pouvez vous y connecter via SSH : `ssh -i ma-cle-ssh.pem ubuntu@15.236.142.87`.
**Intérêt dans un contexte AWS :**
- Complémentaire à **Terraform** : Terraform crée l'infrastructure (VPC, EC2, RDS…), Ansible configure les instances après leur lancement.
- Pas d'agent à installer sur les instances EC2 — idéal pour les environnements éphémères.
- Intégration native avec les services AWS via la collection `amazon.aws`.
- Utilisé dans les pipelines CI/CD avec CodePipeline ou Jenkins.

📎 [Ansible Documentation officielle](https://docs.ansible.com/)
📎 [Collection Ansible AWS (amazon.aws)](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)

#### CloudFormation — Infrastructure as Code AWS-native

Enfin, il existe **AWS CloudFormation**, l'outil **native AWS** pour déployer l'infrastructure via du code YAML/JSON. Ce template déclare deux ressources : une instance EC2 et un bucket S3. AWS les crée dans le bon ordre, sans commande manuelle :

```yaml
# Exemple CloudFormation : créer une instance EC2 et un bucket S3
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro

  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-bucket
```

> [!TIP]
> **Résultat attendu — après `aws cloudformation deploy --template-file stack.yaml --stack-name ma-stack` :**
> ```
> Waiting for changeset to be created...
> Waiting for stack create/update to complete...
>
> Successfully created/updated stack - ma-stack
>
> Stack ID   : arn:aws:cloudformation:eu-west-3:123456789012:stack/ma-stack/abc12345
> Status     : CREATE_COMPLETE
> Resources  :
>   - MyInstance  → i-0abc123def456789  (AWS::EC2::Instance)    CREATE_COMPLETE
>   - MyBucket    → my-app-bucket       (AWS::S3::Bucket)        CREATE_COMPLETE
> ```
> Les deux ressources ont été créées par CloudFormation dans le bon ordre. En cas de suppression, `aws cloudformation delete-stack --stack-name ma-stack` supprimera l'EC2 et le bucket ensemble — ce qui garantit qu'il ne reste pas de ressources orphelines.
**Intérêt :** CloudFormation est intégré nativement à AWS. Vous versionnez votre infrastructure comme du code et pouvez la recréer en quelques clics.

#### Comparaison simplifiée

| Outil | Modèle | Langage | Cas d'usage | Intégration AWS |
|-------|--------|---------|-------------|-----------------|
| **Chef** | Impératif (recipes) | Ruby | Configurations complexes ou parc historique | Solution Chef maintenue hors OpsWorks |
| **Puppet** | Déclaratif | Puppet DSL | Gestion de flotte à grande échelle | Solution Puppet maintenue hors OpsWorks |
| **Ansible** | Déclaratif (agentless) | YAML | Config management, post-provisioning | Via collection `amazon.aws` |
| **CloudFormation** | Déclaratif (IaC) | YAML/JSON | Déploiement d'infrastructure AWS | **Native, recommandé** |
| **Terraform** | Déclaratif (IaC) | HCL | Multi-cloud, plus flexible | Via AWS provider |

**Recommandation pratique :**
- Pour **débuter avec AWS**, utiliser **CloudFormation** ou **Terraform**.
- Pour **configurer les instances après déploiement**, utiliser **Ansible** — agentless et très bien intégré à AWS.
- Pour **gérer des configurations** sur des centaines de serveurs, évaluer AWS Systems Manager ou une solution Chef/Puppet maintenue ; ne pas créer de dépendance à OpsWorks.
- Pour **Puppet**, privilégier dans les environnements d'entreprise très structurés.
- En pratique : **Terraform + Ansible** est le duo le plus courant aujourd'hui — Terraform provisionne, Ansible configure.

📎 [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
📎 [AWS OpsWorks for Chef Automate](https://aws.amazon.com/opsworks/chef-automate/)
📎 [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)

---

## 10. Bonnes pratiques de démarrage

> [!NOTE]
> La pratique dans **AWS Academy** permet de découvrir la console dans un environnement temporaire, sans créer de compte AWS personnel.
### 10.1 Démarrer et sécuriser une session AWS Academy

Le lab fournit un compte, un rôle et des autorisations temporaires. La séquence de démarrage est la suivante :

1. Ouvrir le cours et le lab indiqués dans **AWS Academy**.
2. Cliquer sur **Start Lab** et attendre que la console soit disponible.
3. Ouvrir la console avec le bouton **AWS** fourni par le lab.
4. Vérifier le rôle et la **région active** avant toute création de ressource.
5. Utiliser **AWS CloudShell** pour les commandes CLI : aucune installation ni configuration locale n'est requise.

> [!TIP]
> **Ce que confirme la commande `aws sts get-caller-identity` dans CloudShell :**
> ```bash
> aws sts get-caller-identity
> ```
> ```json
> {
>     "UserId": "AROAEXAMPLE:academy-session",
>     "Account": "123456789012",
>     "Arn": "arn:aws:sts::123456789012:assumed-role/LabRole/academy-session"
> }
> ```
> L'ARN confirme que la session utilise un rôle temporaire du lab. N'exécutez pas `aws configure` et ne copiez jamais d'identifiants sur votre poste.
### 10.2 Repères pour la navigation dans la console AWS

Voici les éléments essentiels à repérer dans la console — vous les retrouverez présentés en vidéo dans le module AWS Academy correspondant.

**Éléments clés :**

1. **Sélecteur de région** : situé en haut à droite, il permet de choisir la région AWS dans laquelle les ressources seront déployées.
2. **Barre de recherche des services** : permet d'accéder rapidement à n'importe quel service AWS (EC2, S3, VPC, IAM, RDS…).
3. **Tableau de bord (Dashboard)** : affiche les services récemment utilisés et les informations de facturation.
4. **Panneau IAM** : permet de gérer les utilisateurs, groupes, rôles et stratégies de sécurité.
5. **Billing & Cost Management** : donne accès au suivi de la consommation et à la facturation.

📎 [Guide de démarrage AWS Console](https://docs.aws.amazon.com/awsconsole/latest/userguide/)
📎 [AWS Billing Documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html)
📎 [IAM Documentation](https://docs.aws.amazon.com/iam/)

Retenons que certaines ressources sont **spécifiques à une région** et que la facturation dépend de cette localisation.

> [!WARNING]
> **Piège fréquent — mauvaise région active :**
> Si vous créez des ressources dans une région autre que celle attendue (ex. `us-east-1` au lieu de `eu-west-3`), vous ne les verrez pas dans votre vue habituelle et continuerez à les payer. Vérifiez toujours le sélecteur de région en haut à droite de la console avant toute création de ressource.
---

## 11. Points importants et pièges fréquents

| Piège | Réalité |
|-------|---------|
| **« AWS, c'est juste des serveurs dans le cloud »** | AWS est une **plateforme complète** couvrant 200+ services — calcul, stockage, réseau, IA, sécurité, DevOps, etc. |
| **« Je suis en sécurité, AWS gère tout »** | **Non.** AWS gère la sécurité *du* cloud, vous gérez la sécurité *dans* le cloud (IAM, chiffrement, configuration). |
| **« Le Cloud public n'est pas conforme RGPD »** | **Faux.** AWS est conforme RGPD. C'est votre **usage** qui doit être conforme — localisation des données, consentement, droit à l'oubli, etc. |
| **« Les coûts AWS sont imprévisibles »** | Avec une **bonne gouvernance** (budgets, alertes, AWS Cost Explorer), les coûts sont très maîtrisables. |
| **« Un conteneur = une VM plus légère »** | **Non.** Un conteneur ne contient pas d'OS complet — il partage le noyau de l'hôte. C'est une architecture radicalement différente. |
| **« Je dois mettre toutes mes données en cloud »** | Non. Certaines données peuvent rester **on-premise** pour des raisons légales, réglementaires ou métier. Le **cloud hybride** existe pour ça. |
| **« IaaS, PaaS, SaaS — pareil pareil »** | Non. Chaque modèle **décale les responsabilités**. En IaaS, vous gérez plus ; en SaaS, AWS gère presque tout. |
| **« Je peux utiliser n'importe quelle région »** | Non. Certaines régions n'ont pas tous les services, et les données sensibles doivent rester dans des zones spécifiques (ex. RGPD en UE). |

---

Comprendre l'infrastructure ne suffit pas — encore faut-il la sécuriser. Le Chapitre 2 est entièrement consacré à la sécurité et à la gestion des identités dans AWS : comment **IAM** structure les droits et les accès, comment appliquer les bonnes pratiques de moindre privilège, et comment mettre en place une gouvernance solide.

## Ressources

### Documentation officielle AWS
- [AWS Documentation](https://docs.aws.amazon.com/fr_fr/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Free Tier](https://aws.amazon.com/fr/free/)
- [Régions et zones de disponibilité AWS](https://aws.amazon.com/fr/about-aws/global-infrastructure/regions_az/)

---

## Quiz interactif du chapitre

Choisissez une réponse : la correction expliquée apparaît immédiatement. Les propositions changent d’ordre à chaque nouvelle tentative.

<iframe class="quiz-frame" src="https://diablotynne.github.io/aws-initiation-approfondissement/static/quiz-aws/quiz-chapitre-1.html" title="Quiz interactif du chapitre 1" loading="lazy"></iframe>
