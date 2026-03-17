# 3. Distribuer les workflows avec des webhooks

> **Résumé** : Transformez vos workflows locaux en services REST accessibles depuis d'autres machines via webhooks  
> **Temps estimé** : 75-90 minutes  
> **Difficulté** : Intermédiaire/Avancé ⭐⭐⭐  
> **Étape précédente** : [2. Split Workflow](../2.%20split%20workflow/README.md)

---

## 🎯 Objectifs pédagogiques

À la fin de cette étape, vous serez capable de :

1. **Comprendre la différence** entre "Execute Workflow" (interne) et webhooks (distribués)
2. **Configurer des webhooks POST** dans n8n pour exposer vos workflows LLM comme APIs REST
3. **Remplacer les nœuds "Execute Workflow Trigger"** par des nœuds "Webhook"
4. **Créer un workflow client** utilisant des requêtes HTTP pour interroger les webhooks
5. **Exposer n8n sur le réseau local** en modifiant la configuration Docker
6. **Tester depuis une autre machine** l'accès aux workflows distribués
7. **Sécuriser les webhooks** avec des credentials (Basic Auth ou API Key)

Cette étape est essentielle pour construire une **architecture distribuée** et préparer le déploiement d'un comparateur de LLM multi-serveurs.

---

## 📚 Prérequis

### Étape précédente requise

- ✅ **Étape 2 complétée** : Vous devez avoir un workflow client et 4 sous-workflows LLM fonctionnels
  - 📖 Voir : [2. Split Workflow](../2.%20split%20workflow/README.md)

### Connaissances requises

- 🌐 **Protocoles HTTP et APIs REST**
  - 📖 Voir : [/ressources/http&API/README.md](../../ressources/http&API/README.md)
  
- 🔐 **Gestion des credentials et authentification**
  - 📖 Voir : [/ressources/credentials/README.md](../../ressources/credentials/README.md)

- 🐳 **Docker et configuration réseau**
  - 📖 Voir : [/ressources/docker/README.md](../../ressources/docker/README.md)

- 🔌 **Bases du réseau (IP, ports, localhost vs 0.0.0.0)**
  - 📖 Voir : [/ressources/bases_reseau/README.md](../../ressources/bases_reseau/README.md)

### Outils requis

- **Docker et docker-compose** : pour modifier l'exposition réseau de n8n
- **Accès à une deuxième machine** (ou VM) : pour tester l'accès distant (optionnel mais recommandé)
- **Postman ou cURL** : pour tester les webhooks manuellement

### Ressources externes

- [n8n Documentation - Webhook Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)
- [n8n Documentation - Webhook Development](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/workflow-development/)
- [n8n Documentation - Webhook Common Issues](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/common-issues/)

---

## 📊 Architecture visuelle

### Différence clé : Execute Workflow vs Webhook

```mermaid
graph TB
    subgraph "Étape 2: Execute Workflow (Interne)"
        E2_MAIN["Workflow Principal"]
        E2_SUB1["Sub-workflow 1"]
        E2_SUB2["Sub-workflow 2"]
        
        E2_MAIN -->|Execute Workflow<br/>Appel interne| E2_SUB1
        E2_MAIN -->|Execute Workflow<br/>Appel interne| E2_SUB2
    end
    
    subgraph "Étape 3: Webhook (Distribué)"
        E3_MAIN["Workflow Principal"]
        
        E3_MAIN -->|HTTP POST<br/>Réseau| E3_WH1
        E3_MAIN -->|HTTP POST<br/>Réseau| E3_WH2
        
        subgraph "Serveur 1 (peut être distant)"
            E3_WH1["Webhook 1"]
            E3_SUB1["LLM Workflow 1"]
            E3_WH1 --> E3_SUB1
        end
        
        subgraph "Serveur 2 (peut être distant)"
            E3_WH2["Webhook 2"]
            E3_SUB2["LLM Workflow 2"]
            E3_WH2 --> E3_SUB2
        end
    end
    
    style E2_MAIN fill:#9B59B6,color:#fff
    style E2_SUB1 fill:#3498DB,color:#fff
    style E2_SUB2 fill:#3498DB,color:#fff
    style E3_MAIN fill:#E67E22,color:#fff
    style E3_WH1 fill:#27AE60,color:#fff
    style E3_WH2 fill:#27AE60,color:#fff
```

**Avantages de l'architecture avec webhooks** :
- ✅ Les workflows LLM peuvent être sur des machines différentes
- ✅ Chaque workflow est accessible comme une API REST indépendante
- ✅ Scalabilité : facile d'ajouter de nouveaux serveurs
- ✅ Interopérabilité : n'importe quel client HTTP peut interroger les workflows (pas seulement n8n)

