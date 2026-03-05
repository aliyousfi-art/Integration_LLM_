# ✅ Vérification de l'Installation

Ce document vous guide dans la vérification complète de votre installation pour vous assurer que tous les services fonctionnent correctement.

---

## 📋 Checklist Complète d'Installation

### Phase 1 : Prérequis Système

- [ ] **Docker installé**
  ```bash
  docker --version
  # Attendu : Docker version 20.10.0 ou supérieur
  ```

- [ ] **Docker Compose installé**
  ```bash
  docker compose version
  # Attendu : Docker Compose version 2.0.0 ou supérieur
  ```

- [ ] **Fichier .env créé**
  ```bash
  ls -la .env
  # Attendu : Le fichier .env existe
  ```

- [ ] **Mots de passe modifiés dans .env**
  ```bash
  grep "strong_password_here" .env
  # Attendu : Aucun résultat (tous les mots de passe ont été changés)
  ```

---

### Phase 2 : Lancement des Services

- [ ] **Tous les conteneurs sont démarrés**
  ```bash
  docker ps
  # Attendu : 4 conteneurs en état "Up"
  # - n8n
  # - postgres
  # - pgadmin
  # - ollama
  ```

- [ ] **Aucun conteneur en erreur**
  ```bash
  docker ps -a --filter "status=exited"
  # Attendu : Aucun conteneur (liste vide)
  ```

- [ ] **Les volumes sont créés**
  ```bash
  docker volume ls | grep integration-llm
  # Attendu : 5 volumes
  # - n8n_data
  # - postgres_data
  # - pgadmin_data
  # - ollama_data
  # - ollama_models
  ```

- [ ] **Le réseau est créé**
  ```bash
  docker network ls | grep custom_network
  # Attendu : 1 réseau custom_network
  ```

---

### Phase 3 : Vérification de n8n

- [ ] **n8n est accessible**
  - Navigateur : `http://localhost:5678`
  - Attendu : Page de connexion n8n

- [ ] **Authentification fonctionne**
  - Username : `admin` (ou celui défini dans .env)
  - Password : celui défini dans .env
  - Attendu : Accès à l'interface n8n

- [ ] **Healthcheck n8n OK**
  ```bash
  docker inspect n8n | grep -A5 Health
  # Attendu : "Status": "healthy"
  ```

- [ ] **Base de données connectée**
  - Dans n8n, aller dans Settings → About
  - Attendu : Database type: postgresdb

- [ ] **Dossier /files accessible**
  ```bash
  ls -la n8n_files/
  # Attendu : Dossiers corrections/, input/, output/
  ```

---

### Phase 4 : Vérification de PostgreSQL

- [ ] **PostgreSQL répond**
  ```bash
  docker exec postgres pg_isready -U n8n_user -d n8n
  # Attendu : "accepting connections"
  ```

- [ ] **Port PostgreSQL accessible**
  ```bash
  nc -zv localhost 5434
  # OU
  telnet localhost 5434
  # Attendu : Connexion réussie
  ```

- [ ] **Healthcheck PostgreSQL OK**
  ```bash
  docker inspect postgres | grep -A5 Health
  # Attendu : "Status": "healthy"
  ```

- [ ] **pgvector extension disponible**
  ```bash
  docker exec postgres psql -U n8n_user -d n8n -c "SELECT * FROM pg_extension WHERE extname='vector';"
  # Attendu : 1 ligne avec l'extension vector
  # Si non, l'installer : CREATE EXTENSION IF NOT EXISTS vector;
  ```

---

### Phase 5 : Vérification de pgAdmin

- [ ] **pgAdmin est accessible**
  - Navigateur : `http://localhost:5050`
  - Attendu : Page de connexion pgAdmin

- [ ] **Connexion à pgAdmin réussie**
  - Email : celui défini dans .env (défaut: admin@admin.com)
  - Password : celui défini dans .env
  - Attendu : Accès à l'interface pgAdmin

- [ ] **Serveur PostgreSQL ajouté**
  - Clic droit sur Servers → Register → Server
  - General → Name : `n8n_postgres`
  - Connection :
    - Host : `postgres`
    - Port : `5432`
    - Database : `n8n`
    - Username : `n8n_user`
    - Password : celui défini dans .env
  - Attendu : Connexion réussie et serveur visible

- [ ] **Tables n8n visibles**
  - Servers → n8n_postgres → Databases → n8n → Schemas → public → Tables
  - Attendu : Tables n8n (credentials_entity, execution_entity, etc.)

---

### Phase 6 : Vérification d'Ollama

- [ ] **Ollama API répond**
  ```bash
  curl http://localhost:11434/api/tags
  # Attendu : JSON avec liste des modèles (peut être vide au début)
  ```

- [ ] **Healthcheck Ollama OK**
  ```bash
  docker inspect ollama | grep -A5 Health
  # Attendu : "Status": "healthy"
  ```

- [ ] **Modèle Mistral téléchargé**
  ```bash
  docker exec ollama ollama list
  # Attendu : Ligne avec "mistral:latest"
  ```
  
  Si non téléchargé :
  ```bash
  docker exec ollama ollama pull mistral
  # Attendre le téléchargement (peut prendre 5-10 minutes)
  ```

- [ ] **Modèle Mistral fonctionne**
  ```bash
  docker exec ollama ollama run mistral "Dis bonjour"
  # Attendu : Réponse du modèle en français
  ```

- [ ] **Llama Guard 3 téléchargé (optionnel pour étape 9)**
  ```bash
  docker exec ollama ollama pull llama-guard3
  docker exec ollama ollama list
  # Attendu : Ligne avec "llama-guard3:latest"
  ```

---

### Phase 7 : Vérification des Workflows d'Exemple

