---
title: "Travaux pratiques — VPC et bases de données"
description: "Chapitre 4 — Amazon VPC et bases de données AWS - Travaux pratiques — VPC et bases de données"
---

<nav class="page-sequence"><a href="cours/chapitre-4/fiche-memoire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/ressources">Suivant</a></nav>

| Parcours | Modules et ateliers recensés |
|---|---|
| Cloud Foundations | Module 5 — Mise en réseau et diffusion de contenu ; Module 8 — Bases de données ; Atelier 2 — Création d'un VPC et lancement d'un serveur web ; Atelier 5 — Création d'un serveur RDS |
| Cloud Architecting | Module 6 — Ajout d'une couche de base de données ; Module 7 — Création d'un environnement réseau ; Module 8 — Connexion de réseaux |
| Ateliers associés | RDS ; migration vers RDS ; création d'un VPC ; environnement réseau du café ; appairage de VPC |

<details><summary><strong>Parcours guidé</strong></summary>

1. Lire le CIDR et prévoir les subnets avant toute création.
2. Relier chaque subnet à sa table de routage et identifier la cible de la route par défaut.
3. Distinguer le routage des contrôles Security Group et NACL.
4. Placer la base dans les subnets privés prévus et n'autoriser le port qu'à partir du Security Group applicatif.
5. Pour le peering, contrôler les CIDR et les routes des deux côtés avant de tester le flux.

**Validation observable :** tracer le chemin navigateur → instance → base de données et distinguer Multi-AZ d'une réplique de lecture.
</details>

---

<nav class="page-sequence"><a href="cours/chapitre-4/fiche-memoire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/ressources">Suivant</a></nav>
