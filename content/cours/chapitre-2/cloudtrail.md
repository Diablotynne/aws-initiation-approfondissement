---
title: "6. Traçabilité et surveillance avec CloudTrail"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - 6. Traçabilité et surveillance avec CloudTrail"
---

<nav class="page-sequence"><a href="cours/chapitre-2/organizations">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/points-attention">Suivant</a></nav>

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
> **Protéger la journalisation CloudTrail.** Surveillez notamment `DeleteTrail` et `StopLogging`, centralisez les journaux et définissez leur durée de conservation à partir des exigences réglementaires et internes applicables. S3 Object Lock peut contribuer à une stratégie d'immutabilité lorsque sa configuration répond au besoin retenu.


### 6.10 Comprendre les coûts de CloudTrail et d'AWS Config

La facture dépend des fonctions activées et du volume observé. Il faut donc raisonner sur les **unités de consommation**, puis appliquer les tarifs affichés pour la région et la date de l'estimation.

| Service | Principaux facteurs de coût | Point de contrôle |
|---|---|---|
| **CloudTrail** | Types d'événements collectés, volume d'événements, nombre de copies de trails, stockage et analyse des journaux | Sélectionner uniquement les événements nécessaires à l'objectif d'audit ; distinguer événements de gestion et événements de données |
| **AWS Config** | Nombre de ressources enregistrées, fréquence des changements, évaluations de règles et éventuels packs de conformité | Limiter l'enregistrement et les règles au périmètre réellement surveillé |

**Méthode d'estimation** : relever les volumes dans le compte, choisir la région dans la page tarifaire officielle, puis calculer séparément la collecte, l'évaluation, l'analyse et le stockage. Une estimation sans région, sans période et sans volume n'est pas exploitable.

📎 [AWS CloudTrail Pricing](https://aws.amazon.com/cloudtrail/pricing/)
📎 [AWS Config Pricing](https://aws.amazon.com/config/pricing/)

---

<nav class="page-sequence"><a href="cours/chapitre-2/organizations">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/points-attention">Suivant</a></nav>
