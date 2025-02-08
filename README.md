# API Restful avec Express JS et SQLite3 🌐

## OBJECTIF(S) 🎯
- Création d’une API Rest avec **Express JS** 🚀
- Utilisation des bonnes pratiques pour les API Restful 🔧

## OUTILS UTILISÉS 🛠️
- **Node.js** ⚙️
- **Express.js** 🌟
- **SQLite3** 💾

### Utilisation du framework **Express JS** 📦
Express est un framework d'application Web **Node.js** minimal et flexible qui fournit un ensemble robuste de fonctionnalités pour les applications Web et mobiles.

---

## Étapes de Création 📝

### Étape 1 : Initialisation du Projet 🏗️
1. Créez un nouveau dossier pour votre projet.
2. Ouvrez un terminal et naviguez vers ce dossier.
3. Initialisez un nouveau projet **Node.js** avec :
    ```bash
    npm init -y
    ```
4. Installez **Express.js** et **SQLite3** avec :
    ```bash
    npm install express sqlite3
    ```

### Étape 2 : Configuration de **SQLite3** 🗃️
1. Créez un nouveau fichier `database.js`.
2. Dans `database.js`, configurez **SQLite3** pour se connecter à une base de données :
    ```javascript
    const sqlite3 = require('sqlite3').verbose();

    // Connexion à la base de données SQLite
    const db = new sqlite3.Database('./maBaseDeDonnees.sqlite', sqlite3.OPEN_READWRITE | sqlite3.OPEN_CREATE, (err) => {
        if (err) {
            console.error(err.message);
        } else {
            console.log('Connecté à la base de données SQLite.');
            db.run(`CREATE TABLE IF NOT EXISTS personnes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nom TEXT NOT NULL
            )`, (err) => {
                if (err) {
                    console.error(err.message);
                } else {
                    // Insertion de données initiales
                    const personnes = ['Bob', 'Alice', 'Charlie'];
                    personnes.forEach((nom) => {
                        db.run(`INSERT INTO personnes (nom) VALUES (?)`, [nom]);
                    });
                }
            });
        }
    });

    module.exports = db;
    ```

### Étape 3 : Mise en Place de l'API 🖥️
1. Créez un fichier `index.js`.
2. Dans `index.js`, importez **Express** et configurez les routes suivantes :

    ```javascript
    const express = require('express');
    const db = require('./database');
    const app = express();
    app.use(express.json());
    const PORT = 3000;

    // Route de base
    app.get('/', (req, res) => {
        res.json("Registre de personnes! Choisissez le bon routage! 🔄")
    })

    // Récupérer toutes les personnes
    app.get('/personnes', (req, res) => {
        db.all("SELECT * FROM personnes", [], (err, rows) => {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({ "message": "success ✅", "data": rows });
        });
    });

    // Récupérer une personne par ID
    app.get('/personnes/:id', (req, res) => {
        const id = req.params.id;
        db.get("SELECT * FROM personnes WHERE id = ?", [id], (err, row) => {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({ "message": "success ✅", "data": row });
        });
    });

    // Créer une nouvelle personne
    app.post('/personnes', (req, res) => {
        const nom = req.body.nom;
        db.run(`INSERT INTO personnes (nom) VALUES (?)`, [nom], function(err) {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({
                "message": "success ✅",
                "data": { id: this.lastID }
            });
        });
    });

    // Mettre à jour une personne
    app.put('/personnes/:id', (req, res) => {
        const id = req.params.id;
        const nom = req.body.nom;
        db.run(`UPDATE personnes SET nom = ? WHERE id = ?`, [nom, id], function(err) {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({ "message": "success ✅" });
        });
    });

    // Supprimer une personne
    app.delete('/personnes/:id', (req, res) => {
        const id = req.params.id;
        db.run(`DELETE FROM personnes WHERE id = ?`, id, function(err) {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({ "message": "success ✅" });
        });
    });

    app.listen(PORT, () => { console.log(`Server running on port ${PORT} 🏃‍♂️`); });
    ```

### Étape 4 : Modification de la Structure de la Base de Données 🛠️
1. Ajoutez une colonne `adresse` à votre table `personnes`.
2. Mettez à jour le fichier `database.js` pour inclure cette modification :
    ```javascript
    db.run(`CREATE TABLE IF NOT EXISTS personnes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nom TEXT NOT NULL,
        adresse TEXT
    )`, (err) => {
        if (err) {
            console.error(err.message);
        } else {
            // Insertion de données initiales avec adresses
        }
    });
    ```

3. Modifiez les routes POST et PUT pour accepter et traiter l'adresse :
    ```javascript
    app.post('/personnes', (req, res) => {
        const { nom, adresse } = req.body;
        db.run(`INSERT INTO personnes (nom, adresse) VALUES (?, ?)`, [nom, adresse], function(err) {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({
                "message": "success ✅",
                "data": { id: this.lastID }
            });
        });
    });
    ```

### Étape 5 : Test avec **Postman** 🧪
1. **Installation et Configuration de Postman** : Téléchargez et installez **Postman**, puis configurez une nouvelle collection pour votre API.
2. **Test des Routes** : Testez chaque route avec les bonnes méthodes (GET, POST, PUT, DELETE) en utilisant **Postman**.
3. **Exécution et Validation des Tests** : Exécutez les tests pour vérifier les réponses et la gestion des erreurs.

### Étape 6 : (Optionnelle) **OAuth 2.0 avec Keycloak** 🔑
1. Installez `keycloak-connect` :
    ```bash
    npm install keycloak-connect
    ```

2. Créez un fichier de configuration `keycloak-config.json`.

3. Modifiez `index.js` pour intégrer **Keycloak** et protéger certaines routes :
    ```javascript
    const session = require('express-session');
    const Keycloak = require('keycloak-connect');
    const memoryStore = new session.MemoryStore();
    app.use(session({
        secret: 'api-secret',
        resave: false,
        saveUninitialized: true,
        store: memoryStore
    }));

    const keycloak = new Keycloak({ store: memoryStore }, './keycloak-config.json');
    app.use(keycloak.middleware());

    app.get('/secure', keycloak.protect(), (req, res) => {
        res.json({ message: 'Vous êtes authentifié ! ✅' });
    });

    app.get('/personnes', keycloak.protect(), (req, res) => {
        db.all("SELECT * FROM personnes", [], (err, rows) => {
            if (err) {
                res.status(400).json({ "error": err.message });
                return;
            }
            res.json({ "message": "success ✅", "data": rows });
        });
    });
    ```

---

## Conclusion 🎉
Cette API permet de gérer des personnes avec une base de données **SQLite3**. On a créé des routes **RESTful** pour les opérations de base : récupérer, ajouter, mettre à jour et supprimer des données. Optionnellement, On peut ajouter une authentification **OAuth** avec **Keycloak** pour sécuriser nos routes 🔒.
