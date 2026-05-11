# ✦ Constellation School — Guide d'installation complet

**Plateforme de gestion scolaire PWA — Cameroun**
**Fondateur : Patrick TALLA — Yaoundé, 2025**

---

## Structure du projet

```
constellation-school/
├── app.js                    ← Serveur Express.js principal
├── package.json              ← Dépendances Node.js
├── .env.example              ← Modèle de configuration (copier en .env)
├── config/
│   └── database.js           ← Pool de connexions MySQL
├── middleware/
│   └── auth.js               ← Vérification JWT + rôles
├── routes/
│   ├── auth.js               ← Connexion / déconnexion
│   ├── eleves.js             ← CRUD élèves + inscriptions
│   ├── classes.js            ← Gestion des classes
│   ├── notes.js              ← Saisie notes + calcul moyennes
│   ├── bulletins.js          ← Génération bulletins
│   ├── paiements.js          ← Paiements + reçus + Monetbil
│   ├── personnel.js          ← Enseignants + salaires
│   ├── emplois.js            ← Emplois du temps
│   ├── communication.js      ← WhatsApp + historique
│   └── dashboard.js          ← Statistiques tableau de bord
├── services/
│   └── whatsapp.js           ← Service d'envoi WhatsApp
├── src/
│   └── offline-db.js         ← IndexedDB — base locale hors ligne
└── public/
    ├── index.html            ← Page principale (SPA)
    ├── manifest.json         ← PWA — installable sur mobile/PC
    ├── service-worker.js     ← Mode hors ligne + sync
    ├── register-sw.js        ← Enregistrement SW + détection connexion
    ├── offline.html          ← Page affichée sans internet
    └── icons/
        ├── icon-72.png
        ├── icon-192.png
        └── icon-512.png
```

---

## Installation rapide (5 minutes)

### 1. Cloner le projet
```bash
git clone https://github.com/VOTRE_COMPTE/constellation-school.git
cd constellation-school
```

### 2. Installer les dépendances
```bash
npm install
```

### 3. Configurer l'environnement
```bash
cp .env.example .env
nano .env   # Remplir DB_HOST, DB_USER, DB_PASS, JWT_SECRET, etc.
```

### 4. Créer la base de données
```bash
mysql -u root -p -e "CREATE DATABASE constellation_school CHARACTER SET utf8mb4;"
mysql -u root -p constellation_school < Constellation_School_Base_de_Donnees.sql
```

### 5. Lancer le serveur
```bash
# Développement
npm run dev

# Production avec PM2
npm run pm2
```

L'application sera accessible sur : **http://localhost:3000**

---

## Déploiement en production (VPS)

### Prérequis serveur (Ubuntu 22.04)
```bash
apt update && apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs mysql-server nginx certbot python3-certbot-nginx
npm install -g pm2
```

### Configurer Nginx
```nginx
# /etc/nginx/sites-available/constellation
server {
    server_name constellation.school www.constellation.school;

    location / {
        proxy_pass         http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection 'upgrade';
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Real-IP $remote_addr;
    }

    # Pas de cache pour le service worker
    location /service-worker.js {
        proxy_pass http://localhost:3000;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # Cache long pour les assets statiques
    location /icons/ {
        proxy_pass http://localhost:3000;
        add_header Cache-Control "public, max-age=2592000, immutable";
    }
}
```

```bash
ln -s /etc/nginx/sites-available/constellation /etc/nginx/sites-enabled/
certbot --nginx -d constellation.school
nginx -t && systemctl reload nginx
```

### Lancer avec PM2
```bash
pm2 start app.js --name constellation-school
pm2 save
pm2 startup
```

---

## Variables d'environnement obligatoires

| Variable       | Description                            |
|----------------|----------------------------------------|
| DB_HOST        | Adresse MySQL (localhost)              |
| DB_NAME        | Nom de la base (constellation_school)  |
| DB_USER        | Utilisateur MySQL                      |
| DB_PASS        | Mot de passe MySQL                     |
| JWT_SECRET     | Clé secrète pour les tokens JWT        |
| MONETBIL_KEY   | Clé API Monetbil (paiements)           |
| WA_PHONE_ID    | ID du numéro WhatsApp Business         |
| WA_TOKEN       | Token d'accès WhatsApp API             |

---

## Comptes par défaut (à modifier impérativement)

| Rôle        | Email                           | Mot de passe |
|-------------|----------------------------------|--------------|
| Directeur   | directeur@constellation.school  | Changer!2025 |
| Enseignant  | enseignant@constellation.school | Changer!2025 |
| Comptable   | comptable@constellation.school  | Changer!2025 |

---

## Mode hors ligne (PWA)

La plateforme fonctionne **partiellement sans connexion internet** :

✅ **Disponible hors ligne :**
- Consultation des élèves (depuis le cache)
- Saisie des notes (stockées dans IndexedDB)
- Enregistrement des paiements (file d'attente)
- Consultation de l'emploi du temps
- Page de connexion

❌ **Nécessite internet :**
- Génération des bulletins PDF
- Envoi WhatsApp aux parents
- Paiements Orange Money / MTN MoMo en temps réel
- Nouvelles inscriptions

**Synchronisation automatique** dès le retour de la connexion internet.

---

## Support

- 📱 WhatsApp : +237 XXX XXX XXX
- 📧 Email : support@constellation.school
- 🕐 Lundi–Vendredi : 7h30–18h00

---

*✦ Constellation School — Ensemble, éclairons l'éducation camerounaise*
*Patrick TALLA — Yaoundé, Cameroun — © 2025*
