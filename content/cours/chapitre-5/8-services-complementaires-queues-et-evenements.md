---
title: "8. Services complémentaires — Queues et événements"
description: "\"Chapitre 5 — Automatisation, supervision et reprise d'activité\" - 8. Services complémentaires — Queues et événements"
---

<nav class="page-sequence"><a href="cours/chapitre-5/7-aws-well-architected-framework-mise-en-pratique">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/9-certifications-aws-objectif-saa-c03">Suivant</a></nav>

### 8.1 Amazon SQS — File d'attente de messages

**Amazon SQS** (Simple Queue Service) est une **file d'attente de messages** complètement gérée.

**Cas d'usage** : Découpler des composants d'une application.

```
Analogie : SQS est comme une BOÎTE AUX LETTRES

Sans SQS (couplage fort) :
  Producteur (app web) → appelle directement Consommateur (worker)
  Si worker est en panne → producteur attend → demande utilisateur bloquée

Avec SQS (découplage) :
  Producteur → envoie message dans SQS → retour immédiat
  Consommateur → consomme messages quand il est prêt (même en panne, pas de perte)
```

Voici la séquence complète : créer la queue, envoyer un message, le lire, puis le supprimer. La suppression explicite est obligatoire — SQS ne supprime pas automatiquement un message après lecture (pour éviter la perte en cas d'échec).

```bash
# Créer une queue SQS
aws sqs create-queue --queue-name MonQueue

# Envoyer un message
aws sqs send-message \
  --queue-url https://sqs.eu-west-1.amazonaws.com/123456789/MonQueue \
  --message-body "Bonjour depuis CloudFormation"

# Consommer un message (avec delete)
aws sqs receive-message \
  --queue-url https://sqs.eu-west-1.amazonaws.com/123456789/MonQueue \
  --max-number-of-messages 1

# Supprimer le message de la queue
aws sqs delete-message \
  --receipt-handle <receipt-handle>
```

> [!tip]
> **Résultat attendu :**
> ```
> # create-queue :
> {
>     "QueueUrl": "https://sqs.eu-west-1.amazonaws.com/123456789012/MonQueue"
> }
>
> # send-message :
> {
>     "MD5OfMessageBody": "9c7c6f0f3f748bdfa5a5e6e7c8d9e0f1",
>     "MessageId": "msg-0abc123def456789a"
> }
>
> # receive-message :
> {
>     "Messages": [
>         {
>             "MessageId": "msg-0abc123def456789a",
>             "ReceiptHandle": "AQEB...longstring...",
>             "MD5OfBody": "9c7c6f0f3f748bdfa5a5e6e7c8d9e0f1",
>             "Body": "Bonjour depuis CloudFormation"
>         }
>     ]
> }
>
> # delete-message : pas de sortie si succès (HTTP 200)
> ```

---

### 8.2 Amazon SNS — Notifications pubsub

**Amazon SNS** (Simple Notification Service) est un service de **notifications pub/sub**.

```
Différence SQS vs SNS :
  SQS : 1 producteur → 1 queue → 1 consommateur (FIFO ou parallèle)
  SNS : 1 producteur → N abonnés (email, SMS, SQS, Lambda, HTTP)
```

Voici comment créer un topic SNS, y abonner une adresse email, et publier un message qui sera envoyé à tous les abonnés simultanément :

```bash
# Créer un sujet SNS
aws sns create-topic --name MonTopicAlarmes

# Ajouter un abonnement email
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --protocol email \
  --notification-endpoint admin@example.com

# Publier un message
aws sns publish \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --message "Alerte : CPU élevé détecté !"
```

> [!tip]
> **Résultat attendu :**
> ```
> # create-topic :
> {
>     "TopicArn": "arn:aws:sns:eu-west-1:123456789012:MonTopicAlarmes"
> }
>
> # subscribe :
> {
>     "SubscriptionArn": "pending confirmation"
> }
> # → Un email est envoyé avec un lien de confirmation
>
> # publish :
> {
>     "MessageId": "pub-0abc123def456789a"
> }
> # → Tous les abonnés (email, SQS, Lambda) reçoivent le message instantanément
> ```

> **Référence** : [Amazon SQS](https://docs.aws.amazon.com/sqs/)
> **Référence** : [Amazon SNS](https://docs.aws.amazon.com/sns/)

---

### 8.3 Architectures découplées, microservices et sans serveur

Le schéma suivant rassemble les briques d'une architecture sans serveur orientée événements. L'objectif n'est pas de mémoriser une succession d'icônes : suivez le trajet d'une requête, puis identifiez les points où l'application absorbe un pic, conserve un état ou isole une panne.

<a class="schema-zoom" href="assets/schemas/aws-serverless-architecture.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-serverless-architecture.svg" alt="Architecture AWS sans serveur avec API Gateway, Lambda, files de messages et services de données" style="display:block; margin:auto; width:90%"></a>

API Gateway reçoit les appels synchrones, tandis qu'une file ou un bus d'événements permet de différer certains traitements. Lambda exécute le code sans serveur à administrer, mais le client reste responsable des permissions IAM, des dépendances, de l'idempotence, des erreurs partielles et du cycle de vie des données.

SQS et SNS résolvent le découplage entre deux composants pris isolément. Une architecture applicative complète va plus loin : elle assemble plusieurs services managés pour qu'aucun composant ne dépende directement de la disponibilité d'un autre, et que chaque brique puisse évoluer, tomber en panne ou être remplacée sans effet domino sur le reste du système. C'est le principe des **microservices** : découper une application monolithique en plusieurs services indépendants, chacun responsable d'une capacité métier précise (paiement, catalogue, notifications), communiquant entre eux par API ou par messages plutôt que par appels de fonction directs en mémoire.

**Couplage fort vs couplage faible — le test décisif**

```
Couplage fort (monolithe classique) :
  Service A appelle directement Service B (HTTP synchrone ou appel de fonction)
  → Si B est lent ou en panne, A attend, se bloque, ou échoue en cascade
  → Faire évoluer B (changer sa techno, sa capacité) impose de coordonner A

Couplage faible (architecture découplée) :
  Service A publie un événement/message, sans savoir qui le consommera
  → B (et C, D…) consomment à leur rythme, indépendamment de la disponibilité de A
  → B peut tomber, redémarrer, ou être remplacé sans qu'A ne le sache
```

Le test décisif pour repérer un couplage fort évitable : si le service producteur doit attendre une réponse synchrone du consommateur pour continuer son propre travail, alors qu'il n'a besoin d'aucune information en retour, c'est un candidat naturel au découplage via SQS ou SNS.

**Amazon API Gateway — le point d'entrée unifié**

Dans une architecture microservices, chaque service pourrait exposer sa propre adresse réseau — mais cela oblige les clients (applications mobiles, sites web, partenaires) à connaître et gérer N adresses différentes, et complique la sécurisation (authentification à répliquer partout). **Amazon API Gateway** résout ce problème en offrant un point d'entrée HTTP unique, qui route chaque requête vers le bon service backend (Lambda, conteneur, serveur EC2) selon l'URL et la méthode appelées.

```
Client mobile/web
       │
       ▼
┌─────────────────┐
│  API Gateway     │  ← authentification centralisée, throttling, cache
│  /users   → Lambda A
│  /orders  → Lambda B
│  /catalog → ECS Fargate
└─────────────────┘
```

API Gateway prend en charge, sans code supplémentaire à écrire dans chaque microservice : l'authentification (intégration IAM, Cognito, ou clé API), la limitation de débit (*throttling*) pour protéger les backends d'un pic de trafic, la mise en cache des réponses, et la transformation de requêtes/réponses (mapping de formats).

```bash
# Créer une API REST
aws apigateway create-rest-api --name "MonAPI-Catalogue"

# Créer une ressource sous la racine
aws apigateway create-resource \
  --rest-api-id <api-id> \
  --parent-id <root-resource-id> \
  --path-part "produits"

# Associer une méthode GET à une fonction Lambda (intégration proxy)
aws apigateway put-integration \
  --rest-api-id <api-id> \
  --resource-id <resource-id> \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:eu-west-1:lambda:path/2015-03-31/functions/arn:aws:lambda:eu-west-1:123456789:function:lister-produits/invocations
```

> [!tip]
> **Résultat attendu (create-rest-api) :**
> ```
> {
>     "id": "a1b2c3d4e5",
>     "name": "MonAPI-Catalogue",
>     "createdDate": "2026-07-27T10:00:00Z",
>     "apiKeySource": "HEADER",
>     "endpointConfiguration": {
>         "types": ["EDGE"]
>     }
> }
> ```

**AWS Step Functions — orchestrer plusieurs services dans un workflow**

Certains traitements ne se résument pas à un simple message transmis d'un service à l'autre : ils enchaînent plusieurs étapes avec de la logique conditionnelle (si le paiement échoue, annuler la réservation), des étapes parallèles (vérifier le stock et calculer les frais de port en même temps), et des reprises sur erreur. **AWS Step Functions** modélise ce type de workflow sous forme de machine à états (*state machine*), où chaque état est typiquement une fonction Lambda, un appel à un autre service AWS, ou une branche de décision.

```json
{
  "Comment": "Traitement d'une commande e-commerce",
  "StartAt": "VerifierStock",
  "States": {
    "VerifierStock": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:verifier-stock",
      "Next": "StockDisponible"
    },
    "StockDisponible": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.stock_ok",
          "BooleanEquals": true,
          "Next": "TraiterPaiement"
        }
      ],
      "Default": "AnnulerCommande"
    },
    "TraiterPaiement": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:traiter-paiement",
      "Catch": [
        {
          "ErrorEquals": ["PaiementRefuse"],
          "Next": "AnnulerCommande"
        }
      ],
      "Next": "ConfirmerCommande"
    },
    "ConfirmerCommande": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:confirmer-commande",
      "End": true
    },
    "AnnulerCommande": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:eu-west-1:123456789:function:annuler-commande",
      "End": true
    }
  }
}
```

L'intérêt par rapport à enchaîner ces appels directement dans le code d'une seule fonction Lambda : chaque état est visible, monitorable et rejouable indépendamment dans la console AWS (Step Functions affiche un diagramme visuel de l'exécution, avec l'état exact atteint en cas d'échec) — un débogage bien plus direct qu'une pile d'appels imbriqués dans les logs CloudWatch d'une fonction monolithique.

**Architecture microservices sans serveur complète — exemple**

L'illustration ci-dessous assemble les briques vues dans ce chapitre et les précédents en une architecture microservices sans serveur cohérente pour une application de commande en ligne :

```
                        ┌──────────────┐
   Client (web/mobile) ─▶ API Gateway   │
                        └──────┬───────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        Lambda: Catalogue  Lambda: Commande  Lambda: Compte
              │                │                │
              ▼                ▼                ▼
        DynamoDB          Step Functions    Cognito
        (produits)        (workflow         (utilisateurs)
                            commande)
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                SQS (paiement)      SNS (notifications)
                    │                     │
                    ▼                     ▼
              Lambda: Paiement     Email / SMS client
```

Aucun composant de ce schéma ne connaît directement l'adresse réseau d'un autre composant en amont : le client ne connaît que l'URL d'API Gateway, les Lambdas ne connaissent que les ARN des ressources qu'elles invoquent, et la communication asynchrone (SQS/SNS) élimine toute dépendance temporelle stricte entre le traitement de la commande et l'envoi de la notification. C'est cette absence de dépendance directe qui permet à chaque brique d'être mise à l'échelle, remplacée ou de tomber en panne sans effet domino sur le reste de l'architecture — le principe même du découplage appliqué à l'échelle d'un système complet.

> [!warning]
> **Piège fréquent :** multiplier les microservices sans réel besoin métier augmente la complexité opérationnelle (plus de composants à surveiller, plus de latence réseau entre services, plus de scénarios d'échec partiel à gérer) sans bénéfice proportionnel. Le découpage en microservices se justifie quand des équipes différentes doivent déployer indépendamment, quand des composants ont des besoins de mise à l'échelle très différents (le service de paiement encaisse un pic le vendredi soir, le catalogue reste stable), ou quand la résilience d'un composant ne doit jamais bloquer les autres. Un monolithe bien structuré reste souvent le bon choix pour une application simple ou une petite équipe.

> **Référence** : [Amazon API Gateway](https://docs.aws.amazon.com/apigateway/)
> **Référence** : [AWS Step Functions](https://docs.aws.amazon.com/step-functions/)

---

<nav class="page-sequence"><a href="cours/chapitre-5/7-aws-well-architected-framework-mise-en-pratique">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/9-certifications-aws-objectif-saa-c03">Suivant</a></nav>
