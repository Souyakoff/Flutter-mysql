### Rapport : Application Flutter avec connexion a une Base de Données MySQL distante

#### **Objectifs du TP**
1. Configurer un backend avec Node.js pour interagir avec une base de données MySQL situé sur une machine "distante" (VM).
2. Développer une application Flutter intégrée à ce backend via une API REST.
3. Afficher et manipuler dynamiquement des données (utilisateurs) dans l'application flutter.

---

#### **Étapes Réalisées**

##### **1. Backend avec Node.js et MySQL**
- **Création de l’API REST** :
  - Endpoint GET : Récupère la liste des utilisateurs avec leurs informations (nom, email, téléphone).
  - Endpoint POST : Permet d’ajouter un utilisateur (nom, email, téléphone).

  **Code du backend :**
  ```javascript
  // Endpoint GET pour récupérer les utilisateurs
  app.get('/api/utilisateurs', (req, res) => {
      connection.query('SELECT * FROM users', (err, results) => {
          if (err) throw err;
          res.json(results); // Renvoie les résultats de la base.
      });
  });

  // Endpoint POST pour ajouter un utilisateur
  app.post('/api/utilisateurs', (req, res) => {
      const { nom, email, telephone } = req.body;
      if (!nom || !email || !telephone) {
          return res.status(400).json({ message: 'Nom, email et téléphone sont requis.' });
      }
      connection.query(
          'INSERT INTO users (nom, email, telephone) VALUES (?, ?, ?)',
          [nom, email, telephone],
          (err, result) => {
              if (err) throw err;
              res.status(201).json({ id: result.insertId, nom, email, telephone });
          }
      );
  });
  ```

- **Configuration de la base de données** :
  - Création de la base `gestion_utilisateurs`.
  - Ajout d'une table `users` avec les colonnes `id`, `nom`, `email`, et `telephone`(la colonne "telephone" a été rajouté ultérieurement).

  **Commandes SQL :**
  ```sql
  CREATE DATABASE gestion_utilisateurs;

  USE gestion_utilisateurs;

  CREATE TABLE users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      nom VARCHAR(50),
      email VARCHAR(50),
      telephone VARCHAR(15)/*à été rajouté ultérieurement*/
  );

  INSERT INTO users (nom, email, telephone) VALUES
      ('Alice Dupont', 'alice@example.com', '0601234567'),
      ('Bob Martin', 'bob@example.com', '0612345678');
  ```

---

##### **2. Application Flutter**

- **Fichier `api_service.dart`** : Communication avec le backend.
  - **Récupération des utilisateurs** :
    ```dart
    // Fonction pour récupérer les utilisateurs depuis l'API
    Future<List<dynamic>> fetchUtilisateurs() async {
        // Envoi d'une requête GET à l'API pour récupérer la liste des utilisateurs
        final response = await http.get(Uri.parse('$baseUrl/utilisateurs'));

         // Vérification de la réponse HTTP
        if (response.statusCode == 200) {
            // Si la requête est réussie on retourne la réponse en format json
            return json.decode(response.body);
        } else {
            // Si la requête échoue, on a une erreur
            throw Exception('Erreur lors de la récupération des utilisateurs');
        }
    }
    ```
  - **Ajout d'un utilisateur** :
    ```dart
    // Fonction pour ajouter un utilisateur via l'API
    Future<void> addUtilisateur(String nom, String email, String telephone) async {
        final response = await http.post(
            Uri.parse('$baseUrl/utilisateurs'),
            headers: {'Content-Type': 'application/json'},
            body: json.encode({'nom': nom, 'email': email, 'telephone': telephone}),
        );
        if (response.statusCode != 201) {
            throw Exception('Erreur lors de l\'ajout de l\'utilisateur');
        }
    }
    ```

- **Fichier `main.dart`** : Interface utilisateur.
  - **Affichage des utilisateurs** :
    Utilisation de `FutureBuilder` pour récupérer et afficher la liste des utilisateurs sous forme de `ListView`.
  - **Formulaire pour ajouter un utilisateur** :
    Comprend trois champs : nom, email, et téléphone, avec validation pour s'assurer que les données sont renseignées.

---

Voici une version améliorée et plus détaillée du schéma d'architecture pour mieux représenter les interactions entre les différentes composantes de l'application :

#### **Schéma d’Architecture**
```plaintext
+----------------------------+              +-----------------------+
|                            |   HTTP API   |                       |
|        Flutter             | <----------> |      Node.js          |
|  (Interface Utilisateur)   |              | (Backend et API REST) |
|                            |              |                       |
+----------------------------+              +-----------------------+
             |                                    |
             |                                    | Connexion à la base
             |                                    | 
             v                                    v
+----------------------------+              +-----------------------+
|                            |              |                       |
|       MySQL                | <----------> |       Backend         |
|  (Base de Données)         |              |                       |
|                            |              |                       |
+----------------------------+              +-----------------------+
```
---

#### **Résultats**
- **Affichage dynamique des utilisateurs** :
  
![Alt text](images.png)
  - Liste des utilisateurs récupérée depuis le backend.
  - Mise à jour de la liste après l’ajout d’un nouvel utilisateur.
- **Ajout d’un utilisateur** :
  
![Alt text](image-1.PNG)
  - Utilisation d’un formulaire pour saisir les informations nécessaires.
  - Données envoyées au backend et ajoutées dans la base MySQL.

---

#### **Améliorations Apportées**
1. Ajout du champ `telephone` dans :
   - Le backend (`server.js`).
   - L’application Flutter (`main.dart`, `api_service.dart`).
   - La base de données MySQL.
2. Gestion des erreurs :
   - Validation des données dans l'API.
   - Gestion des erreurs HTTP côté Flutter.

---

#### **Évaluation**
- **Fonctionnalité des endpoints** : Les endpoints GET et POST fonctionnent parfaitement.
- **Interface utilisateur** :
  - Liste des utilisateurs affichée dynamiquement.
  - Formulaire simple et fonctionnel.
- **Gestion des erreurs** :
  - Messages d’erreur affichés pour les cas de requêtes incorrectes.
- **Rapport** : Documentation complète avec le code source et les étapes suivies.

---

#### **Conclusion**
Ce projet a permis de créer une application mobile Flutter connectée à un backend Node.js et une base de données MySQL. L'application est extensible et peut intégrer d'autres fonctionnalités comme la modification ou la suppression d'utilisateurs.
