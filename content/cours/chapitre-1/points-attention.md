---
title: "11. Points importants et pièges fréquents"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - 11. Points importants et pièges fréquents"
---

<nav class="page-sequence"><a href="cours/chapitre-1/bonnes-pratiques">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/fiche-memoire">Suivant</a></nav>

| Piège | Réalité |
|-------|---------|
| **« AWS, c'est juste des serveurs dans le cloud »** | AWS propose un catalogue étendu couvrant notamment calcul, stockage, réseau, données, IA, sécurité et exploitation. |
| **« Je suis en sécurité, AWS gère tout »** | **Non.** AWS gère la sécurité *du* cloud, vous gérez la sécurité *dans* le cloud (IAM, chiffrement, configuration). |
| **« Le Cloud public n'est pas conforme RGPD »** | **Faux.** AWS est conforme RGPD. C'est votre **usage** qui doit être conforme — localisation des données, consentement, droit à l'oubli, etc. |
| **« Les coûts AWS sont imprévisibles »** | Avec une **bonne gouvernance** (budgets, alertes, AWS Cost Explorer), les coûts sont très maîtrisables. |
| **« Un conteneur = une VM plus légère »** | **Non.** Un conteneur ne contient pas d'OS complet — il partage le noyau de l'hôte. C'est une architecture radicalement différente. |
| **« Je dois mettre toutes mes données en cloud »** | Non. Certaines données peuvent rester **on-premise** pour des raisons légales, réglementaires ou métier. Le **cloud hybride** existe pour ça. |
| **« IaaS, PaaS, SaaS — pareil pareil »** | Non. Chaque modèle **décale les responsabilités**. En IaaS, vous gérez plus ; en SaaS, AWS gère presque tout. |
| **« Je peux utiliser n'importe quelle région »** | Non. Certaines régions n'ont pas tous les services, et les données sensibles doivent rester dans des zones spécifiques (ex. RGPD en UE). |

---

Comprendre l'infrastructure ne suffit pas — encore faut-il la sécuriser. Le Chapitre 2 est entièrement consacré à la sécurité et à la gestion des identités dans AWS : comment **IAM** structure les droits et les accès, comment appliquer les bonnes pratiques de moindre privilège, et comment mettre en place une gouvernance solide.

<nav class="page-sequence"><a href="cours/chapitre-1/bonnes-pratiques">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/fiche-memoire">Suivant</a></nav>
