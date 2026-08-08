---
title: "9. Les services AWS les plus utilisés"
description: "\"Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS\" - 9. Les services AWS les plus utilisés"
---

<nav class="page-sequence"><a href="cours/chapitre-1/8-aws-management-console">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/10-bonnes-pratiques-de-demarrage">Suivant</a></nav>

AWS (Amazon Web Services) propose **plus de 200 services complets** couvrant le calcul, le stockage, le réseau, les bases de données, l'IA, la sécurité et bien plus encore.
Sa philosophie est simple : offrir à chaque entreprise, quelle que soit sa taille, **la même puissance technologique** qu'Amazon utilise pour ses propres opérations mondiales.

| Domaine | Service | Explication | Cas d'usage | Analogie |
|---------|---------|-------------|-----------|----------|
| **Calcul** | EC2 | Machines virtuelles configurables (CPU, RAM, OS) | Hébergement web, backend, batch | Louer un serveur dans le cloud |
| | Lambda | Exécution de code sans serveur, déclenchée par événements | Microservices, automatisation, API | Interrupteur intelligent : déclenche une action sans infrastructure |
| **Stockage** | S3 | Stockage d'objets scalable et durable | Sauvegardes, fichiers, hébergement statique | Entrepôt numérique : chaque fichier est une boîte |
| | EBS | Stockage bloc attaché à EC2, persistant | Bases de données, stockage système | Disque dur virtuel |
| | EFS | Système de fichiers partagé pour EC2 | Dossiers partagés, applications multi-instances | Dossier réseau partagé |
| **Bases de données** | RDS | Base relationnelle gérée (MySQL, PostgreSQL…) | ERP, CRM, applications transactionnelles | Serveur SQL géré par AWS |
| | DynamoDB | Base NoSQL scalable sans schéma | IoT, mobile, jeux, logs | Carnet de notes sans structure fixe |
| **Réseau** | VPC | Réseau privé isolé avec sous-réseaux et sécurité | Isolation d'environnements, segmentation | Immeuble privé avec étages et portiers |
| | CloudFront | CDN mondial pour accélérer les contenus | Sites web, vidéos, fichiers statiques | Réseau de relais pour livrer plus vite |
| **Sécurité** | IAM | Gestion des identités et des accès | Contrôle des permissions, MFA, rôles | Badge d'accès pour chaque utilisateur |
| | KMS | Gestion des clés de chiffrement | Sécurité des données, RGPD, PCI-DSS | Coffre-fort numérique pour les clés |
| **DevOps** | CloudFormation | Déploiement d'infrastructure via des fichiers YAML/JSON | Automatisation, standardisation | Plan d'architecte pour construire automatiquement |
| | CodePipeline | Orchestration CI/CD pour le déploiement | Intégration continue, tests, livraison | Chaîne de montage automatisée |
| **IA/ML** | SageMaker | Plateforme de machine learning gérée | Modèles prédictifs, classification, NLP | Laboratoire de data science automatisé |
| | Rekognition | Analyse d'images et vidéos par IA | Détection faciale, modération, OCR | Caméra intelligente qui comprend les images |
| **Analyse** | Athena | Requêtes SQL sur des fichiers S3 | Exploration de données, logs, BI | Loupe SQL sur vos fichiers |
| | Redshift | Entrepôt de données pour BI et reporting | Tableaux de bord, KPIs, analytique | Base de données XXL pour l'analyse |

