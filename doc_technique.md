# Documentation Technique  Cyber-CTI

1.  ## Architecture Technique Détaillée
- ## Composants principaux

    -   **Scraper** : module Python (cron) qui extrait chaque jour les
        vulnérabilités du site **TUN-CERT**.

    -   **Agents AutoGen** orchestrés :

        -   **StixTransformerAgent** → transforme en objets STIX 2.1.

        -   **MapperTTPAgent** → associe vulnérabilités aux techniques
            MITRE ATT&CK.

        -   **HypotheseAgent** → génère des hypothèses d'attaque.

        -   **PlayBookAgent** → propose des actions concrètes
            (playbooks).

    -   **Base de données PostgreSQL** : stockage des vulnérabilités,
        STIX, TTPs, hypothèses, playbooks.

    -   ## API REST (FastAPI) : expose /cti_mappings/ et autres
        endpoints.

    -   ## Dashboard (Bootstrap/JS) : interface utilisateur consommant
        l'API.
- **Interactions & orchestration**

    -   **L'exécution se déroule en deux temps :**

        1.  **Scraping (cron système)** : collecte des vulnérabilités et
            insertion en base.

        2.  **Orchestration AutoGen (cron dédié)** : deux heures après
            le scraping, AutoGen orchestre les agents pour enrichir les
            données.

    -   AutoGen assure la coordination des agents :

-   L'agent STIX reçoit la vulnérabilité, génère l'objet STIX.

-   Le MapperTTP interroge Azure AI (pipeline RAG) pour identifier les
    TTPs pertinents.

-   L'HypotheseAgent et le PlayBookAgent, tous deux basés sur GPT-4.1,
    enrichissent le résultat.

    -   Les résultats sont persistés en base puis exposés via API.

    -   Le dashboard consomme ces données pour affichage dynamique.

2.  ## Cron & Automatisation

    a.  **Cron scraping (exemple 6h)**

<img width="806" height="98" alt="image" src="https://github.com/user-attachments/assets/32b0a815-2b72-4850-be3a-57efd1bfbd1c" />


-   Étapes déclenchées :

    1.  Scraping de la liste TUN-CERT.

    2.  Insertion en base.

    a.  **Cron AutoGen orchestration (exemple 8h)**

<img width="813" height="98" alt="image" src="https://github.com/user-attachments/assets/afef3d5b-66de-4bc3-a57d-ea3b83ff685b" />


-   Étapes orchestrées par AutoGen :\
- *StixTransformerAgent → MapperTTPAgent → HypotheseAgent →
    PlayBookAgent**.

-   Ce délai de 2h permet de s'assurer que toutes les vulnérabilités
    collectées ont été correctement insérées avant enrichissement.

3.  ## Pipeline RAG Azure AI

    a.  **Objectif**

> Relier une vulnérabilité décrite en langage naturel aux **TTPs MITRE
> ATT&CK** pertinents.

b.  **Modèles utilisés**

-   **GPT-4.1** : raisonnement, sélection des TTPs, génération
    d'hypothèses et playbooks.

-   **text-embedding-3-small** : vectorisation des descriptions de
    vulnérabilités.

    a.  **Base d'entraînement**
    
-   Données **MITRE ATT&CK** (techniques, sous-techniques, descriptions,
    mitigations, détections).

-   Format STIX importé depuis le repo officiel GitHub de MITRE ATT&CK.

    a.  **Indexation & recherche**

-   **Azure Cognitive Search (vector search)** stocke les embeddings
    MITRE.

-   Flux :

    1.  Description vulnérabilité → embedding.

    2.  Azure Search → recherche top-k techniques.

    3.  GPT-4.1 → raisonnement pour sélectionner la meilleure
        correspondance + justification.

4.  ## API (FastAPI)

    a.  **Principaux endpoints**

-   GET /cti_mappings/

    -   **Description** : retourne toutes les vulnérabilités enrichies.

    -   **Réponse** (extrait simplifié) :

      ```json
      {

          "mapping_id": 9,

          "vuln_id": "tunCERT/Vuln.2025-411",

          "stix": { ... },

          "ttp": [{ "technique_id": "T1036", "technique_name":"Masquerading" }],

          "hypotheses": { "playbook": [...], "hypotheses":[...] },

          "agents_status": { "HypotheseAgent": "Done","MapperTTpAgent": "Done", "stixTransformer": "Done" }

       }
      ```

