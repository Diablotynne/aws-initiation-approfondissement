---
title: "7. AWS Well-Architected Framework"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - 7. AWS Well-Architected Framework"
---

<nav class="page-sequence"><a href="cours/chapitre-1/ecosysteme-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/console-aws">Suivant</a></nav>

Le **Well-Architected Framework** est une méthodologie officielle AWS qui permet de **concevoir des architectures Cloud robustes**, sécurisées, performantes, optimisées en coûts et **durables**.

C'est une **philosophie de design** qui s'applique à chaque projet AWS, du plus petit au plus grand.

Ce framework repose sur **six piliers**, souvent représentés par le sigle **SOPREC** :

<a class="schema-zoom" href="assets/schemas/well-architected-six-piliers.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/well-architected-six-piliers.svg" alt="Les six piliers AWS Well-Architected reliés à une même charge de travail"></a>

**Lecture du schéma.** Les piliers ne forment pas une suite d'étapes. Ils servent à examiner simultanément une même charge de travail et à rendre visibles les compromis d'architecture.

- 🏛️ **Security** → Sécurité, IAM, chiffrement
- ⚙️ **Operational Excellence** → Monitoring, automatisation, processus
- 💪 **Reliability** → Résilience, haute disponibilité
- ⚡ **Performance** → Efficacité des ressources
- 💰 **Cost Optimization** → Optimisation des dépenses
- 🌱 **Sustainability** → Impact environnemental, efficacité

> [!tip]
> **Vérification AWS CLI — liste des piliers WAF disponibles pour une review :**
> ```bash
> aws wellarchitected list-lenses --lens-type AWS_OFFICIAL
> ```
> ```json
> {
>     "LensSummaries": [
>         { "LensAlias": "wellarchitected", "LensName": "AWS Well-Architected Framework",
>           "LensVersion": "2023-04-10", "Description": "Six pillars review" },
>         { "LensAlias": "serverless", "LensName": "Serverless Lens", "LensVersion": "3.0" },
>         { "LensAlias": "saas", "LensName": "SaaS Lens", "LensVersion": "1.0" }
>     ]
> }
> ```
> Le Framework Well-Architected est disponible depuis la console et via API. Vous pouvez lancer une revue avec **AWS Well-Architected Tool** : les questions sont organisées selon les six piliers. Ce service ne doit pas être confondu avec **AWS WAF**, le pare-feu applicatif web.


#### 1. Security (Sécurité)

Mettre en œuvre des mesures de protection fortes à tous les niveaux — **IAM**, **chiffrement**, **sécurité réseau**, **MFA**, **audit**.

**En pratique :**
- Créer des rôles IAM spécifiques plutôt que d'utiliser le compte root.
- Chiffrer les données au repos et en transit.
- Utiliser des security groups et NACL pour isoler les ressources.
- Activer CloudTrail pour auditer les accès.

#### 2. Operational Excellence (Excellence opérationnelle)

Surveiller et améliorer continuellement les opérations — **mise en place de processus**, **d'automatisation** et de **monitoring**.

**En pratique :**
- Utiliser CloudWatch pour surveiller les ressources.
- Automatiser les déploiements avec CloudFormation ou Terraform.
- Documenter les procédures d'incident.
- Faire des reviews régulières de l'architecture.

#### 3. Reliability (Fiabilité / Résilience)

Assurer la **résilience** et la **tolérance aux pannes** — **redondance**, **sauvegarde**, **failover automatique**.

**En pratique :**
- Déployer les applications dans plusieurs **Availability Zones** pour la haute disponibilité.
- Mettre en place des sauvegardes automatiques (EBS snapshots, RDS backups).
- Utiliser des auto-scaling groups pour ajuster le nombre d'instances en cas de charge.
- Tester régulièrement les scénarios de récupération d'urgence (RTO/RPO).

#### 4. Performance Efficiency (Efficacité des performances)

Utiliser les ressources de manière **efficace et évolutive** — **choix des bonnes instances**, **optimisation des requêtes**.

**En pratique :**
- Choisir le bon type d'instance EC2 selon le workload (t3 pour web, m5 pour applications, c5 pour calcul).
- Utiliser des caches (ElastiCache, CloudFront) pour réduire la latence.
- Optimiser les requêtes de bases de données.
- Utiliser AWS Compute Optimizer pour recommander les bonnes tailles.

#### 5. Cost Optimization (Optimisation des coûts)

Éviter les dépenses inutiles et **ajuster la capacité à la demande** — **monitoring des coûts**, **réservation d'instances**, **suppression des ressources non utilisées**.

**En pratique :**
- Utiliser des **Reserved Instances** ou **Savings Plans** pour réduire les coûts.
- Mettre en place des **budgets d'alerte** dans AWS Billing.
- Arrêter les ressources non utilisées (instances EC2, RDS développement).
- Analyser les logs de coûts avec Cost Explorer ou AWS Cost Anomaly Detection.

#### 6. Sustainability (Durabilité) — **Le pilier le plus récent** ✨

Minimiser l'**impact environnemental** de votre infrastructure Cloud — **efficacité énergétique**, **utilisation optimale des ressources**, **choix des régions**.

**En pratique :**
- Utiliser les régions AWS qui utilisent des énergies renouvelables (ex : `eu-west-1` en Irlande, alimentée à >80% par des renouvelables).
- Éteindre les ressources inutilisées plutôt que de les laisser tourner.
- Utiliser le **Compute Optimizer** pour ajuster les tailles d'instances et réduire la consommation.
- Préférer les architectures **serverless** (Lambda, Fargate) qui consomment moins que les EC2 permanentes.
- Monitorer l'empreinte carbone de vos déploiements via le **AWS Customer Carbon Footprint Tool**.

C'est un enjeu de plus en plus important pour les entreprises souhaitant atteindre leurs **objectifs de développement durable (ODD)** et de **net-zéro** (carbon-neutral).

📎 [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
📎 [AWS Sustainability — Carbon Footprint Tool](https://aws.amazon.com/about-aws/sustainability/)
#### Appliquer le Framework au quotidien

Lors de **chaque conception d'architecture AWS**, on doit se poser ces questions :

- **🔒 Sécurité** : Les données sont-elles protégées ? IAM bien configuré ?
- **⚙️ Opérations** : Comment monitorer et alerter ? Comment déployer sans risque ?
- **💪 Résilience** : Que se passe-t-il si un service tombe ? Avons-nous des sauvegardes ?
- **⚡ Performance** : Les ressources sont-elles bien dimensionnées ? Y a-t-il des goulots ?
- **💰 Coûts** : Payons-nous ce que nous utilisons réellement ? Pouvons-nous optimiser ?
- **🌱 Durabilité** : Quel est notre impact environnemental ? Pouvons-nous le réduire ?

Cette approche sera **reprise au fil des jours**, notamment lors des ateliers EC2, S3, RDS et VPC.

---

<nav class="page-sequence"><a href="cours/chapitre-1/ecosysteme-aws">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/console-aws">Suivant</a></nav>
