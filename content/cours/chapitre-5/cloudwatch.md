---
title: "6. Amazon CloudWatch — Supervision et alarmes"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 6. Amazon CloudWatch — Supervision et alarmes"
---

<nav class="page-sequence"><a href="cours/chapitre-5/beanstalk">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/well-architected">Suivant</a></nav>

### 6.1 Qu'est-ce que CloudWatch ?

**Amazon CloudWatch** est le service de **monitoring centralisé** d'AWS. Il collecte, stocke et affiche des métriques sur :

- **Instances EC2** : CPU, mémoire réseau, I/O disque
- **Bases RDS** : connexions actives, CPU, I/O
- **Load Balancers** : requêtes/seconde, latence
- **Applications custom** : envoi de métriques via API

<a class="schema-zoom" href="assets/schemas/cloudwatch-architecture.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/cloudwatch-architecture.svg"
     alt="Architecture Amazon CloudWatch"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Les services et applications publient métriques et journaux dans CloudWatch. Les tableaux de bord servent à observer, tandis que les alarmes évaluent des conditions. Une alarme peut déclencher une notification ou une automatisation, mais elle ne corrige pas un incident sans action associée.

---

### 6.2 Créer une alarme CloudWatch (CLI)

```bash
# 1. Créer une alarme sur la métrique CPU d'une instance EC2
# L'alarme se déclenche si CPU > 80% pendant 2 périodes consécutives (10 min)
aws cloudwatch put-metric-alarm \
  --alarm-name "MonInstance-CPU-Élevé" \
  --alarm-description "Alerte CPU élevé instance EC2" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-12345

# 2. Ajouter une action SNS (envoyer un email)
# D'abord, créer un sujet SNS
aws sns create-topic --name MonTopicAlarmes
# Output : TopicArn: arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes

# S'abonner au sujet (recevoir les notifications)
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes \
  --protocol email \
  --notification-endpoint admin@example.com

# 3. Modifier l'alarme pour envoyer une notification
aws cloudwatch put-metric-alarm \
  --alarm-name "MonInstance-CPU-Élevé" \
  --alarm-description "Alerte CPU élevé instance EC2" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-12345 \
  --alarm-actions arn:aws:sns:eu-west-1:123456789:MonTopicAlarmes

# 4. Lister toutes les alarmes
aws cloudwatch describe-alarms

# 5. Supprimer une alarme
aws cloudwatch delete-alarms --alarm-names "MonInstance-CPU-Élevé"
```

> [!tip]
> **Résultat attendu :**
> ```json
> # sns create-topic :
> {
>     "TopicArn": "arn:aws:sns:eu-west-1:123456789012:MonTopicAlarmes"
> }
>
> # sns subscribe :
> {
>     "SubscriptionArn": "pending confirmation"
> }
> # → Un email est envoyé à admin@example.com avec un lien de confirmation
>
> # put-metric-alarm : (pas de sortie si succès — code HTTP 200)
>
> # describe-alarms (extrait) :
> {
>     "MetricAlarms": [
>         {
>             "AlarmName": "MonInstance-CPU-Élevé",
>             "AlarmDescription": "Alerte CPU élevé instance EC2",
>             "StateValue": "OK",
>             "MetricName": "CPUUtilization",
>             "Threshold": 80.0,
>             "Period": 300,
>             "EvaluationPeriods": 2
>         }
>     ]
> }
> ```


---

### 6.3 Envoyer des métriques custom depuis une application

Applications peuvent envoyer des métriques CloudWatch pour monitorer des KPI métier :

```bash
# Exemple : envoyer une métrique custom "OrdersPerMinute"
aws cloudwatch put-metric-data \
  --namespace "MonApplication" \
  --metric-name "OrdersPerMinute" \
  --value 42 \
  --unit Count \
  --timestamp 2025-03-24T14:30:00Z
```

> [!tip]
> **Résultat attendu :**
> ```text
> # put-metric-data : pas de sortie si succès (HTTP 200)
> # La métrique est visible dans CloudWatch Console sous "MonApplication > OrdersPerMinute"
> # après environ 1 minute de délai d'ingestion.
> ```


```bash
# Ou dans un script Python :
import boto3

cloudwatch = boto3.client('cloudwatch')

# Envoyer une métrique custom
cloudwatch.put_metric_data(
    Namespace='MonApplication',
    MetricData=[
        {
            'MetricName': 'PedidosProcessados',
            'Value': 42,
            'Unit': 'Count',
            'Timestamp': datetime.utcnow()
        }
    ]
)
```

---

### 6.4 Tableaux de bord CloudWatch

CloudWatch permet de créer des **dashboards** personnalisés affichant plusieurs métriques :

```bash
# Créer un dashboard JSON
cat > dashboard.json << 'EOF'
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          [ "AWS/EC2", "CPUUtilization", { "stat": "Average" } ],
          [ ".", "NetworkIn", { "stat": "Sum" } ]
        ],
        "period": 300,
        "stat": "Average",
        "region": "eu-west-1",
        "title": "Métriques EC2"
      }
    }
  ]
}
EOF

# Créer le dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "MonDashboard" \
  --dashboard-body file://dashboard.json
```

> [!tip]
> **Résultat attendu :**
> ```json
> # put-dashboard :
> {
>     "DashboardValidationMessages": []
> }
> # Le tableau de bord "MonDashboard" est maintenant visible dans la console CloudWatch.
> # Accès : CloudWatch → Dashboards → MonDashboard
> ```


