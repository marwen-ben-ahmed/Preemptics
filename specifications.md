# Cahier de Charges Détaillé - Application CTI Intelligence

## 1. 📋 Présentation du Projet V0

### 1.1 Contexte et Enjeux Stratégiques V0

**Objectif principal V0** : Développer une application web responsive CTI focalisée sur l'analyse et la corrélation des sources Wazuh/OpenVAS avec restitution selon framework MITRE ATT&CK et langage Kestrel.

**Enjeux critiques V0** :
- Centralisation et corrélation intelligente des alertes Wazuh/OpenVAS
- Mapping automatique vers techniques MITRE ATT&CK
- Interface de hunting Kestrel intégrée avec sources actuelles
- Dashboard temps réel pour amélioration réactivité SOC

### 1.2 Périmètre Fonctionnel V0

**Sources d'ingestion V0** : Wazuh SIEM, OpenVAS Scanner, OTX (AlienVault/AT&T)
**Standards supportés** : MITRE ATT&CK Enterprise, STIX 2.1, Kestrel
**Restitution** : Dashboard temps réel, alertes intelligentes, hunt reports
**Architecture** : Application web responsive avec RAG Azure OpenAI intégré

**Modules V0** :
- 🏠 **Dashboard Principal** : Alertes temps réel, KPIs, timeline tactique
- 🛡️ **Sources CTI** : Intégration Wazuh SIEM + OpenVAS Scanner + OTX
- 🔗 **Liaison Alertes → MITRE → Kestrel** : Mapping automatique + hunt transparent
- 🤖 **RAG Intelligence** : Azure OpenAI pour analyse TTPs et enrichissement

---

## 2. 🎯 Exigences Fonctionnelles V0

### 2.1 Module Dashboard Principal V0

**F001 - Dashboard Temps Réel Unifié**
- Vue centralisée des alertes Wazuh + vulnérabilités OpenVAS + pulses OTX
- **Système de scoring tri-sources** : Criticité Wazuh (0-15) + CVSS OpenVAS (0-10) + Confidence OTX (0-10)
- **Regroupement par technique ATT&CK** : Organisation intelligente des alertes selon MITRE
- Timeline tactique des 7 derniers jours avec évolution des menaces
- Filtrage avancé (criticité, source, statut, date, host, technique)

**F002 - Workflow de Gestion d'Alertes Mixte**
- **Auto-assignation** : Alertes critiques (score >25) → Analyste Senior automatique
- **Assignation manuelle** : Alertes moyennes → Pool d'analystes disponibles
- Workflow : Nouveau → Assigné → En cours → Résolu → Archivé
- Actions rapides : Assign, Close, Escalate, **Generate Hunt with RAG**
- Notifications temps réel avec WebSocket et historique complet

**Critères d'acceptation F001-F002** :
- ✅ Temps de chargement dashboard rapide
- ✅ Auto-refresh intelligent sans perte de performance
- ✅ Support 100+ alertes simultanées sans dégradation
- ✅ Interface responsive desktop/tablette/mobile
- ✅ Filtrage temps rapide

### 2.2 Module Sources CTI V0

**F003 - Intégration Wazuh SIEM**
- Connexion API REST Wazuh avec authentification sécurisée
- Récupération alertes temps réel et logs historiques
- Parsing automatique des règles et événements Wazuh
- Extraction IOCs depuis logs (IPs, domaines, processus, fichiers)
- Mapping automatique règles Wazuh → techniques MITRE ATT&CK

**F004 - Intégration OpenVAS Scanner**
- Connexion API GMP (Greenbone Management Protocol)
- Import automatique rapports de vulnérabilités XML/PDF
- Extraction CVE et scores CVSS avec criticité
- Corrélation CVE → techniques d'exploitation MITRE ATT&CK
- Priorisation par criticité et exploitabilité

**Critères d'acceptation F003-F004** :
- ✅ Connexion Wazuh/OpenVAS/OTX validée sur environnement client
- ✅ Synchronisation temps réel opérationnelle
- ✅ Mapping ATT&CK automatique avec précision > 85%
- ✅ Gestion d'erreurs robuste avec retry automatique

### 2.3 Module Liaison Alertes → MITRE → Kestrel V0

