---
title: "10. AWS Lambda — Le calcul sans serveur"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 10. AWS Lambda — Le calcul sans serveur"
---

<nav class="page-sequence"><a href="cours/chapitre-3/auto-scaling">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/architecture">Suivant</a></nav>

### 10.1 Pourquoi Lambda, quand on a déjà EC2 et Auto Scaling ?

Vous venez de voir comment EC2 et Auto Scaling permettent d'adapter dynamiquement une flotte de serveurs à la charge. Mais même avec Auto Scaling, une instance EC2 minimale **tourne en permanence** — vous la payez même quand elle ne traite aucune requête.

**AWS Lambda** pousse le modèle serverless plus loin : au lieu de faire tourner un serveur en continu, vous déployez une **fonction** — un bloc de code — qu'AWS exécute uniquement quand un événement le déclenche (requête HTTP, fichier déposé sur S3, message dans une file, tâche planifiée...). Entre deux exécutions, **aucune ressource ne tourne, donc rien n'est facturé**.

| | EC2 (même avec Auto Scaling) | Lambda |
|---|---|---|
| **Ce que vous gérez** | OS, runtime, mises à jour, capacité | Uniquement votre code |
| **Facturation** | À l'heure/seconde tant que l'instance tourne | À l'exécution (durée × mémoire allouée) |
| **Charge nulle** | Coût minimal non nul (au moins 1 instance) | **0 $** — aucune exécution, aucun coût |
| **Démarrage** | Minutes (boot instance) ou secondes (déjà démarrée) | Millisecondes à quelques secondes (cold start) |
| **Durée d'exécution max** | Illimitée | **15 minutes** par exécution |

### 10.2 Fonctionnement d'une fonction Lambda

Une fonction Lambda est un paquet de code (Python, Node.js, Java, Go, etc.) associé à une configuration : mémoire allouée (128 Mo à 10 Go), timeout maximal, et un ou plusieurs **triggers** — les événements qui la déclenchent.

```text
Événement déclencheur                    Fonction Lambda                Résultat
─────────────────────                    ────────────────                ────────
Requête HTTP (API Gateway)      ──►    Exécute le code       ──►    Réponse HTTP
Fichier déposé sur S3            ──►    (runtime + mémoire     ──►    Traitement du fichier
Message dans une file SQS        ──►     alloués à la demande) ──►    Traitement du message
Planification (EventBridge)      ──►                            ──►    Tâche exécutée
```

Le CPU alloué est proportionnel à la mémoire configurée — une fonction à 1 769 Mo de RAM obtient l'équivalent d'un vCPU complet. AWS gère entièrement l'infrastructure sous-jacente : vous ne choisissez ni AMI, ni type d'instance, ni Security Group pour la fonction elle-même.

### 10.3 Créer et invoquer une fonction Lambda en CLI

> [!info]
> Une activité pratique permet d’approfondir la création et l’invocation de fonctions Lambda.


Cette séquence crée une fonction Lambda Python minimale, l'invoque manuellement, puis vérifie les logs d'exécution dans CloudWatch.

```bash
# 1. Écrire le code de la fonction (fichier local)
cat > lambda_function.py << 'EOF'
def lambda_handler(event, context):
    nom = event.get('nom', 'monde')
    return {
        'statusCode': 200,
        'body': f'Bonjour, {nom} ! Fonction exécutée avec succès.'
    }
EOF

# 2. Empaqueter le code en ZIP (format attendu par Lambda)
zip function.zip lambda_function.py

# 3. Créer le rôle IAM que la fonction va assumer (permissions d'exécution)
aws iam create-role \
  --role-name formation-lambda-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# 4. Attacher la policy minimale pour écrire les logs CloudWatch
aws iam attach-role-policy \
  --role-name formation-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# 5. Créer la fonction Lambda
aws lambda create-function \
  --function-name formation-bonjour \
  --runtime python3.12 \
  --role arn:aws:iam::123456789012:role/formation-lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --memory-size 128 \
  --timeout 10

# 6. Invoquer la fonction avec un événement de test
aws lambda invoke \
  --function-name formation-bonjour \
  --payload '{"nom": "Formation AWS"}' \
  --cli-binary-format raw-in-base64-out \
  reponse.json

cat reponse.json
```

> [!tip]
> **Résultat attendu :**
> ```json
> {
>   "StatusCode": 200,
>   "ExecutedVersion": "$LATEST"
> }
> ```
> Contenu de `reponse.json` :
> ```json
> {"statusCode": 200, "body": "Bonjour, Formation AWS ! Fonction exécutée avec succès."}
> ```
> La fonction s'est exécutée en quelques centaines de millisecondes. Aucune instance EC2 n'a été provisionnée — Lambda a alloué l'environnement d'exécution le temps de traiter cette seule invocation, puis l'a libéré.


```bash
# 7. Consulter les logs d'exécution (CloudWatch Logs, créés automatiquement)
aws logs tail /aws/lambda/formation-bonjour --follow
```

> [!info]
> **Le rôle IAM est la seule "sécurité réseau" de Lambda par défaut.** Contrairement à EC2, une fonction Lambda n'a pas de Security Group tant qu'elle n'est pas explicitement rattachée à un VPC (`--vpc-config`). Une Lambda simple qui n'a besoin que d'appeler d'autres services AWS (S3, DynamoDB) via leurs API n'a généralement pas besoin d'être dans un VPC — le rôle IAM suffit à contrôler ce qu'elle a le droit de faire.


### 10.4 Comment estimer le coût de Lambda ?

Lambda facture principalement le **nombre de requêtes** et la **durée d'exécution pondérée par la mémoire allouée**. D'autres postes peuvent s'ajouter : concurrence provisionnée, stockage éphémère supplémentaire, journaux CloudWatch, transfert réseau et services déclencheurs.

```text
consommation de calcul = nombre d'exécutions × durée moyenne × mémoire allouée
coût total = requêtes + calcul + options Lambda + services associés
```

Cette formule explique le modèle sans figer un tarif. Pour comparer Lambda à EC2, utilisez le même trafic et la même exigence de disponibilité : Lambda convient souvent aux charges intermittentes, tandis qu'une capacité durable correctement dimensionnée peut être plus économique pour une charge continue. Le résultat doit être confirmé dans AWS Pricing Calculator.

📎 [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
📎 [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)

---

<nav class="page-sequence"><a href="cours/chapitre-3/auto-scaling">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/architecture">Suivant</a></nav>
