---
title: "1. Introduction"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 1. Introduction"
---

<nav class="page-sequence"><a href="cours/chapitre-1/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/2-fondamentaux-du-cloud-computing">Suivant</a></nav>

### 1.1 Qu'est-ce qu'AWS et pourquoi le découvrir ?

**Amazon Web Services (AWS)** est la division cloud du groupe Amazon. Elle propose aujourd'hui plus de 200 services : calcul, stockage, bases de données, sécurité, intelligence artificielle, DevOps.

Ce catalogue n'a pas surgi d'un coup. Il est le résultat de vingt ans de croissance quasi continue — la section suivante en retrace les jalons.

Ce qui rend AWS incontournable dans une formation cloud n'est pas seulement son ancienneté. C'est sa position de force sur le marché. AWS capte aujourd'hui un peu plus de **30 % du marché mondial des infrastructures cloud**. Cette part dépasse celle de ses deux principaux concurrents, Microsoft Azure et Google Cloud, réunis.

Cette domination a une conséquence directe pour vous. Les compétences AWS restent, pour l'instant, les plus recherchées sur le marché de l'emploi IT. Une grande partie des architectures cloud que vous croiserez en entreprise — quel que soit le secteur — auront été conçues avec ces services en tête. Apprendre AWS en premier, c'est donc apprendre un vocabulaire et des mécanismes qui se retrouvent, sous des noms différents, chez tous les fournisseurs cloud : régions, zones de disponibilité, modèle de responsabilité partagée.

Vous trouverez AWS dans quasiment tous les secteurs d'activité, pas seulement chez des startups qui testent une idée en quelques jours. Amazon documente publiquement des cas d'usage chez des entreprises que vous connaissez déjà :

- **Netflix** héberge sur AWS l'intégralité de son infrastructure de diffusion vidéo, jusqu'à un studio de production virtuel dans le cloud.
- **Airbnb** y fait tourner ses annonces, ses paiements sécurisés et son moteur de recommandation.
- La bourse **Nasdaq** y traite des données de marché critiques.
- La **NASA** y stocke et analyse les données de ses missions spatiales.
- Dans la santé, des groupes pharmaceutiques comme **Pfizer** s'appuient sur AWS pour leurs travaux de recherche.

Ces exemples ne sont pas anecdotiques. Ils montrent que le même socle de services — calcul, stockage, bases de données, sécurité — s'adapte à un site e-commerce comme à une mission spatiale ou un système bancaire. C'est cette polyvalence que ce chapitre pose comme fondation, avant d'entrer dans le détail des services.

### 1.2 Histoire d'AWS — d'un mémo interne au leader mondial du cloud

L'histoire d'AWS ne commence pas en 2006 avec le lancement grand public de S3 et EC2. Elle commence trois ans plus tôt, en interne chez Amazon.

En 2003, l'entreprise impose à toutes ses équipes une architecture orientée services (SOA). Chaque système doit communiquer avec les autres via des API bien définies, plutôt que par des accès directs à des bases de données partagées. Cette contrainte technique visait d'abord à fiabiliser le site e-commerce d'Amazon. Mais elle révèle un constat inattendu : les équipes passent un temps disproportionné à réinventer chacune leur propre infrastructure de serveurs et de stockage.

Deux ingénieurs, **Chris Pinkham** et **Benjamin Black**, proposent alors un mémo interne. Il décrit des services de calcul virtualisés standardisés. L'idée germe : Amazon pourrait vendre cette infrastructure interne à d'autres entreprises.

>🔎 **Qui sont ces personnes ?**
>
>- **Chris Pinkham** — ingénieur Amazon. Il dirigera ensuite le développement d'EC2 depuis une nouvelle filiale créée au Cap, en Afrique du Sud.
>- **Benjamin Black** — ingénieur Amazon, co-auteur du mémo fondateur avec Pinkham. Il quittera l'entreprise avant le lancement public d'AWS en 2006.
>- **Andy Jassy** — à l'époque cadre chez Amazon, pas encore dirigeant du groupe. Il porte le projet politiquement en interne et devient le premier responsable d'AWS, avant de diriger Amazon tout entier en 2021 (voir plus bas).

**Andy Jassy** porte le projet. Il rédige la note de six pages — un "six-pager", format classique chez Amazon pour formaliser une proposition stratégique. Le feu vert tombe : la première équipe est recrutée.

