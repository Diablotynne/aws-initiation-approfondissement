---
title: "6. Présentation d'AWS et de son écosystème"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 6. Présentation d'AWS et de son écosystème"
---

<nav class="page-sequence"><a href="cours/chapitre-1/5-fondamentaux-techniques-virtualisation-et-conteneurs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/7-aws-well-architected-framework">Suivant</a></nav>

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

<a class="schema-zoom" href="assets/schemas/aws-global-infrastructure-animated.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-global-infrastructure-animated.svg" alt="Distribution animée d'une charge entre les trois zones de disponibilité de la région AWS Paris" style="display:block; margin:auto; width:90%"></a>

Les trois AZ appartiennent à la même région et communiquent par le réseau régional AWS, mais constituent des domaines de défaillance distincts. Déployer une instance dans `eu-west-3a` et une base uniquement dans cette même AZ reste une architecture mono-AZ, même si la région en propose trois.

L'un des piliers majeurs d'AWS est son **infrastructure mondiale** extrêmement étendue et redondée.
Elle est structurée en trois grandes composantes, organisées selon une hiérarchie précise :

<a class="schema-zoom" href="assets/schemas/infrastructure-mondiale-aws.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/infrastructure-mondiale-aws.svg"
     alt="Hiérarchie de l'infrastructure AWS : une région contient plusieurs zones de disponibilité, chaque zone contenant plusieurs datacenters ; les points de présence (Edge Locations) sont indépendants des régions et rapprochent le contenu des utilisateurs"
     style="display:block; margin:auto; width:90%"></a>

Ce schéma montre l'emboîtement des trois niveaux : une région regroupe plusieurs zones de disponibilité, chaque zone regroupant elle-même plusieurs datacenters physiques. Les points de présence, à l'inverse, ne font pas partie de cette hiérarchie régionale — ils sont disséminés indépendamment, au plus près des utilisateurs finaux.

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

<div class="concept-check">
<strong>Décision d'architecture — maintenant que vous connaissez régions et AZ</strong>
<p>Une application française contient des données sensibles et doit rester disponible lors de la panne d'un datacenter. Quel premier choix faut-il formuler ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Choisir une région conforme aux contraintes de localisation (ex. <code>eu-west-3</code> pour des données devant rester en France/UE), puis répartir les composants sur plusieurs zones de disponibilité de cette région. Une région protège du choix de localisation légale ; une AZ protège de la panne d'un datacenter — ce sont deux niveaux de panne différents, qui se cumulent plutôt que se remplacent.</p></details>
</div>

### 6.5 Principes de tarification AWS

L'un des aspects stratégiques de l'adoption du Cloud est la **tarification à l'usage**.
AWS applique un modèle de facturation souple et transparent basé sur la consommation réelle.

#### Principes clés :

- **Pay-as-you-go** : paiement en fonction de l'utilisation réelle des ressources.
- **Sans engagement initial** : pas de coût d'entrée, contrairement au modèle CAPEX.
- **Économies d'échelle** : les tarifs peuvent diminuer avec l'usage.
- **Facturation par service** : chaque service (EC2, S3, RDS…) est facturé séparément selon ses métriques propres.

Selon le service et l'engagement pris, plusieurs modes de tarification existent — à la demande, instances réservées, Savings Plans, Spot Instances. Le Chapitre 4 détaille chacun de ces modes avec des cas métier chiffrés, au moment où vous configurerez vos premières instances EC2.

📎 [AWS Pricing Overview](https://aws.amazon.com/pricing/)
📎 [AWS Pricing Calculator](https://calculator.aws/)

---

<nav class="page-sequence"><a href="cours/chapitre-1/5-fondamentaux-techniques-virtualisation-et-conteneurs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/7-aws-well-architected-framework">Suivant</a></nav>
