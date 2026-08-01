---
title: "11. Architecture complète : Illustration e-commerce"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 11. Architecture complète : Illustration e-commerce"
---

<nav class="page-sequence"><a href="cours/chapitre-3/lambda">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/haute-disponibilite">Suivant</a></nav>

Scénario réaliste — montée en charge pendant les soldes, puis retour à la normale :

### 11.1 Avant les soldes (charge normale)

Les clients normaux passent par l'Application Load Balancer, qui répartit le trafic sur trois instances EC2 (EC2-1, EC2-2, EC2-3). L'Auto Scaling Group est configuré avec Min=2, Max=10, Desired=3.

### 11.2 Pendant les soldes (pic de trafic)

Le trafic est multiplié par 5, le CPU moyen passe à 85% — cela déclenche le scale-out : 4 instances supplémentaires sont ajoutées derrière l'Application Load Balancer, portant le total à 7 instances actives (Desired Capacity = 7).

### 11.3 Après les soldes (retour à la normale)

```text
Trafic revient à la normale

    CPU chute à 40%

    Déclenchement du scale-in

    Retirer 4 instances

    Desired Capacity = 3
    Coûts réduits
```

---

<nav class="page-sequence"><a href="cours/chapitre-3/lambda">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/haute-disponibilite">Suivant</a></nav>
