---
title: "5. AWS Elastic Beanstalk — Déploiement simplifié d'applications"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 5. AWS Elastic Beanstalk — Déploiement simplifié d'applications"
---

# 5. AWS Elastic Beanstalk — Déploiement simplifié d'applications

<nav class="page-sequence"><a href="cours/chapitre-5/systems-manager">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/cloudwatch">Suivant</a></nav>

### 5.1 Qu'est-ce que Elastic Beanstalk ?

**AWS Elastic Beanstalk** est une plateforme PaaS managée qui permet de **déployer des applications web** sans gérer l'infrastructure sous-jacente.

Contrairement à CloudFormation où vous décrivez **chaque ressource manuellement**, Beanstalk **abstrait** la complexité : vous uploadez simplement votre code, et Beanstalk s'occupe de :

- Créer/gérer les instances EC2
- Configurer l'Auto Scaling
- Mettre en place le Load Balancer
- Activer le monitoring CloudWatch
- Gérer les mises à jour de l'OS et du runtime

```text
Analogie : Elastic Beanstalk vs CloudFormation

CloudFormation = "Je veux décrire exactement mon infrastructure"
  - Créer VPC, subnets, security groups, EC2, ALB, RDS, etc.
  - Contrôle total mais responsabilité complète

Elastic Beanstalk = "Je veux juste déployer mon app, pas m'embêter avec l'infra"
  - Upload le code (Node.js, Python, Java, .NET)
  - Beanstalk crée l'infrastructure automatiquement
  - Mise en échelle automatique en cas de charge
  - Moins de contrôle mais moins de friction
```

---

### 5.2 Runtimes et plateformes supportées

Elastic Beanstalk supporte plusieurs langages et frameworks :

| Langage | Framework | Exemple |
|---------|-----------|---------|
| **Node.js** | Express, Fastify | Application web Node.js classique |
| **Python** | Flask, Django | Application Flask avec routes |
| **Java** | Spring Boot | Application Spring Boot JAR |
| **.NET** | ASP.NET Core | Application web .NET |
| **PHP** | Laravel, Symfony | Brochure web PHP |
| **Go** | Gin, Echo | Microservice Go |
| **Docker** | N'importe quel container | Flexibilité maximale |

> **Référence** : [Elastic Beanstalk Platforms](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html)

---

### 5.3 Déployer une app Node.js avec Elastic Beanstalk (CLI)

Elastic Beanstalk gère toute l'infrastructure à votre place : vous fournissez juste le code. La commande `eb create` provisionne automatiquement une instance EC2, un load balancer, un Auto Scaling Group et CloudWatch. Vous n'avez qu'à pousser votre code.

```bash
# 1. Créer une application Node.js simple (app.js)
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Bonjour depuis Elastic Beanstalk !');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# 2. Créer un package.json
cat > package.json << 'EOF'
{
  "name": "monappbeanstalk",
  "version": "1.0.0",
  "description": "Petite app Express sur Beanstalk",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# 3. Initialiser un environnement Elastic Beanstalk
eb init -p node.js-18 monappbeanstalk --region eu-west-1

# 4. Créer et déployer l'environnement
# AWS crée automatiquement :
#   - Une application Beanstalk
#   - Un environnement (Dev, Staging, Prod, etc.)
#   - 1+ instances EC2 (t2.micro par défaut)
#   - Un Load Balancer
#   - Auto Scaling Group
#   - Monitoring CloudWatch
eb create monappbeanstalk-env --instance-type t2.micro --envvars NODE_ENV=production

# 5. Vérifier l'état du déploiement
eb status

# 6. Voir l'URL publique de l'application
eb open

# 7. Voir les logs en temps réel
eb logs -f

# 8. Augmenter le nombre minimum d'instances
eb scale 3

# 9. Deployer une nouvelle version du code
# (après modifi app.js)
eb deploy

# 10. Supprimer l'application et l'environnement
eb terminate monappbeanstalk-env
```

:::success
**Résultat attendu :**
```text
# eb create (extrait de la progression) :
Creating application version archive "app-v1".
Uploading monappbeanstalk/app-v1.zip to S3. This may take a while.
Upload Complete.
Environment details for: monappbeanstalk-env
  Application name: monappbeanstalk
  Region: eu-west-1
  Deployed Version: app-v1
  Environment ID: e-abc123defg
  Platform: arn:aws:elasticbeanstalk:eu-west-1::platform/Node.js 18 running on 64bit Amazon Linux 2023
  Tier: WebServer-Standard-1.0
  CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
  Updated: 2026-03-24 10:45:00.000000+00:00
  Status: Launching
  Health: Grey
...
INFO: Successfully launched environment: monappbeanstalk-env

# eb status :
Environment details for: monappbeanstalk-env
  Status: Ready
  Health: Green
  CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com

# L'application répond "Bonjour depuis Elastic Beanstalk !" à http://monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
```
:::

:::warning
**Cold start Elastic Beanstalk :** Le premier déploiement (`eb create`) prend généralement 5 à 10 minutes car AWS provisionne l'infrastructure complète (EC2, ELB, Auto Scaling Group). Les déploiements suivants (`eb deploy`) sont plus rapides (1 à 3 minutes). Si `eb status` reste sur `Launching` trop longtemps, consultez les logs avec `eb logs`.
:::

