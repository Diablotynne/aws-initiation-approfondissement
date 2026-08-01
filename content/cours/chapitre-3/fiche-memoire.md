---
title: "Cheat sheet — Stockage et calcul"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - Cheat sheet — Stockage et calcul"
---

<nav class="page-sequence"><a href="cours/chapitre-3/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/travaux-pratiques">Suivant</a></nav>

### Choisir un stockage

| Besoin | Service | Unité manipulée | Point de vigilance |
|---|---|---|---|
| Objets accessibles par API | Amazon S3 | Objet dans un bucket | Ce n'est ni un disque ni un système de fichiers POSIX |
| Volume bloc pour une instance | Amazon EBS | Volume attaché dans une zone | Le volume et l'instance doivent être dans la même zone de disponibilité |
| Fichiers partagés entre plusieurs clients | Amazon EFS | Système de fichiers NFS régional | Vérifier réseau, points de montage, performances et coût |
| Disque temporaire lié à l'hôte | Instance Store | Volume éphémère | Les données ne constituent pas un stockage persistant |
| Archivage | Classes S3 Glacier | Objet archivé | Délai et coût de restauration varient selon la classe |

### Repères Amazon S3

- Le nom d'un bucket doit être globalement unique dans la partition AWS.
- Un objet est identifié par la combinaison bucket + clé.
- Le versioning conserve plusieurs versions ; il ne remplace pas une stratégie complète de sauvegarde.
- Le blocage de l'accès public constitue une protection importante, mais il faut aussi vérifier politiques de bucket, ACL éventuelles et autorisations IAM.
- Une règle de cycle de vie automatise transitions et expirations selon des critères explicites.

```bash
# Inventaire en lecture seule
aws s3 ls
aws s3api get-public-access-block --bucket nom-du-bucket
aws s3api get-bucket-versioning --bucket nom-du-bucket
aws s3api get-bucket-encryption --bucket nom-du-bucket

# Opérations de lab : vérifier le bucket cible avant exécution
aws s3 cp fichier.txt s3://nom-du-bucket/
aws s3 sync ./dossier/ s3://nom-du-bucket/prefixe/
```

`aws s3` propose des commandes de haut niveau ; `aws s3api` expose plus directement les opérations de l'API S3.

### Choisir un mode de calcul

| Besoin | Service à examiner | Responsabilité principale du client |
|---|---|---|
| Contrôle du système d'exploitation | Amazon EC2 | Système invité, correctifs, application et capacité |
| Fonctions déclenchées par événement | AWS Lambda | Code, dépendances, configuration, permissions et limites |
| Conteneurs sans gérer les serveurs | Amazon ECS avec AWS Fargate | Image, définition de tâche, réseau, permissions et observabilité |
| Déploiement d'application sur plateforme gérée | AWS Elastic Beanstalk | Code, configuration applicative et choix de plateforme |

### Cycle de vie et disponibilité EC2

```text
AMI + type + réseau + stockage + rôle IAM
                    ↓
               instance EC2
        pending → running → stopping → stopped
                         └────────────→ terminated
```

- Arrêter une instance conserve généralement ses volumes EBS, mais pas les données d'Instance Store.
- Terminer une instance peut supprimer son volume racine selon l'attribut `DeleteOnTermination`.
- Une AMI est un modèle de lancement ; un snapshot EBS est une sauvegarde incrémentale d'un volume.
- Un type d'instance décrit une combinaison de processeur, mémoire, réseau et capacités associées.

```bash
# Afficher l'état, le type, la zone et les adresses des instances visibles
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,Placement.AvailabilityZone,PrivateIpAddress]' \
  --output table

# Afficher les volumes EBS et leur rattachement
aws ec2 describe-volumes \
  --query 'Volumes[].[VolumeId,State,Size,AvailabilityZone,Attachments[0].InstanceId]' \
  --output table
```

### Élasticité et répartition de charge

- Elastic Load Balancing distribue les requêtes vers des cibles saines ; il ne crée pas lui-même de nouvelles instances.
- EC2 Auto Scaling ajuste le nombre d'instances entre capacité minimale, souhaitée et maximale.
- Un health check vérifie l'aptitude d'une cible à recevoir du trafic ; choisir un endpoint représentatif de l'application.
- Une architecture multi-AZ nécessite des cibles réellement réparties entre plusieurs zones.

### Checklist avant validation

- Je sais justifier S3, EBS ou EFS par le modèle d'accès attendu.
- Je sais distinguer arrêt, redémarrage et terminaison d'une instance.
- Je vérifie région, zone, identifiant et tags avant toute commande de modification.
- Je sais distinguer sauvegarde, réplication, versioning et haute disponibilité.
- Je sais expliquer les rôles complémentaires d'un load balancer et d'un groupe Auto Scaling.

<nav class="page-sequence"><a href="cours/chapitre-3/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/travaux-pratiques">Suivant</a></nav>