Le premier service à voir le jour, en 2004, est discret : **Amazon SQS** (Simple Queue Service), une file de messages entre applications. Pas spectaculaire, mais c'est la première pierre technique du futur AWS.

Le grand public découvre la plateforme en 2006, avec le lancement quasi simultané de deux services :
- **S3** (mars 2006) — stockage objet.
- **EC2** (août 2006) — calcul virtualisé à la demande.

Ces deux services incarnent encore aujourd'hui les deux piliers historiques du cloud : payer pour stocker des données, payer pour de la puissance de calcul — les deux à l'usage, jamais par achat de matériel.

La table ci-dessous reprend les jalons majeurs qui ont suivi ce lancement fondateur. Les dates comptent moins que ce qu'elles disent : AWS est passé d'un simple hébergeur de serveurs à une plateforme qui couvre aujourd'hui la donnée, la sécurité et l'intelligence artificielle.

| Année | Événement | Explication pédagogique |
|-------|-----------|------------------------|
| **2004** | Lancement discret d'**Amazon SQS** | Premier service AWS jamais commercialisé, une simple file de messages — la brique technique fondatrice, avant même que le nom "AWS" ne soit public. |
| **2006** | Lancement public d'AWS avec **S3** et **EC2** | AWS débute avec deux services fondamentaux : **S3** pour le stockage objet (sauvegardes, fichiers, images) et **EC2** pour le calcul (machines virtuelles). Ces deux services incarnent les piliers du cloud : stockage et puissance de calcul à la demande. |
| **2009** | Introduction de **VPC** et **RDS** | AWS ajoute **VPC** (Virtual Private Cloud) pour créer des réseaux privés isolés, et **RDS** (Relational Database Service) pour gérer des bases de données relationnelles sans administrer les serveurs. Cela marque l'arrivée des services réseau et des bases gérées. |
| **2012** | Lancement de **DynamoDB** | AWS introduit **DynamoDB**, une base NoSQL scalable et sans schéma, adaptée aux applications modernes (web, mobile, IoT). C'est le tournant vers les architectures serverless et les microservices. |
| **2014** | Lancement de **AWS Lambda** | AWS révolutionne le cloud avec **Lambda**, qui permet d'exécuter du code sans serveur. C'est le début du **serverless computing**, où l'infrastructure disparaît derrière la logique métier. |
| **2021** | **Andy Jassy** devient CEO d'Amazon | Le fondateur d'AWS quitte la direction d'AWS pour succéder à Jeff Bezos à la tête du groupe Amazon tout entier — signe du poids stratégique pris par la division cloud. **Adam Selipsky** (l'un des tout premiers VP d'AWS, embauché en 2005) reprend la direction d'AWS. |
| **2024** | **Matt Garman** devient CEO d'AWS | Vétéran d'AWS depuis 18 ans et premier product manager d'EC2 historiquement, Matt Garman succède à Adam Selipsky le 3 juin 2024 et dirige AWS aujourd'hui. |
| **2025** | Plus de **200 services**, chiffre d'affaires record | AWS devient le fournisseur cloud le plus complet, couvrant tous les domaines : calcul, stockage, IA, sécurité, DevOps, IoT, bases de données, etc. La section suivante détaille les chiffres financiers de cette année. |

Ces dates montrent une trajectoire claire : AWS est passé d'un simple fournisseur de serveurs et de stockage à une **plateforme cloud complète** — hébergement, sécurité, automatisation, intelligence artificielle.

Elles montrent aussi une continuité de gouvernance rare dans la tech. De 2003 à 2021, une seule personne — Andy Jassy — a porté le projet, du mémo initial jusqu'à la direction du groupe Amazon tout entier. C'est un signal fort : ce qui n'était au départ qu'un problème d'infrastructure interne a fini par devenir stratégique pour toute l'entreprise.

#### 1.2 bis — Chiffres clés 2025 : le poids réel d'AWS dans le groupe Amazon

Amazon publie chaque année ses chiffres financiers dans son rapport annuel déposé auprès du régulateur américain (formulaire 10-K). Pour l'exercice 2025, déposé en février 2026 :

- Chiffre d'affaires total du groupe Amazon : **716,9 milliards de dollars**.
- Chiffre d'affaires AWS seule : **128,7 milliards de dollars**, soit environ **18 %** du total du groupe.
- Croissance d'AWS sur un an : **+20 %**, portée notamment par la demande liée à l'intelligence artificielle.
- Le seul quatrième trimestre 2025 a vu AWS croître de **24 %** sur un an — sa meilleure progression en treize trimestres.

