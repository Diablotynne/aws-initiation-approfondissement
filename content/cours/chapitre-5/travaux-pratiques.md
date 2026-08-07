---
title: "Travaux pratiques — Automatisation et résilience"
description: "\"Chapitre 5 — Automatisation, supervision et reprise d'activité\" - Travaux pratiques — Automatisation et résilience"
---

<nav class="page-sequence"><a href="cours/chapitre-5/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/quiz">Suivant</a></nav>

### AWS Academy — Cloud Foundations

| Type | Référence | Intitulé |
|---|---|---|
| Module | Module 9 | Architecture cloud et Well-Architected Framework |
| Module | Module 10 | Auto Scaling et surveillance |
| Atelier | Atelier 6 | Mise à l'échelle et équilibrage des charges de votre architecture — 100 points |
| Évaluation | Fin du parcours | Évaluation du cours Cloud Foundations — seuil indiqué : 70/100 |

### AWS Academy — Cloud Architecting

| Type | Référence | Intitulé |
|---|---|---|
| Module | Module 10 | Mise en œuvre de la surveillance, de l'élasticité et de la haute disponibilité |
| Module | Module 11 | Automatisation de votre architecture |
| Module | Module 13 | Création d'architectures découplées |
| Module | Module 14 | Création de microservices et d'architectures sans serveur |
| Module | Module 15 | Modèles d'ingénierie des données |
| Module | Module 16 | Planification des sinistres |
| Module | Module 17 | Passerelle vers la certification |
| Atelier guidé | Module 10 | Création d'un environnement hautement disponible |
| Atelier guidé | Module 11 | Automatisation de l'infrastructure avec AWS CloudFormation |
| Atelier Défi | Module 11 | Automatisation du déploiement d'infrastructures |
| Atelier guidé | Module 13 | Création d'applications découplées à l'aide d'Amazon SQS |
| Atelier guidé | Module 14 | Mise en œuvre d'une architecture sans serveur sur AWS |
| Atelier Défi | Module 14 | Mise en œuvre d'une architecture sans serveur pour le café |
| Atelier guidé | Module 16 | Configuration du stockage hybride et migration avec AWS Storage Gateway |
| Projet | Synthèse | Projet basé sur le cas pratique du café |
| Évaluation | Fin du parcours | Évaluation du cours Cloud Architecting |

### Go Deploy

| Lab | Intitulé |
|---|---|
| Lab 22 | Surveillance avec CloudWatch : alarmes et tableaux de bord |
| Lab 24 | Utiliser AWS Systems Manager pour installer Apache sur EC2 |
| Lab 25 | Gérer des files d'attente avec Amazon SQS |
| Lab 26 | Créer des topics SNS, s'y abonner et ajouter un événement SNS sur S3 |
| Lab 28 | Créer un environnement hautement disponible |

<details><summary><strong>Parcours guidé et validation</strong></summary>

1. Dans l'Atelier 6 Foundations, suivre la chaîne métrique → alarme → politique → capacité → instance.
2. Lire le template CloudFormation avant son déploiement, suivre les événements de stack, puis examiner mise à jour, rollback et drift.
3. Dans l'activité SQS, observer le délai de visibilité, la suppression du message et le besoin d'idempotence.
4. Relier RPO et RTO à un mécanisme de sauvegarde ou de reprise réellement testable.
5. Utiliser les labs Go Deploy sélectionnés pour vérifier CloudWatch, Systems Manager, SQS, SNS ou la haute disponibilité.
6. Terminer par l'évaluation Well-Architected de l'architecture obtenue.

**Validation observable :** expliquer idempotence, rollback et drift, puis défendre les compromis de l'architecture avec les six piliers Well-Architected.
</details>

---

<nav class="page-sequence"><a href="cours/chapitre-5/ressources">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-5/index">Sommaire</a> <a href="cours/chapitre-5/quiz">Suivant</a></nav>
