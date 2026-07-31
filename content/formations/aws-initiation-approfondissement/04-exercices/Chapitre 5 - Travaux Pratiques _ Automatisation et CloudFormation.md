# Chapitre 5 — Travaux Pratiques : Automatisation, CloudFormation & Well-Architected

> [!NOTE]
> Le noyau recommandé est la revue Well-Architected, l'atelier de mise à l'échelle et le Module 11 CloudFormation. Les Modules Architecting 10 et 13 à 17, le projet café et les labs complémentaires restent accessibles comme démonstrations ou approfondissements : ils ne sont pas tous exigés pendant cette dernière séance.
---

## 🎓 AWS Academy Cloud Foundations — Stagiaires académiques

**Modules associés à ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 9 | Architecture cloud (Well-Architected Framework) | ✅ À compléter |
| Module 10 | Auto Scaling et surveillance (ELB, CloudWatch, ASG) | ✅ À compléter |

**Atelier guidé :**

| Atelier | Titre | Points |
|---------|-------|--------|
| **Atelier 6** | Mise à l'échelle et équilibrage des charges de votre architecture | 100 pts |

**Évaluation finale Cloud Foundations :**

| Évaluation | Titre | Note minimale |
|------------|-------|---------------|
| **Évaluation du cours** | AWS Academy Cloud Foundations | 70 / 100 pts |

---

## 🎓 AWS Academy Cloud Architecting — Stagiaires académiques

**Modules à compléter pour ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 10 | Mise en œuvre de la surveillance, de l'élasticité et de la haute disponibilité | ✅ À compléter |
| Module 11 | Automatisation de votre architecture | ✅ À compléter |
| Module 13 | Création d'architectures découplées | ✅ À compléter |
| Module 14 | Création de microservices et d'architectures sans serveur | ✅ À compléter |
| Module 15 | Modèles d'ingénierie des données | ✅ À compléter |
| Module 16 | Planification des sinistres | ✅ À compléter |
| Module 17 | Passerelle vers la certification | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre |
|---------|-------|
| Atelier guidé | Création d'un environnement hautement disponible |
| Atelier guidé | Automatisation de l'infrastructure avec AWS CloudFormation |
| Atelier Défi | Automatisation du déploiement d'infrastructures |
| Atelier guidé | Création d'applications découplées à l'aide d'Amazon SQS |
| Atelier guidé | Mise en œuvre d'une architecture sans serveur sur AWS |
| Atelier Défi | Mise en œuvre d'une architecture sans serveur pour le café |
| Atelier guidé | Configuration du stockage hybride et migration de données avec AWS Storage Gateway |

**Projet basé sur un cas pratique et évaluation finale Cloud Architecting :**

| Évaluation | Titre |
|------------|-------|
| Projet | Projet basé sur un cas pratique (café) — synthèse de tous les modules |
| Évaluation finale | Évaluation du cours AWS Academy Cloud Architecting |

> [!NOTE]
> Ce chapitre regroupe la totalité des modules Architecting restants (10 à 17), car ils partagent tous la même logique de clôture de formation : automatiser, superviser, découpler et préparer la certification — cohérent avec le cours (CloudFormation, Systems Manager, CloudWatch, Well-Architected, SQS/SNS, architectures découplées §8.3, certifications §9). C'est le chapitre le plus dense de la formation ; prévoir du temps supplémentaire si besoin.
---

## Déroulé guidé des modules et ateliers

<div class="lab-route">
<details open><summary><strong>Foundations M9 et M10 + Atelier 6</strong></summary>

1. Évaluer l’architecture étudiée avec les six piliers Well-Architected.
2. Dans l’atelier de mise à l’échelle, repérer le modèle de lancement, le groupe cible, le Load Balancer et l’Auto Scaling Group.
3. Générer la charge prévue par Academy et observer métriques, alarmes et capacité désirée.
4. Attendre le retour à l’équilibre puis expliquer la réduction de capacité.

**Validation** : retracer la chaîne métrique → politique → capacité → instance → contrôle de santé.
</details>

<details open><summary><strong>Architecting M11 — CloudFormation</strong></summary>

1. Lire le template avant de le déployer : paramètres, ressources, dépendances et sorties.
2. Valider le YAML puis créer la stack selon l’atelier Academy.
3. Suivre les événements et relier chaque ressource logique à sa ressource physique.
4. Modifier le template, examiner le changement attendu puis mettre la stack à jour.
5. Supprimer la stack et vérifier la disparition des ressources gérées.

**Validation** : expliquer idempotence, rollback et intérêt d’un change set.
</details>

<details><summary><strong>Architecting M13 — SQS et découplage</strong></summary>

1. Identifier producteur, file, consommateur et message.
2. Exécuter l’atelier SQS et envoyer un message de test.
3. Recevoir le message sans le supprimer, puis observer sa réapparition après le délai de visibilité.
4. Expliquer l’idempotence du consommateur et le rôle d’une dead-letter queue.

**Validation** : expliquer comment la file absorbe un écart de rythme entre deux composants.
</details>

<details><summary><strong>Architecting M14 — Architecture sans serveur</strong></summary>

1. Tracer le chemin événement → API Gateway → Lambda → service de données.
2. Suivre l’atelier serverless Academy et tester une invocation réussie.
3. Consulter les logs CloudWatch et identifier entrée, résultat et erreur éventuelle.
4. Comparer ce modèle à un service EC2 permanent.

**Validation** : justifier serverless avec le profil de charge et les contraintes d’exploitation, pas uniquement avec le mot « moderne ».
</details>

<details><summary><strong>Architecting M15 et M16 — Données et reprise</strong></summary>

1. Identifier ingestion, stockage, transformation et restitution dans le modèle de données étudié.
2. Pour la reprise, distinguer RPO et RTO puis classer sauvegarde/restauration, pilote léger, secours actif-passif et multi-site.
3. Dans l’atelier Storage Gateway, repérer les composants locaux, le service AWS et les données transférées.
4. Proposer un contrôle permettant de prouver qu’une restauration fonctionne réellement.

**Validation** : relier une exigence métier de reprise à une architecture et à son coût.
</details>

<details><summary><strong>Architecting M17 — Synthèse et certification</strong></summary>

1. Reprendre l’architecture du café et nommer chaque décision, hypothèse et risque.
2. Associer chaque décision à un pilier Well-Architected.
3. Identifier les sujets maîtrisés, ceux à revoir et ceux à pratiquer à nouveau.
4. Compléter l’évaluation finale Academy selon les consignes de la formatrice.

**Validation** : défendre une architecture en expliquant ses compromis plutôt qu’en énumérant ses services.
</details>
</div>

- [ ] Je sais lire et faire évoluer un template CloudFormation.
- [ ] Je sais expliquer une mise à l’échelle à partir d’une métrique.
- [ ] Je distingue couplage synchrone et découplage par message.
- [ ] Je sais relier RPO/RTO à une stratégie de reprise.

---

*Fin du Chapitre 5 — Travaux Pratiques*
