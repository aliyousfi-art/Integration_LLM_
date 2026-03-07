# Communication avec des API et Protocole HTTP

> 💡 **En bref** : Maîtriser HTTP et REST pour communiquer avec les APIs (LLM, webhooks...)  
> ⏱️ **Temps de lecture** : 2 heures  
> 🎯 **Niveau** : Débutant à Intermédiaire  
> 📚 **Prérequis** : Bases réseau (IP, ports)

Ce cours est une introduction aux concepts fondamentaux de la communication avec des API, avec un focus particulier sur
le modèle RESTful. Il est divisé en sections : la communication client/serveur, le protocole HTTP, et les API
REST.

---

## 🎯 Objectifs d'Apprentissage

À la fin de ce module, vous serez capable de :

- ✅ Comprendre le modèle client-serveur
- ✅ Maîtriser les méthodes HTTP (GET, POST, PUT, DELETE...)
- ✅ Interpréter les codes de statut (200, 404, 500...)
- ✅ Concevoir des APIs RESTful
- ✅ Utiliser les headers HTTP correctement
- ✅ Tester des APIs avec curl et Postman
- ✅ Intégrer des APIs dans n8n

---

## 1. Communication Client/Serveur

### Description des concepts

La communication client/serveur repose sur une architecture où deux entités principales interagissent entre elles :

- **Client** : Le consommateur de services. C'est souvent une application ou un utilisateur qui envoie une requête pour
  obtenir ou envoyer des données.
- **Serveur** : Le fournisseur de services. Il reçoit les requêtes du client, les traite, et retourne une réponse.

L'objectif de cette architecture est de permettre aux systèmes hétérogènes de communiquer facilement entre eux via un
protocole standard, comme HTTP.

### Exemple

Un exemple simple de communication client/serveur est celui d'un navigateur (client) qui accède à une page web :

1. Le navigateur envoie une requête au serveur (par exemple, pour charger `https://example.com`).
2. Le serveur répond avec le contenu demandé (par exemple, une page HTML ou un fichier JSON).

### 🖼️ Diagramme Client-Serveur

```mermaid
sequenceDiagram
    participant Client as 💻 Client<br/>(Navigateur, n8n, curl)
    participant Server as 🖥️ Serveur<br/>(API, Backend)
    participant DB as 💾 Base de Données
    
    Client->>Server: 1. HTTP Request<br/>GET /api/users
    activate Server
    Server->>DB: 2. Query données
    activate DB
    DB-->>Server: 3. Résultats
    deactivate DB
    Server-->>Client: 4. HTTP Response<br/>200 OK + JSON
    deactivate Server
    
    Note over Client,Server: Communication synchrone<br/>Request-Response
```

**Rôles :**
- **Client** : Initie la communication (demande)
- **Serveur** : Attend et répond aux requêtes
- **Protocole** : HTTP/HTTPS (règles de communication)

**Dans le projet :**
- n8n = Client (appelle APIs LLM)
- APIs OpenAI/Anthropic = Serveurs
- Webhooks n8n = Serveur (reçoit requêtes externes)

### Ressources

