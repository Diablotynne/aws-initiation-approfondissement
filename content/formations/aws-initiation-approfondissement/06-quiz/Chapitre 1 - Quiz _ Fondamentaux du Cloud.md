# Chapitre 1 - Quiz : Formation AWS — Initiation + Approfondissement
## Fondamentaux du Cloud Computing et présentation AWS

---

## Instructions

- **1 seul choix correct** par question (sauf mention contraire)
- Notez vos réponses sur papier (A, B, C ou D) avant de consulter le corrigé
- Le corrigé se trouve en bas de ce document — ne le consultez qu'après avoir répondu à toutes les questions
- Durée recommandée : 1,5 minute par question

---

## Questions

---

**Q1 — Définition NIST du Cloud Computing**

Selon la définition officielle du NIST (National Institute of Standards and Technology), combien de caractéristiques essentielles définissent le Cloud Computing ?

- A) 3 caractéristiques (accès réseau, ressources partagées, facturation à l'usage)
- B) 4 caractéristiques (on-demand, réseau, élasticité, service mesuré)
- C) 5 caractéristiques (on-demand self-service, accès réseau large bande, mutualisation, élasticité rapide, service mesuré)
- D) 6 caractéristiques (les 5 du NIST + haute disponibilité)

---

**Q2 — CAPEX vs OPEX**

Une entreprise décide de migrer son datacenter on-premise vers AWS. Quelle affirmation décrit le mieux le changement de modèle financier ?

- A) L'entreprise passe d'un modèle OPEX (dépenses opérationnelles) à un modèle CAPEX (dépenses d'investissement)
- B) L'entreprise passe d'un modèle CAPEX à un modèle OPEX — les coûts deviennent des charges d'exploitation variables
- C) Le modèle financier ne change pas, seule la localisation des serveurs change
- D) AWS impose toujours un modèle CAPEX via les Reserved Instances

---

**Q3 — Modèle de responsabilité partagée**

Dans le modèle de responsabilité partagée AWS, qui est responsable de la mise à jour du système d'exploitation d'une instance EC2 ?

- A) AWS, car elle gère toute l'infrastructure physique et logicielle
- B) Le client, car la mise à jour de l'OS invité est toujours de sa responsabilité pour une IaaS
- C) AWS pour les instances Linux, le client pour les instances Windows
- D) Cela dépend du contrat de support AWS souscrit

---

**Q4 — Classification des services : EC2**

Dans quel modèle de service cloud Amazon EC2 (Elastic Compute Cloud) est-il classé ?

- A) SaaS (Software as a Service)
- B) PaaS (Platform as a Service)
- C) IaaS (Infrastructure as a Service)
- D) FaaS (Function as a Service)

---

**Q5 — Classification des services : Lambda**

AWS Lambda est un service d'exécution de code sans gestion de serveur. Dans quel modèle de service est-il classé ?

- A) IaaS (Infrastructure as a Service)
- B) PaaS (Platform as a Service)
- C) SaaS (Software as a Service)
- D) FaaS (Function as a Service) — souvent rattaché au modèle Serverless/PaaS

---

**Q6 — Infrastructure AWS : Régions et Zones de Disponibilité**

Quelle affirmation sur les Régions et Zones de Disponibilité (AZ) AWS est correcte ?

- A) Une Région contient exactement 2 Zones de Disponibilité, pour assurer la redondance
- B) Une Zone de Disponibilité est constituée d'un ou plusieurs datacenters physiquement séparés au sein d'une même Région
- C) Les Zones de Disponibilité d'une Région sont toutes situées dans le même datacenter physique
- D) Une Région et une Zone de Disponibilité désignent la même notion — ce sont deux termes synonymes

---

**Q7 — AWS Free Tier**

Quelle affirmation décrit le mieux le Free Tier proposé à un nouveau client qui crée son compte après le 15 juillet 2025 ?

- A) Toutes les ressources EC2, S3 et RDS sont gratuites sans limite pendant douze mois
- B) Le client reçoit 100 USD de crédits à l'inscription, peut gagner jusqu'à 100 USD supplémentaires et le plan gratuit est limité à six mois
- C) Aucun moyen de paiement ni contrôle de facturation n'est nécessaire
- D) Le compte ne peut générer aucun coût tant que le solde de crédits n'est pas épuisé

