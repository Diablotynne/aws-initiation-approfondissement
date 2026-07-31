# Chapitre 3 — Travaux Pratiques : Stockage et Calcul — Amazon S3 & Amazon EC2

> [!NOTE]
> La liste est volontairement exhaustive. En séance, privilégiez un parcours cohérent : S3, lancement EC2 et stockage EBS. Les activités de réplication, accélération, Spot, Windows, AMI et EFS sont sélectionnées par le formateur ou proposées en approfondissement autonome.
---

## 🎓 AWS Academy Cloud Foundations — Stagiaires académiques

**Modules associés à ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 7 | Stockage (S3, EBS, EFS, Glacier) | ✅ À compléter |
| Module 6 | Calcul (EC2, Lambda, Beanstalk) | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre | Points |
|---------|-------|--------|
| **Atelier 3** | Présentation d'Amazon EC2 | 100 pts |
| **Atelier 4** | Utilisation d'EBS | 100 pts |

---

## 🎓 AWS Academy Cloud Architecting — Stagiaires académiques

**Modules à compléter pour ce chapitre :**

| Module | Titre | Contrôle des connaissances |
|--------|-------|---------------------------|
| Module 4 | Ajout d'une couche de stockage avec Amazon S3 | ✅ À compléter |
| Module 5 | Ajout d'une couche de calcul à l'aide d'Amazon EC2 | ✅ À compléter |
| Module 12 | Mise en cache du contenu | ✅ À compléter |

**Ateliers guidés :**

| Atelier | Titre |
|---------|-------|
| Atelier Défi | Création d'un site web statique pour le café (S3) |
| Atelier guidé | Présentation d'Amazon Elastic File System (EFS) |
| Atelier Défi | Création d'un site web dynamique pour le café (EC2) |
| Atelier guidé | Streaming de contenu dynamique avec Amazon CloudFront |

> [!NOTE]
> Le Module 12 (mise en cache avec CloudFront et ElastiCache) prolonge naturellement ce chapitre plutôt que le Chapitre 4 : il s'appuie directement sur le contenu S3/EC2 déjà déployé pour illustrer l'accélération de sa diffusion.
---

## Déroulé guidé des modules et ateliers

<div class="lab-route">
<details open><summary><strong>Foundations M7 + Architecting M4 — Amazon S3</strong></summary>

1. Distinguer stockage objet, bloc et fichier avant d’ouvrir la console.
2. Dans le module S3, repérer bucket, clé d’objet, version, classe de stockage et politique.
3. Dans l’atelier du site statique, suivre l’énoncé Academy pour déposer le contenu et contrôler son accès.
4. Vérifier le blocage de l’accès public, le chiffrement et le versioning selon le scénario.
5. Formuler une règle de cycle de vie adaptée à des données rarement consultées.

**Validation** : expliquer pourquoi un objet n’est ni un fichier EFS ni un bloc EBS.
</details>

<details open><summary><strong>Foundations M6 + Atelier 3 — Amazon EC2</strong></summary>

1. Choisir une AMI, un type d’instance, un subnet, un rôle et un Security Group à partir du besoin.
2. Lancer l’instance en suivant l’atelier Academy, sans élargir les autorisations proposées.
3. Vérifier son état, son adresse, son rôle IAM et ses contrôles de statut.
4. Accéder au service déployé par le lab et relier chaque étape au chemin réseau.
5. Comparer arrêt, redémarrage et terminaison.

**Validation** : identifier les éléments qui persistent après l’arrêt d’une instance.
</details>

<details><summary><strong>Atelier 4 EBS + atelier EFS</strong></summary>

1. Repérer l’AZ de l’instance et du volume EBS.
2. Créer ou attacher le volume prévu par l’énoncé, puis vérifier son montage.
3. Écrire une donnée de test et observer sa persistance indépendamment du processus applicatif.
4. Pour EFS, observer le point de montage partagé entre plusieurs instances.
5. Comparer portée, protocole, concurrence d’accès et usages EBS/EFS.

**Validation** : choisir EBS pour un disque système et EFS pour un partage Linux multi-instance.
</details>

<details><summary><strong>Architecting M12 — CloudFront et mise en cache</strong></summary>

1. Identifier l’origine, la distribution et les emplacements périphériques.
2. Exécuter l’atelier CloudFront proposé dans Academy.
3. Comparer l’URL d’origine et l’URL distribuée.
4. Modifier un objet et expliquer l’effet du TTL et d’une invalidation.

**Validation** : expliquer ce que CloudFront accélère et ce qu’il ne remplace pas.
</details>
</div>

- [ ] Je sais choisir entre S3, EBS et EFS.
- [ ] Je peux justifier l’AMI, le type d’instance et le Security Group.
- [ ] Je comprends le rôle d’un Load Balancer et d’un Auto Scaling Group.

---

*Fin du Chapitre 3 — Travaux Pratiques*
