---
title: "4. Modèles de déploiement : Public, Privé et Hybride"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 4. Modèles de déploiement : Public, Privé et Hybride"
---

<nav class="page-sequence"><a href="cours/chapitre-1/3-modeles-de-services-iaas-paas-et-saas">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/5-fondamentaux-techniques-virtualisation-et-conteneurs">Suivant</a></nav>

Les **modèles de déploiement** définissent *où* sont hébergées les ressources Cloud et *qui* les gère.
Ce choix a un impact direct sur la **gouvernance**, la **sécurité**, la **performance** et le **coût global** d'une solution Cloud.

Il existe trois grands modèles de déploiement reconnus par le NIST et adoptés par la majorité des entreprises : **Cloud Public**, **Cloud Privé** et **Cloud Hybride**.

Le modèle de déploiement choisi dépend souvent d'un compromis entre agilité, sécurité, conformité et maîtrise de l'environnement technique.

### 4.1 Le Cloud Public

Le **Cloud Public** est un modèle dans lequel les ressources sont hébergées dans les **datacenters du fournisseur Cloud** et partagées entre plusieurs clients.
Les infrastructures sont **mutualisées**, mais chaque client bénéficie d'un environnement isolé logiquement.

#### Caractéristiques :

Ce modèle repose sur quelques propriétés techniques qui expliquent à la fois ses forces et ses limites.

- Hébergement sur l'infrastructure du fournisseur.
- Mutualisation des ressources physiques.
- Facturation à l'usage.
- Évolutivité quasi illimitée.
- Accès global via Internet sécurisé.

C'est précisément cette mutualisation des ressources physiques entre plusieurs clients qui permet au fournisseur de proposer une facturation à l'usage et une scalabilité que peu d'entreprises pourraient atteindre seules avec leur propre matériel.

#### Avantages :

Ces caractéristiques techniques se traduisent concrètement par les bénéfices suivants pour l'entreprise cliente :

- Pas de gestion matérielle.
- Déploiement rapide.
- Coût réduit grâce à la mutualisation.
- Accès à un large éventail de services.

Ces avantages ont toutefois une contrepartie directe, qui explique pourquoi certaines entreprises hésitent encore à tout migrer en Cloud public.

#### Inconvénients :

Cette contrepartie se résume à trois limites principales :

- Moins de contrôle sur l'infrastructure physique.
- Dépendance au fournisseur.
- Besoins accrus en gouvernance et sécurité.

Ces inconvénients ne remettent pas en cause le modèle en lui-même, mais imposent de mettre en place des garde-fous (contrats, politiques IAM, plans de sortie) dès l'adoption.

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

Ces services couvrent l'essentiel des besoins d'une application moderne (calcul, stockage, base de données, distribution de contenu) sans qu'aucune infrastructure physique ne soit à gérer par l'entreprise.

📎 [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

### 4.2 Le Cloud Privé

Le **Cloud Privé** est un modèle dans lequel l'entreprise déploie ses propres ressources Cloud sur une **infrastructure dédiée**, souvent hébergée dans ses propres locaux ou dans un datacenter tiers, mais **non mutualisée** avec d'autres clients.

#### Caractéristiques :

Le fait de ne pas mutualiser l'infrastructure avec d'autres clients a des répercussions directes sur ce que l'entreprise gagne — et perd — par rapport au Cloud public.

- Infrastructure dédiée à une seule organisation.
- Contrôle complet sur l'environnement.
- Peut utiliser des technologies de virtualisation similaires à celles du Cloud public.
- Peut être automatisé et orchestré comme un Cloud public.

Ces caractéristiques rapprochent le Cloud privé d'une infrastructure on-premise classique, à ceci près que les outils et pratiques du Cloud public (automatisation, orchestration) peuvent aussi bien s'y appliquer.

#### Avantages :

Ne pas partager l'infrastructure avec d'autres clients se traduit directement par trois bénéfices :

- Meilleure maîtrise de la sécurité et de la conformité.
- Contrôle total sur les configurations.
- Plus de personnalisation possible.

Ce niveau de maîtrise a cependant un coût, qui se traduit par les inconvénients suivants.

#### Inconvénients :

Ce coût se manifeste principalement sur trois plans, financier, organisationnel et humain :

- Coût d'investissement plus élevé (proche d'un modèle CAPEX).
- Moins de flexibilité et de scalabilité.
- Nécessite des équipes internes pour la gestion.

C'est ce compromis entre contrôle et coût qui pousse encore aujourd'hui certaines organisations à conserver un Cloud privé plutôt que de tout basculer en Cloud public, notamment pour des raisons réglementaires.

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

> [!warning]
> **RGPD et localisation des données AWS :**
> Par défaut, AWS peut stocker vos données dans n'importe quelle zone de disponibilité de la région choisie. Pour les données personnelles de citoyens européens, vous devez impérativement choisir une **région européenne** (ex. `eu-west-3` Paris, `eu-central-1` Francfort) et vérifier que les services utilisés ne transfèrent pas les données hors UE sans votre accord explicite.

