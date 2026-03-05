# Architecture du Projet - Intégration LLM avec n8n

Ce document présente l'architecture complète du système et les différents composants utilisés dans ce cours.

> 💡 **Note** : Les diagrammes sont au format Mermaid. Ils s'affichent automatiquement sur GitHub, GitLab, et dans la plupart des éditeurs Markdown modernes (VS Code, Obsidian, etc.).

---

## 🏗️ Vue d'ensemble de l'architecture

### Schéma global des services Docker

```mermaid
graph TB
    subgraph "RÉSEAU DOCKER: custom_network"
        subgraph "Services"
            N8N[("n8n<br/>Port: 5678<br/>Workflows & Automatisation")]
            POSTGRES[("PostgreSQL<br/>Port: 5432<br/>Base de données<br/>pgvector")]
            PGADMIN[("pgAdmin<br/>Port: 80<br/>Interface de gestion DB")]
            OLLAMA[("Ollama<br/>Port: 11434<br/>LLM locaux<br/>Mistral, Llama Guard")]
        end
    end
    
    subgraph "MACHINE HÔTE"
        HOST["localhost<br/>Accès aux services"]
    end
    
    N8N -->|Stocke workflows| POSTGRES
    N8N -->|Appels API| OLLAMA
    PGADMIN -->|Gestion| POSTGRES
    HOST -->|"http://localhost:5678"| N8N
    HOST -->|"localhost:5434"| POSTGRES
    HOST -->|"http://localhost:5050"| PGADMIN
    HOST -->|"http://localhost:11434"| OLLAMA
    
    style N8N fill:#2E86DE,color:#fff
    style POSTGRES fill:#336791,color:#fff
    style PGADMIN fill:#336791,color:#fff
    style OLLAMA fill:#000,color:#fff
    style HOST fill:#27AE60,color:#fff
```

### Mapping des ports

| Service    | Port Interne | Port Externe | URL d'accès             | Description                    |
|------------|--------------|--------------|-------------------------|--------------------------------|
| n8n        | 5678         | 5678         | http://localhost:5678   | Interface web n8n              |
| PostgreSQL | 5432         | 5434         | localhost:5434          | Connexion DB (port non-standard)|
| pgAdmin    | 80           | 5050         | http://localhost:5050   | Interface de gestion DB        |
| Ollama     | 11434        | 11434        | http://localhost:11434  | API LLM locaux                 |

> ⚠️ **Note** : PostgreSQL est mappé sur le port 5434 (au lieu de 5432) pour éviter les conflits avec une installation locale de PostgreSQL.

---

## 🔄 Architecture des Workflows - Progression du Projet

### Étape 0-1 : Chat Simple et Diversity

```mermaid
graph LR
    subgraph "Workflow n8n - Chat Simple"
        INPUT["Chat Input"]
        LLM["LLM Model<br/>(Ollama Mistral)"]
        OUTPUT["Response Output"]
        
        INPUT --> LLM --> OUTPUT
    end
    
    style INPUT fill:#3498DB,color:#fff
    style LLM fill:#E74C3C,color:#fff
    style OUTPUT fill:#2ECC71,color:#fff
```

**Étape 1 - Diversity : Tester plusieurs LLM**

```mermaid
graph TB
    INPUT["Chat Input"]
    
    subgraph "LLM Models"
        OLLAMA["Ollama<br/>Mistral (local)"]
        GEMINI["Google Gemini<br/>(API)"]
        OPENAI["OpenAI GPT<br/>(API)"]
        MISTRAL["Mistral AI<br/>(API)"]
    end
    
    OUTPUT1["Response Ollama"]
    OUTPUT2["Response Gemini"]
    OUTPUT3["Response OpenAI"]
    OUTPUT4["Response Mistral"]
    
    INPUT --> OLLAMA --> OUTPUT1
    INPUT --> GEMINI --> OUTPUT2
    INPUT --> OPENAI --> OUTPUT3
    INPUT --> MISTRAL --> OUTPUT4
    
    style INPUT fill:#3498DB,color:#fff
    style OLLAMA fill:#000,color:#fff
    style GEMINI fill:#4285F4,color:#fff
    style OPENAI fill:#10A37F,color:#fff
    style MISTRAL fill:#FF7000,color:#fff
```

