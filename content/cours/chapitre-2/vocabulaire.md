---
title: "Vocabulaire du chapitre"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Vocabulaire du chapitre"
---

# Vocabulaire du chapitre

<nav class="page-sequence"><a href="cours/chapitre-2/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/iam">Suivant</a></nav>

| Terme | Définition |
|---|---|
| Identité | Entité connue d'un système d'authentification, par exemple une personne, une application ou un service. |
| Principal AWS | Identité qui signe et envoie une requête à AWS : utilisateur IAM, rôle, service ou session fédérée. |
| Authentification | Vérification de l'identité déclarée. |
| Autorisation | Décision qui permet ou refuse une action sur une ressource après authentification. |
| Politique IAM | Document JSON qui décrit des autorisations ou des refus au moyen d'actions, de ressources et de conditions. |
| Rôle IAM | Identité AWS sans identifiants permanents, assumée pour obtenir une session temporaire. |
| MFA | Authentification multifacteur : utilisation d'au moins deux catégories de preuves distinctes. |
| Fédération | Délégation de l'authentification à un fournisseur d'identité externe à AWS. |
| SSO | Authentification unique permettant d'accéder à plusieurs applications ou comptes après une seule connexion. |

---

:::info
Le chapitre précédent a posé les bases théoriques du Cloud Computing et présenté l'écosystème AWS ; ce chapitre entre dans le concret en abordant le premier pilier opérationnel de toute architecture AWS : la sécurité des accès.

**Objectifs du chapitre**

À l'issue de ce chapitre, les stagiaires seront capables de :

- **Expliquer** le rôle stratégique d'IAM et ses concepts fondamentaux (users, groups, roles, policies)
- **Rédiger** une politique IAM au format JSON et comprendre la logique d'évaluation des permissions
- **Appliquer** le modèle de responsabilité partagée AWS au domaine de la sécurité
- **Sécuriser** un compte utilisateur avec le MFA et des politiques conditionnelles
- **Utiliser** AWS STS pour délivrer des identités temporaires (identity broker)
- **Mettre en œuvre** une fédération d'identité avec IAM Identity Center, SAML et AD FS
- **Distinguer** IAM et Amazon Cognito pour la gestion des identités applicatives
- **Concevoir** une stratégie multi-comptes avec AWS Organizations et des Service Control Policies (SCP)
- **Configurer** la traçabilité des activités avec AWS CloudTrail et la comparer à AWS Config
- **Créer** via la CLI des utilisateurs, groupes, rôles et policies IAM, et activer le MFA
:::

---

<nav class="page-sequence"><a href="cours/chapitre-2/index">Sommaire de la journ&eacute;e</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/iam">Suivant</a></nav>