---

### Architecture distribuée complète

```mermaid
graph TB
    USER["👤 Utilisateur"]
    
    subgraph "Machine Principale - n8n"
        MAIN["Workflow Client"]
        HTTP1["HTTP Request<br/>POST /webhook/ollama"]
        HTTP2["HTTP Request<br/>POST /webhook/gemini"]
        HTTP3["HTTP Request<br/>POST /webhook/openai"]
        HTTP4["HTTP Request<br/>POST /webhook/mistral"]
        MERGE["Merge Results"]
        
        MAIN --> HTTP1
        MAIN --> HTTP2
        MAIN --> HTTP3
        MAIN --> HTTP4
        
        HTTP1 --> MERGE
        HTTP2 --> MERGE
        HTTP3 --> MERGE
        HTTP4 --> MERGE
    end
    
    subgraph "Machine(s) Distante(s) - Workflows LLM"
        subgraph "Workflow Ollama"
            WH1["🔗 Webhook<br/>POST /webhook/ollama"]
            LLM1["Ollama LLM"]
            RESP1["Response"]
            WH1 --> LLM1 --> RESP1
        end
        
        subgraph "Workflow Gemini"
            WH2["🔗 Webhook<br/>POST /webhook/gemini"]
            LLM2["Gemini LLM"]
            RESP2["Response"]
            WH2 --> LLM2 --> RESP2
        end
        
        subgraph "Workflow OpenAI"
            WH3["🔗 Webhook<br/>POST /webhook/openai"]
            LLM3["OpenAI LLM"]
            RESP3["Response"]
            WH3 --> LLM3 --> RESP3
        end
        
        subgraph "Workflow Mistral"
            WH4["🔗 Webhook<br/>POST /webhook/mistral"]
            LLM4["Mistral LLM"]
            RESP4["Response"]
            WH4 --> LLM4 --> RESP4
        end
    end
    
    USER --> MAIN
    MERGE --> USER
    
    HTTP1 -.->|http://server:5678/webhook/ollama| WH1
    HTTP2 -.->|http://server:5678/webhook/gemini| WH2
    HTTP3 -.->|http://server:5678/webhook/openai| WH3
    HTTP4 -.->|http://server:5678/webhook/mistral| WH4
    
    RESP1 -.->|JSON Response| HTTP1
    RESP2 -.->|JSON Response| HTTP2
    RESP3 -.->|JSON Response| HTTP3
    RESP4 -.->|JSON Response| HTTP4
    
    style USER fill:#3498DB,color:#fff
    style MAIN fill:#E67E22,color:#fff
    style MERGE fill:#F39C12,color:#fff
    style WH1 fill:#27AE60,color:#fff
    style WH2 fill:#27AE60,color:#fff
    style WH3 fill:#27AE60,color:#fff
    style WH4 fill:#27AE60,color:#fff
```

---

### Configuration réseau : localhost vs réseau

```mermaid
graph LR
    subgraph "docker-compose.yml"
        CONFIG["ports:<br/>- '127.0.0.1:5678:5678'<br/><br/>⬇️ Changement<br/><br/>ports:<br/>- '0.0.0.0:5678:5678'"]
    end
    
    LOCAL["🏠 Localhost<br/>127.0.0.1<br/>Accès local uniquement"]
    NETWORK["🌐 Réseau<br/>0.0.0.0<br/>Accessible depuis<br/>d'autres machines"]
    
    CONFIG -->|Avant| LOCAL
    CONFIG -->|Après| NETWORK
    
    style CONFIG fill:#F39C12,color:#fff
    style LOCAL fill:#E74C3C,color:#fff
    style NETWORK fill:#27AE60,color:#fff
```

