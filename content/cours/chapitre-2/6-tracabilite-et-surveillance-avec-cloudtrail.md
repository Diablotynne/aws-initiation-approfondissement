---
title: "6. Traçabilité et surveillance avec CloudTrail"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 6. Traçabilité et surveillance avec CloudTrail"
---

<nav class="page-sequence"><a href="cours/chapitre-2/5-strategie-multi-comptes-avec-aws-organizations">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/7-gestion-pratique-diam-avec-la-cli">Suivant</a></nav>

### 6.1 Pourquoi surveiller les activités dans AWS ?

En environnement cloud, les utilisateurs peuvent créer, modifier ou supprimer des ressources **en quelques secondes**, sans passer par un processus de validation centralisé comme dans un datacenter traditionnel. Cette vitesse est un atout pour l'agilité, mais elle transforme aussi n'importe quelle erreur de configuration ou action malveillante en un risque qui se matérialise presque instantanément — d'où la nécessité de tout tracer.

L'enjeu de **sécurité** est le plus évident : sur un compte AWS, il faut pouvoir répondre à tout moment à la question *qui* a fait *quoi*, *quand* et *depuis où*, faute de quoi une compromission de compte peut passer inaperçue pendant des semaines. Vient ensuite l'enjeu de **conformité** : de nombreux référentiels réglementaires (ISO 27001, RGPD, PCI-DSS) exigent explicitement une traçabilité des accès et des modifications, et l'absence de journal d'audit peut à elle seule faire échouer une certification. L'**audit interne** dépend lui aussi directement de cette traçabilité : en cas d'incident (suppression accidentelle d'une base de données, fuite de données), c'est l'historique des actions qui permet de reconstituer la chronologie et d'identifier l'origine du problème en quelques minutes plutôt qu'en plusieurs jours. Enfin, cette même traçabilité sert à l'**optimisation** de la gouvernance cloud : en observant qui utilise réellement quels services, une équipe peut ajuster ses permissions IAM au principe du moindre privilège plutôt que de les laisser trop larges par précaution.

Pour cela, AWS fournit **CloudTrail**, un service de **traçabilité des actions** dans un compte ou une organisation.

---

### 6.2 AWS CloudTrail — Définition

> **CloudTrail est le service AWS qui enregistre toutes les actions réalisées via la console, les API et la CLI.**

Chaque événement contient :

* Date et heure de l'action
* Identité de l'utilisateur (ou rôle IAM)
* Adresse IP d'origine
* Service AWS concerné
* Action exécutée (ex. `ConsoleLogin`, `RunInstances`, `DeleteBucket`)

CloudTrail permet ainsi :

* de **visualiser l'historique des actions** dans la console,
* de **stocker les logs dans S3**,
* et de les **exploiter dans CloudWatch ou EventBridge** pour créer des alertes.

---

### 6.3 CloudTrail et AWS Organizations

Lorsque vous utilisez **AWS Organizations**, vous pouvez :

* Activer **un seul trail au niveau du compte de management**
* Appliquer ce trail à **tous les comptes enfants** (OU ou organisation complète)
* Centraliser les logs dans un **seul bucket S3**.

Cela permet de suivre les activités de tous les comptes de votre organisation depuis un **point unique**.

---

### 6.4 Exemples d'événements courants enregistrés

| Événement CloudTrail | Description | Cas d'usage |
|---|---|---|
| `ConsoleLogin` | Connexion à la console AWS | Détecter les connexions non MFA |
| `RunInstances` | Création d'une instance EC2 | Suivi de consommation / sécurité |
| `CreateBucket` | Création d'un bucket S3 | Audit stockage |
| `PutUserPolicy` | Modification d'une policy IAM | Traçabilité sécurité IAM |
| `DeleteTrail` | Suppression du trail CloudTrail | Détection activité critique |

---

### 6.5 CloudTrail vs CloudWatch

| CloudTrail | CloudWatch |
|---|---|
| Enregistre les événements historiques | Surveille les métriques et ressources |
| Trace les actions utilisateurs/API | Crée des alertes sur seuils ou logs |
| Stocke les logs dans S3 | Affiche en temps réel |
| Focus "qui a fait quoi" | Focus "ce qui se passe" |

Les deux sont complémentaires :
- CloudTrail = audit et traçabilité
- CloudWatch = surveillance opérationnelle

---

### 6.6 Exploiter les journaux CloudTrail : de la traçabilité à l'analyse

CloudTrail ne se limite pas à enregistrer les actions : il permet aussi de les **analyser**, de les **interroger**, et de **restreindre leur accès** selon les rôles. Pour cela, AWS propose une chaîne d'outils complémentaires, chacun jouant un rôle précis dans le traitement des logs.

#### Étapes d'intégration : de CloudTrail à Lake Formation

1. **Stockage dans S3**

   Les événements CloudTrail sont automatiquement archivés dans un bucket S3, sous forme de fichiers JSON. Ce stockage est durable, centralisé, et interrogeable.

2. **Catalogage avec AWS Glue**

   Glue agit comme un annuaire technique : il décrit la structure des fichiers (colonnes, types, formats) et les rend accessibles aux outils d'analyse comme Athena. Sans Glue, Athena ne saurait pas comment lire les logs.

3. **Analyse avec Amazon Athena**

   Athena permet d'exécuter des requêtes SQL directement sur les fichiers CloudTrail dans S3. C'est un moteur d'analyse serverless, sans base de données à déployer. On peut par exemple extraire tous les événements `DeleteBucket` du mois ou filtrer par utilisateur IAM.

4. **Gouvernance avec AWS Lake Formation**

   Lake Formation ajoute une couche de sécurité fine : il permet de contrôler **qui peut accéder à quelles données**, jusqu'au niveau colonne. Cela garantit que seuls les rôles autorisés peuvent interroger les logs sensibles.

📎 [Analyser CloudTrail avec Athena](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html)

---

### 6.7 AWS Config vs CloudTrail

Si CloudTrail trace les **actions**, AWS Config trace les **états**. Ensemble, ils offrent une vision complète du comportement et de la conformité des ressources AWS.

| Fonction | AWS Config | AWS CloudTrail |
|----------|------------|----------|
| Ce qui est suivi | État des ressources | Actions API des utilisateurs |
| Exemple | "Ce bucket est-il toujours privé ?" | "Qui a modifié ce bucket ?" |
| Type de données | Historique de configuration | Journal d'activité |
| Objectif | Conformité, détection de dérive | Audit, traçabilité |
| Intégration | Conformance Packs, remédiation automatique | EventBridge, Athena, SIEM |

📎 [AWS Config Documentation](https://docs.aws.amazon.com/config/latest/developerguide/)

---

### 6.8 Cas d'usage

> *Un administrateur souhaite identifier tous les appels `PutUserPolicy` effectués par des utilisateurs non autorisés. Il utilise CloudTrail pour collecter les événements, Glue pour cataloguer les logs, Athena pour interroger les données, et Lake Formation pour restreindre l'accès aux résultats.*

---

### 6.9 Bonnes pratiques CloudTrail

* Activer CloudTrail **au niveau de l'organisation**.
* Centraliser les logs dans un **bucket S3 sécurisé**.
* Activer la **chiffrement SSE-S3 ou SSE-KMS**.
* Mettre en place des **alertes EventBridge** sur les événements sensibles :
  * Connexion root
  * Connexion sans MFA
  * Suppression de logs
  * Création de ressources critiques
* Définir une **politique de rétention** et un plan d'audit régulier.

📎 [AWS CloudTrail Console](https://console.aws.amazon.com/cloudtrail/)

📎 [Documentation CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)

📎 [ConsoleLogin event](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-aws-console-sign-in-events.html)

📎 [CloudTrail + Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-cloudtrail.html)

> [!warning]
> **Ne jamais désactiver CloudTrail en production.** La suppression ou la désactivation d'un trail est elle-même un événement critique enregistré. Configurez des alertes EventBridge sur l'événement `DeleteTrail` et `StopLogging`. En conformité ISO 27001 ou PCI-DSS, les logs CloudTrail doivent être conservés au minimum 1 an et être immuables (activer S3 Object Lock).

### 6.10 Combien coûtent CloudTrail et Config ?

Ces deux services ne sont **pas gratuits au-delà d'un premier niveau minimal** — un point souvent ignoré, car "activer l'audit" semble aller de soi sans qu'on en vérifie le coût.

| Service | Gratuit | Facturé |
|---|---|---|
| **CloudTrail** | 1 trail de gestion (événements de gestion) par région, journalisé automatiquement, consultable 90 jours dans l'historique des événements | Trails supplémentaires : 2,00 $ par 100 000 événements de gestion. Événements de données (S3 objet par objet, invocations Lambda) : 0,10 $ par 100 000 événements — peut grimper vite sur un bucket S3 à fort trafic |
| **AWS Config** | — (pas de niveau gratuit) | 0,003 $ par élément de configuration enregistré, plus 0,001 à 0,0012 $ par évaluation de règle de conformité |

**Exemple concret** : un compte avec 500 ressources suivies par Config, réévaluées 4 fois par jour (2 000 évaluations/jour ≈ 60 000/mois) sur 10 règles de conformité coûte environ 500 × 0,003 $ (enregistrement) + 60 000 × 10 × 0,001 $ (évaluations) = 1,50 $ + 600 $ — **le coût des évaluations de règles domine largement**, pas l'enregistrement des ressources elles-mêmes. Limiter le nombre de règles actives et leur fréquence de déclenchement est le principal levier d'optimisation.

**CloudTrail avec événements de données** : ces événements ne sont pas activés par défaut et entraînent toujours des frais. Par exemple, avec un tarif hypothétique de 0,10 USD pour 100 000 événements, dix millions d'événements représenteraient 10 USD avant les autres coûts. Ce calcul sert à comprendre l'ordre de grandeur ; vérifiez le tarif de la région et la fonctionnalité choisie sur la page officielle CloudTrail Pricing avant toute estimation. Utilisez des sélecteurs d'événements précis plutôt qu'une collecte exhaustive sans objectif d'audit.

📎 [AWS CloudTrail Pricing](https://aws.amazon.com/cloudtrail/pricing/)
📎 [AWS Config Pricing](https://aws.amazon.com/config/pricing/)

---

<nav class="page-sequence"><a href="cours/chapitre-2/5-strategie-multi-comptes-avec-aws-organizations">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/7-gestion-pratique-diam-avec-la-cli">Suivant</a></nav>
