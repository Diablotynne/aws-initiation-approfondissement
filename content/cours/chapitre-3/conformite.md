---
title: "13. Conformité et sécurité pour les données sensibles"
description: "Chapitre 3 — Stockage Amazon S3 et calcul Amazon EC2 - 13. Conformité et sécurité pour les données sensibles"
---

# 13. Conformité et sécurité pour les données sensibles

<nav class="page-sequence"><a href="cours/chapitre-3/haute-disponibilite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/points-attention">Suivant</a></nav>

Les environnements soumis à des réglementations (RGPD, HIPAA, PCI-DSS) nécessitent des garanties strictes.

### 13.1 Frameworks de conformité AWS

| Framework | Objectif | Services AWS applicables |
|-----------|----------|-------------------------|
| **RGPD** | Protection des données personnelles UE | Encryption, Data Residency, CloudTrail |
| **HIPAA** | Confidentialité des données santé | Dedicated Instance, Encrypted EBS, Audit logs |
| **PCI-DSS** | Sécurité des données cartes bancaires | VPC isolé, Encryption, Firewall |
| **ISO 27001** | Gestion de la sécurité informatique | IAM, KMS, CloudTrail, Monitoring |

### 13.2 Bonnes pratiques de conformité pour S3

- ✅ Chiffrement : SSE-KMS (clés maîtrisées)
- ✅ Versioning : actif (trace des modifications)
- ✅ Bucket Policy : restreint à IP/domaine
- ✅ Logging : S3 Access Logs dans un bucket séparé
- ✅ CloudTrail : audit API dans le compte AWS
- ✅ MFA Delete : protection contre la suppression
- ✅ Block Public : tous les accès publics bloqués
- ✅ Lifecycle : archivage des données obsolètes
- ✅ Réplication CRR : backup multi-région

### 13.3 Bonnes pratiques de conformité pour EC2

- ✅ Dedicated Instance : pas de partage d'hôte physique
- ✅ EBS chiffré : SSE-KMS pour tous les volumes
- ✅ Security Group : minimaliste (moindre privilège)
- ✅ IAM Role : permissions spécifiques au rôle
- ✅ CloudWatch Agent : logs applicatifs
- ✅ VPC privé : pas d'accès Internet direct
- ✅ Snapshots EBS : conservés X années
- ✅ Patch Management : système à jour
- ✅ Monitoring : alertes sur anomalies

### 13.4 Exemple : Architecture RGPD multi-région

```text
Région EU (Ireland)
- VPC Privé
   - Subnets privés (applications)
   - Subnet public (NAT Gateway seulement)
   - Security Group très restrictif
- S3 Bucket
   - Versioning activé
   - SSE-KMS (clé EU managée)
   - Bucket Policy : IP/IAM restrictifs
   - CloudTrail logging
- EC2 Instances
   - Dedicated Instance
   - EBS chiffré KMS
   - Snapshots quotidiens → S3
   - CloudWatch + CloudTrail

Région EU (Frankfurt) — Backup
- S3 Réplication CRR du bucket EU-Ireland
   - Préservé 7 ans (conformité)
```

### 13.5 Audit et certification

**AWS Artifact** : plateforme d'AWS pour les certifications de conformité. Les rapports téléchargés (SOC 2, ISO 27001) servent à prouver à vos clients ou auditeurs qu'AWS respecte les normes de sécurité.

```bash
# Dans la console AWS → Security, Identity & Compliance → Artifact
# Télécharger :
# - AWS Compliance Summary
# - SOC 2 Type II reports
# - ISO 27001 certificates
# Utile pour les audits externes
```

**CloudTrail** pour l'audit — ces commandes permettent de lister les trails actifs et de rechercher les actions effectuées sur une ressource précise (ici, un bucket S3) :
```bash
aws cloudtrail describe-trails --region eu-west-1
aws cloudtrail list-events \
    --region eu-west-1 \
    --max-results 50 \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=mon-bucket
```

:::success
**Résultat attendu :**
```json
# describe-trails retourne :
{
    "trailList": [
        {
            "Name": "management-events-trail",
            "S3BucketName": "my-cloudtrail-logs-bucket",
            "IncludeGlobalServiceEvents": true,
            "IsMultiRegionTrail": true,
            "HomeRegion": "eu-west-1",
            "TrailARN": "arn:aws:cloudtrail:eu-west-1:123456789012:trail/management-events-trail",
            "LogFileValidationEnabled": true
        }
    ]
}

# list-events retourne des événements du type :
{
    "Events": [
        {
            "EventId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
            "EventName": "PutObject",
            "ReadOnly": "false",
            "EventTime": "2024-05-17T14:23:05+00:00",
            "Username": "stagiaire-demo",
            "Resources": [
                {
                    "ResourceType": "AWS::S3::Object",
                    "ResourceName": "mon-bucket/documents/rapport.pdf"
                }
            ]
        }
    ]
}
```
:::

---

<nav class="page-sequence"><a href="cours/chapitre-3/haute-disponibilite">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-3/index">Sommaire</a> <a href="cours/chapitre-3/points-attention">Suivant</a></nav>
