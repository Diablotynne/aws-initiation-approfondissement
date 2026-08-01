---
title: "2. Fondamentaux du Cloud Computing"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - 2. Fondamentaux du Cloud Computing"
---

# 2. Fondamentaux du Cloud Computing

<nav class="page-sequence"><a href="cours/chapitre-1/introduction">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/modeles-service">Suivant</a></nav>

### 2.1 Qu'est-ce que le Cloud Computing ?

Le **Cloud Computing** est une évolution majeure dans la manière dont les systèmes d'information sont conçus, déployés et exploités.

Pendant des décennies, les entreprises ont investi massivement dans leurs propres infrastructures : datacenters internes, salles machines, serveurs physiques, systèmes de climatisation, sécurité, licences logicielles, équipes d'exploitation dédiées.

Ce modèle repose sur des investissements lourds (**CAPEX**) et des cycles de décision longs.

Le Cloud vient bouleverser cette logique en offrant un modèle **flexible**, **scalable** et **orienté services** : les ressources sont **louées à la demande** et **payées à l'usage**.

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

:::warning
**Attention aux coûts AWS — le modèle OPEX peut surprendre :**
Une instance EC2 laissée tournante 24h/7j sans utilisation reste facturée. Contrairement à un investissement CAPEX amorti sur plusieurs années, les coûts OPEX s'accumulent en temps réel. Activez toujours des **alertes de budget (AWS Budgets)** dès le premier jour.
:::

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

Le tableau compare EC2, RDS et Lambda. Plus le service est **managé** — c'est-à-dire exploité techniquement par AWS — plus AWS prend en charge de couches techniques. Le client reste toutefois responsable de la classification de ses données et des autorisations qu'il accorde.

Le modèle de responsabilité partagée AWS répartit la sécurité entre deux parties, et confondre les deux moitiés est l'une des erreurs les plus fréquentes chez les entreprises qui découvrent le cloud.

AWS porte la responsabilité de la **sécurité du cloud**, c'est-à-dire de tout ce qui constitue l'infrastructure sous-jacente sur laquelle les clients construisent leurs services. Cela couvre la sécurité physique des data centers — accès contrôlé, vidéosurveillance, alimentation redondante — que le client n'a jamais à gérer lui-même. Cela couvre aussi le matériel et la couche de virtualisation qui isole chaque client des autres sur un même serveur physique, ainsi que la disponibilité globale de la plateforme, avec ses multiples régions et zones de disponibilité conçues pour survivre à la panne d'un data center entier.

Le **client**, de son côté, porte la responsabilité de la **sécurité dans le cloud** : tout ce qu'il configure et déploie au-dessus de cette infrastructure reste de son ressort. Il doit gérer les accès et les identités via IAM — qui a le droit de faire quoi sur son compte — et sécuriser ses données par le chiffrement et la rotation régulière des clés. Il doit configurer correctement son réseau (VPC, Security Groups, NACL) pour éviter d'exposer par erreur des ressources sensibles sur Internet. Et lorsqu'il utilise un service de type IaaS comme EC2, où AWS fournit la machine virtuelle mais pas ce qui tourne dessus, il reste responsable des sauvegardes, des correctifs de sécurité et des mises à jour du système d'exploitation.

Cela signifie que même si AWS est hautement sécurisé, **une mauvaise configuration côté client peut compromettre la sécurité** (par ex. un bucket S3 public par erreur).

:::danger
**Responsabilité client — erreurs fréquentes en production :**
- Un bucket S3 configuré en **accès public par erreur** expose toutes vos données sur Internet.
- L'utilisation du **compte root** pour les opérations quotidiennes est une faille de sécurité majeure.
- Un **Security Group ouvert sur 0.0.0.0/0 port 22** expose vos instances SSH au monde entier.
- L'absence de **MFA** sur le compte root est la première cause de compromission de compte AWS.

AWS ne peut pas vous protéger de vos propres erreurs de configuration — c'est votre responsabilité.
:::

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

<nav class="page-sequence"><a href="cours/chapitre-1/introduction">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/modeles-service">Suivant</a></nav>