- [Comprendre l'architecture client/serveur](https://www.techtarget.com/whatis/definition/client-server-architecture)

---

## 2. Le protocole HTTP

HTTP (Hypertext Transfer Protocol) est un protocole utilisé pour la communication entre un client et un serveur sur le
web. Il fonctionne selon un modèle de **requêtes** et de **réponses**.

- Requête HTTP : Composée d’une méthode (GET, POST, PUT, DELETE, etc.), d'une URL et parfois d’un corps de requête.
- Réponse HTTP : Elle contient un **code de statut** (exemple : 200 pour succès, 404 pour non trouvé) et des données.

### Principales méthodes HTTP

- **GET** : Récupérer des ressources (lecture).
- **POST** : Envoyer des données pour les créer sur le serveur.
- **PUT** : Mettre à jour ou remplacer une ressource existante.
- **DELETE** : Supprimer une ressource sur le serveur.

### URL, URI et paramètres de requête

Définition d'URL et d'URI

- **URL (Uniform Resource Locator)** : Il s'agit de l'adresse complète qui identifie une ressource sur un réseau. Elle
  contient des informations de localisation et le protocole utilisé pour accéder à cette ressource (exemple :
  `https://example.com/page`).

- **URI (Uniform Resource Identifier)** : Il s'agit d'un concept plus général qui inclut les URL et tout autre
  identifiant permettant de nommer ou d'identifier une ressource.

En résumé :

- Une URL est toujours une URI, mais une URI n'est pas forcément une URL.

Une URL est composée de plusieurs parties. Par exemple, pour l'URL suivante :

`https://example.com:8080/articles?page=2&sort=asc#section1`

Les différentes parties sont définies comme suit :

- **Protocole** : `https` - Indique le protocole à utiliser pour accéder à la ressource.
- **Nom de domaine** : `example.com` - Spécifie le nom de l'hôte ou du serveur.
- **Port** : `8080` - Définit le port, généralement optionnel. Par défaut, il est implicite pour les protocoles,
comme 80 pour HTTP ou 443 pour HTTPS.
- **Chemin** : `/articles` - Définit la ressource ou l'endroit sur le serveur où se trouve l'information.
- **Paramètres de requête** : `?page=2&sort=asc` - Contiennent des paires clé/valeur passées après un point
d'interrogation (?) pour transmettre des données au serveur.
- **Fragment** : `#section1` - Fait référence à une section spécifique d'une page ou ressource.

Cela permet de structurer les URL, facilitant la communication et navigation dans le cadre du protocole HTTP.

### Structure d'une Requête HTTP

Les trames HTTP sont les structures fondamentales utilisées pour échanger des données entre un client et un serveur.
Elles contiennent des informations qui permettent de spécifier l'action demandée et les données nécessaires.

Une **requête HTTP** se compose généralement de trois parties principales :

1. **Ligne de requête** : Spécifie le type d'action avec la méthode HTTP, l'URL, et la version du protocole HTTP.
2. **En-têtes** : Contiennent des métadonnées sur la requête (type de contenu, authentification, etc.).
3. **Corps (facultatif)** : Contient des données pour certaines méthodes comme POST ou PUT.

### Exemple de requête HTTP :

```http
POST /posts HTTP/1.1
Host: jsonplaceholder.typicode.com
Content-Type: application/json

{
  "title": "Nouvel Article",
  "body": "Contenu de l'article"
}
```

#### Explications des différentes parties de la requête :

1. **Ligne de requête** :
    - `POST /posts HTTP/1.1` :
        - **POST** : Méthode HTTP indiquant qu'on souhaite créer une nouvelle ressource.
        - **/posts** : URL ou chemin identifié la ressource ou le point de terminaison cible sur le serveur.
        - **HTTP/1.1** : Version du protocole HTTP utilisée pour la requête.

2. **En-têtes (Headers)** :
    - `Host: jsonplaceholder.typicode.com` :
        - Requis pour indiquer l'hôte réseau où la requête est envoyée.
    - `Content-Type: application/json` :
        - Indique le type des données envoyées dans le corps de la requête. Ici, il s'agit de données au format JSON.

3. **Corps (Body)** :
    - Contient les données à envoyer au serveur pour créer une ressource.
    - Dans cet exemple :
      ```json
      {
        "title": "Nouvel Article",
        "body": "Contenu de l'article"
      }
      ```
        - **title** : Titre de l'article à créer.
        - **body** : Contenu ou texte associé à l'article.

#### Réponse à la requête

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 101,
  "title": "Nouvel Article",
  "body": "Contenu de l'article",
  "userId": 1
}
```

#### Explications des différentes parties de la réponse :

1. **Statut de la réponse** :
    - `HTTP/1.1 201 Created` :
        - **201 Created** : Code de statut HTTP indiquant que la requête a réussi et qu'une nouvelle ressource a été
          créée sur le serveur.

2. **En-têtes (Headers)** :
    - `Content-Type: application/json` :
        - Indique que les données de la réponse sont envoyées au format JSON.

3. **Corps (Body)** :
    - Contient les données de la ressource nouvellement créée.
    - Dans cet exemple :
      ```json
      {
        "id": 101,
        "title": "Nouvel Article",
        "body": "Contenu de l'article",
        "userId": 1
      }
      ```
        - **id** : Identifiant unique attribué à la ressource sur le serveur.
        - **title** et **body** : Informations envoyées dans la requête, renvoyées ici pour confirmer leur
          enregistrement.
        - **userId** : Identifiant de l'utilisateur associé à la création de cette ressource.

### Exemple de Requête/Response

Voici un exemple avec l'outil `cURL` :

#### Requête GET

```bash
curl -X GET https://jsonplaceholder.typicode.com/posts/1
```

#### Réponse

```json
{
  "userId": 1,
  "id": 1,
  "title": "Sample Title",
  "body": "This is a test body"
}
```

### Ressources

- [Introduction au protocole HTTP sur Mozilla](https://developer.mozilla.org/fr/docs/Web/HTTP)

---

## 3. Les API REST

### Description des concepts

REST (Representational State Transfer) est une architecture qui s’appuie sur HTTP pour créer des services web. Voici ses
principes fondamentaux :

1. **Basé sur des ressources** : Tout élément manipulé par l’API est une ressource identifiée via une URL unique.
   Exemple : `/users/1` identifie un utilisateur spécifique.
2. **Méthodes HTTP standardisées** : REST utilise des méthodes HTTP comme GET, POST, PUT et DELETE.
3. **Stateless** : Chaque requête est indépendante. Le serveur ne conserve pas de contexte entre deux requêtes.
4. **Utilisation des formats standard** : Les réponses sont souvent encodées au format JSON ou XML, ce qui les rend
   facilement lisibles par les machines.

### Exemple de création et récupération avec une API REST

Prenons une API REST pour gérer une liste d'utilisateurs :

#### Création d’un utilisateur (POST)

Requête :

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "John Doe"}' https://example.com/api/users
```

