---
title: "Cheat sheet — Automatisation, supervision et reprise"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - Cheat sheet — Automatisation, supervision et reprise"
---

<nav class="page-sequence"><a href="cours/chapitre-5/certifications">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/travaux-pratiques">Suivant</a></nav>

### Choisir le bon mécanisme

| Besoin | Service ou mécanisme | Validation attendue |
|---|---|---|
| Décrire et versionner l'infrastructure | AWS CloudFormation | Template validé, changement relu, stack dans l'état attendu |
| Administrer une instance sans ouvrir SSH | AWS Systems Manager Session Manager | Instance gérée, rôle IAM correct et agent opérationnel |
| Exécuter une commande sur un parc | Systems Manager Run Command | Cibles maîtrisées, résultat et erreurs consultables |
| Déployer une application sur une plateforme gérée | AWS Elastic Beanstalk | Environnement sain et version applicative identifiable |
| Mesurer, visualiser et alerter | Amazon CloudWatch | Métrique, période, statistique, seuil et action cohérents |
| Découpler producteur et consommateur | Amazon SQS | Messages consommés, reprise sur erreur et file de lettres mortes prévue |
| Diffuser un événement à plusieurs abonnés | Amazon SNS | Abonnements confirmés et filtrage vérifié |
| Router des événements entre services | Amazon EventBridge | Pattern d'événement et cible testés |
| Définir la reprise | RPO, RTO, sauvegardes et procédure testée | Restauration mesurée, pas seulement sauvegarde déclarée |

### CloudFormation : cycle de travail

```text
template YAML/JSON → validation → change set → déploiement → événements → drift → mise à jour/suppression
```

```bash
# Vérifier la syntaxe et les propriétés comprises par CloudFormation
aws cloudformation validate-template --template-body file://template.yaml

# Consulter une stack et ses événements
aws cloudformation describe-stacks --stack-name nom-de-stack
aws cloudformation describe-stack-events --stack-name nom-de-stack \
  --query 'StackEvents[].[Timestamp,ResourceStatus,LogicalResourceId,ResourceStatusReason]' \
  --output table

# Détecter un écart entre le template et les ressources prises en charge
aws cloudformation detect-stack-drift --stack-name nom-de-stack
```

- Une **stack** est l'ensemble de ressources géré comme une unité à partir d'un template.
- Un **change set** montre les changements prévus avant leur exécution.
- Le **drift** est un écart entre l'état déclaré et l'état réel détectable par CloudFormation pour les ressources prises en charge.
- L'idempotence signifie qu'appliquer de nouveau le même état désiré ne doit pas créer une duplication non prévue.

### CloudWatch : construire une alarme utile

| Élément | Question |
|---|---|
| Métrique | Mesure-t-elle réellement le symptôme ou le risque recherché ? |
| Statistique | Moyenne, maximum, somme ou percentile : laquelle représente le phénomène ? |
| Période | La fenêtre est-elle assez courte pour réagir et assez longue pour éviter le bruit ? |
| Seuil | Est-il fondé sur une exigence ou une valeur arbitraire ? |
| Points de données | Combien de périodes doivent être en anomalie ? |
| Données manquantes | Doivent-elles être considérées comme normales, en alarme ou ignorées ? |
| Action | Qui ou quel système agit lorsque l'alarme change d'état ? |

```bash
aws cloudwatch describe-alarms \
  --query 'MetricAlarms[].[AlarmName,StateValue,MetricName,Threshold]' \
  --output table
aws logs describe-log-groups --query 'logGroups[].logGroupName' --output table
```

### SQS, SNS et EventBridge

| Service | Modèle | À retenir |
|---|---|---|
| SQS | File de messages | Le consommateur extrait puis supprime le message ; gérer visibilité, répétition et lettres mortes |
| SNS | Publication/abonnement | Un message est poussé vers une ou plusieurs destinations abonnées |
| EventBridge | Bus et règles d'événements | Les événements sont routés selon leur structure vers des cibles |

Concevoir les consommateurs comme idempotents : un même message ou événement peut être traité plus d'une fois selon le service et le scénario.

### Reprise d'activité

| Terme | Question métier |
|---|---|
| RPO — Recovery Point Objective | Jusqu'à quel point dans le passé accepte-t-on de revenir, donc quelle quantité de données peut être perdue ? |
| RTO — Recovery Time Objective | Combien de temps le service peut-il rester indisponible ? |

```text
incident
   ├── perte de données tolérée ≤ RPO
   └── délai de rétablissement ≤ RTO
```

Une sauvegarde non restaurée lors d'un test ne démontre pas la capacité de reprise. Documenter dépendances, ordre de restauration, identités, réseau, données, application et contrôles de validation.

### Relecture Well-Architected

- Excellence opérationnelle : opérations sous forme de code, observabilité et amélioration continue.
- Sécurité : identité forte, traçabilité, protection des données et défense en profondeur.
- Fiabilité : reprise automatique, capacité adaptée et tests des procédures de restauration.
- Efficacité des performances : choix technologiques et adaptation à la demande.
- Optimisation des coûts : mesure, suppression du gaspillage et arbitrage coût/valeur.
- Durabilité : ressources efficaces et réduction des traitements ou stockages inutiles.

<nav class="page-sequence"><a href="cours/chapitre-5/certifications">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/travaux-pratiques">Suivant</a></nav>
