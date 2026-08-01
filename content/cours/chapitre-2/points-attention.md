---
title: "7. Points importants et pièges fréquents"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - 7. Points importants et pièges fréquents"
---

# 7. Points importants et pièges fréquents

<nav class="page-sequence"><a href="cours/chapitre-2/cloudtrail">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/choix-authentification">Suivant</a></nav>

| Piège courant | Réalité | Solution |
|---|---|---|
| "Les SCP donnent des permissions" | Les SCP **limitent** les permissions, elles ne les donnent pas. | Toujours combiner SCP + policies IAM. |
| "Un Deny peut être contourné par un Allow" | Un Deny **explicite** est prioritaire. Toujours. | Reconnaître que Deny > Allow dans l'évaluation. |
| "IAM et Cognito, c'est pareil" | IAM = accès AWS administratif. Cognito = accès application. | Utiliser IAM pour IT, Cognito pour utilisateurs finaux. |
| "MFA, c'est juste un code SMS" | MFA peut être TOTP, YubiKey, passkey, biométrie. | Proposer plusieurs types selon la sensibilité. |
| "Pas besoin de fédération si on a IAM" | La fédération centralise la gestion et réduit les comptes statiques. | Préférer la fédération en environnement d'entreprise. |
| "CloudTrail ralentit AWS" | CloudTrail est activé implicitement et n'impacte pas les perfs. | L'activer sans crainte pour l'audit. |
| "Un utilisateur sans policy n'a aucun accès" | Correct : le moindre privilège s'applique par défaut. | Toujours attacher une policy minimale. |
| "On peut récupérer une clé d'accès perdue" | Non. Les clés ne s'affichent qu'à la création. | Conserver les clés en lieu sûr, utiliser AWS Secrets Manager. |

---

<nav class="page-sequence"><a href="cours/chapitre-2/cloudtrail">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/choix-authentification">Suivant</a></nav>