> **Résultat attendu :** `eb create` affiche la progression en temps réel (création VPC, EC2, Load Balancer…) et se termine avec l'URL publique de l'application. `eb open` ouvre cette URL dans votre navigateur — vous devez voir "Bonjour depuis Elastic Beanstalk !".

---

### 5.4 Avantages et limitations

| Avantage | Limitation |
|----------|-----------|
| Déploiement simple du code | Contrôle limité sur l'infrastructure |
| Scalabilité automatique | Moins flexible que CloudFormation |
| Monitoring intégré | Pas adapté aux architectures très complexes |
| Mises à jour OS automatiques | Coûts potentiellement plus élevés |

> **Référence** : [Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)


### 5.5 Elastic Beanstalk vs Lambda — Quand choisir quoi ?

Ces deux services répondent à la même question — "comment déployer du code sans gérer de serveurs" — mais avec des philosophies radicalement différentes.

| | Beanstalk | Lambda |
|---|---|---|
| Exécution | EC2 toujours allumé, Load Balancer, Auto Scaling Group, CloudWatch | Conteneur éphémère, créé à la demande et détruit après exécution |
| Paradigme | Lift & Shift | Event-Driven |

#### Tableau de comparaison

| Critère | Elastic Beanstalk | Lambda |
|---------|------------------|--------|
| **Paradigme** | PaaS — application toujours en cours | FaaS — fonction déclenchée à la demande |
| **Infrastructure** | EC2 + ELB + ASG (gérés automatiquement) | Aucune instance visible |
| **Démarrage** | Toujours chaud | Cold start possible (ms à quelques s) |

:::warning
**Lambda Cold Starts :** Lors du premier appel d'une fonction Lambda (ou après une longue période d'inactivité), AWS doit initialiser le conteneur d'exécution — c'est le **cold start**. La latence peut aller de quelques dizaines de ms (Node.js/Python) à plusieurs secondes (Java). Solutions : **Provisioned Concurrency** (maintient N conteneurs chauds en permanence, payant), ou choisir un runtime léger (Node.js/Python) pour les APIs sensibles à la latence.
:::
| **Durée max d'exécution** | Illimitée | **15 minutes** |
| **Mémoire max** | Celle de l'instance (jusqu'à 384 Go) | **10 Go** |
| **Stockage local** | EBS persistent | **/tmp : 10 Go seulement** |
| **Langages supportés** | Java, Node.js, Python, Ruby, PHP, Go, .NET | Java, Node.js, Python, Ruby, Go, .NET, Rust + custom runtime |
| **Modèle de coût** | Ressources sous-jacentes : instances, équilibrage, stockage et transfert | Requêtes, durée d'exécution, mémoire et services associés |
| **Scaling** | Auto Scaling Group (minutes) | Instantané, jusqu'à 1 000 exécutions parallèles |
| **État** | Stateful possible (session, fichiers) | Stateless obligatoire |
| **Réseau** | VPC natif, Security Groups | VPC optionnel |
| **Déploiement** | ZIP, WAR, Docker, `eb deploy` | ZIP, container, `aws lambda update-function-code` |

#### Comparer les modèles de coût

**Elastic Beanstalk** n'ajoute pas de frais de service propres, mais crée des ressources facturables. L'estimation doit inclure les instances EC2, l'équilibreur de charge, les volumes EBS, les adresses et le transfert de données. Une capacité maintenue en fonctionnement continue d'être facturée même lorsqu'elle reçoit peu de trafic.

**Lambda** se calcule à partir du nombre de requêtes et des ressources consommées pendant l'exécution. Il faut aussi compter les services périphériques : API Gateway, journaux CloudWatch, stockage, transfert et éventuelle concurrence provisionnée.

La comparaison correcte utilise le **même scénario de charge** : nombre de requêtes, durée moyenne, mémoire, trafic réseau et disponibilité attendue. Les tarifs et offres gratuites évoluent ; saisissez ces valeurs dans [AWS Pricing Calculator](https://calculator.aws/) au moment de l'étude.

#### Quand utiliser lequel ?

```text
CHOISIR BEANSTALK si :
  ✅ Application web traditionnelle (Django, Express, Spring Boot, WordPress)
  ✅ Traitement long (> 15 minutes)
  ✅ Besoin de sessions persistantes côté serveur
  ✅ Migration d'une app existante ("lift & shift")
  ✅ Besoin d'accès à des fichiers locaux entre requêtes
  ✅ Équipe non familière avec l'architecture event-driven

CHOISIR LAMBDA si :
  ✅ API REST légère (avec API Gateway)
  ✅ Traitement d'événements (upload S3, message SQS, stream DynamoDB)
  ✅ Tâches planifiées (cron CloudWatch Events)
  ✅ Traitement de fichiers (redimensionnement images, parsing CSV)
  ✅ Webhooks, notifications, automatisations
  ✅ Trafic très variable (pics et creux importants)
  ✅ Budget serré avec faible volumétrie
```

> 💡 **En pratique** : beaucoup d'architectures modernes combinent les deux. Beanstalk pour le frontend/API principale, Lambda pour les traitements en arrière-plan (envoi d'emails, génération de rapports, nettoyage de données).

📎 [Documentation AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

---

<nav class="page-sequence"><a href="cours/chapitre-5/systems-manager">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/cloudwatch">Suivant</a></nav>
