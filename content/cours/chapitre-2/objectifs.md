---
title: "Objectifs du chapitre"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Objectifs du chapitre"
---

<nav class="page-sequence"><a href="cours/chapitre-2/index">Sommaire du chapitre</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/vocabulaire">Suivant</a></nav>

> [!info]
> Le chapitre précédent a posé les bases théoriques du Cloud Computing et présenté l'écosystème AWS ; ce chapitre entre dans le concret en abordant le premier pilier opérationnel de toute architecture AWS : la sécurité des accès.
>
> À l'issue de ce chapitre, vous saurez :
>
> - **Expliquer** le rôle stratégique d'IAM et ses concepts fondamentaux (users, groups, roles, policies)
> - **Rédiger** une politique IAM au format JSON et comprendre la logique d'évaluation des permissions
> - **Appliquer** le modèle de responsabilité partagée AWS au domaine de la sécurité
> - **Sécuriser** un compte utilisateur avec le MFA et des politiques conditionnelles
> - **Utiliser** AWS STS pour délivrer des identités temporaires (identity broker)
> - **Mettre en œuvre** une fédération d'identité avec IAM Identity Center, SAML et AD FS
> - **Distinguer** IAM et Amazon Cognito pour la gestion des identités applicatives
> - **Concevoir** une stratégie multi-comptes avec AWS Organizations et des Service Control Policies (SCP)
> - **Configurer** la traçabilité des activités avec AWS CloudTrail et la comparer à AWS Config
> - **Créer** via la CLI des utilisateurs, groupes, rôles et policies IAM, et activer le MFA

<a class="schema-zoom" href="assets/schemas/evaluation-autorisation-iam.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/evaluation-autorisation-iam.svg" alt="Chemin d'évaluation d'une requête AWS par IAM"></a>

**Lecture du schéma.** IAM évalue la requête dans son contexte complet. Un refus explicite prévaut ; en son absence, une autorisation explicite doit permettre l'action sur la ressource concernée.

---

<nav class="page-sequence"><a href="cours/chapitre-2/index">Sommaire du chapitre</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/vocabulaire">Suivant</a></nav>
