---
title: "10. Bonnes pratiques de démarrage"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - 10. Bonnes pratiques de démarrage"
---

<nav class="page-sequence"><a href="cours/chapitre-1/services-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/points-attention">Suivant</a></nav>

### 10.1 Repères pour la navigation dans la console AWS

Voici les éléments essentiels à repérer dans la console pendant la démonstration.

**Éléments clés :**

1. **Sélecteur de région** : situé en haut à droite, il permet de choisir la région AWS dans laquelle les ressources seront déployées.
2. **Barre de recherche des services** : permet d'accéder rapidement à n'importe quel service AWS (EC2, S3, VPC, IAM, RDS…).
3. **Tableau de bord (Dashboard)** : affiche les services récemment utilisés et les informations de facturation.
4. **Panneau IAM** : permet de gérer les utilisateurs, groupes, rôles et stratégies de sécurité.
5. **Billing & Cost Management** : donne accès au suivi de la consommation et à la facturation.

📎 [Guide AWS Management Console](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/what-is.html)
📎 [AWS Billing Documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html)
📎 [IAM Documentation](https://docs.aws.amazon.com/iam/)

Retenons que certaines ressources sont **spécifiques à une région** et que la facturation dépend de cette localisation.

> [!warning]
> **Piège fréquent — mauvaise région active :**
> Si vous créez des ressources dans une région autre que celle attendue (ex. `us-east-1` au lieu de `eu-west-3`), vous ne les verrez pas dans votre vue habituelle et continuerez à les payer. Vérifiez toujours le sélecteur de région en haut à droite de la console avant toute création de ressource.


---

<nav class="page-sequence"><a href="cours/chapitre-1/services-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/points-attention">Suivant</a></nav>