-   GET /cti_mappings/{id}

    -   Détail d'une vulnérabilité enrichie.

-   GET /stats

    -   Statistiques agrégées (par date, classification, CVSS).

    a.  **Formats**

-   **Entrée** : JSON (optionnel pour filtres).

-   **Sortie** : JSON enrichi conforme au modèle ci-dessus.

5.  ## Base de Données
- *5.1 Modèle simplifié**

-   **vulnerability**

    -   id, vuln_id, title, publication_date, classification.

-   **vulnerability_detail**

    -   id, vuln_id, description, CVEs, CVSS, documentation, impacts.

-   **cti_mapping**

    -   mapping_id, vuln_id, stix (JSONB), ttp\[\], hypotheses\[\],
        playbook\[\].
- *5.2 Gestion des mises à jour**

-   Déduplication sur vuln_id.

-   Si une vulnérabilité existe déjà → mise à jour des champs enrichis +
    horodatage.

-   **⚠️ TODO** : vérifier les cas où **plusieurs vulnérabilités
    partagent la même référence (vuln_id)**.

    -   Exemple : différentes versions ou déclinaisons peuvent utiliser
        le même identifiant de référence.

    -   Une logique d'**unicité combinée** (vuln_id + date_version ou
        vuln_id + reference_number) devra être confirmée et testée.

6.  ## Dashboard
- *6.1 Technologie**

-   HTML5, Bootstrap 5, Vanilla JS.

-   Consommation directe de l'API via fetch().
- *6.2 Fonctionnalités**

-   Recherche en texte libre.

-   Pagination par 10.

-   Tri par date de publication.

-   Visualisation détaillée avec :

    -   Métadonnées (CVSS, impacts, plateformes).

    -   **TTPs mappés**.

    -   **Hypothèses**.

    -   **Playbooks**.

    -   **STIX brut**.

7.  ## Sécurité & Conformité

-   ## API : authentification par clé API ou JWT.

-   **Logs d'accès** : audit complet des requêtes.

-   **RGPD & réglementation tunisienne** :

    -   Pas de données personnelles collectées.

    -   Stockage limité à la durée utile.

8.  ## Performances & Indicateurs

-   **Mise à jour** : scraping quotidien (6h), enrichissement AutoGen
    (8h).

-   **Précision TTPs** : mesurée via taux de confiance (confidence
    score).

-   **Couverture** : % vulnérabilités enrichies avec TTPs.

-   **Faux positifs** : suivi par analystes CTI.

9.  ## Limites & Évolutions Futures

-   **Limites actuelles** :

    -   Source unique (TUN-CERT).

    -   Dépendance à Azure AI.

-   **Évolutions prévues** :

    -   Ajout d'autres sources (NVD, exploit-db).

    -   Intégration SOC (SIEM/SOAR).

    -   Partage TAXII/MISP.

    -   Graphes relationnels vulnérabilité ↔ TTP ↔ campagne.

10. # Annexes
- *10.1 Diagrammes**

-   Schéma d'architecture (cf. flow.png).

-   Structure API (cf. API-STRUCTURE.png).

-   Interface utilisateur (cf. interface-final.png).
- *10.2 Glossaire technique**

-   **STIX 2.1** : standard de données CTI.

-   **AutoGen** : framework d'orchestration multi-agents.

-   **RAG** : Retrieval Augmented Generation.

-   **TTP** : Tactiques, Techniques, Procédures (MITRE).

-   **Playbook** : suite d'actions opérationnelles.

-   **Hypothèse** : scénario potentiel d'attaque.

-   **Cron** : planificateur de tâches Linux.

-   **Azure AI Search** : moteur vectoriel pour indexation/recherche.

-   **GPT-4.1** : modèle de raisonnement.

-   **text-embedding-3-small** : modèle d'embedding.

-   **FastAPI** : framework Python pour APIs.

-   **PostgreSQL** : SGBD relationnel avec support JSONB.