---

### Étape 2-3 : Split & Distribute Workflows

```mermaid
graph TB
    subgraph "WORKFLOW CLIENT (Main)"
        CHAT_INPUT["Chat Input"]
        DISTRIBUTE["Distribute Logic"]
        MERGE["Merge Results"]
        DISPLAY["Display to User"]
        
        CHAT_INPUT --> DISTRIBUTE
        MERGE --> DISPLAY
    end
    
    subgraph "SUBWORKFLOWS (Étape 2) / WEBHOOKS (Étape 3)"
        subgraph "Ollama Workflow"
            WH1["Webhook POST<br/>ou Execute Workflow"]
            LLM1["Ollama LLM"]
            RESP1["Response"]
            WH1 --> LLM1 --> RESP1
        end
        
        subgraph "Gemini Workflow"
            WH2["Webhook POST<br/>ou Execute Workflow"]
            LLM2["Gemini LLM"]
            RESP2["Response"]
            WH2 --> LLM2 --> RESP2
        end
        
        subgraph "OpenAI Workflow"
            WH3["Webhook POST<br/>ou Execute Workflow"]
            LLM3["OpenAI LLM"]
            RESP3["Response"]
            WH3 --> LLM3 --> RESP3
        end
        
        subgraph "Mistral Workflow"
            WH4["Webhook POST<br/>ou Execute Workflow"]
            LLM4["Mistral LLM"]
            RESP4["Response"]
            WH4 --> LLM4 --> RESP4
        end
    end
    
    DISTRIBUTE --> WH1
    DISTRIBUTE --> WH2
    DISTRIBUTE --> WH3
    DISTRIBUTE --> WH4
    
    RESP1 --> MERGE
    RESP2 --> MERGE
    RESP3 --> MERGE
    RESP4 --> MERGE
    
    style CHAT_INPUT fill:#3498DB,color:#fff
    style DISTRIBUTE fill:#9B59B6,color:#fff
    style MERGE fill:#E67E22,color:#fff
    style DISPLAY fill:#2ECC71,color:#fff
    style WH1 fill:#34495E,color:#fff
    style WH2 fill:#34495E,color:#fff
    style WH3 fill:#34495E,color:#fff
    style WH4 fill:#34495E,color:#fff
```

**Différence Étape 2 vs Étape 3:**
- **Étape 2** : Utilise `Execute Workflow` (subworkflows internes)
- **Étape 3** : Utilise `Webhook POST` (workflows distribués, accessibles via HTTP)

---

### Étape 4 : Forms - Interface Utilisateur

```mermaid
graph TB
    USER["👤 Utilisateur"]
    
    subgraph "Workflow avec Formulaire"
        FORM["Form Trigger<br/>Champ: Question"]
        DISTRIBUTE["Distribute to<br/>4 Webhooks"]
        
        subgraph "LLM Calls"
            WH1["Webhook Ollama"]
            WH2["Webhook Gemini"]
            WH3["Webhook OpenAI"]
            WH4["Webhook Mistral"]
        end
        
        MERGE["Merge Results"]
        DISPLAY["Display Results<br/>to User"]
        
        FORM --> DISTRIBUTE
        DISTRIBUTE --> WH1
        DISTRIBUTE --> WH2
        DISTRIBUTE --> WH3
        DISTRIBUTE --> WH4
        
        WH1 --> MERGE
        WH2 --> MERGE
        WH3 --> MERGE
        WH4 --> MERGE
        
        MERGE --> DISPLAY
    end
    
    USER -->|Accède à l'URL| FORM
    DISPLAY -->|Affiche| USER
    
    style USER fill:#3498DB,color:#fff
    style FORM fill:#9B59B6,color:#fff
    style DISTRIBUTE fill:#E67E22,color:#fff
    style MERGE fill:#E67E22,color:#fff
    style DISPLAY fill:#2ECC71,color:#fff
```

