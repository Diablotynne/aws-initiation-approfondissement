---
title: AWS — Initiation et approfondissement
description: Support de cours théorique sur le Cloud Computing et les principaux services AWS.
---

<!-- introduction-integree -->

# AWS — Initiation et approfondissement

<section class="course-intro-hero">
  <p class="module-hero__eyebrow">Support de cours · Cloud Computing et Amazon Web Services</p>
  <h2>Fondamentaux du cloud, services AWS, sécurité et architecture</h2>
  <p>Cinq chapitres couvrent les fondamentaux et offres AWS, la sécurité IAM, le stockage et le calcul, le réseau et les bases de données, puis l'automatisation et la reprise d'activité.</p>
  <div class="aws-actions"><a href="#objectifs-de-la-formation">Voir les objectifs</a><a href="cours/chapitre-1/">Commencer le cours</a></div>
</section>

## Parcours de formation

<div class="learning-card-grid">
  <a class="learning-card" href="cours/chapitre-1/"><span class="learning-card__number">01</span><span class="learning-card__meta">Cloud Computing et AWS</span><strong>Fondamentaux du cloud et présentation d'AWS</strong><small>NIST, modèles cloud, virtualisation, infrastructure mondiale, services, coûts et Well-Architected.</small></a>
  <a class="learning-card" href="cours/chapitre-2/"><span class="learning-card__number">02</span><span class="learning-card__meta">Sécurité</span><strong>Identités, authentification et autorisations avec IAM</strong><small>Utilisateurs, groupes, rôles, politiques, MFA, fédération, Cognito, comptes multiples et CloudTrail.</small></a>
  <a class="learning-card" href="cours/chapitre-3/"><span class="learning-card__number">03</span><span class="learning-card__meta">Stockage et calcul</span><strong>Amazon S3, EC2 et services de stockage associés</strong><small>Stockage objet, AMI, instances, EBS, EFS, tarification, équilibrage et Auto Scaling.</small></a>
  <a class="learning-card" href="cours/chapitre-4/"><span class="learning-card__number">04</span><span class="learning-card__meta">Réseau et données</span><strong>Amazon VPC et bases de données AWS</strong><small>RDS, Aurora, DynamoDB, migration, adressage, sous-réseaux, routage, sécurité et DNS.</small></a>
  <a class="learning-card" href="cours/chapitre-5/"><span class="learning-card__number">05</span><span class="learning-card__meta">Exploitation</span><strong>Automatisation, supervision et reprise d'activité</strong><small>RTO, RPO, sauvegarde, CloudFormation, Systems Manager, Elastic Beanstalk et CloudWatch.</small></a>
</div>

<!-- contenu-introduction:debut -->

Cette page rassemble les informations pratiques à connaître avant d'aborder les chapitres techniques.

## Finalité de la formation

La formation apporte les bases nécessaires pour analyser une architecture AWS, comprendre les responsabilités de sécurité, sélectionner les principaux services de calcul, de stockage, de réseau et de données, puis introduire l'automatisation et la supervision.

Le support sépare volontairement trois activités :

- le **cours**, consacré aux concepts, au fonctionnement des services et aux décisions d'architecture ;
- les **travaux pratiques**, réalisés dans l'environnement temporaire remis au début de la formation ;
- les **quiz**, utilisés pour vérifier immédiatement la compréhension de chaque chapitre.

## Tour de table

Avant d'aborder les concepts techniques, ce tour de table permet d'adapter les exemples et le niveau d'accompagnement au contexte professionnel du groupe :

- Quel est votre métier et votre environnement technique actuel ?
- Avez-vous déjà utilisé un cloud public ou administré une infrastructure virtualisée ?
- Quels services ou sujets AWS rencontrez-vous dans vos projets ?
- Quel objectif professionnel souhaitez-vous atteindre avec cette formation ?
- Existe-t-il une contrainte particulière à prendre en compte : sécurité, réseau, coûts, exploitation ou architecture ?

Les réponses servent à choisir les exemples et le niveau d'approfondissement sans modifier la couverture du programme.

## Objectifs de la formation

À l'issue de la formation, vous saurez :

