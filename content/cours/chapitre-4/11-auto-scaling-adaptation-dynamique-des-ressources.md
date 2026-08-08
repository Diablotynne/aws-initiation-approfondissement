---
title: "11. Auto Scaling — Adaptation dynamique des ressources"
description: "\"Chapitre 4 — Stockage Amazon S3 et calcul Amazon EC2\" - 11. Auto Scaling — Adaptation dynamique des ressources"
---

<nav class="page-sequence"><a href="cours/chapitre-4/10-elastic-load-balancing-elb-repartition-du-trafic">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/12-aws-lambda-le-calcul-sans-serveur">Suivant</a></nav>

### 11.1 Qu'est-ce qu'Auto Scaling ?

Un **Auto Scaling Group (ASG)** est un groupe d'instances EC2 géré automatiquement. Il peut être associé à un Load Balancer pour garantir que :

- Les nouvelles instances sont automatiquement **enregistrées** auprès du Load Balancer.
- Les instances défaillantes sont **retirées** du pool.
- Le trafic est toujours dirigé vers les **ressources disponibles**.

Cette intégration avec le Load Balancer n'est qu'une face d'Auto Scaling ; l'autre face concerne les règles qui décident du nombre d'instances à faire tourner à un instant donné.

### 11.2 Politiques de scaling — Fondamentaux

Les politiques définissent **quand et comment** ajouter ou retirer des instances, dans deux directions symétriques.

#### Scale-out (Agrandissement)

Voici la séquence déclenchée lorsque la charge augmente :

```
Charge CPU dépasse 70% pendant 5 min
                 ▼
Ajouter 2 instances supplémentaires
                 ▼
Attendre que les instances démarrent
                 ▼
Health check OK : instances intégrées au LB
```

À l'opposé de cette montée en charge, un mécanisme symétrique retire les instances devenues inutiles pour éviter de payer une capacité surdimensionnée.

#### Scale-in (Réduction)

Et voici la séquence inverse, déclenchée lorsque la charge redescend :

```
Charge CPU chute à 30% pendant 10 min
                 ▼
Retirer 1 instance
                 ▼
Attendre que les requêtes actuelles finissent
                 ▼
Fermer l'instance, libérer les ressources
```

Notez l'asymétrie des délais (5 min pour scale-out, 10 min pour scale-in) : c'est volontaire, mieux vaut réagir vite à une surcharge et prudemment à une baisse, pour éviter d'osciller sans arrêt entre ajout et retrait d'instances.

![](assets/schemas/ch3-capture-04-88735023.png)

### 11.3 Configuration d'Auto Scaling

Un ASG typique comporte :

Min Size : 2 instances · Max Size : 10 instances · Desired Capacity : 4 instances
Launch Template : my-ami-config · Load Balancer : my-alb

Scaling Policies : Target CPU 70% · Scale out +2 instances/5 min · Scale in -1 instance/10 min

**Paramètres clés** :
- **Min Size** : minimum d'instances (au moins 2 pour la haute disponibilité)
- **Max Size** : limite supérieure pour éviter les coûts explosifs
- **Desired Capacity** : nombre d'instances cible en ce moment
- **Launch Template** : modèle (AMI, type, security group, etc.) pour les nouvelles instances

Cette configuration de base repose entièrement sur le CPU pour déclencher le scaling ; Auto Scaling peut en réalité observer bien d'autres signaux pour piloter ses décisions.

### 11.4 Métriques CloudWatch et politiques de scaling avancées

Auto Scaling peut se baser sur **plusieurs métriques CloudWatch**, pas seulement CPU.

#### Métriques disponibles

Selon la nature de l'application, une métrique différente du CPU peut être plus pertinente pour déclencher le scaling :

| Métrique | Source | Cas d'usage typique |
|----------|--------|-------------------|
| **CPU Utilization** | CloudWatch | Charge générale, serverless |
| **NetworkIn / NetworkOut** | CloudWatch | Applications réseau intensives |
| **ALB Target Count** | CloudWatch + ELB | Nombre de requêtes traitées |
| **Request Count Per Target** | CloudWatch + ELB | Répartition de charge par instance |
| **Target Response Time** | CloudWatch + ELB | Dégradation de performance |
| **Memory Utilization** | CloudWatch Agent | Applications mémoire-intensives |
| **Queue Depth** (SQS) | SQS | Traitement asynchrone |

Rien n'empêche de combiner plusieurs de ces métriques sur un même Auto Scaling Group, comme le montre l'exemple suivant avec CPU et débit réseau.

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
> ```
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

Les deux politiques créées ci-dessus configurent chacune un délai de stabilisation après une action de scaling, pour éviter les décisions trop rapprochées :

- **ScaleOutCooldown** (120-300s) : attend avant la prochaine augmentation.
  - Évite les oscillations rapides (scaling de "ping-pong").
  - Laisse le temps aux instances de démarrer.

- **ScaleInCooldown** (300-900s) : plus long que scale-out.
  - Garantit l'équilibre avant réduction.
  - Préserve la performance en cas de pics rapides.

**Exemple réaliste** :
```
T=0s    CPU = 80% → Déclenche scale-out (+2 instances)
T=120s  Instances démarrent (cool-down scale-out)
T=180s  CPU = 60% → Pourrait déclencher scale-in MAIS...
T=240s  Attendre le cooldown scale-in
T=540s  CPU toujours < 30% → Scale-in (-1 instance)
```

**Conseil** : Définir des cooldowns asymétriques (court pour scale-out, long pour scale-in) pour favorer la disponibilité.

📎 [Auto Scaling EC2](https://docs.aws.amazon.com/autoscaling/ec2/)

### 11.5 Avantages combinés Load Balancer + Auto Scaling

Utilisés ensemble, Load Balancer et Auto Scaling forment le duo de base d'une architecture web résiliente sur AWS :

- **Résilience** : les instances défaillantes sont automatiquement **remplacées**.
- **Scalabilité** : le nombre d'instances s'adapte à la **charge** en temps réel.
- **Performance** : le trafic est réparti de manière **optimale** entre les ressources disponibles.
- **Économie** : vous payez uniquement pour les ressources utilisées.
- **Sécurité** : le Load Balancer peut gérer les **certificats SSL/TLS** pour sécuriser les communications.

Cette combinaison reste néanmoins bâtie sur des instances EC2 qui tournent en continu, même au repos. Le modèle serverless présenté dans la section suivante pousse l'élasticité un cran plus loin.

---

<nav class="page-sequence"><a href="cours/chapitre-4/10-elastic-load-balancing-elb-repartition-du-trafic">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-4/index">Sommaire</a> <a href="cours/chapitre-4/12-aws-lambda-le-calcul-sans-serveur">Suivant</a></nav>