📎 [AWS Products](https://aws.amazon.com/products/)

Nous n'entrons pas ici dans les détails techniques de chaque service — cela sera traité dans les jours suivants (IAM, S3, EC2, RDS, VPC…).

### 9.1 Automatisation et orchestration : Chef, Puppet et OpsWorks

Au-delà des services de base, AWS propose des outils pour **automatiser le déploiement et la gestion de l'infrastructure**.
Ces outils sont essentiels dans une approche **DevOps** et **Infrastructure as Code (IaC)**.

#### AWS OpsWorks — Orchestration managée par AWS

**AWS OpsWorks** était une famille de services AWS destinée au déploiement et à la gestion automatisés d'applications et d'infrastructures.

Il supporte deux moteurs d'automation populaires :
- **Chef** (propriétaire de OpsWorks Chef)
- **Puppet** (partenariat AWS)

> [!warning]
> **Toute la famille OpsWorks est désormais en fin de vie et désactivée.** OpsWorks for Chef Automate et OpsWorks for Puppet Enterprise ont été arrêtés le 5 mai 2024 ; OpsWorks Stacks l'a été le 26 mai 2024. Cette section sert uniquement à reconnaître une infrastructure historique et à comprendre les principes de Chef et Puppet. Pour une nouvelle architecture AWS, privilégiez notamment AWS Systems Manager, les images automatisées, les services de conteneurs ou une solution de gestion de configuration encore maintenue.

**Cas d'usage typiques :**
- Provisionner automatiquement des serveurs EC2 avec une stack applicative complète.
- Gérer les configurations à grande échelle sans intervention manuelle.
- Assurer la conformité des serveurs (tous les serveurs web ont la même configuration).
- Automatiser les déploiements continus (CI/CD).

Ces cas d'usage reposaient concrètement sur deux moteurs d'automatisation aux philosophies différentes, que nous détaillons ci-dessous à titre historique.

#### Chef — Automation as Code

**Chef** est un outil d'**automatisation de configuration** très populaire. Il fonctionne sur un modèle de **recettes (recipes)** écrites en Ruby qui décrivent **l'état souhaité** d'une machine.

```ruby
# Exemple simplifié d'une recipe Chef
package 'apache2' do
  action :install
end

service 'apache2' do
  action [:enable, :start]
end

template '/var/www/html/index.html' do
  source 'index.html.erb'
end
```

> [!tip]
> **Résultat attendu — exécution de `chef-client` :**
> ```
> [2024-01-15T09:12:34+00:00] INFO: Starting Chef Infra Client Run
> [2024-01-15T09:12:35+00:00] INFO: Installing package apache2
> [2024-01-15T09:12:42+00:00] INFO: package[apache2] installed version 2.4.57
> [2024-01-15T09:12:43+00:00] INFO: service[apache2] enabled and started
> [2024-01-15T09:12:43+00:00] INFO: template[/var/www/html/index.html] created file
> [2024-01-15T09:12:43+00:00] INFO: Chef Infra Client Run complete in 9.123 seconds.
> ```
> Apache est installé, démarré et la page d'accueil est en place. Si vous relancez `chef-client`, rien ne change — c'est l'idempotence en action.

**Intérêt :** Au lieu de cliquer dans une UI ou d'écrire des scripts bash, vous décrivez le **résultat attendu** (Apache installé, service actif, fichiers à jour).
Chef **idempotent** — si vous exécutez la recipe 10 fois, le résultat sera toujours le même.

📎 [Chef Documentation](https://docs.chef.io/)

#### Puppet — Configuration Management

**Puppet** est un outil comparable à Chef, mais avec une approche **déclarative** différente.
Au lieu de recettes, Puppet utilise un **langage déclaratif** qui décrit l'état souhaité en termes de ressources. Voici le même objectif (Apache installé et actif) exprimé en syntaxe Puppet :

```puppet
# Exemple simplifié de Puppet
package { 'apache2':
  ensure => present,
}

service { 'apache2':
  ensure => running,
  enable => true,
}
```

> [!tip]
> **Résultat attendu — exécution de `puppet agent --test` :**
> ```
> Info: Caching catalog for node01.example.com
> Info: Applying configuration version '1705312800'
> Notice: /Stage[main]/Main/Package[apache2]/ensure: created
> Notice: /Stage[main]/Main/Service[apache2]/ensure: ensure changed 'stopped' to 'running'
> Notice: Applied catalog in 8.54 seconds
> ```
> Puppet a appliqué l'état souhaité : `apache2` installé et le service actif. Tout est tracé dans les logs Puppet Master.

**Intérêt :** Comme Chef, il permet d'automatiser des déploiements complexes à grande échelle.

📎 [Puppet Documentation](https://puppet.com/docs/)

#### Ansible — Automatisation agentless et modules AWS

**Ansible** est aujourd'hui l'un des outils d'automatisation les plus utilisés dans les environnements Cloud, notamment en **complément de Terraform** pour la gestion de configuration.

Contrairement à Chef ou Puppet, Ansible est **agentless** : il n'installe rien sur les machines cibles — il se connecte via SSH (Linux) ou WinRM (Windows) et exécute les tâches définies dans des **playbooks** YAML. Voici un playbook typique pour configurer un serveur Apache sur une instance EC2 :

```yaml
# Exemple de playbook Ansible — installation Apache sur EC2
---
- name: Configurer un serveur web Apache sur EC2
  hosts: ec2_instances
  become: yes

  tasks:
    - name: Installer Apache
      apt:
        name: apache2
        state: present

    - name: Démarrer et activer le service
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Copier la page d'accueil
      copy:
        src: index.html
        dest: /var/www/html/index.html
```

> [!tip]
> **Résultat attendu — `ansible-playbook site.yml -i inventory.ini` :**
> ```
> PLAY [Configurer un serveur web Apache sur EC2] ****************************
>
> TASK [Gathering Facts] *****************************************************
> ok: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Installer Apache] ****************************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Démarrer et activer le service] **************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> TASK [Copier la page d'accueil] ********************************************
> changed: [ec2-18-234-56-78.eu-west-3.compute.amazonaws.com]
>
> PLAY RECAP *****************************************************************
> ec2-18-234-56-78.eu-west-3.compute.amazonaws.com : ok=4  changed=3  unreachable=0  failed=0
> ```
> Apache est installé et opérationnel sur l'instance EC2. La ligne `changed=3` confirme que les trois tâches ont apporté des modifications. Si vous relancez le playbook, vous verrez `changed=0` — c'est l'idempotence Ansible.

Ansible propose également une **collection dédiée AWS** (`amazon.aws`) permettant de piloter directement les ressources AWS depuis un playbook :

```yaml
# Exemple : créer une instance EC2 avec le module Ansible AWS
- name: Lancer une instance EC2
  amazon.aws.ec2_instance:
    name: "mon-serveur-web"
    instance_type: t3.micro
    image_id: ami-0c55b159cbfafe1f0
    region: eu-west-3
    key_name: ma-cle-ssh
    security_groups:
      - sg-xxxxxxxxxx
    tags:
      Env: production
      Owner: formation
```

> [!tip]
> **Résultat attendu — `ansible-playbook create-ec2.yml` :**
> ```
> TASK [Lancer une instance EC2] *********************************************
> changed: [localhost]
>
> PLAY RECAP *****************************************************************
> localhost : ok=1  changed=1  unreachable=0  failed=0
>
> Instance créée : i-0a1b2c3d4e5f67890
> IP publique    : 15.236.142.87
> Région         : eu-west-3
> Statut         : running
> ```
> L'instance EC2 `mon-serveur-web` est lancée en `eu-west-3`. Elle est tagguée `Env: production` et visible dans la console AWS sous EC2 > Instances. Vous pouvez vous y connecter via SSH : `ssh -i ma-cle-ssh.pem ubuntu@15.236.142.87`.

**Intérêt dans un contexte AWS :**
- Complémentaire à **Terraform** : Terraform crée l'infrastructure (VPC, EC2, RDS…), Ansible configure les instances après leur lancement.
- Pas d'agent à installer sur les instances EC2 — idéal pour les environnements éphémères.
- Intégration native avec les services AWS via la collection `amazon.aws`.
- Utilisé dans les pipelines CI/CD avec CodePipeline ou Jenkins.

📎 [Ansible Documentation officielle](https://docs.ansible.com/)
📎 [Collection Ansible AWS (amazon.aws)](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)

#### CloudFormation — Infrastructure as Code AWS-native

Enfin, il existe **AWS CloudFormation**, l'outil **native AWS** pour déployer l'infrastructure via du code YAML/JSON. Ce template déclare deux ressources : une instance EC2 et un bucket S3. AWS les crée dans le bon ordre, sans commande manuelle :

```yaml
# Exemple CloudFormation : créer une instance EC2 et un bucket S3
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro

  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-bucket
```

> [!tip]
> **Résultat attendu — après `aws cloudformation deploy --template-file stack.yaml --stack-name ma-stack` :**
> ```
> Waiting for changeset to be created...
> Waiting for stack create/update to complete...
>
> Successfully created/updated stack - ma-stack
>
> Stack ID   : arn:aws:cloudformation:eu-west-3:123456789012:stack/ma-stack/abc12345
> Status     : CREATE_COMPLETE
> Resources  :
>   - MyInstance  → i-0abc123def456789  (AWS::EC2::Instance)    CREATE_COMPLETE
>   - MyBucket    → my-app-bucket       (AWS::S3::Bucket)        CREATE_COMPLETE
> ```
> Les deux ressources ont été créées par CloudFormation dans le bon ordre. En cas de suppression, `aws cloudformation delete-stack --stack-name ma-stack` supprimera l'EC2 et le bucket ensemble — ce qui garantit qu'il ne reste pas de ressources orphelines.

**Intérêt :** CloudFormation est intégré nativement à AWS. Vous versionnez votre infrastructure comme du code et pouvez la recréer en quelques clics.

#### Comparaison simplifiée

Au-delà du langage et du cas d'usage, ces outils se répartissent selon deux architectures fondamentalement différentes — Chef et Puppet fonctionnent en mode Pull (agent installé sur chaque instance), tandis qu'Ansible et CloudFormation fonctionnent en mode Push (aucun agent, contrôle depuis un poste central) :

<a class="schema-zoom" href="assets/schemas/automatisation-push-vs-pull.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/automatisation-push-vs-pull.svg"
     alt="Comparaison architecturale : modèle Push où un poste de contrôle envoie la configuration via SSH ou l'API AWS sans agent, contre modèle Pull où chaque instance interroge périodiquement un serveur central via un agent installé"
     style="display:block; margin:auto; width:90%"></a>

Cette distinction Push/Pull explique directement pourquoi Ansible et CloudFormation s'intègrent plus naturellement dans un contexte AWS éphémère (instances créées et détruites en continu) : sans agent à installer et maintenir, une nouvelle instance est opérationnelle dès qu'elle est joignable en SSH ou via l'API.

| Outil | Modèle | Langage | Cas d'usage | Intégration AWS |
|-------|--------|---------|-------------|-----------------|
| **Chef** | Impératif (recipes) | Ruby | Configurations complexes ou parc historique | Solution Chef maintenue hors OpsWorks |
| **Puppet** | Déclaratif | Puppet DSL | Gestion de flotte à grande échelle | Solution Puppet maintenue hors OpsWorks |
| **Ansible** | Déclaratif (agentless) | YAML | Config management, post-provisioning | Via collection `amazon.aws` |
| **CloudFormation** | Déclaratif (IaC) | YAML/JSON | Déploiement d'infrastructure AWS | **Native, recommandé** |
| **Terraform** | Déclaratif (IaC) | HCL | Multi-cloud, plus flexible | Via AWS provider |

**Recommandation pratique :**
- Pour **débuter avec AWS**, utiliser **CloudFormation** ou **Terraform**.
- Pour **configurer les instances après déploiement**, utiliser **Ansible** — agentless et très bien intégré à AWS.
- Pour **gérer des configurations** sur des centaines de serveurs, évaluer AWS Systems Manager ou une solution Chef/Puppet maintenue ; ne pas créer de dépendance à OpsWorks.
- Pour **Puppet**, privilégier dans les environnements d'entreprise très structurés.
- En pratique : **Terraform + Ansible** est le duo le plus courant aujourd'hui — Terraform provisionne, Ansible configure.

📎 [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
📎 [AWS OpsWorks for Chef Automate](https://aws.amazon.com/opsworks/chef-automate/)
📎 [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)

---

<nav class="page-sequence"><a href="cours/chapitre-1/8-aws-management-console">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/10-bonnes-pratiques-de-demarrage">Suivant</a></nav>
