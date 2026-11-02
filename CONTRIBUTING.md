# Guide de Contribution — SecureDesk

Guide simple pour collaborer efficacement et en sécurité sur le projet **SecureDesk** (BTS SIO).

---

## 🌿 1. Organisation des Branches

Pour éviter d'écraser le travail des autres, **on ne travaille jamais directement sur la branche `main`**.

* **`main`** : version finale et stable du projet.
* **`dev`** : branche où l'on regroupe les fonctionnalités terminées.
* **Vos branches de travail** : créées à partir de `dev` pour chaque tâche.

### Commandes utiles :
```bash
# 1. Se mettre sur la branche dev et récupérer les nouveautés
git checkout dev
git pull origin dev

# 2. Créer sa branche pour sa tâche (ex: feat/connexion ou fix/affichage-tickets)
git checkout -b feat/nom-de-votre-tache

# 3. Travailler, enregistrer et envoyer sa branche
git add .
git commit -m "feat: ajout du formulaire de connexion"
git push -u origin feat/nom-de-votre-tache
```

---

## 💬 2. Format des Messages de Commit

Utilisez des messages courts et clairs avec un préfixe simple :

* `feat:` pour une nouvelle fonctionnalité (ex: `feat: page de création de ticket`)
* `fix:` pour corriger un bug (ex: `fix: problème de redirection après login`)
* `style:` pour du design / CSS (ex: `style: harmonisation des boutons`)
* `docs:` pour la documentation (ex: `docs: mise à jour du README`)

---

## 🛡️ 3. Règles de Sécurité Simples

Avant de pousser du code, vérifiez ces 4 règles obligatoires :

1. **Aucun mot de passe dans Git** : Ne commitez jamais vos fichiers `.env` ou vos mots de passe de base de données.
2. **Pas d'injections SQL** : Utilisez toujours des requêtes préparées (`prepare()` et `execute()`).
3. **Protection des uploads** : Ne laissez pas un utilisateur envoyer n'importe quel fichier (autoriser uniquement PDF, PNG, JPG, et stocker hors du dossier public).
4. **Respect du RGPD** : Ne collectez que le nécessaire (nom, prénom, email pro). Ne demandez jamais de numéro de sécurité sociale, adresse personnelle ou photo d'identité.

---

## 🚀 4. Proposer son Travail (Pull Request)

Une fois votre tâche terminée et testée :

1. Allez sur GitHub et ouvrez une **Pull Request** de votre branche vers **`dev`**.
2. Expliquez brièvement ce que vous avez fait et comment le tester.
3. Demandez à **un membre de l'équipe** de relire et tester votre code avant de fusionner (*merge*).
4. Une fois validé, fusionnez votre travail et supprimez votre branche de travail.