**Explication** :
- `127.0.0.1` : Écoute uniquement sur localhost (seule la machine hôte peut accéder)
- `0.0.0.0` : Écoute sur toutes les interfaces réseau (accessible depuis d'autres machines du réseau local)

---

## 🧑‍💻 Instructions étape par étape

### Partie 1 : Dupliquer les workflows et préparer l'architecture

#### Étape 1 : Créer un nouveau dossier

1. **Dans n8n, créez un dossier**
   - Menu latéral → Workflows → clic droit sur "projet"
   - **"New Folder"** → Nommez-le `3. distribute workflow`

2. **Planifiez la structure**
   Vous allez créer **5 nouveaux workflows** :
   - **4 workflows LLM avec webhooks** : `webhook_ollama`, `webhook_gemini`, `webhook_mistral`, `webhook_openai`
   - **1 workflow client HTTP** : `http_client`

---

#### Étape 2 : Dupliquer les workflows LLM de l'Étape 2

1. **Dupliquez chaque sous-workflow de l'Étape 2**
   - Ouvrez `basic_ollama` → Menu "..." → "Duplicate"
   - Renommez : `webhook_ollama`
   - Déplacez dans le dossier `3. distribute workflow`

2. **Répétez pour les autres modèles**
   - `basic_gemini` → `webhook_gemini`
   - `basic_mistral` → `webhook_mistral`
   - `basic_openai` → `webhook_openai`

---

### Partie 2 : Remplacer "Execute Workflow Trigger" par "Webhook"

#### Étape 3 : Configurer le premier webhook (Ollama)

1. **Ouvrez le workflow `webhook_ollama`**

2. **Supprimez le nœud "Execute Workflow Trigger"**
   - Sélectionnez-le et appuyez sur Delete

3. **Ajoutez un nœud "Webhook"**
   - Cliquez sur **"+"** → Recherchez `Webhook`
   - Ajoutez-le au début du workflow

4. **Configurez le webhook**
   - Double-cliquez sur le nœud "Webhook"
   - **HTTP Method** : `POST`
   - **Path** : `ollama` (ou `webhook/ollama` pour plus de clarté)
   - **Authentication** : `None` (pour l'instant, on sécurisera plus tard)
   - **Response Mode** : `When Last Node Finishes`
   - **Response Code** : `200`

5. **Connectez Webhook → Ollama**
   - Tracez une connexion du nœud Webhook vers le nœud Ollama

6. **Testez le webhook**
   - Cliquez sur "Listen for Test Event" dans le nœud Webhook
   - n8n affiche l'URL de test : `http://localhost:5678/webhook-test/ollama`
   - Copiez cette URL

7. **Testez avec cURL ou Postman**
   ```bash
   curl -X POST http://localhost:5678/webhook-test/ollama \
     -H "Content-Type: application/json" \
     -d '{"chatInput": "Bonjour, qui es-tu ?"}'
   ```
   - Vous devriez recevoir une réponse JSON du modèle Ollama

8. **Activez le workflow**
   - Toggle "Inactive" → "Active"
   - Une fois activé, l'URL de production sera : `http://localhost:5678/webhook/ollama`

9. **Sauvegardez le workflow**

---

#### Étape 4 : Configurer les autres webhooks

Répétez le processus pour chaque modèle :

1. **Pour Gemini** :
   - Ouvrez `webhook_gemini`
   - Supprimez "Execute Workflow Trigger"
   - Ajoutez "Webhook" avec path = `gemini`
   - Connectez : Webhook → Google Gemini Chat Model
   - Testez, activez, sauvegardez

2. **Pour Mistral** :
   - Ouvrez `webhook_mistral`
   - Supprimez "Execute Workflow Trigger"
   - Ajoutez "Webhook" avec path = `mistral`
   - Connectez : Webhook → Mistral Chat Model
   - Testez, activez, sauvegardez

3. **Pour OpenAI** :
   - Ouvrez `webhook_openai`
   - Supprimez "Execute Workflow Trigger"
   - Ajoutez "Webhook" avec path = `openai`
   - Connectez : Webhook → OpenAI Chat Model
   - Testez, activez, sauvegardez

**Vérification** : À ce stade, vous devez avoir **4 workflows actifs** avec des webhooks accessibles :
- `http://localhost:5678/webhook/ollama`
- `http://localhost:5678/webhook/gemini`
- `http://localhost:5678/webhook/mistral`
- `http://localhost:5678/webhook/openai`

---

### Partie 3 : Créer le workflow client HTTP

#### Étape 5 : Construire le workflow client

1. **Créez un nouveau workflow**
   - Cliquez sur "Add workflow"
   - Renommez : `http_client`

2. **Ajoutez un nœud Chat Trigger**
   - C'est le point d'entrée pour l'utilisateur

---

#### Étape 6 : Ajouter les requêtes HTTP

1. **Ajoutez un nœud "HTTP Request"**
   - Cliquez sur **"+"** après le Chat Trigger
   - Recherchez : `HTTP Request`

2. **Configurez la requête pour Ollama**
   - Double-cliquez sur le nœud
   - **Method** : `POST`
   - **URL** : `http://localhost:5678/webhook/ollama`
   - **Authentication** : `None` (pour l'instant)
   - **Send Body** : activez et sélectionnez `JSON`
   - **JSON Body** :
     ```json
     {
       "chatInput": "={{ $json.chatInput }}"
     }
     ```
   - Renommez le nœud : `Call Ollama Webhook`

3. **Répétez pour les autres modèles**
   - Ajoutez 3 autres nœuds "HTTP Request"
   - Connectez chacun au Chat Trigger
   - Configurez :
     - `Call Gemini Webhook` → URL : `http://localhost:5678/webhook/gemini`
     - `Call Mistral Webhook` → URL : `http://localhost:5678/webhook/mistral`
     - `Call OpenAI Webhook` → URL : `http://localhost:5678/webhook/openai`

---

#### Étape 7 : Fusionner et formater les résultats

1. **Ajoutez un nœud "Merge"**
   - Après les 4 nœuds HTTP Request
   - Configurez : **Mode** = `Multiplex`
   - Connectez les 4 nœuds HTTP Request au Merge

2. **(Optionnel) Ajoutez un nœud "Code" pour formater**
   - Comme dans l'Étape 2, vous pouvez ajouter un formatage des réponses

3. **Sauvegardez et activez le workflow**

---

#### Étape 8 : Tester le workflow client

1. **Ouvrez le chat**
   - Cliquez sur "Open chat" du nœud Chat Trigger

2. **Envoyez un message**
   ```
   Bonjour, peux-tu te présenter en une phrase ?
   ```

3. **Vérifiez que vous recevez les 4 réponses**
   - Cette fois-ci, les réponses proviennent de **requêtes HTTP** (pas d'appels internes)

---

### Partie 4 : Exposer n8n sur le réseau local

#### Étape 9 : Modifier la configuration Docker

⚠️ **Attention** : Cette étape expose n8n sur le réseau local. Assurez-vous d'être sur un réseau de confiance (pas un réseau public).

1. **Localisez votre fichier `docker-compose.yml`**
   - Il se trouve dans le dossier d'installation de n8n (ex: `/installation/`)

2. **Ouvrez le fichier avec un éditeur**
   ```bash
   nano docker-compose.yml
   # Ou
   code docker-compose.yml
   ```

3. **Modifiez la section `ports`**
   
   **Avant** :
   ```yaml
   services:
     n8n:
       ports:
         - "127.0.0.1:5678:5678"
   ```

   **Après** :
   ```yaml
   services:
     n8n:
       ports:
         - "0.0.0.0:5678:5678"
   ```

4. **Ajoutez la variable d'environnement WEBHOOK_URL**
   ```yaml
   services:
     n8n:
       ports:
         - "0.0.0.0:5678:5678"
       environment:
         - WEBHOOK_URL=http://0.0.0.0:5678
   ```

5. **Sauvegardez le fichier** (Ctrl+O, puis Entrée, puis Ctrl+X sous nano)

6. **Redémarrez n8n**
   ```bash
   docker compose down
   docker compose up -d
   ```

7. **Vérifiez que n8n est accessible**
   - Sur la machine locale : `http://localhost:5678`
   - Sur le réseau local : `http://<VOTRE_IP>:5678`

---

#### Étape 10 : Trouver votre adresse IP locale

1. **Sous Linux** :
   ```bash
   ip a
   # Cherchez l'adresse sous "eth0" ou "wlan0" (ex: 192.168.1.100)
   ```

2. **Sous Windows** :
   ```cmd
   ipconfig
   # Cherchez "Adresse IPv4" (ex: 192.168.1.100)
   ```

3. **Sous Mac** :
   ```bash
   ifconfig
   # Cherchez l'adresse sous "en0" ou "en1"
   ```

4. **Notez votre adresse IP locale** (ex: `192.168.1.100`)

---

### Partie 5 : Tester depuis une autre machine

#### Étape 11 : Tester l'accès distant

1. **Depuis une autre machine sur le même réseau local**

2. **Ouvrez un navigateur et accédez à n8n**
   ```
   http://192.168.1.100:5678
   ```
   *(Remplacez par votre IP)*

3. **Vérifiez que l'interface n8n s'affiche correctement**

4. **Testez un webhook avec cURL depuis la machine distante**
   ```bash
   curl -X POST http://192.168.1.100:5678/webhook/ollama \
     -H "Content-Type: application/json" \
     -d '{"chatInput": "Bonjour depuis une autre machine !"}'
   ```

5. **Vérifiez que vous recevez une réponse JSON**

---

### Partie 6 : Sécuriser les webhooks

⚠️ **Important** : Les webhooks non sécurisés exposent vos workflows à n'importe qui sur le réseau. Ajoutez toujours une authentification.

#### Étape 12 : Ajouter une authentification Basic Auth

1. **Créez des credentials HTTP Basic Auth**
   - Menu latéral → Credentials → "Add Credential"
   - Sélectionnez "HTTP Basic Auth"
   - Remplissez :
     - **Name** : `Webhook Auth - Ollama`
     - **User** : `admin` (ou un nom d'utilisateur de votre choix)
     - **Password** : `votre_mot_de_passe_fort`
   - Sauvegardez

2. **Ajoutez l'authentification au webhook Ollama**
   - Ouvrez le workflow `webhook_ollama`
   - Double-cliquez sur le nœud Webhook
   - **Authentication** : sélectionnez `Basic Auth`
   - **Credential** : sélectionnez `Webhook Auth - Ollama`
   - Sauvegardez

3. **Testez avec authentification**
   ```bash
   curl -X POST http://192.168.1.100:5678/webhook/ollama \
     -u admin:votre_mot_de_passe_fort \
     -H "Content-Type: application/json" \
     -d '{"chatInput": "Test avec auth"}'
   ```

4. **Répétez pour les autres webhooks**
   - Créez des credentials pour chaque webhook (ou réutilisez le même)
   - Configurez l'authentification dans chaque nœud Webhook

5. **Mettez à jour le workflow client HTTP**
   - Ouvrez `http_client`
   - Pour chaque nœud "HTTP Request", ajoutez l'authentification :
     - Double-cliquez sur le nœud
     - **Authentication** : `Basic Auth`
     - **Credential** : sélectionnez le credential approprié
   - Sauvegardez

⚠️ **Limitation** : Votre instance n8n n'intègre **pas HTTPS**. Les credentials sont donc transmis **en clair** sur le réseau. Pour un environnement de production, configurez un reverse proxy avec SSL/TLS.

**Référence** : [/ressources/credentials/README.md - Section Sécurité](../../ressources/credentials/README.md)

---

### Partie 7 : Sauvegarde et export

#### Étape 13 : Exporter tous les workflows

1. **Exportez chaque workflow en JSON**
   - `webhook_ollama`, `webhook_gemini`, `webhook_mistral`, `webhook_openai`, `http_client`
   - Menu "..." → "Download"

2. **Organisez les fichiers**
   - Déplacez tous les JSON dans `projet/3. distribute workflow/`

---

## ✅ Critères de validation

### Checklist de réussite

Cochez chaque élément une fois terminé :

- [ ] J'ai créé 4 workflows avec des webhooks (ollama, gemini, mistral, openai)
- [ ] Chaque webhook est configuré en méthode POST
- [ ] Les 4 workflows avec webhooks sont **activés**
- [ ] J'ai créé le workflow client HTTP `http_client`
- [ ] Le workflow client utilise des nœuds "HTTP Request" (pas "Execute Workflow")
- [ ] J'ai testé chaque webhook individuellement avec cURL ou Postman
- [ ] J'ai modifié le `docker-compose.yml` pour exposer n8n sur `0.0.0.0`
- [ ] J'ai redémarré n8n avec `docker compose up -d`
- [ ] J'ai testé l'accès à n8n depuis une autre machine (ou VM)
- [ ] J'ai sécurisé les webhooks avec Basic Auth
- [ ] Le workflow client HTTP fonctionne avec l'authentification
- [ ] Tous les workflows sont exportés en JSON dans `projet/3. distribute workflow/`

### Structure de dossier attendue

```
projet/
└── 3. distribute workflow/
    ├── README.md (ce fichier)
    ├── http_client.json
    ├── webhook_ollama.json
    ├── webhook_gemini.json
    ├── webhook_mistral.json
    └── webhook_openai.json
```

---

### Quiz de compréhension

<details>
<summary><strong>Question 1 :</strong> Quelle est la principale différence entre "Execute Workflow" et un webhook ?</summary>

**Réponse :**

| Aspect | Execute Workflow | Webhook |
|--------|------------------|---------|
| **Type de communication** | Interne (même instance n8n) | Réseau (HTTP) |
| **Accessibilité** | Seule l'instance n8n locale | Toute machine avec accès réseau |
| **Protocole** | Appel de fonction interne | HTTP POST/GET |
| **Sécurité** | Pas de concern (interne) | Nécessite authentification |
| **Latence** | Très faible (~ms) | Plus élevée (réseau, ~100-500ms) |
| **Scalabilité** | Limitée à une machine | Distribuée sur plusieurs machines |
| **Interopérabilité** | Seulement n8n | N'importe quel client HTTP |

**Cas d'usage** :
- **Execute Workflow** : Tout est sur la même machine, pas de distribution
- **Webhook** : Workflows sur différentes machines, ou accès depuis d'autres applications (Python, JavaScript, etc.)
</details>

<details>
<summary><strong>Question 2 :</strong> Pourquoi changer `127.0.0.1` en `0.0.0.0` dans docker-compose.yml ?</summary>

**Réponse :**

**`127.0.0.1` (localhost)** :
- N'écoute que sur l'interface de loopback
- Seule la machine hôte peut accéder au service
- Sécurisé par défaut (pas d'accès externe)

**`0.0.0.0` (toutes les interfaces)** :
- Écoute sur toutes les interfaces réseau de la machine
- Accessible depuis d'autres machines du réseau local (via l'IP locale, ex: 192.168.1.100)
- Nécessite des mesures de sécurité (authentification, firewall)

**Analogie** : 
- `127.0.0.1` = fermer sa porte à clé (personne ne peut entrer)
- `0.0.0.0` = ouvrir sa porte (tout le voisinage peut entrer, il faut un vigile)

**Référence** : [/ressources/bases_reseau/README.md - Section Adresses IP](../../ressources/bases_reseau/README.md)
</details>

<details>
<summary><strong>Question 3 :</strong> Quels sont les risques de sécurité d'un webhook non authentifié ?</summary>

**Réponse :**

Un webhook **sans authentification** expose votre workflow à plusieurs risques :

1. **Accès non autorisé** : N'importe qui sur le réseau peut envoyer des requêtes et consommer vos ressources (LLM, crédits API)

2. **Abus de quota** : Un attaquant peut spammer vos webhooks pour épuiser vos quotas Gemini, Mistral, OpenAI

3. **Injection de données malveillantes** : Un attaquant peut envoyer des prompts malveillants pour :
   - Contourner les limites du LLM (jailbreak)
   - Exfiltrer des données sensibles
   - Générer du contenu illégal

4. **Déni de service (DoS)** : Surcharger votre serveur n8n avec des milliers de requêtes

**Solutions** :
- ✅ Toujours activer l'authentification (Basic Auth minimum)
- ✅ Utiliser HTTPS (reverse proxy avec SSL/TLS)
- ✅ Implémenter un rate limiting (limiter le nombre de requêtes par IP/utilisateur)
- ✅ Valider et sanitiser les entrées (filtrer les prompts suspects)
- ✅ Utiliser un firewall pour restreindre les IPs autorisées

**Référence** : [/ressources/credentials/README.md - Section Sécurité des APIs](../../ressources/credentials/README.md)
</details>

<details>
<summary><strong>Question 4 :</strong> Pourquoi les credentials sont-ils transmis en clair sans HTTPS ?</summary>

**Réponse :**

**HTTP (sans S)** :
- Les données sont transmises **en clair** sur le réseau
- N'importe qui avec un analyseur de paquets (Wireshark, tcpdump) peut intercepter les données
- Cela inclut les credentials (login/password en Basic Auth, API keys, tokens)

**Exemple d'interception** :
Un attaquant sur le même réseau Wi-Fi peut capturer vos requêtes :
```
POST /webhook/ollama HTTP/1.1
Authorization: Basic YWRtaW46bW90ZGVwYXNzZQ==
```
Le header `Authorization` est encodé en Base64 (pas chiffré !), il suffit de décoder :
```bash
echo "YWRtaW46bW90ZGVwYXNzZQ==" | base64 -d
# Résultat : admin:motdepasse
```

**HTTPS (avec S)** :
- Les données sont **chiffrées** avec TLS/SSL
- Même interceptées, les données sont illisibles sans la clé de déchiffrement

**Solution pour la production** :
- Configurer un reverse proxy (Nginx, Traefik) avec SSL/TLS
- Utiliser Let's Encrypt pour obtenir un certificat gratuit
- Rediriger HTTP vers HTTPS automatiquement

**Pour ce projet (environnement local)** :
- Acceptable sur un réseau de confiance (chez soi, réseau local isolé)
- Ne jamais exposer sur Internet sans HTTPS

**Référence** : [/ressources/http&API/README.md - Section HTTPS et TLS](../../ressources/http&API/README.md)
</details>

<details>
<summary><strong>Question 5 :</strong> Comment tester un webhook depuis Python ou JavaScript ?</summary>

**Réponse :**

**Python (avec requests)** :
```python
import requests
import json

url = "http://192.168.1.100:5678/webhook/ollama"
auth = ("admin", "votre_mot_de_passe")
payload = {"chatInput": "Bonjour depuis Python !"}

response = requests.post(url, json=payload, auth=auth)
print(response.json())
```

**JavaScript (avec fetch)** :
```javascript
const url = "http://192.168.1.100:5678/webhook/ollama";
const auth = btoa("admin:votre_mot_de_passe"); // Base64 encode

fetch(url, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Basic ${auth}`
  },
  body: JSON.stringify({ chatInput: "Bonjour depuis JavaScript !" })
})
.then(response => response.json())
.then(data => console.log(data));
```

**Node.js (avec axios)** :
```javascript
const axios = require('axios');

const url = "http://192.168.1.100:5678/webhook/ollama";
const auth = { username: "admin", password: "votre_mot_de_passe" };
const payload = { chatInput: "Bonjour depuis Node.js !" };

axios.post(url, payload, { auth })
  .then(response => console.log(response.data));
```

**cURL** :
```bash
curl -X POST http://192.168.1.100:5678/webhook/ollama \
  -u admin:votre_mot_de_passe \
  -H "Content-Type: application/json" \
  -d '{"chatInput": "Bonjour depuis cURL !"}'
```

**Cela démontre l'interopérabilité des webhooks** : une fois exposés, vos workflows n8n peuvent être appelés depuis n'importe quel langage ou outil !
</details>

---

## 🐛 Dépannage

### Problème : "Connection refused" ou "timeout" lors de l'accès depuis une autre machine

**Symptômes** : Impossible d'accéder à `http://<IP>:5678` depuis une autre machine

**Solutions** :

1. **Vérifiez que n8n écoute bien sur `0.0.0.0`**
   ```bash
   docker compose ps
   # Vérifiez que le port est bien : 0.0.0.0:5678->5678/tcp
   ```

2. **Vérifiez le firewall**
   - Sous Linux (UFW) :
     ```bash
     sudo ufw allow 5678/tcp
     sudo ufw reload
     ```
   - Sous Windows : Panneau de configuration → Pare-feu Windows → Règle entrante → Autoriser port 5678

3. **Vérifiez que les deux machines sont sur le même réseau**
   ```bash
   # Sur la machine cliente, testez le ping
   ping 192.168.1.100
   ```

4. **Vérifiez l'IP avec `ip a` ou `ipconfig`**
   - Assurez-vous d'utiliser la bonne adresse IP (pas 127.0.0.1)

5. **Testez avec telnet**
   ```bash
   telnet 192.168.1.100 5678
   # Si "Connected", le port est accessible
   ```

**Référence** : [/ressources/bases_reseau/README.md - Section Troubleshooting](../../ressources/bases_reseau/README.md)

---

### Problème : "401 Unauthorized" lors de l'appel d'un webhook avec authentification

**Symptômes** : Le webhook retourne une erreur 401 même avec les bons credentials

**Solutions** :

1. **Vérifiez que les credentials sont bien configurés**
   - Dans le workflow, ouvrez le nœud Webhook
   - Vérifiez que le credential sélectionné est le bon

2. **Vérifiez le format de l'en-tête Authorization**
   - En Basic Auth, le format est : `Authorization: Basic <base64(username:password)>`
   - Testez avec cURL :
     ```bash
     curl -v -X POST http://localhost:5678/webhook/ollama \
       -u admin:password \
       -H "Content-Type: application/json" \
       -d '{"chatInput": "test"}'
     ```

3. **Vérifiez qu'il n'y a pas d'espace ou caractère invisible** dans le username/password

4. **Si vous utilisez Postman** :
   - Allez dans l'onglet "Authorization"
   - Sélectionnez "Basic Auth"
   - Entrez username et password (Postman génère automatiquement le header)

---

### Problème : Le webhook retourne une erreur 500 ou les données sont mal formatées

**Symptômes** : Le webhook s'exécute, mais retourne une erreur interne

**Solutions** :

1. **Vérifiez la structure du JSON envoyé**
   - Le nœud LLM attend peut-être un champ spécifique (ex: `message`, `prompt`, `chatInput`)
   - Inspectez les données reçues dans le nœud Webhook (cliquez dessus après exécution)

2. **Ajoutez un nœud "Set" ou "Code" après le webhook**
   - Pour normaliser/transformer les données avant de les passer au LLM
   - Exemple :
     ```javascript
     return [{ 
       json: { 
         message: $input.first().json.chatInput || $input.first().json.message 
       } 
     }];
     ```

3. **Consultez les logs n8n**
   ```bash
   docker compose logs n8n
   # Cherchez les erreurs détaillées
   ```

4. **Testez le workflow manuellement**
   - Exécutez le workflow depuis l'interface n8n (bouton "Execute Workflow")
   - Vérifiez qu'il fonctionne sans erreur

---

### Problème : Les webhooks ne répondent qu'après un long délai (timeout)

**Symptômes** : Les requêtes prennent plus de 30 secondes, ou timeout

**Causes possibles** :
- Le modèle LLM est lent (surtout Ollama sans GPU)
- Timeout configuré trop court dans le client HTTP

**Solutions** :

1. **Augmentez le timeout dans le nœud HTTP Request**
   - Ouvrez le nœud "HTTP Request" dans le workflow client
   - Dans les options avancées, configurez :
     - **Timeout** : `60000` (60 secondes) ou plus

2. **Utilisez un modèle plus rapide**
   - Ollama : `phi`, `tinyllama` au lieu de `mistral`
   - Gemini : `gemini-1.5-flash` au lieu de `gemini-pro`

3. **Implémentez un système de callback asynchrone** (avancé)
   - Le webhook retourne immédiatement "processing"
   - Une fois le LLM terminé, il envoie le résultat à un autre webhook du client

---

### Problème : Les workflows n8n ne démarrent pas après modification du docker-compose.yml

**Symptômes** : Erreur lors de `docker compose up -d`

**Solutions** :

1. **Vérifiez la syntaxe YAML**
   - YAML est sensible à l'indentation (utilisez des espaces, pas des tabs)
   - Validez avec un outil en ligne : [yamllint.com](http://www.yamllint.com/)

2. **Vérifiez les logs Docker**
   ```bash
   docker compose logs n8n
   ```

3. **Essayez de recréer le conteneur**
   ```bash
   docker compose down
   docker compose up -d --force-recreate
   ```

4. **Si le problème persiste, revenez à la configuration précédente**
   - Restaurez le backup de `docker-compose.yml`
   - Consultez [/ressources/docker/README.md](../../ressources/docker/README.md)

---

## 🔗 Ressources complémentaires

### Documentation interne

- 📂 [Protocoles HTTP et APIs REST](../../ressources/http&API/README.md)
- 📂 [Gestion des credentials et authentification](../../ressources/credentials/README.md)
- 📂 [Docker et configuration réseau](../../ressources/docker/README.md)
- 📂 [Bases du réseau (IP, ports, firewall)](../../ressources/bases_reseau/README.md)
- 📂 [Glossaire - Termes réseau et HTTP](../../ressources/GLOSSARY.md)

### Documentation externe

- [n8n - Webhook Node Documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)
- [n8n - HTTP Request Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)
- [n8n - Webhook Security Best Practices](https://docs.n8n.io/security/)
- [Docker - Networking](https://docs.docker.com/network/)
- [MDN - HTTP Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

### Articles et tutoriels

- [Building a Distributed Workflow System with n8n](https://blog.n8n.io/distributed-workflows/)
- [REST API Design Best Practices](https://restfulapi.net/rest-api-design-tutorial-with-example/)
- [Understanding Basic Auth vs Bearer Tokens](https://auth0.com/blog/authentication-vs-authorization/)

---

## ➡️ Étape suivante

Une fois cette étape validée, vous êtes prêt à passer à :

**[Étape 4 : Ajouter des formulaires pour les paramètres](../4.%20forms/README.md)**

Dans l'étape suivante, vous apprendrez à :
- Ajouter des champs de formulaire pour personnaliser les requêtes (température, max_tokens, system prompt)
- Utiliser le nœud "Form Trigger" pour créer des interfaces riches
- Transmettre les paramètres aux webhooks LLM
- Créer une interface utilisateur complète pour votre comparateur

---

## 📝 Notes et observations

Utilisez cet espace pour noter vos observations lors de la réalisation de cette étape :

- **Temps réel passé** : _____ minutes
- **Difficultés rencontrées** : _____________________________
- **IP locale de votre serveur** : _____________________________
- **Test depuis machine distante** : ☐ Réussi ☐ Non testé
- **Questions non résolues** : _____________________________

---

**Félicitations pour avoir complété cette étape !** Votre comparateur est maintenant **distribué** et accessible depuis n'importe quelle machine sur le réseau. 🚀
