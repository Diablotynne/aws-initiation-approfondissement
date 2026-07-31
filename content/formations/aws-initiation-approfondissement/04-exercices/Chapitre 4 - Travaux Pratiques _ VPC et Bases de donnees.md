# Chapitre 4 — Travaux Pratiques : Réseau et Bases de données — VPC, RDS, DynamoDB, Route 53

> [!NOTE]
> Le noyau recommandé couvre un VPC segmenté, sa sécurité, une sortie privée et une base hautement disponible. Les labs Peering, Flow Logs, DynamoDB et DMS sont conservés comme approfondissements possibles ; le formateur annonce la sélection selon la plateforme et le temps disponible.
---

## 🎓 AWS Academy Cloud Foundations — Stagiaires académiques

**Modules associés à ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 5 | Mise en réseau et diffusion de contenu (VPC, Route 53, CloudFront) | ✅ À compléter |
| Module 8 | Bases de données (RDS, DynamoDB, Redshift, Aurora) | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre | Points |
|---------|-------|--------|
| **Atelier 2** | Création de votre VPC et lancement d'un serveur web | 100 pts |
| **Atelier 5** | Création d'un serveur de base de données (RDS) | 100 pts |

---

## 🎓 AWS Academy Cloud Architecting — Stagiaires académiques

**Modules à compléter pour ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 6 | Ajout d'une couche de base de données | ✅ À compléter |
| Module 7 | Création d'un environnement réseau | ✅ À compléter |
| Module 8 | Connexion de réseaux | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre |
|---------|-------|
| Atelier guidé | Création d'une base de données Amazon RDS |
| Atelier Défi | Migration d'une base de données vers Amazon RDS |
| Atelier guidé | Création d'un cloud privé virtuel (VPC) |
| Atelier Défi | Création d'un environnement réseau VPC pour le café |
| Atelier guidé | Création d'une connexion d'appairage de VPC |

> [!NOTE]
> Le Module 8 d'Architecting (Transit Gateway, VPN Site-to-Site, Direct Connect) prolonge directement la section 2.5 du cours de ce chapitre — c'est le niveau approfondi de l'interconnexion réseau multi-VPC/multi-comptes déjà introduite avec le VPC Peering.
---

## Déroulé guidé des modules et ateliers

<div class="lab-route">
<details open><summary><strong>Foundations M5 + Atelier 2 — VPC et serveur web</strong></summary>

1. Lire le CIDR du VPC et prévoir les subnets avant de créer une ressource.
2. Suivre l’atelier Academy pour créer le VPC, les subnets et la passerelle Internet.
3. Associer chaque subnet à sa table de routage et lire la cible de la route par défaut.
4. Lancer le serveur web demandé et limiter le Security Group aux flux utiles.
5. Tester l’accès, puis suivre le paquet depuis le navigateur jusqu’à l’instance.

**Validation** : citer les conditions nécessaires pour qu’une instance soit réellement accessible depuis Internet.
</details>

<details><summary><strong>Architecting M7 et M8 — Réseaux privés et interconnexion</strong></summary>

1. Dans l’atelier VPC, distinguer subnet public, subnet privé, Internet Gateway et NAT Gateway.
2. Vérifier la différence entre routage et filtrage avec les tables, NACL et Security Groups.
3. Dans l’atelier de peering, contrôler les CIDR des deux VPC avant la connexion.
4. Ajouter les routes demandées des deux côtés puis tester la connectivité.
5. Comparer peering, Transit Gateway, VPN et Direct Connect selon l’échelle et le contexte.

**Validation** : expliquer pourquoi une connexion de peering sans routes ne transporte aucun trafic.
</details>

<details open><summary><strong>Foundations M8 + Atelier 5 — Amazon RDS</strong></summary>

1. Choisir moteur, classe, stockage, réseau et stratégie de disponibilité selon l’énoncé.
2. Placer la base dans les subnets privés prévus par le lab.
3. Autoriser le port de la base uniquement depuis le Security Group applicatif.
4. Se connecter depuis la ressource cliente du lab et exécuter la requête de validation.
5. Repérer endpoint, sauvegardes, maintenance, Multi-AZ et métriques.

**Validation** : distinguer une instance Multi-AZ d’une réplique de lecture.
</details>

<details><summary><strong>Architecting M6 — RDS, migration et DynamoDB</strong></summary>

1. Comparer les modèles relationnel et clé-valeur à partir des modèles d’accès.
2. Dans le défi de migration RDS, inventorier source, cible, schéma et contrôles de cohérence.
3. Pour DynamoDB, définir partition key et éventuelle sort key avant de créer la table.
4. Lire une donnée avec sa clé puis expliquer l’usage d’un index secondaire.

**Validation** : choisir RDS pour les jointures et DynamoDB pour des accès prévisibles à grande échelle.
</details>
</div>

- [ ] Je sais lire une table de routage.
- [ ] Je distingue Security Group et NACL.
- [ ] Je peux sécuriser le chemin application → base de données.
- [ ] Je peux justifier RDS ou DynamoDB à partir des accès attendus.

---

*Fin du Chapitre 4 — Travaux Pratiques*
