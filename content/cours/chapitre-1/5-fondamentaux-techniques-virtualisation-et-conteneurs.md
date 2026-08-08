---
title: "5. Fondamentaux techniques : Virtualisation et Conteneurs"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 5. Fondamentaux techniques : Virtualisation et Conteneurs"
---

<nav class="page-sequence"><a href="cours/chapitre-1/4-modeles-de-deploiement-public-prive-et-hybride">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/6-presentation-daws-et-de-son-ecosysteme">Suivant</a></nav>

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

On distingue deux familles d'hyperviseurs selon qu'ils s'installent directement sur le matériel ou par-dessus un OS existant :

- **Type 1 (bare-metal)** : installé directement sur le matériel physique. Exemples : VMware ESXi, Microsoft Hyper-V, KVM.
- **Type 2 (hosted)** : installé au-dessus d'un système d'exploitation. Exemples : VirtualBox, VMware Workstation.

Les hyperviseurs de type 1 sont ceux utilisés en production dans les datacenters (dont ceux d'AWS), car ils offrent de meilleures performances en s'affranchissant d'un OS hôte intermédiaire ; les hyperviseurs de type 2 restent réservés à des usages de poste de travail ou de test.

#### Avantages de la virtualisation :

Ces bénéfices expliquent l'adoption quasi universelle de la virtualisation dans les datacenters modernes :

- Meilleure utilisation des ressources matérielles.
- Isolation entre les environnements.
- Déploiement rapide.
- Maintenance simplifiée.

Ces avantages expliquent pourquoi la virtualisation est restée, plus de vingt ans après son adoption massive, le socle technique sur lequel repose la quasi-totalité des offres Cloud actuelles, AWS compris.

#### Virtualisation et AWS

Sur AWS, la virtualisation est au cœur de nombreux services, en particulier :

- **Amazon EC2 (Elastic Compute Cloud)** : service permettant de créer et gérer des machines virtuelles dans le Cloud AWS.
- **Amazon EBS (Elastic Block Store)** : service de stockage en mode bloc utilisé par les instances EC2.
- **Amazon VPC (Virtual Private Cloud)** : service qui fournit un réseau virtuel isolé dans AWS pour héberger les ressources.

Ces trois services montrent que la virtualisation n'est pas un détail d'implémentation invisible chez AWS : elle est directement exposée à l'utilisateur, qui choisit le type et la taille de sa VM (EC2), son stockage (EBS) et son réseau (VPC).

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

Le fait de partager le noyau de l'hôte plutôt que d'embarquer un OS complet se traduit par plusieurs bénéfices concrets :

- Démarrage quasi instantané.
- Consommation réduite de ressources.
- Facilité de portabilité et de déploiement.
- Standardisation des environnements.

Cette légèreté permet de faire tourner beaucoup plus de conteneurs que de VM sur un même serveur physique, ce qui explique pourquoi les architectures microservices modernes s'appuient massivement sur la conteneurisation plutôt que sur des VM dédiées par service.

#### Outils et services associés

Plusieurs outils et services permettent de créer, exécuter et orchestrer ces conteneurs, du poste de développement jusqu'à la production à grande échelle sur AWS :

- **Docker** : technologie de conteneurisation la plus répandue, permettant de créer, packager et exécuter des conteneurs.
- **Amazon ECS (Elastic Container Service)** : service AWS permettant de déployer et gérer des conteneurs Docker à grande échelle.
- **Amazon EKS (Elastic Kubernetes Service)** : service AWS qui fournit une plateforme Kubernetes managée pour orchestrer des conteneurs.

📎 [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)
📎 [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)

Il est important de comprendre la différence clé — VM = OS complet, Conteneur = application isolée. C'est une **rupture technologique** qui a permis l'essor de l'architecture microservices.

### 5.4 Microservices — Architecture distribuée et scalable

Les **microservices** sont une approche architecturale qui consiste à **découper une application monolithique en petits services indépendants**, chacun responsable d'une fonction métier spécifique.

**Exemple :** Une application de e-commerce ne sera plus UNE grosse application, mais plusieurs petits services :
- Service d'authentification
- Service de gestion de panier
- Service de paiement
- Service de recommandations
- Service de livraison

Ce découpage par fonction métier n'est pas qu'un détail d'organisation du code : il conditionne directement les bénéfices obtenus par rapport à une application monolithique.

#### Avantages des microservices

Le monolithe (une application unique de 500k lignes, une seule technologie, un déploiement lent et une scalabilité limitée) s'oppose aux microservices (Auth, Cart, Payment, Reco découpés en services indépendants), qui autorisent des technologies différentes par service, un déploiement rapide et une scalabilité flexible par équipe.

- **Agilité** : chaque équipe développe son service indépendamment.
- **Scalabilité** : seul le service de paiement reçoit beaucoup de trafic ? Scalez juste ce service.
- **Résilience** : si le service de recommandations est down, le reste de l'app fonctionne.
- **Technologie** : chaque service peut utiliser Node.js, Python, Java… selon le besoin.

Ces quatre bénéfices ont toutefois un prérequis technique implicite : découper une application en dizaines de petits services n'a de sens que si l'on dispose d'un moyen léger de les déployer et de les faire communiquer indépendamment — c'est le rôle des conteneurs.

#### Microservices et conteneurs

Les microservices **sans conteneurs** seraient très compliqués à gérer. C'est pourquoi **Docker + Kubernetes** (ou **AWS ECS**) sont devenus essentiels.

Chaque microservice est empaqueté dans son propre **conteneur Docker**, puis orchestré par un outil qui gère les déploiements, la scalabilité et la résilience.

**Services AWS pour microservices :**

- **Amazon ECS (Elastic Container Service)** : orchestration native AWS, plus simple que Kubernetes.
- **Amazon EKS (Elastic Kubernetes Service)** : Kubernetes managé, si vous avez une flotte massive.
- **AWS Fargate** : exécution serverless de conteneurs (sans gérer les instances EC2).
- **AWS Lambda** : exécution de fonctions sans serveur (cas de microservices très légers).

Le choix entre ces quatre services dépend surtout du niveau de contrôle recherché : ECS et Fargate privilégient la simplicité opérationnelle, EKS convient aux équipes déjà investies dans l'écosystème Kubernetes, et Lambda s'adresse aux microservices les plus légers et événementiels.

#### Un exemple en image

<a class="schema-zoom" href="assets/schemas/aws-microservices-ecs-fargate.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-microservices-ecs-fargate.svg"
     alt="Architecture microservices : CloudFront vers ALB, distribué vers Auth/Cart/Payment/Recomend puis vers RDS/DynamoDB/S3"
     style="display:block; margin:auto; width:90%"></a>

📎 [Microservices Documentation AWS](https://aws.amazon.com/microservices/)

### 5.5 Virtualisation vs conteneurisation — tableau comparatif

<a class="schema-zoom" href="assets/schemas/virtualisation-vs-conteneurisation.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/virtualisation-vs-conteneurisation.svg"
     alt="Architecture comparée : VM avec hyperviseur et OS invité complet par application, contre conteneurs partageant le noyau de l'hôte"
     style="display:block; margin:auto; width:90%"></a>

La différence fondamentale entre les deux approches se voit dans l'empilement des couches : une VM embarque un système d'exploitation invité complet pour chaque application, tandis que plusieurs conteneurs partagent le même noyau hôte via le moteur de conteneurs. C'est ce partage du noyau qui explique l'écart de poids et de vitesse de démarrage entre les deux modèles.

| Élément | Virtualisation (VM) | Conteneurisation |
|--------|--------|---------|
| **Système** | OS complet par machine virtuelle | Noyau partagé, application isolée |
| **Démarrage** | Lent (minutes) | Rapide (secondes) |
| **Consommation de ressources** | Élevée | Faible |
| **Portabilité** | Moins flexible | Très flexible |
| **Cas d'usage typiques** | Migration d'applications legacy, serveurs complets | Microservices, CI/CD, déploiements rapides |

Ce tableau permet de bien fixer les différences entre VM et conteneurs.

---

<nav class="page-sequence"><a href="cours/chapitre-1/4-modeles-de-deploiement-public-prive-et-hybride">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/6-presentation-daws-et-de-son-ecosysteme">Suivant</a></nav>