---

### Étape 5-6 : Store & Load from Database

```mermaid
graph TB
    subgraph "Étape 5: Store to DB"
        FORM1["Form Input"]
        INSERT_Q["INSERT INTO question<br/>RETURNING id"]
        
        subgraph "LLM Processing"
            LLM1["LLM 1"]
            LLM2["LLM 2"]
            LLM3["LLM 3"]
            LLM4["LLM 4"]
        end
        
        INSERT_R["INSERT INTO reponse<br/>(reponse, provider, question_id)"]
        
        FORM1 --> INSERT_Q
        INSERT_Q -->|question_id| LLM1
        INSERT_Q -->|question_id| LLM2
        INSERT_Q -->|question_id| LLM3
        INSERT_Q -->|question_id| LLM4
        
        LLM1 --> INSERT_R
        LLM2 --> INSERT_R
        LLM3 --> INSERT_R
        LLM4 --> INSERT_R
    end
    
    subgraph "Base de Données PostgreSQL"
        direction LR
        TABLE_Q["📋 Table: question<br/>• id (PK)<br/>• question (TEXT)<br/>• date (TIMESTAMP)"]
        TABLE_R["📋 Table: reponse<br/>• id (PK)<br/>• reponse (TEXT)<br/>• provider (TEXT)<br/>• question_id (FK)"]
        
        TABLE_Q -->|1:N| TABLE_R
    end
    
    subgraph "Étape 6: Load from DB"
        SELECT["SELECT q.*, r.*<br/>FROM question q<br/>JOIN reponse r<br/>ON q.id = r.question_id"]
        LOOP["Loop over rows<br/>Process data"]
        
        SELECT --> LOOP
    end
    
    INSERT_Q -.->|Écrit| TABLE_Q
    INSERT_R -.->|Écrit| TABLE_R
    TABLE_Q -.->|Lit| SELECT
    TABLE_R -.->|Lit| SELECT
    
    style FORM1 fill:#3498DB,color:#fff
    style INSERT_Q fill:#E74C3C,color:#fff
    style INSERT_R fill:#E74C3C,color:#fff
    style TABLE_Q fill:#336791,color:#fff
    style TABLE_R fill:#336791,color:#fff
    style SELECT fill:#27AE60,color:#fff
    style LOOP fill:#F39C12,color:#fff
```

**Structure de la base de données:**

```mermaid
erDiagram
    QUESTION ||--o{ REPONSE : "a plusieurs"
    
    QUESTION {
        int id PK
        text question
        timestamp date
    }
    
    REPONSE {
        int id PK
        text reponse
        text provider
        int question_id FK
    }
```

---

### Étape 7 : Enhance Prompt (RAG Simplifié)

```mermaid
graph TB
    subgraph "RAG - Retrieval Augmented Generation"
        LOAD["Load from DB<br/>SELECT avec JOIN"]
        LOOP["Loop over<br/>each question"]
        
        BUILD["Build Enhanced Prompt<br/>┌────────────────────┐<br/>│ Question: {q}      │<br/>│ Réponses:          │<br/>│ • OpenAI: {r1}     │<br/>│ • Mistral: {r2}    │<br/>│ • Gemini: {r3}     │<br/>│ • Ollama: {r4}     │<br/>│                    │<br/>│ Analyse et génère  │<br/>│ un rapport MD      │<br/>└────────────────────┘"]
        
        META_LLM["Meta-LLM Analysis<br/>Génère rapport<br/>Markdown"]
        
        SAVE["Save Report<br/>/files/output/<br/>report.md"]
        
        LOAD --> LOOP
        LOOP --> BUILD
        BUILD --> META_LLM
        META_LLM --> SAVE
    end
    
    DB[("PostgreSQL<br/>Questions +<br/>Réponses")]
    FILE[("📄 Fichier<br/>Markdown")]
    
    DB -.->|Fournit données| LOAD
    SAVE -.->|Écrit| FILE
    
    style LOAD fill:#27AE60,color:#fff
    style LOOP fill:#F39C12,color:#fff
    style BUILD fill:#9B59B6,color:#fff
    style META_LLM fill:#E74C3C,color:#fff
    style SAVE fill:#3498DB,color:#fff
    style DB fill:#336791,color:#fff
    style FILE fill:#95A5A6,color:#fff
```

