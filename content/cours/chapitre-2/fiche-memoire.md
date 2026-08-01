---
title: "Cheat sheet — Identités, autorisations et traçabilité"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - Cheat sheet — Identités, autorisations et traçabilité"
---

<nav class="page-sequence"><a href="cours/chapitre-2/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/travaux-pratiques">Suivant</a></nav>

### Choisir le bon mécanisme

| Besoin | Service ou mécanisme | À retenir |
|---|---|---|
| Accès d'un collaborateur à plusieurs comptes | IAM Identity Center | Fédération, attribution centralisée des accès et sessions temporaires |
| Identité locale à un compte pour un cas spécifique | Utilisateur IAM | Éviter les clés longues durées quand un rôle est possible |
| Permissions d'une charge de travail AWS | Rôle IAM | Identifiants temporaires fournis par AWS STS |
| Utilisateurs d'une application web ou mobile | Amazon Cognito | Ne remplace pas IAM Identity Center pour les collaborateurs |
| Organisation de plusieurs comptes | AWS Organizations | Unités d'organisation, comptes et politiques de contrôle des services |
| Chiffrement avec contrôle des clés | AWS KMS | Politique de clé et autorisations IAM doivent être cohérentes |
| Historique des appels d'API | AWS CloudTrail | Qui a appelé quelle API, quand, depuis où et avec quel résultat |

### Vocabulaire IAM

| Terme | Définition opérationnelle |
|---|---|
| Principal | Entité qui effectue une requête : utilisateur, rôle, service ou principal fédéré |
| Action | Opération d'API visée, par exemple `s3:GetObject` |
| Resource | ARN de la ressource sur laquelle porte l'action |
| Condition | Contrainte supplémentaire : MFA, adresse IP, tag, organisation ou autre clé de contexte |
| Politique d'identité | Politique attachée à un utilisateur, groupe ou rôle |
| Politique de ressource | Politique attachée à une ressource, par exemple un bucket S3 ou une clé KMS |
| Rôle | Identité assumable qui fournit une session temporaire ; ce n'est ni une personne ni un groupe |

### Évaluation d'une autorisation

```text
refus implicite par défaut
        ↓
autorisation explicite applicable ? ── non ──> refus
        ↓ oui
refus explicite applicable ? ──────── oui ──> refus
        ↓ non
      accès autorisé
```

Un refus explicite applicable l'emporte sur une autorisation. Les SCP d'AWS Organizations fixent une limite maximale ; elles n'accordent pas, à elles seules, une permission à une identité.

### Structure minimale d'une politique

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "LireLesRapports",
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": ["arn:aws:s3:::exemple-rapports/*"]
  }]
}
```

- `Sid` nomme la déclaration et facilite la relecture.
- `Action` décrit l'opération autorisée ou refusée.
- `Resource` doit viser le bon type d'ARN : le bucket et ses objets n'ont pas le même ARN.
- Une politique de confiance d'un rôle indique **qui peut assumer le rôle** ; une politique de permissions indique **ce que le rôle peut faire**.

### Diagnostic en lecture seule

```bash
# Vérifier l'identité réellement utilisée par la session
aws sts get-caller-identity

# Afficher les rôles sans exposer de secret
aws iam list-roles --query 'Roles[].[RoleName,Arn]' --output table

# Rechercher les événements récents visibles dans CloudTrail
aws cloudtrail lookup-events --max-results 10 \
  --query 'Events[].[EventTime,EventName,Username]' --output table
```

### Checklist de sécurité

- Utiliser le moindre privilège et réduire progressivement les jokers `*`.
- Préférer les rôles et sessions temporaires aux clés d'accès longue durée.
- Activer une authentification forte pour les accès humains.
- Séparer les comptes et les responsabilités plutôt que tout concentrer dans un compte.
- Protéger également les journaux, les sauvegardes et les clés de chiffrement.
- Ne jamais afficher, copier ou publier les identifiants temporaires du lab.

<nav class="page-sequence"><a href="cours/chapitre-2/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/travaux-pratiques">Suivant</a></nav>
