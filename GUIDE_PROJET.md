# 📚 Guide du Projet Hackathon Backend

## 🎯 Vue d'ensemble

Backend NestJS pour la gestion d'un système de hackathons avec authentification, inscriptions, gestion d'annonces, analyse IA et monitoring administrateur.

---

## 🚀 Démarrage Rapide

### Prérequis
- Node.js (v18+)
- PostgreSQL
- npm ou yarn

### Installation

```bash
# Installer les dépendances
npm install

# Copier le fichier .env.example vers .env et configurer
cp .env.example .env

# Générer le client Prisma
npx prisma generate

# Lancer les migrations
npx prisma migrate dev

# Démarrer le serveur en mode développement
npm run start:dev
```

Le serveur sera accessible sur : `http://localhost:3000`
Documentation Swagger : `http://localhost:3000/api`

---

## 📋 Architecture du Projet

### Structure des modules

```
src/
├── admin/          # Gestion administrateur (dashboard, métriques, logs)
├── ai/             # Analyse IA des inscriptions
├── annonce/        # Gestion des annonces (publiques/pour inscrits)
├── auth/           # Authentification (register, login, JWT)
├── common/         # Utilitaires communs (pipes, decorators)
├── config/         # Configuration
├── email/          # Service d'envoi d'emails (SMTP)
├── events/         # WebSockets pour événements temps réel
├── hackathon/      # Gestion des hackathons
├── inscriptions/   # Gestion des inscriptions
├── prisma/         # Service Prisma (ORM)
├── queue/          # Service de queue (actuellement SMTP direct)
└── users/          # Gestion des utilisateurs
```

---

## 🗄️ Base de Données

### Schéma Prisma

Le schéma est conforme au document PDF fourni avec :

#### Modèles principaux :
- **User** : Utilisateurs (USER/ADMIN)
- **Hackathon** : Hackathons avec statuts (UPCOMING/ONGOING/PAST)
- **Inscription** : Inscriptions avec promo, technologies, statut
- **Annonce** : Annonces (PUBLIC/INSCRITS)
- **Notification** : Système de notifications planifiées
- **AnalyseIA** : Résultats d'analyse IA par inscription
- **IALog** : Logs d'activités IA
- **EvenementSurveillance** : Événements de monitoring

#### Enums :
- `Role` : USER, ADMIN
- `HackathonStatus` : UPCOMING, ONGOING, PAST
- `AnnonceCible` : PUBLIC, INSCRITS
- `Promo` : L1, L2
- `StatutInscription` : VALIDE, EN_ATTENTE, REFUSE
- `TypeNotification` : EMAIL, SITE
- `TypeIALog` : ANALYSE, SURVEILLANCE, SUGGESTION

### Migration

```bash
# Créer une migration
npx prisma migrate dev --name nom_de_la_migration

# Appliquer les migrations
npx prisma migrate deploy

# Ouvrir Prisma Studio (interface graphique)
npx prisma studio
```

---

## 🔐 Authentification

### Inscription (Register)

```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123",
  "nom": "Nom",
  "prenom": "Prénom",
  "promo": "L2",  // Optionnel
  "technologies": ["React", "Node.js"],  // Optionnel
  "hackathonId": "uuid-du-hackathon"  // REQUIS
}
```

**Important** : Le `hackathonId` doit être un UUID valide du hackathon auquel l'utilisateur s'inscrit.