**Concept RAG expliqué:**

```mermaid
graph LR
    subgraph "RAG = Retrieval Augmented Generation"
        R["📚 Retrieval<br/>Récupération<br/>de données"]
        A["🔗 Augmented<br/>Enrichissement<br/>du prompt"]
        G["✨ Generation<br/>Génération<br/>par LLM"]
        
        R --> A --> G
    end
    
    DATA[("Base de<br/>données")]
    PROMPT["Prompt<br/>enrichi"]
    RESULT["Résultat<br/>contextualisé"]
    
    DATA -.->|Fournit contexte| R
    A -.->|Crée| PROMPT
    G -.->|Produit| RESULT
    
    style R fill:#27AE60,color:#fff
    style A fill:#F39C12,color:#fff
    style G fill:#E74C3C,color:#fff
```

---

### Étape 9 : Secure Prompt - Protection contre les injections

```mermaid
graph TB
    USER_INPUT["⌨️ User Input<br/>Question"]
    
    subgraph "SECURITY LAYER (Subworkflow)"
        REGEX["🔍 1. REGEX VALIDATION<br/>• Detect &lt;script&gt; tags<br/>• Detect SQL injection<br/>• Detect command injection"]
        
        GUARD["🛡️ 2. LLAMA-GUARD3<br/>Analyse sémantique<br/>Classification: Safe/Unsafe"]
        
        ENCAPS["📦 3. ENCAPSULATION<br/>┌─────────────────────┐<br/>│ Début de question:  │<br/>│ [user: {input}]     │<br/>│ Fin de question.    │<br/>└─────────────────────┘"]
        
        DECISION{"Sécurisé?"}
    end
    
    REJECT["❌ REJECTED<br/>Prompt dangereux"]
    PROCEED["✅ PROCEED<br/>Vers LLM Workflow"]
    
    USER_INPUT --> REGEX
    REGEX -->|Passe| GUARD
    REGEX -->|Suspect| REJECT
    GUARD -->|Safe| ENCAPS
    GUARD -->|Unsafe| REJECT
    ENCAPS --> DECISION
    DECISION -->|Non| REJECT
    DECISION -->|Oui| PROCEED
    
    style USER_INPUT fill:#3498DB,color:#fff
    style REGEX fill:#E74C3C,color:#fff
    style GUARD fill:#F39C12,color:#fff
    style ENCAPS fill:#9B59B6,color:#fff
    style DECISION fill:#E67E22,color:#fff
    style REJECT fill:#C0392B,color:#fff
    style PROCEED fill:#27AE60,color:#fff
```

**Types d'attaques détectées:**

```mermaid
mindmap
  root((Sécurité<br/>Prompt))
    Regex
      Script injection
        <script>alert</script>
        <img src=x onerror=>
      SQL injection
        ' OR '1'='1
        DROP TABLE users
      Command injection
        ; rm -rf /
        && cat /etc/passwd
    LLM Guard
      Jailbreak attempts
      Prompt manipulation
      Toxic content
      PII leakage
    Encapsulation
      Délimiteurs clairs
      Contexte protégé
      Isolation instructions
```

---

## 📊 Flux de données complet (Toutes étapes)

