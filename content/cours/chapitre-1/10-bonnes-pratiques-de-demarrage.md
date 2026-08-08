---
title: "10. Bonnes pratiques de démarrage"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 10. Bonnes pratiques de démarrage"
---

<nav class="page-sequence"><a href="cours/chapitre-1/9-les-services-aws-les-plus-utilises">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/11-points-importants-et-pieges-frequents">Suivant</a></nav>

> [!info]
> L’environnement temporaire fourni pendant la formation permet de découvrir la console sans créer de compte AWS personnel.

### 10.1 Démarrer et sécuriser une session d'atelier

L'atelier fournit un compte, un rôle et des autorisations temporaires. La séquence de démarrage est la suivante :

1. Ouvrir l'atelier indiqué par la formatrice.
2. Cliquer sur **Start Lab** (le bouton de démarrage de l'environnement AWS Academy) et attendre que la console soit disponible.
3. Ouvrir la console avec le bouton **AWS** fourni par l'atelier.
4. Vérifier le rôle et la **région active** avant toute création de ressource.
5. Utiliser **AWS CloudShell** pour les commandes CLI : aucune installation ni configuration locale n'est requise.

> [!tip]
> **Ce que confirme la commande `aws sts get-caller-identity` dans CloudShell :**
> ```bash
> aws sts get-caller-identity
> ```
> ```json
> {
>     "UserId": "AROAEXAMPLE:academy-session",
>     "Account": "123456789012",
>     "Arn": "arn:aws:sts::123456789012:assumed-role/LabRole/academy-session"
> }
> ```
> L'ARN confirme que la session utilise un rôle temporaire de l'atelier. N'exécutez pas `aws configure` et ne copiez jamais d'identifiants sur votre poste.

### 10.2 Repères pour la navigation dans la console AWS

Voici les éléments essentiels à repérer dans la console pendant la démonstration.

**Éléments clés :**

1. **Sélecteur de région** : situé en haut à droite, il permet de choisir la région AWS dans laquelle les ressources seront déployées.
2. **Barre de recherche des services** : permet d'accéder rapidement à n'importe quel service AWS (EC2, S3, VPC, IAM, RDS…).
3. **Tableau de bord (Dashboard)** : affiche les services récemment utilisés et les informations de facturation.
4. **Panneau IAM** : permet de gérer les utilisateurs, groupes, rôles et stratégies de sécurité.
5. **Billing & Cost Management** : donne accès au suivi de la consommation et à la facturation.

📎 [Guide de démarrage AWS Console](https://docs.aws.amazon.com/awsconsole/latest/userguide/)
📎 [AWS Billing Documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html)
📎 [IAM Documentation](https://docs.aws.amazon.com/iam/)

Retenons que certaines ressources sont **spécifiques à une région** et que la facturation dépend de cette localisation.

> [!warning]
> **Piège fréquent — mauvaise région active :**
> Si vous créez des ressources dans une région autre que celle attendue (ex. `us-east-1` au lieu de `eu-west-3`), vous ne les verrez pas dans votre vue habituelle et continuerez à les payer. Vérifiez toujours le sélecteur de région en haut à droite de la console avant toute création de ressource.

---

<nav class="page-sequence"><a href="cours/chapitre-1/9-les-services-aws-les-plus-utilises">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/11-points-importants-et-pieges-frequents">Suivant</a></nav>