> **Référence** : [CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)

---
### 6.5 AWS SDK pour Développeurs — Automatisation Programmatique

Jusque-là, nous avons utilisé la **CLI AWS** pour exécuter des commandes manuellement. Mais les **SDK AWS** permettent d'**intégrer AWS directement dans du code applicatif** (Python, Node.js, Java, Go, etc.).

#### Qu'est-ce que le SDK AWS ?

**SDK** = **kit de développement** fourni par AWS dans plusieurs langages pour interagir avec les services AWS par programmation.

```python
Analogie : CLI vs SDK

CLI (AWS CLI)
  ↓
Outil en ligne de commande
Exécute des commandes manuellement ou dans des scripts bash
Exemple : aws ec2 describe-instances

SDK (boto3, SDK.js, SDK.java)
  ↓
Bibliothèque logicielle intégrée dans votre code
Votre application Python/Node/Java appelle AWS directement
Exemple : ec2_client.describe_instances()
```

#### SDK AWS Disponibles

| Langage | Nom SDK | Cas d'usage |
|---------|---------|-----------|
| **Python** | `boto3` | Data science, Lambda, backend | |
| **JavaScript/Node.js** | `AWS SDK for JavaScript` | Applications web, serverless | |
| **Java** | `AWS SDK for Java` | Entreprise, Spring Boot | |
| **Go** | `AWS SDK for Go` | CLI tools, microservices | |
| **C#/.NET** | `AWS SDK for .NET` | Windows, applications d'entreprise | |
| **PHP** | `AWS SDK for PHP** | Applications web, Laravel | |

#### Introduction à boto3 (Python)

**boto3** est la **SDK AWS officielle pour Python**. Elle est utilisée dans :
- Scripts d'automatisation
- Applications Lambda
- Tâches cron de maintenance
- Outils de gestion d'infrastructure

##### Installation de boto3

```bash
# Installer boto3
pip install boto3

# Vérifier l'installation
python3 -c "import boto3; print(boto3.__version__)"
```

> [!tip]
> **Résultat attendu :**
> ```python
> Collecting boto3
>   Downloading boto3-1.34.69-py3-none-any.whl (139 kB)
> Successfully installed boto3-1.34.69 botocore-1.34.69 s3transfer-0.10.1
> 1.34.69
> ```


##### Exemple 1 : Lister les instances EC2

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Lister toutes les instances
response = ec2_client.describe_instances()

# Parcourir les instances
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        instance_id = instance['InstanceId']
        instance_type = instance['InstanceType']
        state = instance['State']['Name']

        # Afficher les informations
        print(f"Instance : {instance_id}")
        print(f"  Type : {instance_type}")
        print(f"  État : {state}")
        print()
```

**Résultat :**
```text
Instance : i-0123456789abcdef0
  Type : t3.micro
  État : running

Instance : i-0987654321abcdef0
  Type : t3.small
  État : stopped
```

##### Exemple 2 : Créer une snapshot EBS

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Créer un snapshot du volume vol-12345678
response = ec2_client.create_snapshot(
    VolumeId='vol-12345678',
    Description='Sauvegarde avant migration',
    TagSpecifications=[
        {
            'ResourceType': 'snapshot',
            'Tags': [
                {'Key': 'Name', 'Value': 'backup-migration-2026-03-24'},
                {'Key': 'Environment', 'Value': 'production'}
            ]
        }
    ]
)

# Afficher l'ID du snapshot créé
snapshot_id = response['SnapshotId']
progress = response['Progress']

print(f"Snapshot créé : {snapshot_id}")
print(f"Progression : {progress}")
```

##### Exemple 3 : Arrêter une instance EC2

```python
import boto3

# Créer un client EC2
ec2_client = boto3.client('ec2', region_name='eu-west-3')

# Arrêter une instance
instance_id = 'i-0123456789abcdef0'

response = ec2_client.stop_instances(InstanceIds=[instance_id])

# Vérifier que l'arrêt est en cours
for instance in response['StoppingInstances']:
    print(f"Instance {instance['InstanceId']} est en cours d'arrêt")
    print(f"État précédent : {instance['PreviousState']['Name']}")
    print(f"État courant : {instance['CurrentState']['Name']}")
```

##### Exemple 4 : Créer une alarme CloudWatch

```python
import boto3

# Créer un client CloudWatch
cloudwatch_client = boto3.client('cloudwatch', region_name='eu-west-3')

# Créer une alarme si CPU > 80%
cloudwatch_client.put_metric_alarm(
    AlarmName='CPU-Haute-Production',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Period=300,  # 5 minutes
    Statistic='Average',
    Threshold=80.0,
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:eu-west-3:123456789:AlertesProduction'],
    Dimensions=[
        {
            'Name': 'InstanceId',
            'Value': 'i-0123456789abcdef0'
        }
    ]
)

print("Alarme CloudWatch créée avec succès")
```

#### Bonnes pratiques boto3

```python
✓ Utiliser des variables d'environnement ou des profils AWS pour les credentials
✓ Gérer les erreurs avec try/except
✓ Utiliser des context managers ou des sessions boto3
✓ Documenter chaque appel API avec un commentaire
✓ Tester en environnement non-production d'abord
✓ Utiliser des rôles IAM appropriés (pas de clés d'accès root)
```

---

<nav class="page-sequence"><a href="cours/chapitre-5/beanstalk">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/well-architected">Suivant</a></nav>