```mermaid
sequenceDiagram
    actor User as 👤 Utilisateur
    participant Form as 📝 n8n Form
    participant Security as 🛡️ Security Layer
    participant DB as 🗄️ PostgreSQL
    participant LLM1 as 🤖 Ollama
    participant LLM2 as 🤖 Gemini
    participant LLM3 as 🤖 OpenAI
    participant LLM4 as 🤖 Mistral
    participant RAG as 🧠 Meta-LLM
    participant File as 📄 Report.md
    
    User->>Form: 1. Accède au formulaire
    User->>Form: 2. Soumet question
    Form->>Security: 3. Validation
    
    alt Question sécurisée
        Security->>DB: 4. INSERT question
        DB-->>Security: question_id
        
        par Appels parallèles LLM
            Security->>LLM1: Call avec question_id
            Security->>LLM2: Call avec question_id
            Security->>LLM3: Call avec question_id
            Security->>LLM4: Call avec question_id
        end
        
        par Réponses LLM
            LLM1-->>DB: 5. INSERT reponse
            LLM2-->>DB: 5. INSERT reponse
            LLM3-->>DB: 5. INSERT reponse
            LLM4-->>DB: 5. INSERT reponse
        end
        
        DB-->>User: 6. Affichage résultats
        
        Note over DB,RAG: Étape 7: Analyse ultérieure
        
        RAG->>DB: 7. Load questions + réponses
        DB-->>RAG: Données avec JOIN
        RAG->>RAG: 8. Enhanced Prompt
        RAG->>File: 9. Generate Report
        File-->>User: 10. Download rapport
    else Question dangereuse
        Security-->>User: ❌ Rejet (unsafe)
    end
```

---

## 🔐 Sécurité et Architecture Réseau

### Isolation des services Docker

```mermaid
graph TB
    subgraph "DOCKER HOST"
        subgraph "custom_network (Bridge)"
            N8N["n8n<br/>172.18.0.2"]
            POSTGRES["PostgreSQL<br/>172.18.0.3"]
            PGADMIN["pgAdmin<br/>172.18.0.4"]
            OLLAMA["Ollama<br/>172.18.0.5"]
            
            N8N <-->|Communication<br/>interne| POSTGRES
            N8N <-->|API calls| OLLAMA
            PGADMIN <-->|Gestion| POSTGRES
        end
        
        subgraph "Port Binding"
            BIND["127.0.0.1<br/>(localhost only)"]
        end
    end
    
    EXTERNAL["🌐 Réseau<br/>Externe"]
    
    BIND -->|5678| N8N
    BIND -->|5434| POSTGRES
    BIND -->|5050| PGADMIN
    BIND -->|11434| OLLAMA
    
    EXTERNAL -.->|Bloqué par défaut| BIND
    
    style N8N fill:#2E86DE,color:#fff
    style POSTGRES fill:#336791,color:#fff
    style PGADMIN fill:#336791,color:#fff
    style OLLAMA fill:#000,color:#fff
    style BIND fill:#27AE60,color:#fff
    style EXTERNAL fill:#E74C3C,color:#fff
```

### Couches de sécurité

```mermaid
graph TB
    subgraph "Couches de Sécurité"
        L1["🔐 1. Authentication<br/>• n8n Basic Auth<br/>• PostgreSQL credentials<br/>• pgAdmin login"]
        
        L2["🔒 2. Network Isolation<br/>• Docker private network<br/>• Port binding 127.0.0.1<br/>• Firewall"]
        
        L3["🔑 3. Environment Variables<br/>• .env file (not in Git)<br/>• Strong passwords<br/>• Rotation régulière"]
        
        L4["✅ 4. Input Validation<br/>• Regex checks<br/>• LLM-based validation<br/>• Prompt encapsulation"]
        
        L5["🗄️ 5. Database Security<br/>• User permissions<br/>• Connection privée<br/>• Backups réguliers"]
        
        L1 --> L2 --> L3 --> L4 --> L5
    end
    
    USER["👤 User"] --> L1
    L5 --> DATA[("💾 Protected<br/>Data")]
    
    style L1 fill:#E74C3C,color:#fff
    style L2 fill:#E67E22,color:#fff
    style L3 fill:#F39C12,color:#fff
    style L4 fill:#27AE60,color:#fff
    style L5 fill:#3498DB,color:#fff
```

