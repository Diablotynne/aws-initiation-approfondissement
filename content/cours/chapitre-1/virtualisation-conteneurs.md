---
title: "5. Fondamentaux techniques : Virtualisation et Conteneurs"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - 5. Fondamentaux techniques : Virtualisation et Conteneurs"
---

<nav class="page-sequence"><a href="cours/chapitre-1/modeles-deploiement">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/ecosysteme-aws">Suivant</a></nav>

Derrière le Cloud se cachent des **technologies fondamentales** qui le rendent possible.
Deux piliers en particulier ont permis l'essor massif des infrastructures à la demande :

- La **virtualisation**, qui permet de découper une machine physique en plusieurs machines virtuelles indépendantes.
- La **conteneurisation**, qui permet d'isoler et d'exécuter des applications de manière légère et flexible.

Comprendre ces concepts est essentiel pour appréhender le fonctionnement d'**Amazon Web Services (AWS)** et de nombreux autres fournisseurs Cloud.

<a class="schema-zoom" href="assets/schemas/virtualisation-conteneurs.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/virtualisation-conteneurs.svg" alt="Comparaison des couches d'un serveur traditionnel, de machines virtuelles et de conteneurs"></a>

**Lecture du schéma.** Une machine virtuelle embarque un système invité complet au-dessus de l'hyperviseur. Un conteneur isole l'application et ses dépendances tout en partageant le noyau du système hôte : il est généralement plus léger, mais ce n'est pas une machine virtuelle miniature.

📎 [Documentation Amazon EC2](https://docs.aws.amazon.com/ec2/)
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
- **Scalabilité** : un service fortement sollicité peut être mis à l'échelle indépendamment des autres composants.
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

<a class="schema-zoom" href="assets/schemas/aws-microservices-ecs-fargate.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/aws-microservices-ecs-fargate.svg"
     alt="Architecture microservices : CloudFront vers ALB, distribué vers Auth/Cart/Payment/Recomend puis vers RDS/DynamoDB/S3"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Les requêtes entrent par CloudFront, puis l'Application Load Balancer les répartit vers plusieurs microservices exécutés séparément. Chaque service accède uniquement au stockage adapté à son besoin. Les flèches représentent les flux réseau ; elles ne signifient pas que tous les services partagent automatiquement les mêmes droits IAM.

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

<nav class="page-sequence"><a href="cours/chapitre-1/modeles-deploiement">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/ecosysteme-aws">Suivant</a></nav>