Réponse :

```json
{
  "id": 1,
  "name": "John Doe"
}
```

#### Récupération d’un utilisateur (GET)

Requête :

```bash
curl -X GET https://example.com/api/users/1
```

Réponse :

```json
{
  "id": 1,
  "name": "John Doe"
}
```

### Sécurité des APIs

Lorsqu'on manipule des API, la sécurité est primordiale afin de protéger les données échangées et éviter tout accès non
autorisé. Voici quelques concepts clés pour sécuriser une API :

1. **Authentification** : Vérifier l'identité de l'utilisateur ou du système qui envoie une requête.
2. **Autorisation** : Permettre ou non l'accès à certaines ressources en fonction des droits.
3. **Chiffrement (HTTPS)** : Échanger des données via une connexion cryptée pour éviter leur interception.
4. **Rate Limiting** : Limiter le nombre de requêtes autorisées pour prévenir les abus.
5. **Utilisation de tokens** : Protéger l'accès aux API au moyen de jetons d'authentification, comme les tokens JWT (
   JSON Web Tokens).

#### Utilisation d’un token dans le header

Pour faire une requête authentifiée à une API, un **token** est souvent passé dans l'en-tête HTTP, sous le champ
`Authorization`.

Exemple de requête avec un token dans le header :

```bash
curl -X GET https://example.com/api/protected-resource \
-H "Authorization: Bearer <votre_token>"
```

Dans cet exemple :

- `Authorization` : Nom de l’en-tête HTTP utilisé pour indiquer un token.
- `Bearer` : Type de token utilisé (le plus courant est Bearer Token).
- `<votre_token>` : Token que vous passez au serveur pour vous authentifier.

#### Exemple d’une requête avec cURL

```bash
curl -X GET https://example.com/api/users \
-H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
```

Dans cette requête :

- Le token `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ...` est un exemple de JWT utilisé pour authentifier l’utilisateur.

#### Vérification côté serveur

Le serveur qui reçoit une requête avec un token dans les headers doit :

1. Vérifier la validité du token.
2. Décoder le contenu (par exemple avec une clé secrète si le token est un JWT).
3. Autoriser ou refuser l'accès en fonction des permissions associées au token.

#### Ressources

