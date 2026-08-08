---
title: "AWS — Initiation et approfondissement"
description: "Support de cours théorique sur le Cloud Computing et les principaux services AWS."
---

<!-- introduction-integree -->

# AWS — Initiation et approfondissement

<section class="course-intro-hero">
  <p class="module-hero__eyebrow">Support de cours · Cloud Computing et Amazon Web Services</p>
  <h2>Fondamentaux du cloud, services AWS, sécurité et architecture</h2>
  <p>Cinq chapitres couvrent les fondamentaux et offres AWS, la sécurité IAM, le stockage et le calcul, le réseau et les bases de données, puis l'automatisation et la reprise d'activité.</p>
  <div class="course-actions"><a href="#objectifs-de-la-formation">Voir les objectifs</a><a href="cours/chapitre-1/">Commencer le cours</a></div>
</section>

## Informations administratives

Cette page rassemble les informations pratiques à connaître avant d'aborder les chapitres techniques.

- L'émargement est réalisé deux fois par journée de formation.
- Les besoins et attentes sont recueillis au démarrage de la session.
- Les accès temporaires et les consignes propres aux labs sont communiqués pendant la session et ne sont pas publiés dans ce support.

## Horaires

| Période | Matin | Après-midi |
|---|---:|---:|
| Lundi | 9 h 30 – 12 h 30 | 13 h 30 – 17 h 30 |
| Mardi à vendredi | 9 h 00 – 12 h 30 | 13 h 45 – 17 h 00 |

Une pause de 15 minutes est prévue le matin et l'après-midi.

## Tour de table

Avant d'aborder les concepts techniques, ce tour de table permet d'adapter les exemples et le niveau d'accompagnement au contexte professionnel du groupe :

- Quel est votre métier et votre environnement technique actuel ?
- Avez-vous déjà utilisé un cloud public ou administré une infrastructure virtualisée ?
- Quels services ou sujets AWS rencontrez-vous dans vos projets ?
- Quel objectif professionnel souhaitez-vous atteindre avec cette formation ?
- Existe-t-il une contrainte particulière à prendre en compte : sécurité, réseau, coûts, exploitation ou architecture ?

Les réponses servent à choisir les exemples et le niveau d'approfondissement sans modifier la couverture du programme.

> [!important]
> **Règle d'or de cette formation :** interrompre le formateur dès qu'une notion n'est pas comprise ou qu'une question se présente ; partager son écran en cas de blocage technique afin de permettre un diagnostic collectif ; participer activement aux échanges et aux validations.

## Finalité de la formation

La formation apporte les bases nécessaires pour analyser une architecture AWS, comprendre les responsabilités de sécurité, sélectionner les principaux services de calcul, de stockage, de réseau et de données, puis introduire l'automatisation et la supervision.

Le support sépare volontairement trois activités :

- le **cours**, consacré aux concepts, au fonctionnement des services et aux décisions d'architecture ;
- les **travaux pratiques**, réalisés dans l'environnement temporaire remis au début de la formation ;
- les **quiz**, utilisés pour vérifier immédiatement la compréhension de chaque chapitre.

## Objectifs de la formation

À l'issue de la formation, vous saurez :

- **expliquer** les fondamentaux du Cloud Computing et le modèle de responsabilité partagée pour choisir une région ou une architecture multi-AZ adaptée ;
- **concevoir** une gestion des identités et des accès sécurisée avec IAM (rôles temporaires, moindre privilège, MFA, traçabilité) ;
- **concevoir** un VPC segmenté et choisir un service de données (relationnel, NoSQL ou cache) adapté à un besoin donné ;
- **sélectionner et configurer** les services de calcul et de stockage adaptés (EC2, Lambda, S3, EBS, EFS) en tenant compte de l'élasticité, de la disponibilité et de la reprise d'activité ;
- **automatiser** une infrastructure avec CloudFormation, l'administrer avec Systems Manager et l'évaluer avec CloudWatch et les six piliers du AWS Well-Architected Framework.

## Public et prérequis

Le parcours s'adresse aux professionnels de l'informatique, de l'exploitation, du développement, de la sécurité ou de l'architecture qui souhaitent comprendre ou consolider leur pratique d'AWS.

Les manipulations nécessitent :

- un ordinateur disposant d'un navigateur web récent ;
- une connexion Internet stable ;
- l'accès aux environnements temporaires remis en début de formation.

Aucun compte AWS personnel, aucune carte bancaire et aucune installation locale de la CLI AWS ne sont exigés pour suivre les activités prévues.

## Méthode de travail

1. Lire les objectifs du chapitre.
2. Parcourir les concepts dans l'ordre proposé ou accéder directement à une notion précise.
3. Consulter le glossaire et la cheat sheet depuis les liens du chapitre.
4. Réaliser les modules et labs indiqués dans l'environnement de formation.
5. Terminer par le quiz interactif du chapitre.

## Parcours de formation

<div class="learning-card-grid">
<a class="learning-card" href="cours/chapitre-1/"><span class="learning-card__number">01</span><span class="learning-card__meta">Cloud Computing et AWS</span><strong>Fondamentaux du cloud et présentation d'AWS</strong><small>NIST, modèles cloud, virtualisation, infrastructure mondiale, services, coûts et Well-Architected.</small></a>
  <a class="learning-card" href="cours/chapitre-2/"><span class="learning-card__number">02</span><span class="learning-card__meta">Sécurité</span><strong>Identités, authentification et autorisations avec IAM</strong><small>Utilisateurs, groupes, rôles, politiques, MFA, fédération, Cognito, comptes multiples et CloudTrail.</small></a>
  <a class="learning-card" href="cours/chapitre-3/"><span class="learning-card__number">03</span><span class="learning-card__meta">Stockage et calcul</span><strong>Amazon S3, EC2 et services de stockage associés</strong><small>Stockage objet, AMI, instances, EBS, EFS, tarification, équilibrage et Auto Scaling.</small></a>
  <a class="learning-card" href="cours/chapitre-4/"><span class="learning-card__number">04</span><span class="learning-card__meta">Réseau et données</span><strong>Amazon VPC et bases de données AWS</strong><small>RDS, Aurora, DynamoDB, migration, adressage, sous-réseaux, routage, sécurité et DNS.</small></a>
  <a class="learning-card" href="cours/chapitre-5/"><span class="learning-card__number">05</span><span class="learning-card__meta">Exploitation</span><strong>Automatisation, supervision et reprise d'activité</strong><small>RTO, RPO, sauvegarde, CloudFormation, Systems Manager, Elastic Beanstalk et CloudWatch.</small></a>
</div>

## Annexes du cours

<div class="download-grid"><a href="annexes/glossaire"><strong>Glossaire</strong><span>Définitions des termes techniques employés dans le cours.</span></a><a href="annexes/cheat-sheet"><strong>Cheat sheet</strong><span>Commandes, syntaxes et repères à consulter rapidement.</span></a></div>

## Télécharger le support

<div class="download-grid"><a href="static/downloads/aws-initiation-approfondissement-markdown.zip" download><strong>Archive Markdown</strong><span>Les cinq chapitres, les annexes et leurs illustrations.</span></a><a href="assets/telechargements/aws-initiation-approfondissement.pdf" download><strong>Version PDF</strong><span>Support hors ligne et imprimable.</span></a></div>
