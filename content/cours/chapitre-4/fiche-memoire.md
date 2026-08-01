---
title: "Fiche mémo — Réseau et données"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - Fiche mémo — Réseau et données"
---

<nav class="page-sequence"><a href="cours/chapitre-4/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/travaux-pratiques">Suivant</a></nav>

| Élément | Fonction |
|---|---|
| VPC | Périmètre réseau logiquement isolé dans une région. |
| Subnet | Segment limité à une AZ ; son caractère public dépend de son routage. |
| Internet Gateway | Connectivité Internet du VPC pour les ressources correctement routées et adressées. |
| NAT Gateway | Sortie initiée depuis un subnet privé, sans accepter de connexion entrante spontanée. |
| Security Group | Filtrage stateful associé à une interface réseau. |
| NACL | Filtrage stateless appliqué au niveau du subnet. |
| RDS Multi-AZ | Disponibilité et bascule ; ne constitue pas une stratégie de lecture horizontale. |
| Réplique de lecture | Mise à l'échelle des lectures ; la promotion dépend du moteur et du scénario. |

<nav class="page-sequence"><a href="cours/chapitre-4/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/travaux-pratiques">Suivant</a></nav>
