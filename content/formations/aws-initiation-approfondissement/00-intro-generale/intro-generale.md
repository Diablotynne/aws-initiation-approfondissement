---
title: AWS — Construire une architecture cloud pas à pas
description: Point de départ de la formation AWS, parcours pédagogique, accès AWS Academy et ressources.
---

# AWS — Construire une architecture cloud pas à pas

<section class="course-intro-hero">
  <div>
    <span class="aws-kicker">Formation guidée · AWS Academy · Du concept à l’architecture</span>
    <h2>Comprendre avant de déployer.<br>Observer avant d’automatiser.</h2>
    <p>Ce parcours relie les décisions d’architecture aux manipulations réalisées dans les environnements temporaires AWS Academy. Chaque service est introduit par le problème qu’il résout, puis replacé dans une architecture complète.</p>
    <div class="aws-actions">
      <a href="../03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud">Commencer le chapitre 1 →</a>
      <a href="http://awsacademy.com/vforcesite/LMS_Login#!/blueprint/blueprint_subscriptions/178677/131721040" target="_blank" rel="noopener">Ouvrir AWS Academy ↗</a>
    </div>
  </div>
</section>

![Progression de la formation depuis les fondations jusqu'à l'automatisation d'une architecture AWS](formations/aws-initiation-approfondissement/11-images/aws-parcours-formation.svg)

## Le fil rouge

Vous accompagnez l’évolution d’une application : d’abord comprise comme un besoin métier, elle est sécurisée, hébergée, connectée à ses données, puis automatisée et supervisée.

<div class="aws-learning-path">
  <a href="../03-cours/Chapitre%201%20-%20Fondamentaux%20du%20Cloud"><b>01</b><span>Décider</span><strong>Fondamentaux du cloud</strong><small>Modèles cloud, responsabilité partagée, régions, zones et coûts.</small></a>
  <a href="../03-cours/Chapitre%202%20-%20Securite%20et%20IAM"><b>02</b><span>Protéger</span><strong>Identités et accès</strong><small>IAM, rôles, politiques, MFA, fédération et traçabilité.</small></a>
  <a href="../03-cours/Chapitre%203%20-%20Stockage%20S3%20et%20Calcul%20EC2"><b>03</b><span>Exécuter</span><strong>Calcul et stockage</strong><small>S3, EC2, EBS, EFS, équilibrage de charge et élasticité.</small></a>
  <a href="../03-cours/Chapitre%204%20-%20VPC%20et%20Bases%20de%20donnees"><b>04</b><span>Connecter</span><strong>Réseau et données</strong><small>VPC, routage, sécurité réseau, RDS, Aurora et DynamoDB.</small></a>
  <a href="../03-cours/Chapitre%205%20-%20Automatisation%20et%20CloudFormation"><b>05</b><span>Industrialiser</span><strong>Automatisation et résilience</strong><small>CloudFormation, CloudWatch, découplage et Well-Architected.</small></a>
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
  <div><strong>3 · Manipuler</strong><p>Suivez le module ou le lab indiqué dans AWS Academy, exclusivement dans l’environnement temporaire.</p></div>
  <div><strong>4 · Vérifier</strong><p>Validez le résultat technique, expliquez-le avec vos mots puis complétez le quiz du chapitre.</p></div>
</div>

> [!IMPORTANT] Environnement de formation
> Aucun compte AWS personnel, aucune carte bancaire, aucune clé d’accès permanente et aucune installation locale ne sont nécessaires. Les commandes AWS CLI sont exécutées dans CloudShell avec le rôle temporaire du lab.

## Parcours AWS Academy

Les pages de travaux pratiques ne remplacent pas les énoncés AWS Academy. Elles fournissent un **itinéraire pédagogique** : concepts à observer, ordre conseillé, points de contrôle et questions de compréhension.

<div class="academy-cta">
  <div><strong>AWS Academy Learner Lab</strong><p>Démarrez uniquement le module ou le lab indiqué par la formatrice. Vérifiez le rôle, la région et les restrictions avant toute manipulation.</p></div>
  <a href="http://awsacademy.com/vforcesite/LMS_Login#!/blueprint/blueprint_subscriptions/178677/131721040" target="_blank" rel="noopener">Accéder à AWS Academy ↗</a>
</div>

| Étape | Action | Contrôle attendu |
|---|---|---|
| Accéder | Ouvrir le cours depuis le lien AWS Academy | Le cours attribué apparaît dans le tableau de bord |
| Préparer | Lire les objectifs et repérer la région demandée | Vous savez quelles ressources seront manipulées |
| Démarrer | Cliquer sur **Start Lab**, puis attendre l’état prêt | Le bouton **AWS** ouvre la console temporaire |
| Manipuler | Suivre l’énoncé Academy et les repères du chapitre | Chaque étape produit un résultat vérifiable |
| Expliquer | Relier la manipulation au schéma d’architecture | Vous pouvez justifier le service et sa configuration |
| Terminer | Utiliser **End Lab** lorsque la séance est terminée | La session temporaire est fermée proprement |

## Accès directs

<div class="aws-resource-grid">
  <a href="../04-exercices/"><strong>Activités AWS Academy</strong><span>Modules, labs, déroulés guidés et points de contrôle par chapitre.</span></a>
  <a href="../06-quiz/"><strong>Quiz expliqués</strong><span>Valider les acquis et comprendre chaque correction.</span></a>
  <a href="../07-annexes/"><strong>Ressources techniques</strong><span>Aide-mémoire CLI, glossaire et documentation officielle.</span></a>
</div>

## Télécharger les supports

<div class="download-grid">
  <a href="formations/aws-initiation-approfondissement/10-telechargements/AWS-Initiation-Approfondissement.markdown" download><strong>Markdown</strong><span>Télécharger les cinq chapitres dans un fichier Markdown unique.</span></a>
  <a href="formations/aws-initiation-approfondissement/10-telechargements/AWS-Initiation-Approfondissement.pdf" download><strong>PDF</strong><span>Télécharger le support de cours complet pour une consultation hors ligne.</span></a>
</div>

## Fin de parcours : badge et accompagnement

<div class="learning-mode-grid">
  <div><strong>Badge AWS Academy</strong><p>Après les modules, les labs et l’évaluation finale demandés, vérifiez dans AWS Academy que le badge du cursus est disponible. Le badge constitue l’objectif de clôture du parcours Academy.</p></div>
  <div><strong>Conversation Teams</strong><p>Utilisez la conversation Teams de votre session pour les annonces, questions et échanges avec le groupe. Son lien est communiqué directement par la formatrice et n’est pas publié sur ce site public.</p></div>
</div>

> [!TIP] Votre progression
> Ne cherchez pas à mémoriser un catalogue de services. Pour chaque composant, retenez quatre questions : quel problème résout-il, quelle est sa portée, qui le sécurise et comment vérifier son fonctionnement ?
