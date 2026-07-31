# Chapitre 2 — Travaux Pratiques : Sécurité et gestion des accès — IAM, MFA, CloudTrail, Organizations

> [!NOTE]
> Le noyau recommandé est l'atelier IAM Cloud Foundations et le Module 3 Architecting ciblé. Le Module 9 et ses ateliers constituent un approfondissement : la formatrice précise la sélection réalisable pendant la séance.
---

## 🎓 AWS Academy Cloud Foundations — Stagiaires académiques

**Module associé à ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 4 | Sécurité dans le Cloud AWS | ✅ À compléter |

**Atelier guidé :**

| Atelier | Titre | Points |
|---------|-------|--------|
| **Atelier 1** | Introduction à AWS IAM | 100 pts |

---

## 🎓 AWS Academy Cloud Architecting — Stagiaires académiques

**Modules associés à ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 3 | Sécurisation de l'accès | ✅ À compléter |
| Module 9 | Sécurisation de l'accès utilisateur, aux applications et aux données | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre |
|---------|-------|
| Atelier guidé | Exploration du service AWS IAM (utilisateurs, groupes, rôles, politiques) |
| Atelier guidé | Sécurisation des applications à l'aide d'Amazon Cognito |
| Atelier guidé | Chiffrement des données au repos avec AWS KMS |

> [!NOTE]
> Le Module 3 d'Architecting complète le Module 4 de Foundations sur les fondations IAM (utilisateurs, groupes, rôles, politiques) ; le Module 9 va plus loin avec la fédération d'utilisateurs, la gestion multi-comptes et le chiffrement KMS — cohérent avec les sections 3 à 6 du cours de ce chapitre (SSO/IAM Identity Center, Cognito, Organizations, CloudTrail).
---

## Déroulé guidé des modules et ateliers

<div class="lab-route">
<details open><summary><strong>Foundations M4 + Atelier 1 — Introduction à IAM</strong></summary>

1. Dans le module, distinguer authentification et autorisation, puis identité durable et rôle temporaire.
2. Démarrer l’atelier IAM depuis AWS Academy et ouvrir la console temporaire.
3. Inventorier les utilisateurs, groupes, rôles et politiques déjà fournis par le lab.
4. Suivre l’énoncé Academy pour associer les permissions demandées.
5. Tester un accès autorisé puis un accès refusé ; lire le message d’erreur au lieu de modifier les droits au hasard.
6. Vérifier dans la politique les champs `Effect`, `Action`, `Resource` et, lorsqu’il existe, `Condition`.

**Validation** : expliquer pourquoi l’appartenance à un groupe ne suffit pas si une autre politique contient un refus explicite.
</details>

<details><summary><strong>Architecting M3 — Sécurisation de l’accès</strong></summary>

1. Examiner la différence entre utilisateur IAM, rôle IAM et session STS.
2. Identifier le rôle temporaire utilisé par le lab avec `aws sts get-caller-identity` dans CloudShell.
3. Reconstituer le chemin de décision : principal → politique → action → ressource → condition.
4. Étudier l’atelier d’exploration IAM et relever les permissions minimales nécessaires.

**Validation** : proposer un rôle, plutôt que des clés statiques, pour une application EC2 qui lit un bucket S3.
</details>

<details><summary><strong>Architecting M9 — Cognito et chiffrement</strong></summary>

1. Comparer IAM Identity Center pour les collaborateurs et Cognito pour les utilisateurs d’une application.
2. Dans l’atelier Cognito, suivre le parcours utilisateur → jeton → application → ressource autorisée.
3. Dans l’atelier de chiffrement, repérer la clé utilisée, le service qui chiffre et l’identité autorisée à déchiffrer.
4. Distinguer chiffrement au repos, chiffrement en transit et contrôle d’accès.

**Validation** : expliquer pourquoi le chiffrement ne remplace jamais une politique IAM restrictive.
</details>
</div>

- [ ] Je sais lire une politique IAM simple.
- [ ] Je sais expliquer le moindre privilège.
- [ ] Je distingue IAM, Identity Center et Cognito.
- [ ] Je sais retrouver une action dans CloudTrail.

---

*Fin du Chapitre 2 — Travaux Pratiques*
