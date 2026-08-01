---
title: "14. Points importants et pièges fréquents"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 14. Points importants et pièges fréquents"
---

<nav class="page-sequence"><a href="cours/chapitre-3/conformite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/fiche-memoire">Suivant</a></nav>

| Piège | Réalité | Conséquence |
|-------|---------|------------|
| **S3 a une structure de dossiers** | Non ! C'est du stockage objet, les "dossiers" sont juste des préfixes dans les noms | Impossible de renommer les dossiers, penser en clés, pas en hiérarchies |
| **Versioning S3 ne prend pas de place supplémentaire** | Faux ! Chaque version est stockée complètement | Les coûts explosent vite si vous versionnez des fichiers volumineux |
| **Les instances EC2 garderont leurs données après arrêt** | Seulement si vous utilisez EBS persistant | Les données en instance store (stockage éphémère) sont perdues à l'arrêt |
| **On-Demand est toujours la meilleure option tarifaire** | Non : sa flexibilité se paie, mais elle évite un engagement inadapté | Comparer paiement à la demande, engagements et Spot avec la charge réelle |
| **Auto Scaling remplace les instances défaillantes instantanément** | Non, il faut le temps de démarrage (2-5 min) | Configurer les health checks correctement et accepter un délai |
| **Toute IP EC2 est durable** | Non, les IPs publiques changent à l'arrêt/redémarrage | Utiliser Elastic IP pour les IPs stables ou les DNS |
| **Un Security Group "ouvert" (0.0.0.0/0) sur tous les ports est OK si la machine n'a rien à cacher** | Non ! C'est une faille de sécurité | Les scanners de ports peuvent découvrir la machine, minimiser l'exposition |
| **EBS et S3 sont interchangeables** | Non ! EBS est un disque (bloc), S3 est du stockage objet | Choisir le bon service selon le cas d'usage |

---

<nav class="page-sequence"><a href="cours/chapitre-3/conformite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/fiche-memoire">Suivant</a></nav>
