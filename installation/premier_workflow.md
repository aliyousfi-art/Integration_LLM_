# Tutoriel : Créer un premier workflow "Hello World" avec n8n

Dans ce tutoriel, nous allons créer un workflow simple dans **n8n**, une plateforme d'automatisation puissante.
L'objectif est d'envoyer un message "Hello World" en utilisant un workflow comme introduction.

## Pré-requis

Avant de commencer, assurez-vous d'avoir les éléments suivants :

1. Une installation fonctionnelle de **n8n**. Vous pouvez soit :
    - Installer n8n localement sur votre
      machine ([documentation officielle](https://docs.n8n.io/hosting/installation/)).
    - Utiliser une version hébergée sur le cloud (par exemple via [n8n cloud](https://n8n.io/cloud)).
2. Un navigateur web.

---

## Étapes pour créer un workflow simple

### Étape 1 : Accéder à n8n

1. Lancez **n8n** depuis l'installation que vous avez déjà configurée :
    - Accédez à l'interface web de n8n dans votre navigateur via : `http://localhost:5678` (ou l'adresse et le port
      correspondant à votre installation).

2. Connectez-vous ou inscrivez-vous (si nécessaire).

---

### Étape 2 : Créer un nouveau workflow

1. Cliquez sur **"New Workflow"** depuis le tableau de bord de n8n.
2. Donnez un nom à votre workflow en cliquant sur "Untitled Workflow" (en haut à gauche) et entrez un nom comme
   `Hello World`.

---

### Étape 3 : Ajouter le premier nœud "Start"

1. Un nœud **Start** est automatiquement ajouté dans votre workflow. Ce nœud sert de point de départ au workflow.

---

### Étape 4 : Ajouter un nœud d'exécution JavaScript

1. Cliquez sur le bouton **"+"** sur votre interface.
2. Recherchez et sélectionnez le nœud **"Code"**, qui permet d'exécuter du code JavaScript simple.
3. Connectez ce nœud au nœud **Start** (en cliquant et en glissant pour les lier).

---

### Étape 5 : Ajouter un message "Hello World"

1. Dans le nœud **Code**, remplacez le code par défaut avec le contenu suivant pour afficher "Hello World" dans la
   console :

   ```javascript
   return {
       message: "Hello World"
   };
   ```

2. Cliquez sur le bouton **Execute Node** (en haut à droite du nœud) pour exécuter ce nœud et tester.

---

### Étape 6 : Ajouter un nœud d'affichage

1. Cliquez sur **"+"**, puis ajoutez un nœud **"Set"** (pour manipuler ou visualiser des données).
2. Connectez ce nœud au nœud `Code` précédent.
3. Configurez ce nœud pour récupérer et afficher la variable `message`, issue du nœud précédent.

---

### Étape 7 : Tester et finaliser

1. Cliquez sur **"Execute Workflow"** pour exécuter l'ensemble du workflow.
2. Vérifiez que le message "Hello World" apparaît dans l'interface.
3. Cliquez sur **"Save"** en haut à droite pour enregistrer votre travail.

---

## Conclusion

Félicitations 🎉 ! Vous avez créé votre premier workflow simple dans n8n. Ce tutoriel représente un point de départ pour
comprendre les bases. À partir de là, vous pouvez explorer des scénarios plus complexes comme l'automatisation de tâches
ou l'intégration d'API externes.

Pour davantage d'exemples et d'idées, consultez la [documentation officielle de n8n](https://docs.n8n.io/).