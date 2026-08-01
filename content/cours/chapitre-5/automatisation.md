---
title: "2. Pourquoi automatiser dans le Cloud ?"
description: "Chapitre 5 — Automatisation, supervision et reprise d'activité - 2. Pourquoi automatiser dans le Cloud ?"
---

<nav class="page-sequence"><a href="cours/chapitre-5/reprise">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/cloudformation">Suivant</a></nav>

### 2.1 Le déploiement manuel : une source d'erreurs

Dans un environnement traditionnel, déployer une infrastructure peut prendre **des jours, voire des semaines**. Les équipes IT doivent :

- Commander du matériel physique
- Installer les systèmes d'exploitation
- Configurer le réseau et les accès
- Mettre à jour les pare-feu et les politiques de sécurité
- Documenter tous les changements (quand c'est fait...)

**Le problème** : chaque déploiement manuel génère des **incohérences**. Deux administrateurs ne font jamais exactement la même chose. Certains oublis passent inaperçus :

```text
Infrastructure créée manuellement :
  Admin Alice crée un VPC le 10 janvier
  → Configure 2 subnets, 1 IGW, 1 NAT Gateway
  → Oublie de documenter les CIDR utilisés
  → Quitte l'entreprise 6 mois après

  Admin Bob doit dépliquer l'infrastructure
  → Cherche la documentation (introuvable)
  → Recrée un nouveau VPC similaire MAIS différent
  → Incompatibilité lors du peering : PERTE DE TEMPS

Résultat : deux "mêmes" infrastructures qui ne sont PAS identiques
```

---

### 2.2 L'automatisation dans le Cloud : standardisation et rapidité

Dans le cloud AWS, l'**automatisation** permet de :

- **Standardiser** les environnements → Prod et Dev sont identiques
- **Réduire** les délais de déploiement → Quelques minutes au lieu de semaines
- **Limiter** les erreurs humaines → Le code est testé avant déploiement
- **Faciliter** la montée en charge en appliquant une configuration reproductible à plusieurs ressources

```text
Infrastructure automatisée avec CloudFormation :
  Étape 1 : Décrire l'infrastructure dans un template YAML
  Étape 2 : Valider puis déployer le template en développement
  Étape 3 : Réutiliser le même template dans l'environnement de test
  Étape 4 : Promouvoir la version validée vers la production

  Avantage : zéro divergence entre les environnements
            zéro oubli de configuration
            traçabilité complète (versionning Git du template)
```

---

### 2.3 Les outils AWS pour l'automatisation

| Outil | Fonction | Cas d'usage |
|-------|----------|-----------|
| **CloudFormation** | Déploiement d'infrastructures as code (IaC) | Créer VPC, EC2, RDS, S3 en une seule opération |
| **AWS Solutions Library** | Solutions et guides techniques évalués par AWS | Étudier ou déployer une solution documentée pour un besoin identifié |
| **Systems Manager** | Gestion centralisée et automatisation opérationnelle | Exécuter des scripts, patcher les serveurs, inventorier les ressources |
| **Elastic Beanstalk** | Déploiement PaaS simplifié d'applications web | Déployer une app Node.js sans gérer l'infrastructure |
| **CLI / SDK** | Automatisation par scripts et développement | Orchestrer plusieurs services AWS via Python/Bash |

> **Référence** : [AWS Systems Manager Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html)
> **Référence** : [AWS CloudFormation Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)

---
### 2.4 Réutiliser une solution publiée sans déléguer la décision

La **Bibliothèque de solutions AWS** rassemble des solutions et des guides techniques évalués par AWS pour des cas d'usage identifiés. Certaines entrées fournissent du code déployable ; d'autres décrivent une architecture ou une méthode.

Une solution publiée reste un point de départ. Avant tout déploiement, il faut :

1. lire le diagramme et la documentation d'architecture ;
2. identifier les services, régions et quotas utilisés ;
3. examiner le code d'infrastructure et les permissions demandées ;
4. estimer les coûts des ressources créées ;
5. vérifier la stratégie de mise à jour et de suppression ;
6. adapter les paramètres aux exigences de sécurité et de conformité de l'organisation.

> [!warning]
> Un bouton de déploiement n'atteste ni de l'adéquation au besoin, ni du coût, ni de la conformité de la configuration obtenue. Le code doit être revu et testé comme tout autre composant livré en production.

**Référence officielle :** [AWS Solutions Library](https://aws.amazon.com/solutions/)


---

<nav class="page-sequence"><a href="cours/chapitre-5/reprise">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/cloudformation">Suivant</a></nav>