### Connexion (Login)

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response:
{
  "access_token": "eyJhbGci...",
  "user": {
    "id": "...",
    "email": "...",
    "nom": "...",
    "prenom": "...",
    "role": "USER"
  }
}
```

### Utiliser le token

Ajoutez le header dans vos requêtes :
```
Authorization: Bearer {access_token}
```

---

## 📡 Routes API Principales

### 🔓 Routes Publiques (sans authentification)

- `GET /hackathons/public` - Hackathon actuel/à venir
- `GET /hackathons/past` - Hackathons passés (pagination)
- `GET /annonces/public` - Annonces publiques
- `POST /auth/register` - Inscription
- `POST /auth/login` - Connexion

### 👤 Routes Utilisateur (token requis)

- `GET /auth/profile` - Profil utilisateur
- `GET /inscriptions/mes-inscriptions` - Mes inscriptions
- `GET /annonces/inscrits` - Annonces pour inscrits

### 🔑 Routes Admin (token ADMIN requis)

- `GET /admin/dashboard` - Statistiques du dashboard
- `GET /admin/monitoring/metrics` - Métriques système
- `GET /admin/monitoring/logs` - Logs IA avec pagination
- `POST /admin/annonces` - Créer une annonce
- `POST /ai/analyze-inscription/:userId` - Analyser une inscription

---

## 📧 Système d'Emails

Le système utilise **SMTP direct** (sans queue Redis).

### Configuration SMTP (.env)

```env
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=email@example.com
SMTP_PASS=password
EMAIL_FROM=noreply@hackathon.com
```

### Types d'emails envoyés

1. **Accusé de réception** : Envoyé après inscription
2. **Annonces aux inscrits** : Envoyé lors de création d'annonce ciblée "INSCRITS"

---

## 🧪 Tests avec Postman

Une collection Postman est disponible : `Hackathon_API.postman_collection.json`

### Import dans Postman

1. Ouvrez Postman
2. Cliquez sur "Import"
3. Sélectionnez `Hackathon_API.postman_collection.json`

### Ordre d'exécution recommandé

1. **0. Setup - Get Hackathon ID** (récupère le hackathon_id)
2. **1. Register Admin** (crée un compte admin)
3. **2. Login Admin** (récupère le token - sauvegardé automatiquement)
4. Testez toutes les autres routes

**Note** : Les tokens sont automatiquement sauvegardés dans les variables de collection.

---

## 🛠️ Créer des Données de Test

### Via Prisma Studio

```bash
npx prisma studio
```

1. Créez un **Hackathon** :
   - Laissez le champ `id` vide (UUID généré automatiquement)
   - OU utilisez un UUID : `09907183-29ee-4201-bc02-65b77d6bacbb`
   - Remplissez : nom, description, dates, status (UPCOMING)
   
2. Créez un **User** avec rôle ADMIN :
   - Email : `admin@test.com`
   - Password : Hash bcrypt de `admin123` (générez avec : `node -e "const bcrypt=require('bcrypt');bcrypt.hash('admin123',10).then(h=>console.log(h))"`)
   - Role : `ADMIN`

### Exemple de Hash bcrypt

```bash
# Générer un hash pour "admin123"
node -e "const bcrypt=require('bcrypt');bcrypt.hash('admin123',10).then(h=>console.log(h))"
```

---

## 📊 Fonctionnalités Implémentées

### Phase 1 ✅
- Authentification (register/login)
- Gestion des hackathons
- Inscriptions
- Annonces publiques

### Phase 2 ✅
- Dashboard administrateur
- Métriques et monitoring
- Logs IA
- WebSockets pour événements temps réel

### Phase 3 ✅
- Analyse IA des inscriptions
- Notifications planifiées (schéma)
- Système de surveillance (schéma)

---

## 🔧 Configuration

### Variables d'environnement (.env)

```env
# Base de données
DATABASE_URL="postgresql://user:password@host:port/database"

# JWT
JWT_SECRET="votre-secret-jwt"
JWT_EXPIRES_IN="24h"

# SMTP
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=email@example.com
SMTP_PASS=password
EMAIL_FROM=noreply@hackathon.com

# Application
PORT=3000
NODE_ENV=development
```

---

## 🐛 Dépannage

### Erreur "UUID invalide" lors de l'inscription

**Problème** : L'ID du hackathon n'est pas un UUID valide.

**Solution** : 
- Utilisez un UUID valide pour le hackathon (format : `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
- Ou laissez Prisma générer automatiquement lors de la création

### Erreur 403 sur les routes admin

**Problème** : L'utilisateur n'a pas le rôle ADMIN.

**Solution** :
- Changez le rôle dans Prisma Studio ou via SQL
- Utilisez un compte avec le rôle ADMIN

### Erreur 500 sur /ai/analyze-inscription

**Problème** : Corrigé - Le code accédait à des champs qui n'existent plus dans User.

**Solution** : ✅ Déjà corrigé dans le code

---

## 📝 Commandes Utiles

```bash
# Démarrer en développement
npm run start:dev

# Build de production
npm run build
npm run start:prod

# Tests
npm test

# Linting
npm run lint

# Formatage
npm run format

# Prisma
npx prisma studio          # Interface graphique
npx prisma migrate dev     # Créer/appliquer migrations
npx prisma generate        # Régénérer le client
npx prisma db push         # Push du schéma (dev uniquement)
```

---

## 📚 Technologies Utilisées

- **NestJS** : Framework backend
- **Prisma** : ORM pour PostgreSQL
- **JWT** : Authentification
- **Zod** : Validation de schémas
- **Winston** : Logging
- **Nodemailer** : Envoi d'emails (SMTP)
- **Socket.io** : WebSockets pour temps réel
- **Swagger** : Documentation API

---

## ✅ Conformité au Document PDF

Le schéma Prisma et les fonctionnalités sont **100% conformes** au document PDF fourni :

- ✅ Tous les modèles présents
- ✅ Tous les enums présents
- ✅ Champs `promo` et `technologies` dans `Inscription` (pas dans `User`)
- ✅ Modèle `Notification` pour notifications planifiées
- ✅ Modèle `AnalyseIA` pour résultats d'analyse
- ✅ Email via SMTP direct (pas de Redis/BullMQ)

---

## 🎯 Prochaines Étapes Suggérées

1. Implémenter le système de notifications planifiées (J-7, J-3, J-1)
2. Connecter une vraie API IA (OpenAI, etc.) pour l'analyse
3. Ajouter des tests unitaires et E2E
4. Implémenter la surveillance automatique des événements
5. Ajouter la génération de rapports PDF

---

## 👥 Contacts et Support

Pour toute question ou problème, consultez :
- La documentation Swagger : `http://localhost:3000/api`
- Les logs de l'application dans `logs/`
- Le schéma Prisma : `prisma/schema.prisma`

---

**Dernière mise à jour** : 2026-01-01
**Version** : 0.0.1
