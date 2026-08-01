---
title: "12. Bonnes pratiques — Architecture hautement disponible"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 12. Bonnes pratiques — Architecture hautement disponible"
---

# 12. Bonnes pratiques — Architecture hautement disponible

<nav class="page-sequence"><a href="cours/chapitre-3/architecture">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/conformite">Suivant</a></nav>

### 12.1 Architecture résiliente S3

**Multi-région** :
- S3 est déjà multi-AZ au sein d'une région.
- Pour une résilience maximale, activer la **réplication cross-région** (CRR).
- Utile pour respecter des exigences de conformité (RGPD, etc.).

**Versioning + Lifecycle** :
- Toujours activer le versioning sur les buckets de production.
- Combiner avec des règles de cycle de vie pour éviter les coûts exponentiels.
- Exemple : garder 30 versions actives, archiver le reste en Glacier.

**Monitoring et alertes** :
- CloudWatch pour surveiller les métriques S3.
- CloudTrail pour auditer les accès et modifications.
- S3 Access Analyzer pour vérifier les politiques d'accès.

### 12.2 Architecture résiliente EC2

**Load Balancer + Auto Scaling minimum** :
- Toujours au minimum 2 instances (haute disponibilité).
- Répartir sur **plusieurs zones de disponibilité (AZ)**.
- Configurer les health checks correctement.

**Sécurité en couches** :
- Security Groups : bloquer les ports inutiles.
- IAM Roles : donner uniquement les permissions nécessaires.
- Subnets privés pour les instances sans accès Internet.

**Sauvegardes** :
- Snapshots EBS réguliers (quotidiens ou hebdomadaires).
- Externaliser les sauvegardes sur S3.
- Tester la restauration régulièrement !

### 12.3 Optimisation des coûts

**S3 Coûts** :
- Utiliser **Intelligent-Tiering** si l'accès est imprévisible.
- Activer les **lifecycle policies** agressives pour archiver.
- Monitorer la bande passante (les téléchargements hors AWS coûtent cher).

:::warning
**Attention aux postes de coût S3** — Le stockage n'est qu'une partie de la facture. Les requêtes, les transitions de classe, la récupération d'archives, la surveillance des objets et le transfert sortant peuvent devenir dominants. Pour un lac de données composé de nombreux petits objets, comptez les opérations autant que les gigaoctets. Utilisez **Cost Explorer** et les rapports de coûts pour identifier les postes réels.
:::

**EC2 Coûts** :
- Étudier les **Savings Plans** pour les charges stables, après analyse de l'utilisation réelle.
- **Spot** pour les job batch ou CI/CD tolérants aux interruptions.
- AWS Compute Optimizer pour dimensionner correctement.
- Éteindre les ressources de dev/test en fin de journée.

**Estimateur AWS** :
- Utiliser le **Pricing Calculator** pour estimer les coûts futurs.
- Vérifier les coûts inattendus via la **Cost Explorer**.

### 12.4 Performance et scalabilité

**S3 Performance** :
- Utiliser des **préfixes intelligents** pour éviter les goulots d'étranglement (ex. `2024/03/24/log-xxxxx`).
- Activer **S3 Transfer Acceleration** pour les uploads volumineux (MultiPart Upload).
- CloudFront comme CDN pour la distribution worldwide.

**EC2 Performance** :
- **Monitoring continu** : CPU, mémoire, réseau, disque.
- **Auto Scaling** sur **multiple métriques** : CPU, mémoire, débit réseau.
- Usar **Read Replicas** pour les bases de données.
- **Connexion pooling** pour les applications critiques.

---

<nav class="page-sequence"><a href="cours/chapitre-3/architecture">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/conformite">Suivant</a></nav>
