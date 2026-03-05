# 🚀 Configuration GPU NVIDIA pour Ollama

Ce guide vous explique comment activer l'accélération GPU NVIDIA pour Ollama afin d'obtenir de meilleures performances avec les modèles LLM locaux.

> ⚠️ **Attention** : Cette configuration nécessite une carte graphique NVIDIA et peut prendre du temps à configurer. Ne procédez que si vous avez du matériel compatible.

---

## 📋 Prérequis

Avant de commencer, vérifiez que vous avez :

- ✅ Une carte graphique NVIDIA compatible (GTX 10xx series ou plus récent)
- ✅ Pilotes NVIDIA installés et à jour
- ✅ Docker version 19.03 ou supérieure
- ✅ Docker Compose version 1.28 ou supérieure
- ✅ Au moins 6 Go de VRAM pour les modèles standards

### Vérifier votre configuration GPU

```bash
# Vérifier que les pilotes NVIDIA sont installés
nvidia-smi

# Vérifier la version de Docker
docker --version

# Vérifier la version de Docker Compose
docker compose version
```

---

## 🔧 Installation des Outils NVIDIA

### Option 1 : Linux (Ubuntu/Debian)

#### 1. Installer les pilotes NVIDIA

```bash
# Mettre à jour la liste des paquets
sudo apt update

# Installer les pilotes recommandés
sudo ubuntu-drivers autoinstall

# OU installer une version spécifique
sudo apt install nvidia-driver-535

# Redémarrer le système
sudo reboot

# Après redémarrage, vérifier l'installation
nvidia-smi
```

#### 2. Installer NVIDIA Container Toolkit

```bash
# Ajouter la clé GPG de NVIDIA
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Ajouter le dépôt
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Mettre à jour et installer
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Configurer Docker pour utiliser le runtime NVIDIA
sudo nvidia-ctk runtime configure --runtime=docker

# Redémarrer Docker
sudo systemctl restart docker
```

#### 3. Tester l'installation

```bash
# Tester que Docker peut accéder au GPU
docker run --rm --gpus all nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi

# Si cette commande affiche les informations de votre GPU, c'est bon ! ✅
```

### Option 2 : Windows avec WSL 2

#### 1. Installer WSL 2

```powershell
# Dans PowerShell en tant qu'administrateur
wsl --install
```

#### 2. Installer les pilotes GPU pour WSL

1. Téléchargez les pilotes depuis : https://developer.nvidia.com/cuda/wsl
2. Installez le pilote NVIDIA pour WSL
3. Redémarrez votre machine

#### 3. Vérifier dans WSL

```bash
# Ouvrir WSL (Ubuntu)
wsl

# Vérifier que le GPU est accessible
nvidia-smi
```

#### 4. Configurer Docker Desktop

1. Ouvrir Docker Desktop
2. Aller dans **Settings** → **Resources** → **WSL Integration**
3. Activer l'intégration avec votre distribution WSL
4. Aller dans **Settings** → **Docker Engine**
5. Ajouter la configuration NVIDIA :

```json
{
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```

6. Redémarrer Docker Desktop

### Option 3 : macOS

> ⚠️ **Note** : macOS ne supporte pas les GPU NVIDIA avec Docker. Les Mac avec GPU Apple Silicon (M1/M2/M3) ne sont pas compatibles avec CUDA.

---

## 🐳 Utilisation du Docker Compose GPU

### Utiliser le fichier docker-compose-gpu.yml

Le fichier `docker-compose-gpu.yml` est déjà configuré pour utiliser le GPU :

```bash
# Lancer avec support GPU
docker compose -f docker-compose-gpu.yml up -d

# Vérifier que les conteneurs sont lancés
docker ps

# Voir les logs d'Ollama
docker logs ollama
```

### Différences clés avec docker-compose.yml standard

```yaml
# Configuration GPU dans docker-compose-gpu.yml
ollama:
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            count: all
            capabilities: [gpu]
  environment:
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,utility
```

---

## 📥 Télécharger et Tester un Modèle

### Télécharger Mistral avec GPU

```bash
# Se connecter au conteneur Ollama
docker exec -it ollama bash

# À l'intérieur du conteneur, télécharger Mistral
ollama pull mistral

# Tester le modèle
ollama run mistral "Bonjour, comment vas-tu ?"

# Quitter le conteneur
exit
```

### Vérifier l'utilisation du GPU

Pendant que le modèle s'exécute, dans un autre terminal :

```bash
# Surveiller l'utilisation du GPU en temps réel
watch -n 1 nvidia-smi
```

Vous devriez voir :
- 📊 Utilisation GPU augmenter
- 💾 Mémoire GPU utilisée
- ⚡ Power draw (consommation énergétique)

---

## 🔍 Dépannage

### Problème : "no NVIDIA GPU found"