Ce chiffre de 18 % du chiffre d'affaires masque le poids réel d'AWS : sa rentabilité. Sur ce même exercice 2025 :

- Résultat opérationnel total du groupe Amazon : **79,9 milliards de dollars**.
- Résultat opérationnel généré par AWS seule : **45,6 milliards de dollars**.

Autrement dit : AWS produit à elle seule plus de la **moitié du profit opérationnel de tout le groupe Amazon**, alors qu'elle ne représente qu'un cinquième de son chiffre d'affaires. Ce déséquilibre explique pourquoi la presse économique décrit souvent AWS comme le véritable "moteur de profit" d'Amazon. Les activités de vente en ligne (Amazon.com) génèrent l'essentiel du volume d'affaires. Mais elles opèrent avec des marges nettement plus faibles que celles du cloud — structurellement plus rentable une fois l'infrastructure de base amortie.

> [!info]
> **À retenir :** AWS ≈ 18 % du chiffre d'affaires d'Amazon, mais plus de 55 % de son résultat opérationnel. Un participant qui comprend ce déséquilibre comprend aussi pourquoi Amazon continue d'investir massivement dans de nouvelles régions et de nouveaux services cloud : c'est la partie la plus rentable du groupe.

📖 [Amazon — rapport annuel (formulaire 10-K, SEC)](https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=0001018724&type=10-K)
📎 [Success-stories de clients AWS (Netflix, Airbnb, NASA, Nasdaq...)](https://aws.amazon.com/solutions/case-studies/)

### 1.3 Certifications AWS

AWS structure son parcours de certification en quatre niveaux, pensés pour accompagner une montée en compétence progressive plutôt que pour classer les candidats par mérite. Chaque niveau valide un périmètre de responsabilités différent en entreprise, et il est tout à fait normal — recommandé, même — de les passer dans l'ordre.

Le niveau **Fondamental** (Cloud Practitioner) s'adresse à toute personne qui doit comprendre le vocabulaire et les grands principes du cloud AWS sans nécessairement configurer quoi que ce soit elle-même : commerciaux, chefs de projet, décideurs, ou participants qui découvrent AWS pour la première fois. L'examen porte sur les concepts (modèles de tarification, responsabilité partagée, services principaux) plutôt que sur la mise en œuvre technique.

Le niveau **Associate** cible les professionnels qui déploient et opèrent concrètement des infrastructures AWS au quotidien. Il se décline selon le métier visé : **Solutions Architect – Associate** pour la conception d'architectures, **CloudOps Engineer – Associate (SOA-C03)** pour l'exploitation et la supervision — ce nom remplace SysOps Administrator depuis 2025 — et **Developer – Associate** pour l'intégration d'applications avec les API et services AWS. Le catalogue comprend également des certifications Associate orientées données et machine learning.

Le niveau **Professional** approfondit les deux certifications Architecte et DevOps du niveau Associate avec des scénarios de complexité réelle : migrations à grande échelle, architectures multi-comptes, optimisation fine des coûts et de la résilience. Ces examens supposent une expérience pratique significative — AWS les recommande après plusieurs années d'usage professionnel du cloud.

Le niveau **Spécialité** (Specialty) valide une expertise pointue plutôt qu'une vision généraliste. Le catalogue évolue régulièrement : Database Specialty et plusieurs autres examens ont été retirés en 2024, puis Machine Learning Specialty le 31 mars 2026. Il faut donc consulter le catalogue officiel avant de présenter une liste d'examens comme actuelle. Security Specialty et Advanced Networking Specialty restent des repères pertinents pour les domaines abordés ici.

Pour cette formation, le niveau visé est le **Solutions Architect Associate (SAA-C03)** : c'est la certification la plus demandée sur le marché de l'emploi et celle qui couvre le plus largement les services abordés dans les chapitres suivants (EC2, S3, VPC, RDS, IAM, CloudFormation). Le Chapitre 5 détaillera la pondération exacte des domaines de l'examen une fois tous les services vus.

![](assets/schemas/ch1-capture-01-6cc3f203.png)

📎 [Certifications AWS](https://aws.amazon.com/certification/)

---

<nav class="page-sequence"><a href="cours/chapitre-1/objectifs">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/2-fondamentaux-du-cloud-computing">Suivant</a></nav>