---

**Q8 — Code de la région Paris**

Quel est le code identifiant de la région AWS de Paris ?

- A) `eu-central-1`
- B) `eu-south-1`
- C) `eu-west-3`
- D) `fr-paris-1`

---

**Q9 — Commande CLI d'identification**

Vous souhaitez vérifier quel utilisateur ou rôle IAM est associé à vos credentials AWS CLI actuels. Quelle commande utilisez-vous ?

- A) `aws iam get-user`
- B) `aws sts get-caller-identity`
- C) `aws configure list`
- D) `aws iam whoami`

---

**Q10 — Modèle de déploiement et souveraineté des données**

FormaTech doit héberger les données RH de ses 80 salariés (données personnelles soumises au RGPD). L'entreprise souhaite conserver un contrôle maximal sur ces données tout en bénéficiant d'une infrastructure moderne. Quel modèle de déploiement cloud est le plus adapté ?

- A) Cloud public AWS uniquement — AWS est certifié RGPD et gère la conformité
- B) Cloud hybride — les données RH restent on-premise ou dans un Cloud privé, les autres charges vont sur AWS public
- C) Cloud communautaire partagé entre PME lyonnaises — pour mutualiser les coûts de conformité
- D) Aucun cloud — les données RGPD ne peuvent jamais quitter un serveur physique en France

---

## Corrigé

> Ne lisez cette section qu'après avoir répondu aux 10 questions.

---

**Q1 — Réponse : C**

Le NIST définit 5 caractéristiques essentielles du Cloud Computing :
1. On-demand self-service (accès autonome sans intervention humaine)
2. Broad network access (accès via le réseau standard)
3. Resource pooling (mutualisation des ressources entre clients)
4. Rapid elasticity (élasticité et scalabilité rapides)
5. Measured service (facturation et monitoring à l'usage)

Ces 5 caractéristiques sont la base de toute définition du Cloud — à retenir pour la certification Cloud Practitioner.

---

**Q2 — Réponse : B**

Migrer vers le Cloud transforme les investissements en infrastructure (CAPEX — achat de serveurs, licences, immobilisation) en charges d'exploitation variables (OPEX — factures mensuelles proportionnelles à l'usage). C'est l'un des arguments économiques majeurs du Cloud : on passe d'un budget prévisionnel fixe à un coût variable optimisable.

---

**Q3 — Réponse : B**

Dans le modèle de responsabilité partagée AWS :
- **AWS** est responsable de la sécurité **de** l'infrastructure cloud (hyperviseurs, réseau physique, datacenters, hardware)
- **Le client** est responsable de la sécurité **dans** le cloud (OS, patches, applications, données, configuration réseau)

Pour EC2 (IaaS), la mise à jour de l'OS invité incombe toujours au client, quel que soit l'OS. Les services managés (RDS, Lambda) transfèrent plus de responsabilités vers AWS.

---

**Q4 — Réponse : C**

EC2 est un service **IaaS** (Infrastructure as a Service). AWS fournit la machine virtuelle et l'accès à ses ressources (CPU, RAM, stockage, réseau) — mais l'installation et la gestion du système d'exploitation, des middlewares et des applications restent entièrement à la charge du client.

---

**Q5 — Réponse : D**

AWS Lambda est un service **FaaS (Function as a Service)**, catégorie appartenant au mouvement Serverless. Le client écrit uniquement le code de sa fonction — AWS gère entièrement l'infrastructure sous-jacente, la mise à l'échelle automatique et la facturation à l'exécution (à la milliseconde). C'est le niveau d'abstraction le plus élevé dans la hiérarchie IaaS → PaaS → FaaS/SaaS.

---

**Q6 — Réponse : B**

Une **Zone de Disponibilité (AZ)** est composée d'un ou plusieurs datacenters au sein d'une même Région. Les AZ sont physiquement séparées et reliées par un réseau dédié à faible latence, haut débit et forte redondance. Les régions AWS actuellement documentées disposent d'au moins trois AZ, mais une architecture ne devient hautement disponible que si la charge est réellement répartie sur plusieurs AZ.

