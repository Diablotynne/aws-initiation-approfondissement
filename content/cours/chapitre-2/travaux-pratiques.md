---
title: "Travaux pratiques — IAM et traçabilité"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Travaux pratiques — IAM et traçabilité"
---

<nav class="page-sequence"><a href="cours/chapitre-2/fiche-memoire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>

| Parcours | Modules et ateliers recensés |
|---|---|
| Cloud Foundations | Module 4 — Sécurité dans le cloud AWS ; Atelier 1 — Introduction à AWS IAM |
| Cloud Architecting | Module 3 — Sécurisation de l'accès ; Module 9 — Sécurisation de l'accès utilisateur, aux applications et aux données |
| Ateliers associés | Exploration d'IAM ; sécurisation avec Cognito ; chiffrement au repos avec AWS KMS |

<details><summary><strong>Parcours guidé</strong></summary>

1. Inventorier les utilisateurs, groupes, rôles et politiques déjà fournis par le lab.
2. Tester une opération autorisée puis une opération refusée sans élargir arbitrairement les droits.
3. Lire `Effect`, `Action`, `Resource` et `Condition` dans la politique impliquée.
4. Utiliser `aws sts get-caller-identity` lorsque CloudShell est disponible pour identifier la session temporaire.
5. Rechercher l'appel correspondant dans CloudTrail si l'activité le prévoit.

**Validation observable :** proposer un rôle temporaire, plutôt qu'une clé statique, pour une application EC2 qui lit un bucket S3 et justifier le moindre privilège.
</details>

---

<nav class="page-sequence"><a href="cours/chapitre-2/fiche-memoire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>
