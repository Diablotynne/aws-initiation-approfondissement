---
title: AWS — Construire une architecture cloud pas à pas
description: Point de départ de la formation AWS, parcours pédagogique et ressources.
---

# AWS — Construire une architecture cloud pas à pas

<section class="course-intro-hero">
  <div>
    <span class="aws-kicker">Formation guidée · Du concept à l’architecture</span>
    <h2>Comprendre avant de déployer.<br>Observer avant d’automatiser.</h2>
    <p>Ce parcours relie les décisions d’architecture aux manipulations réalisées pendant la formation. Chaque service est introduit par le problème qu’il résout, puis replacé dans une architecture complète.</p>
    <div class="aws-actions">
      <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud">Commencer le chapitre 1 →</a>
    </div>
  </div>
</section>

![Progression de la formation depuis les fondations jusqu'à l'automatisation d'une architecture AWS](formations/aws-initiation-approfondissement/11-images/aws-parcours-formation.svg)

## Le fil rouge

Vous accompagnez l’évolution d’une application : d’abord comprise comme un besoin métier, elle est sécurisée, hébergée, connectée à ses données, puis automatisée et supervisée.

<div class="aws-learning-path">
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud"><b>01</b><span>Décider</span><strong>Fondamentaux du cloud</strong><small>Modèles cloud, responsabilité partagée, régions, zones et coûts.</small></a>
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%202%20-%20Securite%20et%20IAM"><b>02</b><span>Protéger</span><strong>Identités et accès</strong><small>IAM, rôles, politiques, MFA, fédération et traçabilité.</small></a>
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%203%20-%20Stockage%20S3%20et%20Calcul%20EC2"><b>03</b><span>Exécuter</span><strong>Calcul et stockage</strong><small>S3, EC2, EBS, EFS, équilibrage de charge et élasticité.</small></a>
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%204%20-%20VPC%20et%20Bases%20de%20donnees"><b>04</b><span>Connecter</span><strong>Réseau et données</strong><small>VPC, routage, sécurité réseau, RDS, Aurora et DynamoDB.</small></a>
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%205%20-%20Automatisation%20et%20CloudFormation"><b>05</b><span>Industrialiser</span><strong>Automatisation et résilience</strong><small>CloudFormation, CloudWatch, découplage et Well-Architected.</small></a>
</div>

## Ce que vous saurez faire

À la fin du parcours, vous pourrez :

- expliquer le fonctionnement du cloud et situer les responsabilités d’AWS et du client ;
- choisir une région, un modèle de service et les composants adaptés à un besoin ;
- appliquer le moindre privilège avec les identités, rôles et politiques IAM ;
- associer calcul, stockage, réseau et données dans une architecture cohérente ;
- identifier les points uniques de défaillance et proposer une architecture résiliente ;
- automatiser une infrastructure avec CloudFormation ;
- observer son comportement avec les métriques, journaux et alarmes AWS ;
- justifier vos choix avec les principes du AWS Well-Architected Framework.

## Comment travailler avec ce support

<div class="learning-mode-grid">
  <div><strong>1 · Comprendre</strong><p>Lisez le schéma et formulez le problème résolu avant d’étudier le service.</p></div>
  <div><strong>2 · Décider</strong><p>Répondez aux situations proposées et comparez votre raisonnement à la correction dépliable.</p></div>
  <div><strong>3 · Manipuler</strong><p>Suivez les consignes et les accès communiqués directement par la formatrice.</p></div>
  <div><strong>4 · Vérifier</strong><p>Validez le résultat technique, expliquez-le avec vos mots puis complétez le quiz du chapitre.</p></div>
</div>

> [!IMPORTANT] Environnement de formation
> Aucun compte AWS personnel, aucune carte bancaire, aucune clé d’accès permanente et aucune installation locale ne sont nécessaires. Les commandes AWS CLI sont exécutées dans CloudShell avec le rôle temporaire du lab.

## Accès directs

<div class="aws-resource-grid">
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud"><strong>Commencer le cours</strong><span>Parcourir les chapitres dans l’ordre, depuis les fondamentaux jusqu’à l’automatisation.</span></a>
  <a href="formations/aws-initiation-approfondissement/03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud#quiz-interactif-du-chapitre"><strong>Quiz interactifs</strong><span>Répondre en fin de chapitre et afficher l’explication uniquement après le clic.</span></a>
  <a href="formations/aws-initiation-approfondissement/07-annexes/glossaire-aws-initiation-approfondissement"><strong>Glossaire AWS</strong><span>Retrouver les principaux services, acronymes et concepts techniques.</span></a>
</div>

## Télécharger les supports

<div class="download-grid">
  <a href="formations/aws-initiation-approfondissement/10-telechargements/AWS-Initiation-Approfondissement.markdown" download><strong>Markdown</strong><span>Télécharger les cinq chapitres dans un fichier Markdown unique.</span></a>
  <a href="formations/aws-initiation-approfondissement/10-telechargements/AWS-Initiation-Approfondissement.pdf" download><strong>PDF</strong><span>Télécharger le support de cours complet pour une consultation hors ligne.</span></a>
</div>

## Accompagnement

<div class="learning-mode-grid">
  <div><strong>Conversation Teams</strong><p>Utilisez la conversation Teams de votre session pour les annonces, questions et échanges avec le groupe. Son lien est communiqué directement par la formatrice et n’est pas publié sur ce site public.</p></div>
</div>

> [!TIP] Votre progression
> Ne cherchez pas à mémoriser un catalogue de services. Pour chaque composant, retenez quatre questions : quel problème résout-il, quelle est sa portée, qui le sécurise et comment vérifier son fonctionnement ?