**Solution** :
```bash
# Vérifier que nvidia-smi fonctionne
nvidia-smi

# Vérifier que Docker voit le runtime NVIDIA
docker run --rm --gpus all nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi

# Reconfigurer le runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### Problème : "driver version mismatch"

**Solution** :
```bash
# Vérifier la version du driver
nvidia-smi

# Mettre à jour les drivers
sudo apt update
sudo apt upgrade nvidia-driver-*

# Redémarrer
sudo reboot
```

### Problème : Ollama utilise toujours le CPU

**Solution** :
```bash
# Vérifier que le conteneur a bien accès au GPU
docker exec ollama nvidia-smi

# Si erreur, vérifier les logs
docker logs ollama

# Recréer le conteneur
docker compose -f docker-compose-gpu.yml down
docker compose -f docker-compose-gpu.yml up -d
```

### Problème : Out of Memory (OOM)

**Solution** :
- Utilisez un modèle plus petit (ex: `mistral:7b` au lieu de `mistral:70b`)
- Vérifiez la VRAM disponible : `nvidia-smi`
- Libérez de la mémoire GPU : fermez les autres applications GPU

---

## 📊 Performances Attendues

### Comparaison CPU vs GPU

| Modèle | CPU (tokens/s) | GPU (tokens/s) | Gain |
|--------|---------------|----------------|------|
| Mistral 7B | 2-5 | 30-80 | 10-15x |
| Llama 2 7B | 2-4 | 25-70 | 12-17x |
| Llama Guard 3 | 3-6 | 35-90 | 10-15x |

> 📈 Les performances varient selon votre GPU, la longueur du prompt et la complexité de la requête.

---

## ⚙️ Configuration Avancée

### Utiliser plusieurs GPU

Si vous avez plusieurs GPU :

```yaml
# Dans docker-compose-gpu.yml
environment:
  # Utiliser les GPU 0 et 1
  - NVIDIA_VISIBLE_DEVICES=0,1
```

### Limiter l'utilisation GPU

```yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: 1  # Utiliser seulement 1 GPU
          capabilities: [gpu]
```

### Variables d'environnement Ollama

```yaml
environment:
  - OLLAMA_NUM_PARALLEL=2        # Nombre de requêtes parallèles
  - OLLAMA_MAX_LOADED_MODELS=2   # Modèles chargés en mémoire
  - OLLAMA_KEEP_ALIVE=5m         # Durée de maintien en mémoire
```

---

## 📚 Ressources Complémentaires

### Documentation Officielle

- **NVIDIA Container Toolkit** : https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/
- **Docker GPU Support** : https://docs.docker.com/compose/gpu-support/
- **Ollama Documentation** : https://github.com/ollama/ollama/blob/main/docs/gpu.md
- **CUDA Compatibility** : https://docs.nvidia.com/deploy/cuda-compatibility/

### Tutoriels Vidéo

- [NVIDIA Docker Setup](https://www.youtube.com/watch?v=9LQZE1XQVVY)
- [Ollama with GPU](https://www.youtube.com/results?search_query=ollama+gpu+docker)

### Compatibilité des Modèles

Liste des modèles compatibles avec GPU :
```bash
# Vérifier les modèles disponibles
docker exec ollama ollama list

# Modèles recommandés pour GPU
ollama pull mistral       # 7B params - Rapide
ollama pull llama2        # 7B params - Polyvalent
ollama pull codellama     # 7B params - Code
ollama pull llama-guard3  # Sécurité prompts
```

---

## ✅ Checklist de Validation

Avant de considérer l'installation terminée :

- [ ] `nvidia-smi` affiche votre GPU
- [ ] `docker run --rm --gpus all nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi` fonctionne
- [ ] `docker compose -f docker-compose-gpu.yml up -d` lance sans erreur
- [ ] `docker exec ollama nvidia-smi` affiche le GPU dans le conteneur
- [ ] `docker exec ollama ollama pull mistral` télécharge le modèle
- [ ] `docker exec ollama ollama run mistral "test"` utilise le GPU (vérifier avec `nvidia-smi`)
- [ ] Les performances sont nettement meilleures qu'avec CPU

---

## 🎯 Retour en Mode CPU

Si vous rencontrez des problèmes ou préférez revenir au mode CPU :

```bash
# Arrêter la version GPU
docker compose -f docker-compose-gpu.yml down

# Lancer la version standard (CPU)
docker compose up -d
```

Les modèles téléchargés restent disponibles car ils sont stockés dans le volume `ollama_data`.

---

**🚀 Votre configuration GPU est maintenant prête ! Profitez de performances jusqu'à 15x plus rapides avec vos modèles LLM locaux.**

Pour toute question ou problème, consultez :
- Le README.md principal
- Les issues GitHub d'Ollama : https://github.com/ollama/ollama/issues
- La communauté Discord de n8n