---

**Q7 — Réponse : B**

Pour un compte créé après le 15 juillet 2025, AWS annonce **100 USD de crédits à l'inscription** et jusqu'à **100 USD supplémentaires** obtenus via des activités. Le plan gratuit est limité à six mois. Un dépassement de crédit ou l'utilisation d'un service non couvert peut entraîner une facturation au tarif standard ; il faut donc consulter la page Free Tier et la facturation du compte actif. Les comptes antérieurs à cette date peuvent encore relever du régime historique.

---

**Q8 — Réponse : C**

Le code de la région Paris est **`eu-west-3`**. Mémo :
- `eu-west-1` = Irlande (Dublin)
- `eu-west-2` = Royaume-Uni (Londres)
- `eu-west-3` = France (Paris)
- `eu-central-1` = Allemagne (Francfort)

La région Paris a été ouverte en décembre 2017 et dispose de 3 Zones de Disponibilité.

---

**Q9 — Réponse : B**

La commande `aws sts get-caller-identity` retourne trois informations sur l'identité associée aux credentials actifs :
- **UserId** : identifiant unique de l'entité IAM
- **Account** : numéro de compte AWS (12 chiffres)
- **Arn** : ARN complet de l'utilisateur ou du rôle

`aws iam get-user` ne fonctionne pas avec les rôles (uniquement les users IAM). `aws configure list` affiche la configuration CLI mais pas l'identité réelle. `aws iam whoami` n'existe pas.

---

**Q10 — Réponse : B**

Le **Cloud hybride** est la solution la plus adaptée pour FormaTech. Il permet de :
- Maintenir les données RH sensibles on-premise ou dans un Cloud privé souverain (ex. : OVHcloud SecNumCloud)
- Migrer les charges applicatives moins sensibles (plateforme e-learning, contenus vidéo) sur AWS public
- Respecter les exigences RGPD sur la localisation et la maîtrise des données personnelles

La réponse A est partiellement vraie (AWS est bien certifié ISO 27001/27701 et offre des garanties RGPD) mais le "contrôle maximal" mentionné dans la question oriente vers le modèle hybride. La réponse D est incorrecte : le RGPD n'interdit pas le Cloud, il impose des garanties contractuelles et techniques.

---

## Barème

| Score | Appréciation | Commentaire |
|-------|-------------|-------------|
| 10/10 | Excellent | Maîtrise complète des fondamentaux Cloud et AWS |
| 8-9/10 | Bien | Bonne compréhension — revoyez les 1-2 points manqués |
| 7/10 | Satisfaisant | Score minimum — les bases sont là, approfondissez les notions floues |
| 5-6/10 | Insuffisant | Relire le cours Chapitre 1 avant de continuer |
| < 5/10 | À revoir | Reprendre le Chapitre 1 depuis le début |

---

## Question ouverte — Réflexion FormaTech (bonus)

> Cette question ne compte pas dans la note — elle vise à ancrer les concepts dans un contexte professionnel concret.

**Énoncé :**

FormaTech hésite entre deux options pour sa prochaine phase de migration :

- **Option A — Cloud Public AWS uniquement** : tout migrer sur AWS, y compris les données RH des salariés et les données des apprenants (nom, email, progression pédagogique)
- **Option B — Cloud Hybride** : plateforme e-learning et contenus sur AWS, données RH et données personnelles des apprenants dans un hébergement privé en France

**Question :** Quel(s) argument(s) défendreriez-vous pour conseiller FormaTech ? Prenez en compte les aspects techniques, juridiques (RGPD), opérationnels et financiers.

**Pistes de réflexion :**
- Quelle est la différence entre données RH (salariés) et données apprenants (clients) au sens RGPD ?
- AWS peut-il garantir que les données restent en France avec eu-west-3 ?
- Quel est le coût opérationnel de maintenir deux infrastructures (hybride vs tout-cloud) ?
- Existe-t-il des alternatives souveraines françaises à AWS (OVHcloud, Scaleway, Outscale) ?

---

*Fin du Chapitre 1 — Quiz*
*Formation AWS Initiation + Approfondissement — Dawan — Formateur Dawan*