- **expliquer** les caractéristiques du Cloud Computing, ses modèles économiques et le positionnement des principaux services AWS ;
- **décrire** l'infrastructure mondiale AWS et choisir une région ou une architecture multi-AZ à partir de contraintes explicites ;
- **appliquer** le modèle de responsabilité partagée et distinguer ce qui relève d'AWS de ce qui reste sous la responsabilité du client ;
- **concevoir** une gestion des identités et des accès fondée sur IAM, les rôles temporaires, le moindre privilège, la MFA et la traçabilité ;
- **sélectionner et configurer** les services de calcul et de stockage adaptés parmi EC2, Lambda, S3, EBS et EFS ;
- **concevoir** un VPC segmenté et raisonner sur les routes, passerelles, Security Groups, NACL et mécanismes d'interconnexion ;
- **choisir** un service de données relationnel, NoSQL ou de cache en fonction du modèle de données, de la disponibilité et de la charge ;
- **mettre en relation** élasticité, équilibrage de charge, sauvegarde, RPO, RTO et reprise d'activité ;
- **décrire et automatiser** une infrastructure avec CloudFormation et administrer des ressources avec Systems Manager ;
- **superviser et évaluer** une architecture avec CloudWatch et les six piliers du AWS Well-Architected Framework ;
- **justifier** une décision d'architecture en tenant compte de la sécurité, de la fiabilité, de la performance, des coûts et de l'exploitation.

## Horaires

| Période | Matin | Après-midi |
|---|---:|---:|
| Lundi | 9 h 30 – 12 h 30 | 13 h 30 – 17 h 30 |
| Mardi à vendredi | 9 h 00 – 12 h 30 | 13 h 45 – 17 h 00 |

Une pause de 15 minutes est prévue le matin et l'après-midi. L'émargement est réalisé deux fois par journée de formation.

## Public et prérequis

Le parcours s'adresse aux professionnels de l'informatique, de l'exploitation, du développement, de la sécurité ou de l'architecture qui souhaitent comprendre ou consolider leur pratique d'AWS.

Les manipulations nécessitent :

- un ordinateur disposant d'un navigateur web récent ;
- une connexion Internet stable ;
- l'accès aux environnements temporaires remis en début de formation.

Aucun compte AWS personnel, aucune carte bancaire et aucune installation locale de la CLI AWS ne sont exigés pour suivre les activités prévues.

## Organisation des ressources

Chaque chapitre commence par ses objectifs puis propose une page par concept. Le vocabulaire, la fiche mémo, les travaux pratiques et le quiz disposent également de pages distinctes afin de limiter le défilement et de permettre un accès direct.

> [!important]
> Les identifiants, modules et labs disponibles dépendent de la session ouverte pour la formation. Ce support ne publie aucun identifiant privé et n'ajoute aucun lab absent des ressources remises.

## Méthode de travail

1. Lire les objectifs du chapitre.
2. Parcourir les concepts dans l'ordre proposé ou accéder directement à une notion précise.
3. Consulter la fiche mémo avant l'activité pratique.
4. Réaliser les modules et labs indiqués dans l'environnement de formation.
5. Terminer par le quiz interactif du chapitre.

<!-- contenu-introduction:fin -->

## Télécharger le support

<div class="download-grid">
  <a href="static/downloads/aws-initiation-approfondissement-markdown.zip" download><strong>Archive Markdown (.zip)</strong><span>Les cinq chapitres sans navigation Quartz, accompagnés des schémas SVG.</span></a>
  <a href="assets/telechargements/aws-initiation-approfondissement.pdf" download><strong>Version PDF</strong><span>Document mis en page pour lecture hors ligne et impression.</span></a>
</div>

## Utilisation du support

Chaque chapitre développe d'abord les notions théoriques et leur fonctionnement technique. Les schémas servent à représenter les relations entre les composants. Un quiz interactif placé en fin de chapitre permet ensuite de vérifier les acquis ; l'explication apparaît après la réponse.

> [!NOTE]
> Les travaux pratiques sont réalisés dans les environnements temporaires remis pour la session. Ce support ne demande pas de créer un compte AWS personnel et ne contient ni consigne de création de compte ni identifiant d'accès.
