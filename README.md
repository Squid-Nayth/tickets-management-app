# SecureDesk — Application de Gestion de Tickets Sécurisée

[![Sécurité](https://img.shields.io/badge/Security-By_Design-blue?style=flat-square&logo=shield)](https://owasp.org)
[![Confidentialité](https://img.shields.io/badge/Privacy-RGPD_Compliant-green?style=flat-square&logo=eff)](https://www.cnil.fr)
[![Architecture](https://img.shields.io/badge/Architecture-V1_S%C3%A9curis%C3%A9e-orange?style=flat-square)](https://github.com)
[![Statut](https://img.shields.io/badge/Statut-En_D%C3%A9veloppement-success?style=flat-square)](https://github.com)

**SecureDesk** est une plateforme web d'assistance technique interne développée pour les collaborateurs de **NOVATECH SERVICES** (180 salariés). Elle centralise les déclarations d'incidents, optimise le travail des techniciens de support et intègre dès sa conception les exigences de **cybersécurité** (*Security by Design*) et de **protection de la vie privée** (*Privacy by Design*).

---

## 📑 Sommaire

1. [Présentation & Périmètre Métier](#-présentation--périmètre-métier)
2. [Rôles & Habilitations (RBAC)](#-rôles--habilitations-rbac)
3. [Cycle de Vie d'un Incident](#-cycle-de-vie-dun-incident)
4. [Aperçu de l'Interface](#-aperçu-de-linterface)
5. [Architecture Technique Cible (V1)](#-architecture-technique-cible-v1)
6. [Exigences Techniques & Sécurité](#-exigences-techniques--sécurité)
7. [Politique des Données & RGPD](#-politique-des-données--rgpd)
8. [Feuille de Route & Issues GitHub](#-feuille-de-route--issues-github)
9. [Arborescence du Dépôt](#-arborescence-du-dépôt)

---

## 🏢 Présentation & Périmètre Métier

L'application remplace les canaux informels (téléphone, e-mails directs, messageries instantanées) pour apporter :
* **Centralisation & Traçabilité** : Aucun incident oublié, historique horodaté et assignation univoque.
* **Ségrégation des Accès** : Cloisonnement strict entre demandeurs, techniciens et administrateurs.
* **Sécurité des Échanges** : Chiffrement systématique, contrôle strict des pièces jointes et protection contre les attaques web courantes.

---

## 👥 Rôles & Habilitations (RBAC)

| Rôle | Périmètre et Droits applicatifs |
| :--- | :--- |
| **Utilisateur (Salarié)** | Création d'un ticket, choix de catégorie, description du besoin, ajout de pièces jointes autorisées, consultation de ses propres demandes et ajout de réponses/commentaires. |
| **Technicien Support** | Visualisation de la file des tickets assignés, traitement des incidents, changement d'état, rédaction de notes techniques internes, clôture des demandes. |
| **Administrateur Applicatif** | Gestion des comptes (activation/désactivation), attribution des rôles, paramétrage des catégories de tickets, consultation des journaux d'audit applicatifs. |
| **Équipe DevOps / Dev** | Maintenance du code source, gestion de la CI/CD et déploiements sur environnements cloisonnés (sans accès direct aux données de production). |

---

## 🔄 Cycle de Vie d'un Incident

```mermaid
stateDiagram-v2
    [*] --> Nouveau : Création par le salarié
    Nouveau --> Attribué : Assignation technicien
    Attribué --> En_cours : Début d'investigation
    En_cours --> En_attente : Attente d'élément utilisateur / tiers
    En_attente --> En_cours : Précisions reçues
    En_cours --> Résolu : Solution validée
    Résolu --> Fermé : Clôture définitive
    Résolu --> En_cours : Réouverture si anomalie persistante
    Fermé --> [*]
```

---

## 🖥️ Aperçu de l'Interface

| Module | Aperçu |
| :--- | :--- |
| **Tableau de Bord**<br>Métriques clés, tickets en cours et répartition des urgences | ![Dashboard](public/images/dashboard.png) |
| **Suivi des Demandes**<br>Filtres par statut, catégorie, priorité et assignation | ![Liste des tickets](public/images/all%20titkets%20page.png) |
| **Traitement d'un Incident**<br>Échanges avec le demandeur, historique et notes d'intervention | ![Édition de ticket](public/images/edit%20ticket%20page.png) |
| **Gestion des Utilisateurs**<br>Annuaire interne et contrôle des rôles d'accès | ![Gestion des utilisateurs](public/images/users%20page.png) |

---

## 🏗️ Architecture Technique Cible (V1)

Pour corriger les vulnérabilités de l'architecture initiale (HTTP en clair, root MySQL, uploads dans le webroot, dev/prod mutualisés), l'architecture cible V1 est isolée en zones de confiance :

```mermaid
flowchart TD
    subgraph CLIENTS["Accès Utilisateurs"]
        UserRemote["Salarié en Télétravail"]
        UserLAN["Salarié / Technicien sur site"]
    end

    subgraph DMZ["DMZ / Entrée Sécurisée"]
        Proxy["Reverse Proxy Nginx / WAF\n(Terminaison HTTPS TLS 1.3 / Port 443)"]
    end

    subgraph APP["Zone Applicative Interne"]
        Backend["Application SecureDesk\n(API REST / Utilisateur non-privilégié)"]
        Storage["Stockage Uploads Sécurisé\n(/var/storage/securedesk/uploads/)\n(HORS Webroot - Noms UUID - No-Exec)"]
    end

    subgraph DATA["Zone Données (VLAN Privé)"]
        DB["SGBD MySQL Dédié\n(Compte applicatif aux droits minimaux)"]
        Logs["Serveur de Logs Centralisé\n(Append-only / Immuable)"]
        Backups["Sauvegardes Déportées Chiffrées\n(Règle 3-2-1)"]
    end

    UserRemote -- "HTTPS (TLS 1.3)" --> Proxy
    UserLAN -- "HTTPS (TLS 1.3)" --> Proxy
    Proxy -- "Reverse Proxy interne" --> Backend

    Backend -- "Requêtes préparées (PDO)" --> DB
    Backend -- "Upload contrôlé & scan" --> Storage
    Backend -- "Journaux d'audit" --> Logs
    DB -. "Dumps chiffrés AES-256" .-> Backups
```

---

## 🛡️ Exigences Techniques & Sécurité

Toute contribution au projet doit obligatoirement respecter les règles suivantes :

1. **Contrôle d'accès côté serveur (Zero Trust)** :
   * Chaque endpoint d'API ou action vérifie le jeton de session et les permissions associées du demandeur (prévention des IDOR et élévations de privilèges).
2. **Requêtes préparées systématiques** :
   * Utilisation exclusive de requêtes SQL préparées (PDO / ORM sécurisé). Interdiction absolue de la concaténation de variables dans les requêtes.
3. **Téléversement sécurisé étanche** :
   * Stockage des fichiers **hors de la racine web publique**.
   * Renommage systématique des fichiers via un UUID aléatoire.
   * Vérification du type MIME côté serveur (inspection des *magic bytes*) et interdiction des extensions exécutables (`.php`, `.sh`, `.exe`, `.svg`, etc.).
   * Téléchargement uniquement via un contrôleur tampon sécurisé.
4. **Chiffrement & Sécurité des données** :
   * HTTPS obligatoire (TLS 1.3).
   * Hachage des mots de passe en **Argon2id** ou **bcrypt** avec sel unique.
   * Clés d'API et identifiants de BDD chargés via variables d'environnement (`.env`), jamais versionnés dans Git.
5. **Supervision & Imputabilité** :
   * Journalisation des actions sensibles (connexions, modifications de rôles, suppressions) sur un serveur de logs dédié, sans stockage des mots de passe en clair.

---

## 🔒 Politique des Données & RGPD

Conformément au principe de **minimisation des données** (Art. 5 RGPD) :

* **Données strictement autorisées :** Nom, prénom, adresse e-mail professionnelle, service/département, rôle applicatif et détails techniques liés aux incidents.
* **Données exclues du périmètre applicatif :** Numéro de sécurité sociale (NIR), pièces d'identité, coordonnées personnelles (téléphone perso, adresse privée) et géolocalisation GPS.
* **Politique de rétention :**
  * Tickets fermés : Anonymisation après 12 mois.
  * Pièces jointes : Purge automatique après 3 mois suivant la clôture.
  * Journaux de connexion : Rétention maximale de 6 mois.
  * Salariés sortants : Révocation immédiate et suppression après 30 jours.

---

## 📋 Feuille de Route & Issues GitHub

Le développement du projet est découpé en lots de travail configurables en **Issues GitHub** :

* **Infra & Sécurité :** Déploiement du reverse proxy Nginx HTTPS, isolation des environnements Dev/Staging/Prod, gestionnaire de secrets.
* **Backend & API :** Système d'authentification sécurisé, contrôleur RBAC, module de gestion des tickets, contrôleur d'upload étanche.
* **Frontend & UX :** Intégration du tableau de bord, interface de gestion des tickets et formulaires d'administration.
* **Conformité & RGPD :** Mise en place des scripts CRON d'anonymisation et de purge automatique des pièces jointes.

Pour les consignes de développement, la stratégie de branches et le format des commits, consultez le [Guide de Contribution (CONTRIBUTING.md)](CONTRIBUTING.md).

---

## 📂 Arborescence du Dépôt

```text
tickets-management-app/
├── README.md                           # Documentation générale et spécifications du projet
├── CONTRIBUTING.md                     # Guide de contribution, conventions Git et règles de sécurité
├── public/
│   ├── docs/
│   │   └── SecureDesk_Lot_1.docx       # Cahier des charges de référence (Lot 1)
│   └── images/
│       ├── dashboard.png               # Capture écran du tableau de bord
│       ├── all titkets page.png        # Capture écran du listing des tickets
│       ├── edit ticket page.png        # Capture écran du détail / édition d'un ticket
│       └── users page.png              # Capture écran du module utilisateurs
```
