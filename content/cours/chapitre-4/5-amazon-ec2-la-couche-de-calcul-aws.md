---
title: "5. Amazon EC2 : La couche de calcul AWS"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 5. Amazon EC2 : La couche de calcul AWS"
---

<nav class="page-sequence"><a href="cours/chapitre-4/4-gestion-de-s3-en-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/6-choisir-le-bon-type-dinstance-ami-stockage-et-securite">Suivant</a></nav>

### 5.1 Introduction à EC2

📹 **Vidéo** : [Lancer sa première machine virtuelle Windows avec EC2](https://www.youtube.com/watch?v=aARcLxcGJaU)

Après avoir stocké nos données avec Amazon S3, nous allons voir comment **les traiter, les héberger ou les exécuter** grâce à **Amazon Elastic Compute Cloud (EC2)**.

EC2 est l'un des premiers services historiques d'AWS (2006). Il permet de **louer de la puissance de calcul à la demande**, avec une flexibilité inégalée par rapport aux serveurs physiques traditionnels.

📎 [Documentation officielle Amazon EC2](https://docs.aws.amazon.com/ec2/)

**Amazon EC2 (Elastic Compute Cloud)** est le service AWS qui permet de créer des **machines virtuelles** dans le cloud, appelées **instances EC2**.

### 5.2 Pourquoi utiliser EC2 ?

EC2 reprend le principe familier d'un serveur physique — un système d'exploitation, du CPU, de la RAM, du stockage, une carte réseau — mais en supprime toutes les contraintes matérielles, ce qui explique son adoption massive comme brique de calcul de base sur AWS.

Le **lancement est rapide** : là où commander, recevoir et configurer un serveur physique prenait des semaines, une instance EC2 est prête à l'emploi en quelques clics ou quelques lignes de CLI, avec un système d'exploitation déjà installé. Le service est aussi **flexible** : vous choisissez la puissance de calcul, le système d'exploitation, le type de stockage attaché et la configuration réseau, et vous pouvez faire évoluer ces choix a posteriori si les besoins changent — un projet peut commencer sur une petite instance et migrer vers une plus puissante sans réinstallation. Le modèle est **économique** parce que la facturation suit la consommation réelle plutôt qu'un investissement matériel figé : vous payez à l'heure ou à la seconde pour ce qui tourne, et vous pouvez arrêter une instance dès qu'elle n'est plus utile pour cesser d'être facturé. Enfin, EC2 est nativement **connecté** au reste de l'écosystème AWS : une instance peut lire et écrire dans un bucket S3, s'authentifier via un rôle IAM sans stocker de clé d'accès, et vivre dans un VPC dont vous contrôlez entièrement le découpage réseau — cette intégration native évite d'avoir à recoller manuellement des briques hétérogènes comme sur une infrastructure on-premise.

### 5.3 Les composants essentiels d'une instance EC2

| Composant | Rôle dans l'architecture EC2 |
|---|---|
| **Instance EC2** | Machine virtuelle hébergée chez AWS |
| **AMI** | Image système (Linux, Windows, etc.) utilisée comme modèle |
| **Type d'instance** | Détermine la puissance (CPU, RAM, réseau) |
| **Security Group** | Pare-feu virtuel qui contrôle les accès réseau |
| **EBS** | Disque dur virtuel attaché à l'instance |
| **EFS** | Système de fichiers partagé entre plusieurs instances |
| **Key Pair** | Clé SSH utilisée pour se connecter à l'instance en toute sécurité |

_Lors du lancement d'une instance, vous devez choisir chacun de ces éléments._

### 5.4 Les services de conteneurs AWS — ECS, ECR et Fargate

Une instance EC2 reste une machine virtuelle complète. Elle démarre un système d'exploitation entier, avec son propre noyau, avant même de pouvoir exécuter la moindre application. Pour héberger dix applications indépendantes, il faudrait donc dix machines virtuelles — ou une seule, mal isolée, où toutes les applications se partagent le même système.

Le **conteneur** répond à ce problème autrement. Au lieu de virtualiser un système d'exploitation entier, il isole uniquement le processus applicatif et ses dépendances — bibliothèques, variables d'environnement, fichiers — dans un espace cloisonné qui partage le noyau Linux de la machine hôte. Résultat : dix conteneurs peuvent tourner sur une seule instance EC2. Ils démarrent en quelques secondes, pas en quelques minutes. Et ils s'exécutent identiquement sur un poste de développement, en intégration continue, ou en production, une fois empaquetés. C'est cette portabilité qui a fait le succès de Docker, le format de conteneur devenu standard de facto.

AWS propose trois services complémentaires pour faire tourner des conteneurs en production. Ils ne sont pas concurrents entre eux — chacun a un rôle distinct :

- **Amazon ECR (Elastic Container Registry)** — un entrepôt d'images. De la même manière que S3 stocke des fichiers, ECR stocke des **images de conteneurs** : le paquet figé qui contient l'application et tout ce qu'il lui faut pour s'exécuter. C'est l'équivalent AWS de Docker Hub, privé et intégré à IAM. On y pousse ("push") une image depuis son poste ou sa chaîne CI/CD ; les services d'exécution la récupèrent ("pull") au moment de démarrer un conteneur.
- **Amazon ECS (Elastic Container Service)** — l'orchestrateur. Il décide où et comment exécuter les conteneurs à partir des images stockées dans ECR : démarrer le nombre demandé, redémarrer automatiquement en cas de panne, répartir la charge, arrêter proprement lors d'une mise à jour. C'est une alternative plus simple d'usage que Kubernetes (dont l'équivalent AWS géré est **EKS**, hors périmètre de cette formation), pensée nativement pour l'écosystème AWS.
- **AWS Fargate** — un mode d'exécution pour ECS (et EKS) qui retire la dernière brique manuelle. Plus d'instance EC2 à provisionner ni à administrer pour héberger les conteneurs : AWS alloue la capacité de calcul à la demande, facturée au conteneur réellement exécuté plutôt qu'à l'instance entière. C'est le pendant serverless d'ECS, comme Lambda l'est pour du code sans conteneur.

> [!info]
> **À retenir :** ECR stocke l'image, ECS orchestre son exécution, Fargate retire la gestion des serveurs sous-jacents. On peut très bien utiliser ECS **sans** Fargate (type de lancement "EC2", où l'on gère soi-même les instances qui hébergent les conteneurs) — Fargate est une option, pas une obligation.

<a class="schema-zoom" href="assets/schemas/aws-microservices-ecs-fargate.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-microservices-ecs-fargate.svg" alt="Architecture ECS/Fargate : du push d'image ECR au conteneur en exécution" style="display:block; margin:auto; width:90%"></a>

Le schéma ci-dessus illustre le flux complet : une image applicative est construite puis poussée vers un dépôt ECR, un **cluster ECS** définit la capacité disponible (EC2 ou Fargate), une **définition de tâche** (task definition) décrit quelle image lancer avec quelles ressources CPU/mémoire, et un **service ECS** maintient en permanence le nombre de tâches demandé — en relançant automatiquement toute tâche qui s'arrête de manière inattendue. C'est ce mécanisme de service qui rapproche ECS d'Auto Scaling pour des instances EC2 (vu en section 11) : dans les deux cas, AWS surveille l'état réel et le compare à l'état désiré pour corriger tout écart sans intervention humaine.

Passons à la pratique en créant un dépôt ECR, en y poussant une image, puis en la déployant sur un cluster ECS en mode Fargate.

```bash
# Créer un dépôt ECR pour stocker l'image de l'application
aws ecr create-repository \
  --repository-name demo-app \
  --image-scanning-configuration scanOnPush=true
# --image-scanning-configuration scanOnPush=true : ECR analyse automatiquement
# chaque image poussée à la recherche de vulnérabilités connues (CVE)
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>     "repository": {
>         "repositoryArn": "arn:aws:ecr:eu-west-3:123456789012:repository/demo-app",
>         "repositoryUri": "123456789012.dkr.ecr.eu-west-3.amazonaws.com/demo-app",
>         "imageScanningConfiguration": { "scanOnPush": true }
>     }
> }
> ```

Le `repositoryUri` retourné est l'adresse à utiliser pour pousser l'image — il faut d'abord authentifier Docker auprès d'ECR, puis étiqueter l'image locale avec cette adresse avant de la transférer.

```bash
# S'authentifier auprès d'ECR : récupère un jeton temporaire et le transmet à Docker
aws ecr get-login-password --region eu-west-3 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.eu-west-3.amazonaws.com

# Étiqueter l'image locale avec l'adresse complète du dépôt ECR
docker tag demo-app:latest 123456789012.dkr.ecr.eu-west-3.amazonaws.com/demo-app:latest

# Pousser l'image vers ECR
docker push 123456789012.dkr.ecr.eu-west-3.amazonaws.com/demo-app:latest
```

> [!warning]
> **Erreur fréquente :** le jeton d'authentification `get-login-password` expire au bout de 12 heures — un `docker push` qui échoue avec une erreur 401 après une pause déjeuner ne signifie pas que les droits IAM sont mauvais, mais qu'il faut simplement relancer la commande de login.

Une fois l'image disponible dans ECR, il reste à créer le cluster ECS (le regroupement logique de capacité) puis à y déployer un service Fargate qui référence cette image :

```bash
# Créer un cluster ECS — avec Fargate, ce cluster ne contient aucune instance EC2 à gérer
aws ecs create-cluster --cluster-name demo-cluster

# Enregistrer une définition de tâche (task definition) qui référence l'image ECR
# Le fichier task-definition.json décrit : l'image, le CPU/mémoire alloués,
# et le mode réseau (awsvpc, obligatoire avec Fargate)
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Créer le service qui maintient en permanence 2 tâches actives sur ce cluster
aws ecs create-service \
  --cluster demo-cluster \
  --service-name demo-service \
  --task-definition demo-app \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-0123456789abcdef0],securityGroups=[sg-0123456789abcdef0],assignPublicIp=ENABLED}"
# --launch-type FARGATE : AWS fournit la capacité de calcul, aucune instance EC2 à créer
# --desired-count 2 : ECS relance automatiquement toute tâche qui s'arrête pour revenir à 2
```

> [!tip]
> **Résultat attendu (extrait) :**
> ```json
> {
>     "service": {
>         "serviceName": "demo-service",
>         "status": "ACTIVE",
>         "desiredCount": 2,
>         "runningCount": 0,
>         "launchType": "FARGATE"
>     }
> }
> ```
> Le `runningCount` remonte progressivement à 2 en quelques dizaines de secondes, le temps que Fargate provisionne la capacité et démarre les conteneurs.

Cette bascule vers les conteneurs pose une question légitime : pourquoi ne pas tout faire en Lambda (vu en section 12), qui est déjà serverless ? La réponse tient à la nature de la charge : Lambda est pensé pour des exécutions courtes et déclenchées par un événement (une requête HTTP, un fichier déposé sur S3), avec une durée d'exécution plafonnée à 15 minutes et un environnement recréé à chaque invocation. ECS/Fargate convient mieux à une application qui tourne en continu (un serveur web, une API qui doit rester disponible en permanence, un traitement de longue durée) ou qui a besoin d'un contrôle plus fin sur son environnement d'exécution (dépendances système précises, taille mémoire supérieure aux limites Lambda). Les deux approches sont complémentaires plutôt qu'exclusives dans une architecture réelle.

📖 [Amazon ECS — Documentation officielle](https://docs.aws.amazon.com/ecs/)
📖 [Amazon ECR — Documentation officielle](https://docs.aws.amazon.com/AmazonECR/latest/userguide/)
🎬 [Conteneurs sur AWS — chaîne AWS France](https://www.youtube.com/@amazonwebservicesfrance) — rechercher "ECS Fargate" pour les démonstrations les plus récentes

---

<nav class="page-sequence"><a href="cours/chapitre-4/4-gestion-de-s3-en-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/6-choisir-le-bon-type-dinstance-ami-stockage-et-securite">Suivant</a></nav>
