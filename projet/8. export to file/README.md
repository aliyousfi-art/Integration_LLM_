# 8. Exporter les résultats dans différents formats

> **Résumé** : Exportez les comparaisons LLM en Markdown, CSV, JSON ou PDF pour partage et archivage  
> **Temps estimé** : 45-60 minutes  
> **Difficulté** : Intermédiaire ⭐⭐  
> **Étape précédente** : [7. Enhance Prompt](../7.%20enhance%20prompt/README.md)

---

## 🎯 Objectifs pédagogiques

À la fin de cette étape, vous serez capable de :

1. **Exporter des données** depuis n8n vers des fichiers (Markdown, CSV, JSON)
2. **Configurer des volumes Docker** pour accéder aux fichiers générés
3. **Formater automatiquement** les données selon le format cible
4. **Créer des rapports téléchargeables** via webhooks
5. **Gérer les noms de fichiers** avec timestamps et identifiants uniques

Cette étape rend vos analyses **portables** et **partageables** avec d'autres outils ou équipes.

---

## 📚 Prérequis

### Étapes précédentes requises

- ✅ **Étape 5 complétée** : Données stockées en base
- ✅ **Étape 6 complétée** : Chargement des données
- ✅ **Étape 7 complétée** (optionnel) : Rapports Markdown générés

### Connaissances requises

- 📊 **Formats de données (CSV, JSON, Markdown)**
  - 📖 Voir : [/ressources/formats_donnees/README.md](../../ressources/formats_donnees/README.md)
  
- 🐳 **Docker volumes et montage de fichiers**
  - 📖 Voir : [/ressources/docker/README.md](../../ressources/docker/README.md)

### Ressources externes

