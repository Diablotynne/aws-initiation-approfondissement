---
title: "5. Amazon EC2 : La couche de calcul AWS"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 5. Amazon EC2 : La couche de calcul AWS"
---

<nav class="page-sequence"><a href="cours/chapitre-3/s3-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/instance-ec2">Suivant</a></nav>

### 4.1 Introduction à EC2

<div class="video-embed"><iframe src="https://www.youtube-nocookie.com/embed/aARcLxcGJaU" title="Lancer une machine virtuelle Windows avec Amazon EC2" loading="lazy" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Après avoir stocké nos données avec Amazon S3, nous allons voir comment **les traiter, les héberger ou les exécuter** grâce à **Amazon Elastic Compute Cloud (EC2)**.

EC2 est l'un des premiers services historiques d'AWS (2006). Il permet de **louer de la puissance de calcul à la demande**, avec une flexibilité inégalée par rapport aux serveurs physiques traditionnels.

📎 [Documentation officielle Amazon EC2](https://docs.aws.amazon.com/ec2/)

**Amazon EC2 (Elastic Compute Cloud)** est le service AWS qui permet de créer des **machines virtuelles** dans le cloud, appelées **instances EC2**.

### 4.2 Pourquoi utiliser EC2 ?

EC2 reprend le principe familier d'un serveur physique — un système d'exploitation, du CPU, de la RAM, du stockage, une carte réseau — mais en supprime toutes les contraintes matérielles, ce qui explique son adoption massive comme brique de calcul de base sur AWS.

Le **lancement est rapide** : là où commander, recevoir et configurer un serveur physique prenait des semaines, une instance EC2 est prête à l'emploi en quelques clics ou quelques lignes de CLI, avec un système d'exploitation déjà installé. Le service est aussi **flexible** : vous choisissez la puissance de calcul, le système d'exploitation, le type de stockage attaché et la configuration réseau, et vous pouvez faire évoluer ces choix a posteriori si les besoins changent — un projet peut commencer sur une petite instance et migrer vers une plus puissante sans réinstallation. Le modèle est **économique** parce que la facturation suit la consommation réelle plutôt qu'un investissement matériel figé : vous payez à l'heure ou à la seconde pour ce qui tourne, et vous pouvez arrêter une instance dès qu'elle n'est plus utile pour cesser d'être facturé. Enfin, EC2 est nativement **connecté** au reste de l'écosystème AWS : une instance peut lire et écrire dans un bucket S3, s'authentifier via un rôle IAM sans stocker de clé d'accès, et vivre dans un VPC dont vous contrôlez entièrement le découpage réseau — cette intégration native évite d'avoir à recoller manuellement des briques hétérogènes comme sur une infrastructure on-premise.

### 4.3 Les composants essentiels d'une instance EC2

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

---

<nav class="page-sequence"><a href="cours/chapitre-3/s3-cli">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/instance-ec2">Suivant</a></nav>