---

## 📦 Volumes Docker - Persistance des données

```mermaid
graph LR
    subgraph "Docker Volumes"
        subgraph "n8n_data"
            N8N_WF["workflows/"]
            N8N_CRED["credentials/"]
            N8N_CONF["config/"]
        end
        
        subgraph "postgres_data"
            PG_BASE["base/<br/>(données tables)"]
            PG_WAL["pg_wal/<br/>(logs)"]
            PG_STAT["pg_stat/<br/>(stats)"]
        end
        
        subgraph "pgadmin_data"
            PGA_SESS["sessions/"]
            PGA_STOR["storage/"]
        end
        
        subgraph "ollama_data"
            OL_MOD["models/<br/>(Mistral, etc.)"]
        end
    end
    
    subgraph "Bind Mount"
        FILES["./n8n_files/<br/>├─ corrections/<br/>├─ input/<br/>└─ output/"]
    end
    
    CONT_N8N["Container n8n"]
    CONT_PG["Container postgres"]
    CONT_PGA["Container pgadmin"]
    CONT_OL["Container ollama"]
    
    CONT_N8N -.-> N8N_WF
    CONT_N8N -.-> FILES
    CONT_PG -.-> PG_BASE
    CONT_PGA -.-> PGA_SESS
    CONT_OL -.-> OL_MOD
    
    style n8n_data fill:#2E86DE,color:#fff
    style postgres_data fill:#336791,color:#fff
    style pgadmin_data fill:#336791,color:#fff
    style ollama_data fill:#000,color:#fff
    style FILES fill:#95A5A6,color:#fff
```

---

## 🚀 Évolution de l'Architecture

### Vue chronologique de la progression

```mermaid
timeline
    title Évolution du Projet par Étapes
    
    section Fondations
        Étape 0 : Chat Simple
                : 1 workflow
                : 1 LLM local
        
        Étape 1 : Diversity
                : 4 workflows
                : 4 LLM testés
    
    section Modularisation
        Étape 2 : Split
                : Subworkflows
                : Architecture modulaire
        
        Étape 3 : Distribute
                : Webhooks
                : Architecture distribuée
    
    section Interface & Données
        Étape 4 : Forms
                : Interface utilisateur
                : Formulaires web
        
        Étape 5 : Store to DB
                : Persistance
                : PostgreSQL
        
        Étape 6 : Load from DB
                : Récupération
                : Analyse
    
    section Avancé
        Étape 7 : RAG
                : Enrichissement prompts
                : Meta-analyse
        
        Étape 9 : Security
                : Protection injections
                : Multi-couches
```

### Comparaison Architecture Simple vs Complète

```mermaid
graph TB
    subgraph "Architecture Initiale (Étape 0)"
        S1["User Input"] --> S2["LLM"] --> S3["Output"]
    end
    
    subgraph "Architecture Complète (Étape 9)"
        C1["User"] --> C2["Form"]
        C2 --> C3["Security"]
        C3 --> C4["Database"]
        C4 --> C5["4x Webhooks"]
        C5 --> C6["4x LLM"]
        C6 --> C7["Store Results"]
        C7 --> C8["RAG Analysis"]
        C8 --> C9["Report"]
        C9 --> C1
    end
    
    style S1 fill:#3498DB,color:#fff
    style S2 fill:#E74C3C,color:#fff
    style S3 fill:#2ECC71,color:#fff
    
    style C1 fill:#3498DB,color:#fff
    style C3 fill:#E74C3C,color:#fff
    style C4 fill:#336791,color:#fff
    style C5 fill:#9B59B6,color:#fff
    style C6 fill:#E67E22,color:#fff
    style C8 fill:#F39C12,color:#fff
    style C9 fill:#2ECC71,color:#fff
```

---

## 🎯 Architecture de Déploiement