- [ ] **Exemples importés**
  ```bash
  docker exec n8n n8n import:workflow --separate --input=/files/corrections/
  # Attendu : Message de succès pour chaque workflow
  ```

- [ ] **Workflows visibles dans n8n**
  - Dans n8n, vérifier la liste des workflows
  - Attendu : Plusieurs workflows importés

- [ ] **Dossier organisé créé**
  - Créer un dossier "Exemples" dans n8n
  - Déplacer les workflows importés dedans

---

### Phase 8 : Test d'un Workflow Simple

- [ ] **Créer un workflow de test**
  - Dans n8n : New Workflow
  - Ajouter un nœud "Manual Trigger"
  - Ajouter un nœud "Basic LLM Chain"
  - Configurer : Model → Ollama → Mistral
  - Connecter les nœuds

- [ ] **Configurer les credentials Ollama**
  - Dans le nœud Ollama
  - Credentials : Create New
  - Base URL : `http://ollama:11434`
  - Sauvegarder

- [ ] **Tester le workflow**
  - Prompt : "Bonjour, qui es-tu ?"
  - Execute Workflow
  - Attendu : Réponse du modèle Mistral

---

### Phase 9 : Test de Persistance (Étape 5+)

- [ ] **Base de données comparateur_llm créée**
  ```bash
  docker exec postgres psql -U n8n_user -d n8n -c "CREATE DATABASE comparateur_llm;"
  # OU via pgAdmin : Create Database → comparateur_llm
  ```

- [ ] **Tables créées**
  ```sql
  -- Dans pgAdmin ou psql
  \c comparateur_llm
  
  CREATE TABLE question (
      id SERIAL PRIMARY KEY,
      question TEXT NOT NULL,
      date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  
  CREATE TABLE reponse (
      id SERIAL PRIMARY KEY,
      reponse TEXT NOT NULL,
      provider TEXT NOT NULL,
      question_id INTEGER NOT NULL REFERENCES question(id)
  );
  ```
  
  Vérification :
  ```bash
  docker exec postgres psql -U n8n_user -d comparateur_llm -c "\dt"
  # Attendu : Tables question et reponse
  ```

---

### Phase 10 : Test Réseau (Étape 3)

- [ ] **Communication interne entre services**
  ```bash
  docker exec n8n ping -c 2 postgres
  docker exec n8n ping -c 2 ollama
  # Attendu : Réponses ping réussies
  ```

- [ ] **DNS interne fonctionne**
  ```bash
  docker exec n8n nslookup postgres
  docker exec n8n nslookup ollama
  # Attendu : Résolution DNS vers IP interne
  ```

- [ ] **Ports internes accessibles**
  ```bash
  docker exec n8n wget -qO- http://ollama:11434/api/tags
  # Attendu : JSON avec liste des modèles
  ```

---

## 🔍 Diagnostic en Cas de Problème

### Vérifier les Logs

```bash
# Logs de tous les services
docker compose logs

# Logs d'un service spécifique
docker compose logs n8n
docker compose logs postgres
docker compose logs ollama
docker compose logs pgadmin

# Suivre les logs en temps réel
docker compose logs -f
```

### Redémarrer un Service

```bash
# Redémarrer un service spécifique
docker compose restart n8n

# Redémarrer tous les services
docker compose restart

# Recréer un service
docker compose up -d --force-recreate n8n
```

### Nettoyer et Recommencer

```bash
# Arrêter tous les services
docker compose down

# Supprimer les volumes (ATTENTION : perte de données)
docker compose down -v

# Relancer
docker compose up -d
```

---

## 📊 Dashboard de Statut

### Créer un Script de Vérification

Créez un fichier `check-status.sh` :

```bash
#!/bin/bash

echo "=== STATUS DES SERVICES ==="
echo ""

# n8n
echo "n8n:"
curl -s -o /dev/null -w "  HTTP: %{http_code}\n" http://localhost:5678

# PostgreSQL
echo "PostgreSQL:"
docker exec postgres pg_isready -U n8n_user -d n8n 2>&1 | grep -q "accepting connections" && echo "  Status: OK" || echo "  Status: ERROR"

# pgAdmin
echo "pgAdmin:"
curl -s -o /dev/null -w "  HTTP: %{http_code}\n" http://localhost:5050

# Ollama
echo "Ollama:"
curl -s -o /dev/null -w "  HTTP: %{http_code}\n" http://localhost:11434/api/tags

echo ""
echo "=== UTILISATION RESSOURCES ==="
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

Exécution :
```bash
chmod +x check-status.sh
./check-status.sh
```

---

## ✅ Résumé de Validation

Si tous les tests passent, vous devriez avoir :

| Service | Port | Status | Description |
|---------|------|--------|-------------|
| n8n | 5678 | ✅ | Interface workflow accessible |
| PostgreSQL | 5434 | ✅ | Base de données fonctionnelle |
| pgAdmin | 5050 | ✅ | Interface gestion DB accessible |
| Ollama | 11434 | ✅ | API LLM fonctionnelle |

**Étapes suivantes :**
1. Consultez `premier_workflow.md` pour créer votre premier workflow
2. Suivez le projet guidé dans `/projet/0. chat/README.md`
3. Explorez les ressources dans `/ressources/`

---

## 🆘 Obtenir de l'Aide

Si des problèmes persistent :

1. **Consultez les logs** : `docker compose logs`
2. **Vérifiez la documentation** : README.md dans chaque dossier
3. **Ressources externes** :
   - Documentation n8n : https://docs.n8n.io/
   - Documentation Ollama : https://github.com/ollama/ollama
   - Docker Compose : https://docs.docker.com/compose/

---

**🎉 Félicitations ! Si tous les tests passent, votre environnement est prêt pour le cours.**
