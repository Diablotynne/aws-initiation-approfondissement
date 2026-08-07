---
title: "16. Points importants et pièges fréquents"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 16. Points importants et pièges fréquents"
---

<nav class="page-sequence"><a href="cours/chapitre-4/15-conformite-et-securite-pour-les-donnees-sensibles">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/17-ressources">Suivant</a></nav>

| Piège | Réalité | Conséquence |
|-------|---------|------------|
| **S3 a une structure de dossiers** | Non ! C'est du stockage objet, les "dossiers" sont juste des préfixes dans les noms | Impossible de renommer les dossiers, penser en clés, pas en hiérarchies |
| **Versioning S3 ne prend pas de place supplémentaire** | Faux ! Chaque version est stockée complètement | Les coûts explosent vite si vous versionnez des fichiers volumineux |
| **Les instances EC2 garderont leurs données après arrêt** | Seulement si vous utilisez EBS persistant | Les données en instance store (stockage éphémère) sont perdues à l'arrêt |
| **On-Demand est la meilleure option tarifaire** | Non, c'est la plus chère | Reserved / Spot / Savings Plans peuvent économiser 70-90% |
| **Auto Scaling remplace les instances défaillantes instantanément** | Non, il faut le temps de démarrage (2-5 min) | Configurer les health checks correctement et accepter un délai |
| **Toute IP EC2 est durable** | Non, les IPs publiques changent à l'arrêt/redémarrage | Utiliser Elastic IP pour les IPs stables ou les DNS |
| **Un Security Group "ouvert" (0.0.0.0/0) sur tous les ports est OK si la machine n'a rien à cacher** | Non ! C'est une faille de sécurité | Les scanners de ports peuvent découvrir la machine, minimiser l'exposition |
| **EBS et S3 sont interchangeables** | Non ! EBS est un disque (bloc), S3 est du stockage objet | Choisir le bon service selon le cas d'usage |

---

Réseau, données, calcul et stockage sont posés. Reste à ne plus tout faire à la main : le Chapitre 5 automatise ce déploiement avec CloudFormation, pour que la même architecture puisse être recréée à l'identique en une commande.

<nav class="page-sequence"><a href="cours/chapitre-4/15-conformite-et-securite-pour-les-donnees-sensibles">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/17-ressources">Suivant</a></nav>
