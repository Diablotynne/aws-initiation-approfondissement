---
date: 2026-06-30
tags: aws, cloud, ec2, s3, iam, devops
type: cours
status: active
---
<center><img src="https://upload.wikimedia.org/wikipedia/commons/9/93/Amazon_Web_Services_Logo.svg" alt="logo AWS" width="160" /></center>

---

> **Formateur** : Formateur Dawan — formation@dawan.fr
> **Durée** : 5 jours — Réf. Dawan : CLO100999-F
> **Lien Formation** : [Lien Teams à remplir]()
> **Lien Supports** :
>     - [HedgeDoc — Chapitre 1 : Fondamentaux du Cloud]()
>     - [HedgeDoc — Chapitre 2 : Sécurité et IAM]()
>     - [HedgeDoc — Chapitre 3 : Stockage S3 et Calcul EC2]()
>     - [HedgeDoc — Chapitre 4 : VPC et Bases de données]()
>     - [HedgeDoc — Chapitre 5 : Automatisation et CloudFormation]()
> **Lien HTML** : [HTML interactifs]()
> **Lien Prez** : [Présentation]()
> **Lien TP RECAP** : [TP de révision]()

---

[TOC]

---

# Bienvenue

Cette formation construit une architecture AWS résiliente pas à pas sur 5 jours : de la navigation dans la console le premier jour jusqu'à un template CloudFormation complet le cinquième. Plutôt que de survoler les centaines de services AWS, on se concentre sur les fondamentaux incontournables — IAM, EC2, S3, VPC, RDS — en comprenant pourquoi chaque service existe avant de l'utiliser.

L'objectif final : concevoir et déployer une architecture cloud sécurisée et automatisée sur AWS, en autonomie.

> [!NOTE]
> **Pourquoi cette progression ?** AWS compte plus de 200 services. En 5 jours, l'objectif n'est pas de les cataloguer mais de comprendre l'architecture sous-jacente — régions, zones, comptes, politiques — pour être capable de naviguer seul dans n'importe quel service par la suite.
## Tour de table

Avant de commencer, un tour de table permet de mieux cerner le groupe et d'ajuster le rythme de la semaine :

- Qui êtes-vous, quel est votre parcours ?
- Avez-vous déjà pratiqué un cloud public (AWS, Azure, GCP) ?
- Quel est le contexte qui vous amène à suivre cette formation ?
- Votre OS de prédilection ?

Se rendre également sur [https://moncompte.dawan.fr](https://moncompte.dawan.fr) pour remplir : besoins/attentes, niveau d'entrée, puis chaque jour l'émargement bi-quotidien, et en fin de formation l'évaluation et le niveau de sortie.

> [!NOTE]
> **Règle d'or de cette formation :** interrompre le formateur dès que vous ne comprenez pas ou dès que vous avez une question ; partager votre écran dès que vous rencontrez un blocage technique — le groupe est là pour s'entraider ; la caméra est recommandée, les gestes et expressions comptent autant que les mots.
## Objectifs de la formation

À l'issue de cette formation, chaque stagiaire sera capable de :

- Expliquer les concepts fondamentaux du Cloud Computing et l'infrastructure mondiale AWS
- Configurer IAM avec des politiques de sécurité restrictives, rôles et MFA
- Déployer des instances EC2 et gérer du stockage S3, EBS et EFS
- Concevoir un réseau VPC segmenté et déployer une base de données RDS sécurisée
- Automatiser un déploiement complet avec CloudFormation en suivant le Well-Architected Framework

## Public concerné et prérequis

Cette formation s'adresse aux administrateurs systèmes et développeurs souhaitant maîtriser les fondamentaux d'AWS. Aucune connaissance préalable du cloud n'est requise, mais une aisance de base avec la ligne de commande est utile.

## Programme de la semaine

| Chapitre | Cours | Atelier | Stack en fin de chapitre |
|------|-------------------------------|--------------|------------------------|
| **Chapitre 1** | Introduction au Cloud Computing · Concepts de base (IaaS, PaaS, SaaS) · Modèles de déploiement · Infrastructure mondiale AWS (Régions, Zones) | Création du compte · navigation dans le portail · premier déploiement de ressources statiques | Compte actif · familiarisation portail validée |
| **Chapitre 2** | IAM (Identity and Access Management) · Sécurité · Politiques JSON · Rôles & Utilisateurs · MFA · STS & Broker · IAM Identity Center | Configuration d'une politique de sécurité restrictive · création de groupes · test de droits via CLI | Structure de sécurité IAM en place et testée |
| **Chapitre 3** | Calcul avec EC2 · Stockage avec S3 (Stockage objet) · Volumes EBS et EFS · Cycle de vie des données · AWS CLI stockage | Lancement d'instances EC2 avec volume EBS et partage EFS · gestion et transferts de fichiers S3 par CLI | VMs en exécution · stockage S3 connecté |
| **Chapitre 4** | Architecture réseau (VPC, subnets publics/privés) · Sécurité (Security Groups, ACL) · Bases de données (RDS, Aurora, DynamoDB) | Conception d'un réseau VPC segmenté · configuration des Security Groups · déploiement d'une base RDS PostgreSQL | Réseau VPC isolé · base relationnelle RDS sécurisée et connectée |
| **Chapitre 5** | Automatisation avec CloudFormation (IaC) · AWS Backup · Architecture de production (Well-Architected Framework) | Déploiement automatisé d'une stack complète avec un template CloudFormation YAML | Stack automatisée · validation de la conception Well-Architected |

## Organisation pédagogique

Chaque chapitre alterne apports théoriques et ateliers pratiques sur un compte AWS réel, et se termine par un quiz de validation des connaissances. Une étape de nettoyage des ressources est systématique en fin de TP, pour éviter tout coût résiduel.

## Horaires

- Lundi : 9h30–12h30, 13h30–17h30
- Mardi à vendredi : 9h00–12h30, 13h30–17h00
- 1 pause de 15 min le matin et 1 pause de 15 min l'après-midi

## Matériel et environnement

Cette formation n'utilise pas de machines virtuelles locales. Chaque stagiaire dispose d'un **compte AWS temporaire** fourni avec des crédits de formation.

| Outil | Endpoint | Rôle |
|-------|----------|------|
| **AWS Console** | `https://aws.amazon.com/console` | Portail graphique |
| **AWS CLI** | Disponible dans AWS CloudShell, depuis le lab Academy | Ligne de commande sans installation locale |
| **Cloud Shell** | Intégré à la console AWS | Terminal en ligne sans installation |

> [!WARNING]
> **Avant la formation :** vérifier que les quotas de service permettent le déploiement de 2 instances `t2.micro` ou `t3.micro` par stagiaire dans la région sélectionnée.
## Évaluation

Chaque chapitre se termine par un quiz de validation des connaissances (10 questions).

## Ressources

Un espace de partage (Teams / moncompte.dawan.fr) centralise les supports et les corrections mises à disposition en fin de formation. Les modalités d'accès sont communiquées en début de session.
