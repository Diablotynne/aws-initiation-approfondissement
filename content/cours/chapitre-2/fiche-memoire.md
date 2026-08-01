---
title: "Fiche mémo — Identités et accès"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Fiche mémo — Identités et accès"
---

<nav class="page-sequence"><a href="cours/chapitre-2/choix-authentification">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/travaux-pratiques">Suivant</a></nav>

| Besoin | Service ou mécanisme à examiner |
|---|---|
| Accès humain à plusieurs comptes | IAM Identity Center et sessions temporaires |
| Autorisation d'un service AWS | Rôle IAM associé au service |
| Identités d'une application cliente | Amazon Cognito |
| Limite maximale dans une organisation | Service Control Policy (SCP) |
| Journal des appels API | AWS CloudTrail |
| État attendu d'une ressource | AWS Config |

**Lecture minimale d'une politique IAM :** `Effect` indique autorisation ou refus, `Action` l'opération, `Resource` la cible et `Condition` les contraintes supplémentaires. Un refus explicite reste prioritaire.

<nav class="page-sequence"><a href="cours/chapitre-2/choix-authentification">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/travaux-pratiques">Suivant</a></nav>
