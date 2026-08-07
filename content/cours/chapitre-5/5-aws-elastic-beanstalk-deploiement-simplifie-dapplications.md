---
title: "5. AWS Elastic Beanstalk — Déploiement simplifié d'applications"
description: "\"Chapitre 5 — Automatisation, supervision et reprise d'activité\" - 5. AWS Elastic Beanstalk — Déploiement simplifié d'applications"
---

<nav class="page-sequence"><a href="cours/chapitre-5/4-aws-systems-manager-automatisation-operationnelle">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/6-amazon-cloudwatch-supervision-et-alarmes">Suivant</a></nav>

### 5.1 Qu'est-ce que Elastic Beanstalk ?

**AWS Elastic Beanstalk** est une plateforme PaaS managée qui permet de **déployer des applications web** sans gérer l'infrastructure sous-jacente.

Contrairement à CloudFormation où vous décrivez **chaque ressource manuellement**, Beanstalk **abstrait** la complexité : vous uploadez simplement votre code, et Beanstalk s'occupe de :

- Créer/gérer les instances EC2
- Configurer l'Auto Scaling
- Mettre en place le Load Balancer
- Activer le monitoring CloudWatch
- Gérer les mises à jour de l'OS et du runtime

```
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

> [!tip]
> **Résultat attendu :**
> ```
> # eb create (extrait de la progression) :
> Creating application version archive "app-v1".
> Uploading monappbeanstalk/app-v1.zip to S3. This may take a while.
> Upload Complete.
> Environment details for: monappbeanstalk-env
>   Application name: monappbeanstalk
>   Region: eu-west-1
>   Deployed Version: app-v1
>   Environment ID: e-abc123defg
>   Platform: arn:aws:elasticbeanstalk:eu-west-1::platform/Node.js 18 running on 64bit Amazon Linux 2023
>   Tier: WebServer-Standard-1.0
>   CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
>   Updated: 2026-03-24 10:45:00.000000+00:00
>   Status: Launching
>   Health: Grey
> ...
> INFO: Successfully launched environment: monappbeanstalk-env
>
> # eb status :
> Environment details for: monappbeanstalk-env
>   Status: Ready
>   Health: Green
>   CNAME: monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
>
> # L'application répond "Bonjour depuis Elastic Beanstalk !" à http://monappbeanstalk-env.eu-west-1.elasticbeanstalk.com
> ```

> [!warning]
> **Cold start Elastic Beanstalk :** Le premier déploiement (`eb create`) prend généralement 5 à 10 minutes car AWS provisionne l'infrastructure complète (EC2, ELB, Auto Scaling Group). Les déploiements suivants (`eb deploy`) sont plus rapides (1 à 3 minutes). Si `eb status` reste sur `Launching` trop longtemps, consultez les logs avec `eb logs`.

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

> [!warning]
> **Lambda Cold Starts :** Lors du premier appel d'une fonction Lambda (ou après une longue période d'inactivité), AWS doit initialiser le conteneur d'exécution — c'est le **cold start**. La latence peut aller de quelques dizaines de ms (Node.js/Python) à plusieurs secondes (Java). Solutions : **Provisioned Concurrency** (maintient N conteneurs chauds en permanence, payant), ou choisir un runtime léger (Node.js/Python) pour les APIs sensibles à la latence.
| **Durée max d'exécution** | Illimitée | **15 minutes** |
| **Mémoire max** | Celle de l'instance (jusqu'à 384 Go) | **10 Go** |
| **Stockage local** | EBS persistent | **/tmp : 10 Go seulement** |
| **Langages supportés** | Java, Node.js, Python, Ruby, PHP, Go, .NET | Java, Node.js, Python, Ruby, Go, .NET, Rust + custom runtime |
| **Modèle de coût** | EC2 à la seconde (même si pas de requêtes) | À l'invocation (1M req gratuites/mois) |
| **Scaling** | Auto Scaling Group (minutes) | Instantané, jusqu'à 1 000 exécutions parallèles |
| **État** | Stateful possible (session, fichiers) | Stateless obligatoire |
| **Réseau** | VPC natif, Security Groups | VPC optionnel |
| **Déploiement** | ZIP, WAR, Docker, `eb deploy` | ZIP, container, `aws lambda update-function-code` |

#### Modèle de coût détaillé

**Elastic Beanstalk :**
```
Beanstalk en lui-même = GRATUIT
Vous payez les ressources qu'il crée :

Exemple : app Node.js standard
  1× EC2 t3.small      → 0,023 $/h  → ~17 $/mois
  1× ELB Application   → 0,008 $/h  → ~6 $/mois  + 0,008 $/LCU
  Stockage EBS 20 Go   →            → ~2,5 $/mois
  ─────────────────────────────────────────────────
  TOTAL (1 instance)   →            → ~26 $/mois
  (même si 0 utilisateur cette nuit-là)
```

**Lambda :**
```
Free Tier permanent : 1 000 000 requêtes/mois + 400 000 Go-secondes/mois

Au-delà :
  Requêtes : 0,20 $ / million
  Durée    : 0,0000000167 $ / Go-seconde

Exemple : API Lambda 128 Mo RAM, 200 ms d'exécution, 1M requêtes/mois
  Durée : 1 000 000 × 0,128 Go × 0,2 s = 25 600 Go-secondes → GRATUIT (< 400 000)
  Requêtes : 1 000 000 → GRATUIT (< 1M)
  TOTAL : 0 $ (dans le Free Tier)

Exemple : 10M requêtes/mois
  Durée : 256 000 Go-s supplémentaires → 256 000 × 0,0000000167 = ~0,004 $
  Requêtes : 9M supplémentaires → 9 × 0,20 = 1,80 $
  TOTAL : ~1,80 $/mois
```

#### Quand utiliser lequel ?

```
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

📎 [Elastic Beanstalk vs Lambda — AWS Blog](https://aws.amazon.com/compare/the-difference-between-aws-lambda-and-elastic-beanstalk/)

---

<nav class="page-sequence"><a href="cours/chapitre-5/4-aws-systems-manager-automatisation-operationnelle">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/6-amazon-cloudwatch-supervision-et-alarmes">Suivant</a></nav>
