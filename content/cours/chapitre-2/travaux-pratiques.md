---
title: "Travaux pratiques — Sécurité et gestion des accès"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Travaux pratiques — Sécurité et gestion des accès"
---

<nav class="page-sequence"><a href="cours/chapitre-2/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/quiz">Suivant</a></nav>

### AWS Academy — Cloud Foundations

| Type | Référence | Intitulé |
|---|---|---|
| Module | Module 4 | Sécurité dans le Cloud AWS |
| Atelier | Atelier 1 | Introduction à AWS IAM — 100 points |
| Contrôle | Module 4 | Contrôle des connaissances du module |

### AWS Academy — Cloud Architecting

| Type | Référence | Intitulé |
|---|---|---|
| Module | Module 3 | Sécurisation de l'accès |
| Module | Module 9 | Sécurisation de l'accès utilisateur, aux applications et aux données |
| Atelier guidé | Module 3 | Exploration du service AWS IAM : utilisateurs, groupes, rôles et politiques |
| Atelier guidé | Module 9 | Sécurisation des applications à l'aide d'Amazon Cognito |
| Atelier guidé | Module 9 | Chiffrement des données au repos avec AWS KMS |

### Go Deploy

| Lab | Intitulé |
|---|---|
| Lab 21 | Gestion des identités et des accès AWS — IAM |

<details><summary><strong>Parcours guidé et validation</strong></summary>

1. Suivre le Module 4 et l'Atelier 1 Cloud Foundations pour distinguer utilisateur, groupe, rôle et politique.
2. Tester dans l'atelier un accès autorisé puis un accès refusé, et lire les éléments utiles du message d'erreur.
3. Dans Cloud Architecting, reconstituer la chaîne principal → politique → action → ressource → condition.
4. Comparer IAM Identity Center pour les collaborateurs et Cognito pour les utilisateurs d'une application.
5. Réaliser le Lab 21 Go Deploy selon l'énoncé fourni et contrôler l'effet des autorisations configurées.

**Validation observable :** expliquer pourquoi un refus explicite l'emporte, identifier l'appelant avec STS et justifier l'emploi d'un rôle temporaire plutôt que de clés permanentes.
</details>

---

<nav class="page-sequence"><a href="cours/chapitre-2/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/quiz">Suivant</a></nav>