#### AWS s'est adapté :

Face à ces contraintes réglementaires, AWS propose des services qui rapprochent l'expérience du Cloud public d'une infrastructure restant on-premise ou dédiée.

- Un groupe bancaire peut déployer un **Cloud privé** sur sa propre infrastructure virtualisée avec VMware ou OpenStack, tout en automatisant les déploiements comme dans AWS.

Cette approche permet de retrouver les mêmes outils d'automatisation et d'orchestration que sur AWS, tout en gardant la donnée dans un périmètre maîtrisé par l'entreprise.

📎 [VMware Cloud Foundation](https://www.vmware.com/products/cloud-foundation.html)
📎 [OpenStack](https://www.openstack.org/)

**Services AWS typiques pour le Cloud privé :**

- **AWS Outposts** : infrastructure AWS déployée dans le datacenter du client
- **VMware Cloud on AWS** : environnement VMware géré dans AWS
- **Amazon VPC** : réseau virtuel isolé
- **AWS Direct Connect** : liaison réseau privée entre le client et AWS

![](assets/schemas/ch1-capture-05-ca65c3fb.png)

### 4.3 Le Cloud Hybride

Le **Cloud Hybride** combine le **Cloud Privé** et le **Cloud Public**, en permettant aux applications et données de circuler entre les deux environnements.
C'est aujourd'hui le modèle **le plus répandu** dans les grandes organisations.

#### Caractéristiques :

Ce modèle hérite à la fois des avantages du Cloud public et de ceux du Cloud privé, mais aussi d'une complexité supplémentaire liée à la coexistence des deux environnements.

- Combinaison de ressources internes et publiques.
- Interopérabilité entre les environnements.
- Optimisation des coûts et de la sécurité.
- Support de scénarios de migration progressive.

Ces caractéristiques expliquent pourquoi le Cloud hybride est souvent choisi comme étape intermédiaire lors d'une migration progressive vers le Cloud, plutôt que comme destination finale.

#### Avantages :

Cette combinaison des deux mondes se traduit par quatre bénéfices concrets :

- Flexibilité maximale.
- Répartition intelligente des charges de travail.
- Sécurité renforcée pour les données sensibles.
- Possibilité d'exploiter le meilleur des deux mondes.

Cette flexibilité a néanmoins un prix opérationnel, détaillé dans les inconvénients ci-dessous.

#### Inconvénients :

Faire coexister deux environnements distincts a un coût, principalement organisationnel :

- Gestion plus complexe.
- Nécessite des compétences solides en architecture et interconnexion.
- Risque de dépendances multiples.

Cette complexité de gestion justifie l'existence de services AWS dédiés à l'interconnexion, qui simplifient la mise en œuvre d'une architecture hybride.

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

![](assets/schemas/ch1-capture-06-717544e3.png)

### 4.4 Comparatif synthétique

<a class="schema-zoom" href="assets/schemas/modeles-deploiement-cloud.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/modeles-deploiement-cloud.svg"
     alt="Trois topologies côte à côte : Cloud public avec infrastructure AWS partagée entre plusieurs clients, Cloud privé avec infrastructure dédiée à une seule organisation, Cloud hybride reliant un site interne à AWS via une connexion sécurisée"
     style="display:block; margin:auto; width:90%"></a>

Ce schéma résume visuellement ce qui distingue les trois modèles vus dans cette section : qui possède l'infrastructure, si elle est partagée ou dédiée, et comment les deux mondes se connectent dans le cas hybride. Le tableau ci-dessous détaille ces différences critère par critère.

| Élément | Public Cloud | Private Cloud | Hybrid Cloud |
|--------|--------------|---------------|--------------|
| **Propriété de l'infrastructure** | Fournisseur (ex. AWS) | Entreprise | Mixte |
| **Isolation** | Mutualisée, logique | Dédiée, physique | Mixte |
| **Coûts** | OPEX, à la demande | CAPEX, investissement initial | Mixte |
| **Sécurité** | Standardisée, contrôlée par fournisseur | Complète, contrôlée par l'entreprise | Partagée |
| **Scalabilité** | Très élevée | Limitée | Élevée |
| **Gouvernance** | Fournisseur | Entreprise | Mixte |

Ce tableau montre que le Cloud hybride n'est pas un compromis médiocre entre les deux autres modèles, mais une combinaison qui hérite spécifiquement de la scalabilité du public sur les charges qui le permettent, tout en conservant le contrôle du privé sur celles qui l'exigent.

### 4.5 Recommandations par contexte

Le tableau ci-dessous traduit ces critères techniques en recommandation concrète selon le profil et les contraintes de l'organisation.

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

<nav class="page-sequence"><a href="cours/chapitre-1/3-modeles-de-services-iaas-paas-et-saas">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/5-fondamentaux-techniques-virtualisation-et-conteneurs">Suivant</a></nav>
