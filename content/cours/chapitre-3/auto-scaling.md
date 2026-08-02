---
title: "11. Auto Scaling — Adaptation dynamique des ressources"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 11. Auto Scaling — Adaptation dynamique des ressources"
---

<nav class="page-sequence"><a href="cours/chapitre-3/elb">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/lambda">Suivant</a></nav>

### 9.1 Qu'est-ce qu'Auto Scaling ?

Un **Auto Scaling Group (ASG)** est un groupe d'instances EC2 géré automatiquement. Il peut être associé à un Load Balancer pour garantir que :

- Les nouvelles instances sont automatiquement **enregistrées** auprès du Load Balancer.
- Les instances défaillantes sont **retirées** du pool.
- Le trafic est toujours dirigé vers les **ressources disponibles**.

### 9.2 Politiques de scaling — Fondamentaux

Les politiques définissent **quand et comment** ajouter ou retirer des instances.

#### Scale-out (Agrandissement)

```text
Charge CPU dépasse 70% pendant 5 min
                 ▼
Ajouter 2 instances supplémentaires
                 ▼
Attendre que les instances démarrent
                 ▼
Health check OK : instances intégrées au LB
```

#### Scale-in (Réduction)

```text
Charge CPU chute à 30% pendant 10 min
                 ▼
Retirer 1 instance
                 ▼
Attendre que les requêtes actuelles finissent
                 ▼
Fermer l'instance, libérer les ressources
```

### 9.3 Configuration d'Auto Scaling

Un ASG typique comporte :

Min Size : 2 instances · Max Size : 10 instances · Desired Capacity : 4 instances
Launch Template : my-ami-config · Load Balancer : my-alb

Scaling Policies : Target CPU 70% · Scale out +2 instances/5 min · Scale in -1 instance/10 min

**Paramètres clés** :
- **Min Size** : minimum d'instances (au moins 2 pour la haute disponibilité)
- **Max Size** : limite supérieure pour éviter les coûts explosifs
- **Desired Capacity** : nombre d'instances cible en ce moment
- **Launch Template** : modèle (AMI, type, security group, etc.) pour les nouvelles instances

### 9.4 Métriques CloudWatch et politiques de scaling avancées

Auto Scaling peut se baser sur **plusieurs métriques CloudWatch**, pas seulement CPU.

#### Métriques disponibles

| Métrique | Source | Cas d'usage typique |
|----------|--------|-------------------|
| **CPU Utilization** | CloudWatch | Charge générale, serverless |
| **NetworkIn / NetworkOut** | CloudWatch | Applications réseau intensives |
| **ALB Target Count** | CloudWatch + ELB | Nombre de requêtes traitées |
| **Request Count Per Target** | CloudWatch + ELB | Répartition de charge par instance |
| **Target Response Time** | CloudWatch + ELB | Dégradation de performance |
| **Memory Utilization** | CloudWatch Agent | Applications mémoire-intensives |
| **Queue Depth** (SQS) | SQS | Traitement asynchrone |

#### Exemple de politique de scaling multi-métriques

On crée ici deux politiques indépendantes sur le même ASG : l'une réagit au CPU, l'autre au débit réseau. AWS les évalue en parallèle et déclenche le scaling dès que l'une d'elles est satisfaite.

```bash
# Politique 1 : Scale out si CPU > 75% pendant 2 minutes
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name mon-asg \
    --policy-name cpu-scale-out \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 75.0,
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "ScaleOutCooldown": 120,
        "ScaleInCooldown": 300
    }'

# Politique 2 : Scale out si Network > 1 Gbps
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name mon-asg \
    --policy-name network-scale-out \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 70.0,
        "CustomizedMetricSpecification": {
            "MetricName": "NetworkOut",
            "Namespace": "AWS/EC2",
            "Statistic": "Average"
        }
    }'
```

> [!tip]
> **Résultat attendu :**
> ```json
> # put-scaling-policy (cpu-scale-out) retourne :
> {
>     "PolicyARN": "arn:aws:autoscaling:eu-west-1:123456789012:scalingPolicy:a1b2c3d4:autoScalingGroupName/mon-asg:policyName/cpu-scale-out",
>     "Alarms": [
>         {
>             "AlarmName": "TargetTracking-mon-asg-AlarmHigh-cpu-scale-out",
>             "AlarmARN": "arn:aws:cloudwatch:eu-west-1:123456789012:alarm:TargetTracking-mon-asg-AlarmHigh"
>         }
>     ]
> }
>
> # put-scaling-policy (network-scale-out) retourne de même avec un ARN différent
> ```


#### Cooldown Periods (délais entre actions)

- **ScaleOutCooldown** (120-300s) : attend avant la prochaine augmentation.
  - Évite les oscillations rapides (scaling de "ping-pong").
  - Laisse le temps aux instances de démarrer.

- **ScaleInCooldown** (300-900s) : plus long que scale-out.
  - Garantit l'équilibre avant réduction.
  - Préserve la performance en cas de pics rapides.

**Exemple réaliste** :
```text
T=0s    CPU = 80% → Déclenche scale-out (+2 instances)
T=120s  Instances démarrent (cool-down scale-out)
T=180s  CPU = 60% → Pourrait déclencher scale-in MAIS...
T=240s  Attendre le cooldown scale-in
T=540s  CPU toujours < 30% → Scale-in (-1 instance)
```

**Conseil** : Définir des cooldowns asymétriques (court pour scale-out, long pour scale-in) pour favorer la disponibilité.

📎 [Auto Scaling EC2](https://docs.aws.amazon.com/autoscaling/ec2/)

### 9.5 Avantages combinés Load Balancer + Auto Scaling

- **Résilience** : les instances défaillantes sont automatiquement **remplacées**.
- **Scalabilité** : le nombre d'instances s'adapte à la **charge** en temps réel.
- **Performance** : le trafic est réparti de manière **optimale** entre les ressources disponibles.
- **Économie** : vous payez uniquement pour les ressources utilisées.
- **Sécurité** : le Load Balancer peut gérer les **certificats SSL/TLS** pour sécuriser les communications.

---

<nav class="page-sequence"><a href="cours/chapitre-3/elb">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/lambda">Suivant</a></nav>