- [n8n - Read/Write Binary File](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.readwritefile/)
- [n8n - Convert to/from Binary](https://docs.n8n.io/data/data-structure/)

---

## 🧑‍💻 Instructions étape par étape

### Partie 1 : Configurer le volume Docker

#### Étape 1 : Créer le dossier de sortie

1. **Créez un dossier local** :
   ```bash
   mkdir -p ~/n8n-files/output
   ```

2. **Modifiez docker-compose.yml** :
   ```yaml
   services:
     n8n:
       volumes:
         - ./n8n-files:/files
   ```

3. **Redémarrez n8n** :
   ```bash
   docker compose down && docker compose up -d
   ```

---

### Partie 2 : Export Markdown

#### Étape 2 : Créer le workflow d'export

1. **Dupliquez le workflow de l'Étape 6**
2. **Ajoutez un nœud Code** pour formater en Markdown :
   ```javascript
   const questions = $input.all();
   
   let markdown = `# Rapport Comparateur LLM\n\n`;
   markdown += `Généré le : ${new Date().toLocaleString('fr-FR')}\n\n`;
   markdown += `Total de questions : ${questions.length}\n\n---\n\n`;
   
   for (const item of questions) {
     const q = item.json;
     markdown += `## Question #${q.id}\n\n`;
     markdown += `**Question** : ${q.question}\n\n`;
     markdown += `**Date** : ${new Date(q.date).toLocaleString('fr-FR')}\n\n`;
     
     if (q.responses && q.responses.length > 0) {
       markdown += `### Réponses\n\n`;
       for (const resp of q.responses) {
         markdown += `#### ${resp.provider}\n\n`;
         markdown += `${resp.reponse}\n\n`;
       }
     }
     markdown += `---\n\n`;
   }
   
   return [{
     json: {
       data: markdown,
       filename: `rapport_${Date.now()}.md`
     }
   }];
   ```

3. **Ajoutez "Write Binary File"** :
   - **File Name** : `={{ $json.filename }}`
   - **Input Binary Field** : Laissez vide
   - **Data** : `={{ $json.data }}`
   - **Options** → **File Path** : `/files/output/`

---

### Partie 3 : Export CSV

#### Étape 3 : Formater en CSV

1. **Ajoutez un nœud Code** :
   ```javascript
   const questions = $input.all();
   
   let csv = `"ID","Question","Date","Provider","Reponse"\n`;
   
   for (const item of questions) {
     const q = item.json;
     for (const resp of q.responses || []) {
       const date = new Date(q.date).toISOString();
       const reponse = resp.reponse.replace(/"/g, '""').substring(0, 200);
       csv += `"${q.id}","${q.question}","${date}","${resp.provider}","${reponse}"\n`;
     }
   }
   
   return [{
     json: {
       data: csv,
       filename: `export_${Date.now()}.csv`
     }
   }];
   ```

2. **Utilisez "Write Binary File"** comme précédemment

---

### Partie 4 : Export JSON

#### Étape 4 : Export JSON structuré

1. **Plus simple - le JSON est déjà formaté** :
   ```javascript
   const questions = $input.all().map(item => item.json);
   
   const jsonData = {
     generated_at: new Date().toISOString(),
     version: "1.0",
     total_questions: questions.length,
     questions: questions
   };
   
   return [{
     json: {
       data: JSON.stringify(jsonData, null, 2),
       filename: `export_${Date.now()}.json`
     }
   }];
   ```

---

### Partie 5 : Webhook de téléchargement

#### Étape 5 : Créer un endpoint de téléchargement

1. **Créez un nouveau workflow**
2. **Ajoutez un Webhook** :
   - **Path** : `download/:format`
   - **Method** : `GET`

3. **Ajoutez un Switch** basé sur le format :
   ```
   {{ $json.query.format }}
   ```

4. **Pour chaque branche (md, csv, json)** :
   - Chargez les données depuis PostgreSQL
   - Formatez selon le format
   - Retournez avec les bons headers :
     ```json
     {
       "Content-Type": "text/markdown",
       "Content-Disposition": "attachment; filename=export.md"
     }
     ```

---

## ✅ Critères de validation

### Checklist

- [ ] Le volume Docker est configuré
- [ ] Le dossier `/files/output/` est accessible
- [ ] Export Markdown fonctionne
- [ ] Export CSV fonctionne
- [ ] Export JSON fonctionne
- [ ] Les fichiers sont générés avec timestamps
- [ ] Workflow exporté dans `projet/8. export to file/`

---

### Quiz de compréhension

<details>
<summary><strong>Question 1 :</strong> Pourquoi utiliser des volumes Docker pour les fichiers ?</summary>

**Réponse :**

Sans volume, les fichiers créés **dans le conteneur** sont **perdus** quand le conteneur est supprimé.

**Volume Docker** :
- ✅ Persiste les données hors du conteneur
- ✅ Accessible depuis l'hôte
- ✅ Partageable entre conteneurs

**Configuration** :
```yaml
volumes:
  - ./n8n-files:/files
```
→ Le dossier local `./n8n-files` est monté dans `/files` du conteneur.

</details>

<details>
<summary><strong>Question 2 :</strong> Comment choisir entre CSV, JSON, et Markdown ?</summary>

**Réponse :**

| Format | Avantages | Cas d'usage |
|--------|-----------|-------------|
| **CSV** | Excel/Sheets, léger | Analyse statistique, tableaux |
| **JSON** | Structuré, programmable | APIs, applications |
| **Markdown** | Lisible, documentation | Rapports humains, GitHub |

**Recommandation** : Générez les 3 formats et laissez l'utilisateur choisir !

</details>

<details>
<summary><strong>Question 3 :</strong> Comment gérer les caractères spéciaux en CSV ?</summary>

**Réponse :**

**Problème** : Les guillemets (`"`) et virgules (`,`) cassent le CSV.

**Solution** :
- Échapper les guillemets : `"` → `""`
- Entourer les champs de guillemets : `"champ avec , virgule"`

```javascript
const escaped = text.replace(/"/g, '""');
csv += `"${escaped}"\n`;
```

</details>

---

## 🐛 Dépannage

### Problème : "Permission denied" lors de l'écriture

**Solution** : Vérifiez les permissions du dossier :
```bash
chmod -R 777 ~/n8n-files
```

---

### Problème : Le fichier n'apparaît pas

**Solution** : Vérifiez le chemin dans le conteneur :
```bash
docker exec -it n8n ls /files/output
```

---

## ➡️ Étape suivante

**[Étape 9 : Sécuriser les prompts](../9.%20secure%20prompt/README.md)**

Apprenez à prévenir les attaques de prompt injection.

---

**Félicitations !** Vos analyses sont maintenant exportables. 🎉