- [JSON Web Tokens (JWT)](https://jwt.io/)
- [Sécuriser son API RESTful](https://owasp.org/www-project-api-security/)

### Ressources supplémentaires

Pour aller plus loin dans la compréhension et l'utilisation des API REST, voici des ressources complémentaires :

- [Documentation RESTful API by IBM](https://www.ibm.com/topics/rest-apis)
- [Tutoriel sur REST par REST API Tutorial](https://restfulapi.net/)

#### Vidéos explicatives :

- [REST API Concept and Examples](https://www.youtube.com/watch?v=7YcW25PHnAA) - Une vidéo simple et pédagogique sur les
  concepts des API REST.
- [What is an API?](https://www.youtube.com/watch?v=s7wmiS2mSXY) - Introduction rapide aux API par IBM Cloud.
- [REST APIs : RESTful Design Principles](https://www.youtube.com/watch?v=YourLinkHere) - Vidéo expliquant les principes
  fondamentaux de REST dans une approche de conception.

#### Articles de vulgarisation :

- [Comprendre les API REST](https://www.alsacreations.com/article/lire/1687-rest-et-les-api.html) - Une introduction
  simple et efficace sur le sujet.
- [API REST : Définition et mise en œuvre](https://www.redhat.com/fr/topics/api/what-is-a-rest-api) - Article détaillé
  par Red Hat présentant REST et les applications pratiques.
- [Comment fonctionnent les API REST](https://www.programmez.com/magazine/12) - Un guide pratique à destination des
  développeurs débutants et confirmés.

#### Pages Wikipédia :

- [API - Interface de programmation d'application](https://fr.wikipedia.org/wiki/Application_programming_interface) -
  Page détaillant les bases des API, incluant les API REST.
- [REST - Représentation état de la ressource](https://fr.wikipedia.org/wiki/Representational_State_Transfer) -
  Description technique et historique du modèle architectural REST.

---

## 🖼️ Architecture REST Complète

```mermaid
graph TB
    subgraph "Client Applications"
        Web[🌐 Web App]
        Mobile[📱 Mobile App]
        N8N[⚡ n8n Workflow]
    end
    
    subgraph "API Gateway / Load Balancer"
        Gateway[🚪 API Gateway<br/>Rate Limiting, Auth]
    end
    
    subgraph "REST API Server"
        Routes[📍 Routes<br/>/api/users<br/>/api/posts]
        Controllers[🎮 Controllers<br/>Business Logic]
        Models[📦 Models<br/>Data Structure]
    end
    
    subgraph "Data Layer"
        DB[(💾 Database<br/>PostgreSQL)]
        Cache[(⚡ Cache<br/>Redis)]
    end
    
    Web --> Gateway
    Mobile --> Gateway
    N8N --> Gateway
    
    Gateway --> Routes
    Routes --> Controllers
    Controllers --> Models
    Models --> DB
    Models --> Cache
    
    style Gateway fill:#FFD700
    style Routes fill:#87CEEB
    style Controllers fill:#90EE90
    style Models fill:#FFA07A
    style DB fill:#9370DB
```

**Flux typique d'une requête :**
1. **Client** envoie requête HTTP
2. **API Gateway** vérifie auth, rate limit
3. **Routes** dirigent vers bon controller
4. **Controller** applique logique métier
5. **Model** interagit avec base de données
6. **Response** remonte la chaîne (JSON)

---

## 💻 Exercices Pratiques

### Exercice 1 : Tester une API avec curl

**Objectif** : Appeler l'API JSONPlaceholder

<details>
<summary>📝 Instructions</summary>

Utilisez curl pour :
1. GET liste d'utilisateurs
2. GET un utilisateur spécifique
3. POST créer un post
4. PUT modifier un post
5. DELETE supprimer un post

API : https://jsonplaceholder.typicode.com

</details>

<details>
<summary>✅ Solution</summary>

```bash
# 1. GET tous les utilisateurs
curl https://jsonplaceholder.typicode.com/users

# 2. GET utilisateur #1
curl https://jsonplaceholder.typicode.com/users/1

# 3. POST créer un post
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Mon premier post",
    "body": "Contenu du post",
    "userId": 1
  }'

# 4. PUT modifier post #1
curl -X PUT https://jsonplaceholder.typicode.com/posts/1 \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "title": "Titre modifié",
    "body": "Nouveau contenu",
    "userId": 1
  }'

# 5. DELETE supprimer post #1
curl -X DELETE https://jsonplaceholder.typicode.com/posts/1
```

**Options curl utiles :**
- `-X` : Méthode HTTP
- `-H` : Header
- `-d` : Data (body)
- `-i` : Inclure headers de réponse
- `-v` : Verbose (debug)

**Lien projet :** Étape 3, 7 (appels APIs externes)

</details>

---

### Exercice 2 : Créer un Workflow n8n avec API

**Objectif** : Intégrer une API REST dans n8n

<details>
<summary>📝 Instructions</summary>

Créez un workflow qui :
1. Reçoit un webhook avec un user_id
2. Appelle l'API JSONPlaceholder pour récupérer l'utilisateur
3. Récupère ses posts
4. Retourne un résumé (nom + nombre de posts)

</details>

<details>
<summary>✅ Solution</summary>

**Workflow n8n :**
```
Webhook → HTTP Request (User) → HTTP Request (Posts) → Code (Count) → Respond
```

**Node 1 - Webhook :**
- Method: GET
- Path: user-info

**Node 2 - HTTP Request (Get User) :**
- Method: GET
- URL: `https://jsonplaceholder.typicode.com/users/{{ $json.query.user_id }}`

**Node 3 - HTTP Request (Get Posts) :**
- Method: GET
- URL: `https://jsonplaceholder.typicode.com/posts?userId={{ $json.id }}`

**Node 4 - Code (Summary) :**
```javascript
const user = $input.first().json;
const posts = $("HTTP Request1").all();

return [{
  json: {
    user: {
      name: user.name,
      email: user.email
    },
    stats: {
      total_posts: posts.length,
      posts_titles: posts.map(p => p.json.title)
    }
  }
}];
```

**Node 5 - Respond to Webhook :**
- Retourne le JSON du node précédent

**Test :**
```bash
curl "http://localhost:5678/webhook/user-info?user_id=1"
```

**Résultat attendu :**
```json
{
  "user": {
    "name": "Leanne Graham",
    "email": "Sincere@april.biz"
  },
  "stats": {
    "total_posts": 10,
    "posts_titles": ["...", "..."]
  }
}
```

</details>

---

## ✅ Quiz d'Auto-Évaluation

**1. Quelle méthode HTTP pour récupérer des données ?**

<details><summary>Réponse</summary>
✅ **GET**

GET = Lecture (Read)  
Pas de body, données dans l'URL (query params)
</details>

**2. Quelle méthode HTTP pour créer une ressource ?**

<details><summary>Réponse</summary>
✅ **POST**

POST = Création (Create)  
Données dans le body (JSON généralement)
</details>

**3. Que signifie le code 404 ?**

<details><summary>Réponse</summary>
✅ **Not Found** - Ressource introuvable

L'URL demandée n'existe pas sur le serveur.
</details>

**4. Que signifie le code 200 ?**

<details><summary>Réponse</summary>
✅ **OK** - Succès

La requête a réussi, le serveur retourne la ressource demandée.
</details>

**5. Que signifie le code 500 ?**

<details><summary>Réponse</summary>
✅ **Internal Server Error** - Erreur serveur

Bug côté serveur, pas de la faute du client.
</details>

**6. Différence entre PUT et PATCH ?**

<details><summary>Réponse</summary>
✅ **PUT** = Remplacer entièrement  
✅ **PATCH** = Modifier partiellement

PUT : Envoyer toutes les propriétés  
PATCH : Envoyer seulement ce qui change
</details>

**7. Qu'est-ce qu'une API REST ?**

<details><summary>Réponse</summary>
✅ **Representational State Transfer**

Architecture basée sur HTTP avec :
- Ressources identifiées par URLs
- Méthodes HTTP standard
- Stateless (sans état)
- Format JSON généralement
</details>

**8. Que contient le header Authorization ?**

<details><summary>Réponse</summary>
✅ **Token d'authentification**

Format typique :
```
Authorization: Bearer eyJhbGci...
```

Types : Bearer, Basic, API Key...
</details>

**9. Qu'est-ce que le rate limiting ?**

<details><summary>Réponse</summary>
✅ **Limitation du nombre de requêtes**

Exemple : 100 requêtes/minute maximum

Protège contre abus et DDoS.

Headers :
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000
```
</details>

**10. Différence entre HTTP et HTTPS ?**

<details><summary>Réponse</summary>
✅ **HTTPS = HTTP + TLS/SSL (chiffrement)**

HTTP : Port 80, en clair  
HTTPS : Port 443, chiffré

TOUJOURS utiliser HTTPS en production !
</details>

**Score :** _/10  
- 8-10 : Expert APIs ! 🌐  
- 5-7 : Bien, pratiquez avec des APIs réelles  
- 0-4 : Relisez et testez avec curl

---

## 📊 Cheat Sheet HTTP & APIs

### Méthodes HTTP

| Méthode | Usage | Idempotent | Body | URL Example |
|---------|-------|------------|------|-------------|
| **GET** | Lire | ✅ Oui | ❌ Non | `/api/users?page=1` |
| **POST** | Créer | ❌ Non | ✅ Oui | `/api/users` |
| **PUT** | Remplacer | ✅ Oui | ✅ Oui | `/api/users/123` |
| **PATCH** | Modifier | ❌ Non | ✅ Oui | `/api/users/123` |
| **DELETE** | Supprimer | ✅ Oui | ❌ Non | `/api/users/123` |
| **HEAD** | Headers seulement | ✅ Oui | ❌ Non | `/api/users/123` |
| **OPTIONS** | Méthodes supportées | ✅ Oui | ❌ Non | `/api/users` |

### Codes de Statut HTTP

**2xx - Succès**

| Code | Signification | Usage |
|------|---------------|-------|
| 200 | OK | GET/PUT réussi |
| 201 | Created | POST réussi (ressource créée) |
| 204 | No Content | DELETE réussi (pas de body) |

**3xx - Redirection**

| Code | Signification | Usage |
|------|---------------|-------|
| 301 | Moved Permanently | Ressource déplacée définitivement |
| 302 | Found | Redirection temporaire |
| 304 | Not Modified | Cache valide (pas de changement) |

**4xx - Erreur Client**

| Code | Signification | Usage |
|------|---------------|-------|
| 400 | Bad Request | Requête mal formée (JSON invalide...) |
| 401 | Unauthorized | Non authentifié (pas de token) |
| 403 | Forbidden | Authentifié mais pas les droits |
| 404 | Not Found | Ressource introuvable |
| 429 | Too Many Requests | Rate limit dépassé |

**5xx - Erreur Serveur**

| Code | Signification | Usage |
|------|---------------|-------|
| 500 | Internal Server Error | Bug serveur |
| 502 | Bad Gateway | Proxy/gateway invalide |
| 503 | Service Unavailable | Serveur surchargé/maintenance |
| 504 | Gateway Timeout | Timeout proxy/gateway |

### Headers HTTP Courants

| Header | Rôle | Exemple |
|--------|------|---------|
| `Content-Type` | Format du body | `application/json` |
| `Authorization` | Token auth | `Bearer eyJhbGc...` |
| `Accept` | Format accepté | `application/json` |
| `User-Agent` | Client | `curl/7.68.0` |
| `X-RateLimit-*` | Rate limiting info | `X-RateLimit-Remaining: 45` |

### Commandes curl Essentielles

```bash
# GET simple
curl https://api.example.com/users

# GET avec query params
curl "https://api.example.com/users?page=2&limit=10"

# POST avec JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

# Avec authentification
curl https://api.example.com/users \
  -H "Authorization: Bearer YOUR_TOKEN"

# Afficher headers de réponse
curl -i https://api.example.com/users

# Verbose (debug)
curl -v https://api.example.com/users

# Sauver réponse dans fichier
curl https://api.example.com/users > users.json

# Follow redirects
curl -L https://api.example.com/users
```

### REST URL Design Best Practices

✅ **BON :**
```
GET    /api/users           # Liste
GET    /api/users/123       # Détail
POST   /api/users           # Créer
PUT    /api/users/123       # Remplacer
PATCH  /api/users/123       # Modifier
DELETE /api/users/123       # Supprimer

# Ressources imbriquées
GET    /api/users/123/posts # Posts de l'user 123
```

❌ **MAUVAIS :**
```
GET    /api/getAllUsers
POST   /api/createUser
GET    /api/user_delete?id=123
POST   /api/users/search    # Utilisez GET avec query params
```

---

## 🔗 Liens avec le Projet

| Étape | Utilisation HTTP/API |
|-------|----------------------|
| **0. Chat** | POST vers API LLM (OpenAI, Anthropic...) |
| **1. Diversity** | Appels multiples APIs LLMs |
| **3. Distribute** | Webhooks (recevoir requêtes HTTP) |
| **4. Forms** | POST formulaire vers webhook |
| **7. Enhance** | GET APIs externes (RAG, search...) |
| **9. Secure** | Headers Authorization, rate limiting |

---

## ❓ FAQ

**Q : Quelle différence entre API et REST API ?**  
API = Interface générale (peut être SOAP, GraphQL, gRPC...)  
REST API = API suivant les principes REST (HTTP, ressources, stateless...)

**Q : JSON obligatoire pour REST ?**  
Non, mais c'est le standard de facto. XML possible mais rare aujourd'hui.

**Q : Comment tester une API sans curl ?**  
- **Postman** : Interface graphique complète
- **Insomnia** : Alternative à Postman
- **HTTPie** : CLI plus user-friendly que curl
- **n8n** : Directement dans votre workflow !

**Q : Que faire si rate limit dépassé ?**  
1. Attendre (respecter X-RateLimit-Reset)
2. Implémenter retry avec exponential backoff
3. Cacher les résultats
4. Upgrade plan API si possible

**Q : API key dans URL ou header ?**  
✅ **Header** (plus sécurisé)  
❌ URL (apparaît dans les logs serveur)

---

Ce cours vous présente les bases nécessaires pour comprendre et utiliser efficacement les API. Plongez dans les
ressources fournies pour approfondir vos connaissances et commencez à expérimenter avec des exemples concrets ! 🚀

---

_Dernière mise à jour : Décembre 2024 | Version 1.4.0_