**F006 - Mapping Automatique Alertes → MITRE ATT&CK**
- **Pas de matrice interactive** : Focus sur liaison directe alertes
- Mapping automatique sources Wazuh/OpenVAS/OTX → techniques ATT&CK
- Affichage technique(s) identifiée(s) directement dans contexte alerte
- Base de données ATT&CK en arrière-plan (pas d'interface matrice)
- Export mapping et statistiques de couverture tri-sources

**F007 - Génération Automatique Hunt Kestrel depuis Pipeline RAG**
- **Input structuré RAG** : TTPs MITRE + raisonnement + contexte STIX
- **Génération template** : Requête Kestrel optimisée selon TTPs identifiés par RAG
- **Workflow automatique technique** : 
  1. Données sources → Normalisation STIX 2.1
  2. STIX objects → RAG Analysis → TTPs MITRE + reasoning
  3. TTPs + reasoning → Génération requête Kestrel contextuelle
  4. Exécution hunt automatique avec corrélation tri-sources
  5. Résultats + raisonnement affichés dans contexte alerte
- **Templates intelligents** : Kestrel adapté selon TTPs et environnement client
- **Enrichissement contextuel** : Résultats hunt + raisonnement IA intégrés

**Exemple Pipeline RAG → Kestrel** :
```json
{
   "input_stix":{
      "stix_bundle":{
         "type":"bundle",
         "id":"bundle--e7eae5c3-14c1-4c8d-9f9a-7f8e6c71f7b9",
         "objects":[
            {
               "type":"indicator",
               "spec_version":"2.1",
               "id":"indicator--f54f7f78-2174-4e4f-99d0-54a1e43d52a4",
               "created":"2025-08-14T09:00:00Z",
               "modified":"2025-08-14T09:00:00Z",
               "name":"Suspicious SSH brute force IP",
               "description":"IP address seen in multiple failed SSH login attempts.",
               "pattern":"[ipv4-addr:value = '203.0.113.45']",
               "pattern_type":"stix",
               "valid_from":"2025-08-14T08:55:00Z",
               "labels":[
                  "malicious-activity",
                  "brute-force"
               ]
            },
            {
               "type":"ipv4-addr",
               "spec_version":"2.1",
               "id":"ipv4-addr--f12bcb31-c412-4c67-9731-dc1e593cf35a",
               "value":"203.0.113.45"
            },
            {
               "type":"vulnerability",
               "spec_version":"2.1",
               "id":"vulnerability--a8b7b9d4-7a5b-4d93-aaa2-318fcbcd74d4",
               "name":"OpenSSH User Enumeration Vulnerability",
               "description":"CVE-2024-5678 allows enumeration of valid usernames via timing attacks.",
               "external_references":[
                  {
                     "source_name":"cve",
                     "external_id":"CVE-2024-5678"
                  }
               ]
            },
            {
               "type":"malware",
               "spec_version":"2.1",
               "id":"malware--7b6c3b75-b6f8-4e65-a4b7-5ef485adbc4c",
               "name":"SSH Credential Harvester",
               "is_family":false,
               "description":"Tool designed to capture SSH credentials through brute-force attacks."
            }
         ]
      },
      "options":{
         "min_confidence":0.8,
         "return_kestrel":true,
         "kestrel_hunt_goal":"Detect SSH brute-force and account enumeration",
         "max_tokens":1500,
         "temperature":0.1
      }
   },
   "rag_output":{
      "identified_ttps":[
         {
            "technique":"T1110.001",
            "name":"Brute Force: Password Guessing",
            "confidence":0.9,
            "reasoning":"The indicator describes an IP address involved in multiple failed SSH login attempts, which aligns with the brute force technique of password guessing."
         },
         {
            "technique":"T1588.002",
            "name":"Obtain Capabilities: Tool",
            "confidence":0.8,
            "reasoning":"The malware 'SSH Credential Harvester' is described as a tool for capturing SSH credentials through brute-force attacks, indicating the use of a specific tool for credential harvesting."
         }
      ],
      "overall_reasoning":"The STIX bundle provides indicators of SSH brute force attempts, a specific malware tool for credential harvesting, and a vulnerability that can be exploited for user enumeration. These elements collectively suggest a coordinated effort to compromise SSH credentials through brute force and exploitation techniques. The presence of a specific IP address involved in failed login attempts, a tool designed for credential harvesting, and a vulnerability that aids in user enumeration supports the identification of these MITRE ATT&CK techniques.",
      "generated_kestrel_query":"find ipv4-addr with value = '203.0.113.45'; find indicator where pattern = '[ipv4-addr:value = \"203.0.113.45\"]'; find malware where name = 'SSH Credential Harvester'; find vulnerability where external_references.external_id = 'CVE-2024-5678';",
      "audit":{
         "latency_ms":3774,
         "model":"azure-openai:gpt-4o-cybersec",
         "prompt_version":"rag-v1.0",
         "inputs_hash":"1f60feeed5434be6"
      }
   }
}
```

**Critères d'acceptation F006-F007** :
- ✅ Mapping automatique tri-sources → ATT&CK avec précision importante
- ✅ Génération + exécution hunt automatique sans interface dédiée
- ✅ Workflow intégré dans dashboard alertes uniquement
- ✅ Corrélation intelligente des 3 sources dans hunt
- ✅ Enrichissement alerte avec techniques et résultats hunt tri-sources

### 2.4 Module Intelligence Artificielle RAG V0

**F008 - Pipeline RAG pour Analyse STIX → ATT&CK → Kestrel**
- **Input standardisé** : Données STIX 2.1 normalisées depuis sources tri-sources
- **Analyse contextuelle IA** : Azure OpenAI GPT analyse objets STIX (indicators, observables, patterns)
- **Output structuré** : 
  1. **TTPs MITRE ATT&CK** correspondants avec scores de confiance
  2. **Raisonnement explicite** : Justification mapping STIX → ATT&CK
  3. **Requête Kestrel générée** : Template hunt optimisé pour TTPs identifiés
- **Enrichissement contextuel** : Recommandations de mitigation basées sur environnement
- **Traçabilité complète** : Log du raisonnement IA pour audit et amélioration

**Pipeline RAG Technique Détaillé** :
```
[Sources Tri-Sources] 
        ↓
[Normalisation STIX 2.1] 
        ↓
[RAG Analysis Engine]
├── Input: STIX 2.1 Objects (indicators, patterns, observables)
├── Processing: Azure OpenAI GPT-4 contextual analysis
├── Knowledge Base: MITRE ATT&CK Enterprise + Environmental Context
└── Output: Structured Response
        ↓
[Structured RAG Output]
├── 1. TTPs MITRE ATT&CK: ["T1566.001", "T1059.001"] + confidence scores
├── 2. Reasoning: "STIX email-addr + malware → Spearphishing + PowerShell execution"
└── 3. Kestrel Query: Generated hunt template for identified TTPs
        ↓
[Automatic Hunt Execution]
        ↓
[Enriched Alert Display]
```

**Critères d'acceptation F008** :
- ✅ **Input STIX 2.1** : Processing correct de tous objets STIX depuis tri-sources
- ✅ **TTPs identification** : Précision > 85% mapping STIX → MITRE ATT&CK
- ✅ **Raisonnement explicite** : Justification claire et auditable du mapping
- ✅ **Requête Kestrel** : Génération automatique template hunt fonctionnel
- ✅ **Temps processing** : Pipeline complet et rapide
- ✅ **Traçabilité** : Log raisonnement IA pour amélioration continue

---

## 3. 🔧 Exigences Techniques Détaillées

### 3.1 Architecture Technique Validée

**Frontend** :
- Framework : Angular 18 + TypeScript strict
- UI : Bootstrap 5 + Admin Template professionnel
- Design : Responsive avec sidebar navigation
- State Management : NgRx pour gestion état complexe

**Backend** :
- Framework : Django 4.2+ + Django REST Framework
- API : REST + WebSockets (Socket.IO) pour temps réel
- Authentification : Django Auth + JWT + OAuth 2.0/SAML 2.0

**Base de Données** :
- PostgreSQL : Métadonnées, utilisateurs, configuration
- MongoDB : Données CTI non-structurées, IOCs
- ElasticSearch : Indexation et recherche full-text
- Redis : Cache, sessions, message queuing

**Sécurité** :
- Chiffrement : TLS 1.3 en transit, AES-256 au repos
- Isolation : Sandboxing pour analyse fichiers suspects
- Audit : Logs complets avec traçabilité RGPD

### 3.2 Intégrations Tierces Critiques

**Azure OpenAI** : GPT-4 pour RAG et analyse contextuelle
**ElasticSearch** : Moteur de recherche et indexation
**Redis** : Cache haute performance et gestion sessions
**RabbitMQ** : Files d'attente pour traitement asynchrone
**Docker** : Conteneurisation pour déploiement simplifié

---


## 4. 📊 Scénarios de Test Fonctionnels

**Scénario 1** : STIX → RAG → TTPs → Kestrel (Pipeline Complet)
1. **Alerte Wazuh SSH** → Normalisation STIX 2.1 avec network-traffic objects
2. **RAG Processing** : STIX network patterns → TTPs ["T1110.001"] + reasoning explicite
3. **Kestrel Generation** : "ssh_attempts = NEW network-traffic WHERE [dst_port = 22]"
4. **Hunt automatique** : Exécution requête avec corrélation tri-sources
5. **Résultats enrichis** : Alerte + TTPs + raisonnement + résultats hunt

**Scénario 2** : OTX Pulse → STIX → RAG → Hunt Préventif
1. **Pulse OTX malware** → Conversion STIX 2.1 avec malware + indicator objects
2. **RAG Analysis** : STIX malware patterns → TTPs ["T1055", "T1027"] + confidence scores
3. **Reasoning IA** : "Malware indicators suggest Process Injection + Obfuscation techniques"
4. **Kestrel auto-générée** : Hunt processes suspects + file obfuscation patterns
5. **Hunt préventif** : Recherche traces dans infrastructure avec contexte explicite

**Scénario 3** : Corrélation Tri-Sources → STIX → RAG → Campagne
1. **Multi-sources** : Wazuh emails + OTX domains + OpenVAS web vulns → STIX bundle
2. **RAG Campaign Analysis** : STIX bundle → TTPs campagne ["T1566.001", "T1190", "T1059.001"]
3. **Reasoning détaillé** : "STIX correlation suggests coordinated campaign: Phishing → Web Exploit → PowerShell"
4. **Kestrel campaign hunt** : Requête multi-techniques avec timeline
5. **Rapport campagne** : TTPs + reasoning + timeline + IOCs + recommandations

---

## 6. 🚀 Évolutions Futures (Roadmap)

### Version 2.0
- **Matrice MITRE ATT&CK interactive** : Interface dédiée navigation techniques
- **Module Kestrel Hunting standalone** : Interface dédiée avec éditeur
- **Module Analyse Multi-Format** : Emails, documents PDF, images avec IOCs
- Support sources CTI étendues (MISP, OTX)
- Heat maps et visualisations avancées ATT&CK