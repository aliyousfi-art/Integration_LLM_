# 🔄 Réimporter des Workflows

Ce guide explique comment réimporter des workflows sauvegardés dans n8n.

---

## 📥 Cas d'Usage

- Réinitialisation de n8n (après `docker compose down -v`)
- Migration vers une nouvelle machine
- Restauration après une erreur
- Partage de workflows entre utilisateurs
- Import de workflows d'exemple supplémentaires

---

## 🔁 Procédure de Réimportation

### Étape 1 : Préparer les fichiers

```bash
# Se placer dans le dossier installation
cd installation/

# Créer un dossier pour vos workflows (si non existant)
mkdir -p n8n_files/workflows

# Copier vos fichiers JSON dans ce dossier
# Les fichiers doivent être au format .json
```

**Structure attendue :**
```
n8n_files/
└── workflows/
    ├── workflow1.json
    ├── workflow2.json
    └── workflow3.json
```

### Étape 2 : Importer les workflows

```bash
# Importer tous les workflows du dossier
docker exec n8n n8n import:workflow --separate --input=/files/workflows/

# Pour un fichier spécifique
docker exec n8n n8n import:workflow --input=/files/workflows/workflow1.json
```

**Résultat attendu :**
```
✓ Successfully imported workflow: workflow1
✓ Successfully imported workflow: workflow2
✓ Successfully imported workflow: workflow3
```

### Étape 3 : Vérifier dans n8n

1. Ouvrir n8n : `http://localhost:5678`
2. Aller dans **Workflows**
3. Les workflows importés apparaissent dans la liste
4. Organiser dans des dossiers si nécessaire

---

## 📤 Exporter des Workflows

Pour sauvegarder vos workflows avant réimport :

### Méthode 1 : Interface n8n (Recommandée)

1. Ouvrir le workflow dans n8n
2. Cliquer sur le menu `⋮` (en haut à droite)
3. Cliquer sur **Download**
4. Le fichier JSON est téléchargé
5. Déplacer le fichier dans `n8n_files/workflows/`

### Méthode 2 : Export en masse

```bash
# Exporter tous les workflows
docker exec n8n n8n export:workflow --all --output=/files/backups/

# Les fichiers sont créés dans n8n_files/backups/
```

---

## 🔄 Workflow Complet : Sauvegarde et Restauration

### Sauvegarde

```bash
# 1. Créer un dossier de backup avec la date
BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "n8n_files/backups/$BACKUP_DATE"

# 2. Exporter tous les workflows
docker exec n8n n8n export:workflow --all --output=/files/backups/$BACKUP_DATE/

# 3. Sauvegarder aussi la base de données
docker exec postgres pg_dump -U n8n_user n8n > "n8n_files/backups/$BACKUP_DATE/database.sql"

# 4. Créer une archive
tar -czf "backup_n8n_$BACKUP_DATE.tar.gz" -C n8n_files/backups "$BACKUP_DATE"

echo "✓ Backup créé : backup_n8n_$BACKUP_DATE.tar.gz"
```

### Restauration

```bash
# 1. Extraire l'archive
tar -xzf backup_n8n_20250109_143000.tar.gz -C n8n_files/backups/

# 2. Restaurer les workflows
docker exec n8n n8n import:workflow --separate --input=/files/backups/20250109_143000/

# 3. Restaurer la base de données (optionnel)
docker exec -i postgres psql -U n8n_user n8n < n8n_files/backups/20250109_143000/database.sql

echo "✓ Restauration terminée"
```

---

## 🛠️ Commandes Utiles

### Lister les workflows existants

```bash
# Dans n8n
docker exec n8n n8n list:workflow
```

### Supprimer un workflow

```bash
# Par ID
docker exec n8n n8n delete:workflow --id=<workflow_id>
```

### Valider un fichier JSON avant import

```bash
# Vérifier que le JSON est valide
cat n8n_files/workflows/workflow1.json | jq .

# Si jq n'est pas installé
cat n8n_files/workflows/workflow1.json | python -m json.tool
```

---

## ⚠️ Précautions

1. **Vérifier les credentials** : Les credentials ne sont pas exportés avec les workflows pour des raisons de sécurité. Vous devrez les reconfigurer après import.

2. **Compatibilité des versions** : Assurez-vous que la version de n8n est compatible avec les workflows exportés.

3. **IDs de workflows** : Les IDs des workflows peuvent changer après réimport.

4. **Dépendances** : Si un workflow appelle un autre (subworkflow), assurez-vous que tous sont importés.

---

## 📋 Checklist de Migration Complète

- [ ] Sauvegarder tous les workflows
- [ ] Exporter la configuration des credentials (noter les valeurs)
- [ ] Sauvegarder la base de données PostgreSQL
- [ ] Sauvegarder les fichiers dans `n8n_files/`
- [ ] Tester la restauration sur un nouvel environnement
- [ ] Vérifier que tous les workflows fonctionnent
- [ ] Reconfigurer les credentials
- [ ] Tester les executions

---

## 🔍 Dépannage

### Erreur : "Workflow already exists"

```bash
# Renommer ou supprimer le workflow existant dans n8n
# OU importer avec un nouvel ID (automatique avec --separate)
```

### Erreur : "Invalid JSON format"

```bash
# Valider le JSON
cat workflow.json | jq .

# Si erreur, le fichier est corrompu
# Réexporter depuis n8n ou éditer manuellement
```

### Les workflows importés ne fonctionnent pas

1. Vérifier les credentials (menu Settings → Credentials)
2. Vérifier que tous les subworkflows sont importés
3. Vérifier les chemins de fichiers dans les nœuds
4. Tester chaque nœud individuellement

---

## 📚 Ressources

- **Documentation n8n sur l'import/export** : https://docs.n8n.io/workflows/share/
- **CLI n8n** : https://docs.n8n.io/hosting/cli-commands/
- **Sauvegarde et restauration** : https://docs.n8n.io/hosting/configuration/

---

**💡 Astuce** : Créez un script de backup automatique et planifiez-le avec cron pour sauvegarder régulièrement vos workflows.