### Développement Local (Actuel)

```mermaid
graph TB
    subgraph "Machine Locale"
        subgraph "Docker Compose"
            ALL["Tous les services<br/>sur un seul hôte"]
        end
        
        PROS["✅ Simple<br/>✅ Rapide setup<br/>✅ Pas de config réseau"]
        CONS["❌ Pas scalable<br/>❌ Single point of failure<br/>❌ Ressources limitées"]
    end
    
    ALL --> PROS
    ALL --> CONS
    
    style ALL fill:#3498DB,color:#fff
    style PROS fill:#27AE60,color:#fff
    style CONS fill:#E74C3C,color:#fff
```

### Production (Optionnel - Hors scope)

```mermaid
graph TB
    subgraph "Production Deployment"
        LB["⚖️ Load Balancer"]
        
        subgraph "n8n Cluster"
            N1["n8n Instance 1"]
            N2["n8n Instance 2"]
            N3["n8n Instance 3"]
        end
        
        subgraph "Database Cluster"
            DB_MASTER["PostgreSQL<br/>Master"]
            DB_SLAVE1["PostgreSQL<br/>Replica 1"]
            DB_SLAVE2["PostgreSQL<br/>Replica 2"]
        end
        
        REDIS["Redis<br/>Cache"]
        MONITOR["📊 Monitoring<br/>Grafana + Prometheus"]
    end
    
    USERS["👥 Users"] --> LB
    LB --> N1
    LB --> N2
    LB --> N3
    
    N1 --> REDIS
    N2 --> REDIS
    N3 --> REDIS
    
    N1 --> DB_MASTER
    N2 --> DB_MASTER
    N3 --> DB_MASTER
    
    DB_MASTER --> DB_SLAVE1
    DB_MASTER --> DB_SLAVE2
    
    MONITOR -.->|Surveille| N1
    MONITOR -.->|Surveille| N2
    MONITOR -.->|Surveille| N3
    MONITOR -.->|Surveille| DB_MASTER
    
    style LB fill:#E67E22,color:#fff
    style REDIS fill:#DC382D,color:#fff
    style DB_MASTER fill:#336791,color:#fff
    style DB_SLAVE1 fill:#336791,color:#fff
    style DB_SLAVE2 fill:#336791,color:#fff
    style MONITOR fill:#F39C12,color:#fff
```

---

## 📚 Résumé par Étape

```mermaid
graph LR
    E0["📝 Étape 0<br/>Chat Simple"] --> E1["🔀 Étape 1<br/>Diversity"]
    E1 --> E2["✂️ Étape 2<br/>Split"]
    E2 --> E3["🌐 Étape 3<br/>Distribute"]
    E3 --> E4["📋 Étape 4<br/>Forms"]
    E4 --> E5["💾 Étape 5<br/>Store DB"]
    E5 --> E6["📤 Étape 6<br/>Load DB"]
    E6 --> E7["🧠 Étape 7<br/>RAG"]
    E7 --> E8["📄 Étape 8<br/>Export"]
    E8 --> E9["🔐 Étape 9<br/>Security"]
    E9 --> E10["🎁 Étape 10<br/>Bonus"]
    
    style E0 fill:#3498DB,color:#fff
    style E1 fill:#9B59B6,color:#fff
    style E2 fill:#E67E22,color:#fff
    style E3 fill:#27AE60,color:#fff
    style E4 fill:#F39C12,color:#fff
    style E5 fill:#336791,color:#fff
    style E6 fill:#336791,color:#fff
    style E7 fill:#E74C3C,color:#fff
    style E8 fill:#95A5A6,color:#fff
    style E9 fill:#C0392B,color:#fff
    style E10 fill:#16A085,color:#fff
```

---

Cette architecture évolue progressivement tout au long du cours, de simple (Étape 0) à complexe et sécurisée (Étape 9+).

> 💡 **Astuce** : Référez-vous à ce document à chaque étape pour visualiser où vous en êtes dans l'architecture globale !
