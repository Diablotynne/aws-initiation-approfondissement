---
title: "Cheat sheet — Fondamentaux du cloud et AWS"
description: "Chapitre 1 — Fondamentaux du Cloud et présentation d'AWS - Cheat sheet — Fondamentaux du cloud et AWS"
---

<nav class="page-sequence"><a href="cours/chapitre-1/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/travaux-pratiques">Suivant</a></nav>

Cette fiche rassemble les repères à mobiliser pour lire une offre cloud, situer une ressource AWS et expliquer un choix d'architecture. Les commandes sont à exécuter uniquement dans l'environnement temporaire AWS Academy lorsqu'elles sont demandées par le lab.

### Les cinq caractéristiques du cloud selon le NIST

| Caractéristique | Question de contrôle |
|---|---|
| Libre-service à la demande | La ressource peut-elle être provisionnée sans intervention humaine du fournisseur ? |
| Accès réseau étendu | Le service est-il accessible par des mécanismes réseau standards ? |
| Mutualisation des ressources | Le fournisseur partage-t-il un pool de ressources entre plusieurs clients en conservant leur isolation ? |
| Élasticité rapide | La capacité peut-elle augmenter et diminuer selon la demande ? |
| Service mesuré | La consommation est-elle observée et mesurée selon une unité identifiable ? |

> Haute disponibilité, sauvegarde et sécurité sont des qualités d'architecture importantes, mais ne remplacent aucune des cinq caractéristiques de la définition NIST.

### Repères de décision

| Question | Repère à mobiliser |
|---|---|
| Qui sécurise quoi ? | AWS sécurise **le cloud** ; le client sécurise ce qu'il configure et déploie **dans le cloud**. La limite varie selon le service. |
| IaaS, PaaS ou SaaS ? | Comparer la part administrée par le fournisseur et celle qui reste sous la responsabilité du client. |
| Public, privé, hybride ou multicloud ? | Identifier les environnements réellement reliés et les fournisseurs utilisés ; deux clouds publics constituent généralement un multicloud, pas automatiquement un cloud hybride. |
| CAPEX ou OPEX ? | CAPEX : investissement immobilisé ; OPEX : dépense d'exploitation liée à l'usage. Le cloud modifie le modèle de dépense sans garantir à lui seul une baisse de coût. |
| Quelle région ? | Conformité et résidence des données, disponibilité des services, latence, coût puis continuité d'activité. |
| Combien de zones de disponibilité ? | Une zone ne protège pas d'une panne de zone ; une architecture multi-AZ réduit ce risque si l'application et les données sont aussi conçues pour la redondance. |
| Comment évaluer l'architecture ? | Exigences métier explicites puis six piliers AWS Well-Architected. |

### Infrastructure mondiale AWS

```text
Région AWS
├── zone de disponibilité A
├── zone de disponibilité B
└── zone de disponibilité C
```

- Une **région** est une zone géographique qui contient plusieurs zones de disponibilité.
- Une **zone de disponibilité** est un ou plusieurs centres de données distincts reliés par des réseaux à faible latence.
- Un **point de présence** rapproche notamment la distribution de contenu et certains services réseau des utilisateurs.
- Une ressource **régionale** n'est pas nécessairement répliquée entre zones par défaut : vérifier le comportement du service.

### Modèle de responsabilité partagée

| Toujours sous responsabilité AWS | Responsabilité du client selon le service |
|---|---|
| Centres de données, matériel, réseau physique, hyperviseur et infrastructure des services managés | Données, identités, autorisations, configuration réseau, chiffrement choisi, système invité pour EC2, application et conformité d'usage |

### Commandes de repérage dans CloudShell

```bash
# Afficher l'identité et le compte associés à la session temporaire
aws sts get-caller-identity

# Afficher la région configurée pour la CLI
aws configure get region

# Lister les régions accessibles et les zones de la région active
aws ec2 describe-regions --query 'Regions[].RegionName' --output table
aws ec2 describe-availability-zones \
  --query 'AvailabilityZones[].[ZoneName,State]' --output table
```

`--query` filtre la réponse avec le langage JMESPath ; `--output table` ne modifie pas les ressources, il rend seulement la sortie lisible.

### Contrôle avant de poursuivre

- Je sais distinguer région, zone de disponibilité et point de présence.
- Je sais expliquer pourquoi « présent dans le cloud » ne signifie pas « hautement disponible ».
- Je sais positionner AWS et le client sur le modèle de responsabilité partagée.
- Je sais citer et reconnaître les cinq caractéristiques NIST.
- Je sais utiliser les six piliers Well-Architected comme axes d'analyse, pas comme une certification automatique de l'architecture.

<nav class="page-sequence"><a href="cours/chapitre-1/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-1/index">Sommaire</a> <a href="cours/chapitre-1/travaux-pratiques">Suivant</a></nav>
