📘 Livrable Développeur Web - Sionohmair Insight v2.0 (Suite)
6. Base de données
6.1 MongoDB - Schémas des collections
Collection users
{
  "_id": ObjectId("6574abc..."),
  "name": "Bruno Coldold",
  "email": "bruno@example.com",
  "passwordHash": "$2a$10$...",
  "language": "fr",
  "preferences": {
    "notifications": true,
    "darkMode": false,
    "audioVolume": 70,
    "language": "fr"
  },
  "profile": {
    "avatar": "https://...",
    "bio": "Développeur passionné...",
    "birthDate": ISODate("1990-01-15"),
    "gender": "male"
  },
  "devices": [
    {
      "type": "muse",
      "deviceId": "MUSE-A1B2",
      "name": "Mon Muse 2",
      "lastConnected": ISODate("2024-11-25T10:30:00Z"),
      "addedAt": ISODate("2024-11-01T08:00:00Z")
    }
  ],
  "subscription": {
    "plan": "premium",
    "startDate": ISODate("2024-11-01"),
    "endDate": ISODate("2025-11-01"),
    "autoRenew": true
  },
  "stats": {
    "totalSessions": 45,
    "totalDuration": 13500,
    "lastSessionDate": ISODate("2024-11-25")
  },
  "lastLogin": ISODate("2024-11-25T10:00:00Z"),
  "createdAt": ISODate("2024-11-01T08:00:00Z"),
  "updatedAt": ISODate("2024-11-25T10:00:00Z"),
  "isActive": true,
  "isVerified": true
}
Collection eegsessions
{
  "_id": ObjectId("6574def..."),
  "userId": ObjectId("6574abc..."),
  "deviceType": "muse",
  "deviceId": "MUSE-A1B2",
  "startTime": ISODate("2024-11-25T10:30:00Z"),
  "endTime": ISODate("2024-11-25T10:45:00Z"),
  "duration": 900, // secondes
  "status": "completed",
  "data": [
    {
      "timestamp": ISODate("2024-11-25T10:30:00.123Z"),
      "channels": {
        "TP9": 850.5,
        "AF7": 820.3,
        "AF8": 815.7,
        "TP10": 840.2
      },
      "quality": "good",
      "battery": 85
    },
    // ... milliers de points
  ],
  "statistics": {
    "totalSamples": 9000,
    "averageQuality": "good",
    "channels": {
      "TP9": {
        "mean": 845.2,
        "min": 800.0,
        "max": 900.5,
        "samples": 9000
      },
      // ... autres canaux
    }
  },
  "analysis": {
    "brainwaves": {
      "delta": 15.5,   // 0.5-4 Hz
      "theta": 25.3,   // 4-8 Hz
      "alpha": 45.7,   // 8-13 Hz
      "beta": 12.1,    // 13-30 Hz
      "gamma": 1.4     // 30-100 Hz
    },
    "meditation": 72,
    "focus": 68,
    "relaxation": 80,
    "stress": 25,
    "insights": [
      "État de relaxation profonde détecté",
      "Excellent niveau d'ondes alpha"
    ],
    "performedAt": ISODate("2024-11-25T10:45:30Z")
  },
  "notes": "Session de méditation matinale",
  "tags": ["meditation", "morning"],
  "environment": {
    "location": "Bureau",
    "ambient": "quiet",
    "activity": "meditation"
  },
  "audioSync": {
    "enabled": true,
    "audioSessionId": ObjectId("6574xyz...")
  },
  "createdAt": ISODate("2024-11-25T10:30:00Z"),
  "updatedAt": ISODate("2024-11-25T10:45:30Z")
}
6.2 PostgreSQL - Tables
Table users_metadata
CREATE TABLE users_metadata (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(24) NOT NULL UNIQUE,  -- MongoDB ObjectId
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP,
  login_count INTEGER DEFAULT 0,
  ip_address INET,
  user_agent TEXT,
  
  -- Index
  INDEX idx_user_id (user_id),
  INDEX idx_email (email),
  INDEX idx_created_at (created_at)
);
Table sessions_metadata
CREATE TABLE sessions_metadata (
  id SERIAL PRIMARY KEY,
  session_id VARCHAR(24) NOT NULL UNIQUE,  -- MongoDB ObjectId
  user_id VARCHAR(24) NOT NULL,
  session_type VARCHAR(20) NOT NULL,  -- 'eeg', 'audio', 'combined'
  start_time TIMESTAMP NOT NULL,
  end_time TIMESTAMP,
  duration INTEGER,  -- en secondes
  data_points INTEGER DEFAULT 0,
  device_type VARCHAR(50),
  
  -- Analyse rapide
  avg_quality VARCHAR(20),
  meditation_score INTEGER,
  focus_score INTEGER,
  
  -- Métadonnées
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Index
  INDEX idx_user_sessions (user_id, start_time DESC),
  INDEX idx_session_id (session_id)
);
Table analytics_events
CREATE TABLE analytics_events (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(24),
  event_type VARCHAR(50) NOT NULL,  -- 'session_start', 'device_connect', etc.
  event_data JSONB,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_user_events (user_id, timestamp DESC),
  INDEX idx_event_type (event_type)
);
6.3 Scripts d'initialisation
MongoDB init script
// backend/scripts/init-mongodb.js
const mongoose = require('mongoose');
require('dotenv').config();

async function initMongoDB() {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log('✓ Connecté à MongoDB');

    // Créer les index
    const db = mongoose.connection.db;

    // Index pour users
    await db.collection('users').createIndex({ email: 1 }, { unique: true });
    await db.collection('users').createIndex({ createdAt: -1 });
    
    // Index pour eegsessions
    await db.collection('eegsessions').createIndex({ userId: 1, startTime: -1 });
    await db.collection('eegsessions').createIndex({ status: 1 });
    
    console.log('✓ Index créés');

    // Créer utilisateur de test (optionnel)
    const bcrypt = require('bcryptjs');
    const User = require('../models/User');
    
    const testUser = await User.findOne({ email: 'test@sionohmair.com' });
    if (!testUser) {
      const passwordHash = await bcrypt.hash('Test1234!', 10);
      await User.create({
        name: 'Test User',
        email: 'test@sionohmair.com',
        passwordHash,
        language: 'fr'
      });
      console.log('✓ Utilisateur de test créé');
    }

    await mongoose.connection.close();
    console.log('✓ Initialisation terminée');

  } catch (error) {
    console.error('✗ Erreur:', error);
    process.exit(1);
  }
}

initMongoDB();
PostgreSQL init script
-- backend/scripts/init-postgres.sql

-- Créer la base de données
CREATE DATABASE sionohmair_db;

\c sionohmair_db;

-- Table users_metadata
CREATE TABLE users_metadata (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(24) NOT NULL UNIQUE,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP,
  login_count INTEGER DEFAULT 0,
  ip_address INET,
  user_agent TEXT
);

CREATE INDEX idx_user_id ON users_metadata(user_id);
CREATE INDEX idx_email ON users_metadata(email);
CREATE INDEX idx_created_at ON users_metadata(created_at);

-- Table sessions_metadata
CREATE TABLE sessions_metadata (
  id SERIAL PRIMARY KEY,
  session_id VARCHAR(24) NOT NULL UNIQUE,
  user_id VARCHAR(24) NOT NULL,
  session_type VARCHAR(20) NOT NULL,
  start_time TIMESTAMP NOT NULL,
  end_time TIMESTAMP,
  duration INTEGER,
  data_points INTEGER DEFAULT 0,
  device_type VARCHAR(50),
  avg_quality VARCHAR(20),
  meditation_score INTEGER,
  focus_score INTEGER,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_sessions ON sessions_metadata(user_id, start_time DESC);
CREATE INDEX idx_session_id ON sessions_metadata(session_id);

-- Table analytics_events
CREATE TABLE analytics_events (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(24),
  event_type VARCHAR(50) NOT NULL,
  event_data JSONB,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_events ON analytics_events(user_id, timestamp DESC);
CREATE INDEX idx_event_type ON analytics_events(event_type);

-- Fonction pour mettre à jour updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger sur users_metadata
CREATE TRIGGER update_users_metadata_updated_at
  BEFORE UPDATE ON users_metadata
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- Utilisateur de test
INSERT INTO users_metadata (user_id, email, login_count)
VALUES ('test_user_id_123', 'test@sionohmair.com', 0);
7. Intégration Bluetooth/Capteurs
7.1 Dispositifs supportés
Dispositif
Type
Canaux
Fréquence
SDK
Muse 2
EEG
4 (TP9, AF7, AF8, TP10)
256 Hz
Muse SDK
Muse S
EEG
4
256 Hz
Muse SDK
Emotiv Insight
EEG
5
128 Hz
Emotiv SDK
OpenBCI
EEG
8-16
250 Hz
OpenBCI
NeuroSky
EEG
1
512 Hz
ThinkGear
7.2 UUIDs Bluetooth (Muse)
// frontend/src/utils/bleConstants.js

export const MUSE_SERVICE_UUID = '0000fe8d-0000-1000-8000-00805f9b34fb';

export const MUSE_CHARACTERISTICS = {
  // Canaux EEG
  TP9: '273e0003-4c4d-454d-96be-f03bac821358',
  AF7: '273e0004-4c4d-454d-96be-f03bac821358',
  AF8: '273e0005-4c4d-454d-96be-f03bac821358',
  TP10: '273e0006-4c4d-454d-96be-f03bac821358',
  
  // Contrôle
  CONTROL: '273e0001-4c4d-454d-96be-f03bac821358',
  STATUS: '273e0002-4c4d-454d-96be-f03bac821358',
  
  // Gyroscope/Accéléromètre
  GYRO: '273e0009-4c4d-454d-96be-f03bac821358',
  ACCEL: '273e000a-4c4d-454d-96be-f03bac821358',
  
  // Batterie
  BATTERY: '273e000b-4c4d-454d-96be-f03bac821358'
};

// Commandes de contrôle Muse
export const MUSE_COMMANDS = {
  START_STREAM: 'd', // 0x64
  STOP_STREAM: 'h',  // 0x68
  RESET: 'v'         // 0x76
};
7.3 Parser de données EEG
// frontend/src/services/eegParser.js

export class EEGDataParser {
  constructor(deviceType) {
    this.deviceType = deviceType;
    this.samplingRate = this.getSamplingRate();
  }

  getSamplingRate() {
    const rates = {
      muse: 256,
      emotiv: 128,
      openbci: 250,
      neurosky: 512
    };
    return rates[this.deviceType] || 256;
  }

  // Parser pour Muse
  parseMuseData(base64Value, channel) {
    const buffer = Buffer.from(base64Value, 'base64');
    const values = [];

    // Muse envoie 12 échantillons par paquet
    for (let i = 0; i < 12; i++) {
      const index = i * 2;
      if (index + 1 < buffer.length) {
        // Convertir 2 bytes en valeur 16-bit (big-endian)
        const value = (buffer[index] << 8) | buffer[index + 1];
        // Convertir en microvolts
        const microvolts = (value - 2048) * 0.48828125;
        values.push(microvolts);
      }
    }

    return values;
  }

  // Parser pour Emotiv
  parseEmotivData(base64Value) {
    const buffer = Buffer.from(base64Value, 'base64');
    // Format spécifique Emotiv
    // ... implémentation selon la doc Emotiv
    return {};
  }

  // Parser universel
  parse(base64Value, channel) {
    switch (this.deviceType) {
      case 'muse':
        return this.parseMuseData(base64Value, channel);
      case 'emotiv':
        return this.parseEmotivData(base64Value);
      default:
        return [];
    }
  }

  // Calculer la qualité du signal
  calculateSignalQuality(values) {
    if (!values || values.length === 0) return 'poor';

    // Calculer l'écart-type
    const mean = values.reduce((a, b) => a + b, 0) / values.length;
    const variance = values.reduce((sum, val) => {
      return sum + Math.pow(val - mean, 2);
    }, 0) / values.length;
    const stdDev = Math.sqrt(variance);

    // Critères de qualité (ajuster selon l'expérience)
    if (stdDev < 10) return 'poor';        // Signal trop stable (probablement déconnecté)
    if (stdDev < 50) return 'fair';        // Signal faible
    if (stdDev < 100) return 'good';       // Signal acceptable
    if (stdDev < 200) return 'excellent';  // Signal excellent
    return 'poor';                         // Trop de bruit
  }
}
7.4 Permissions Android
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  
  <!-- Permissions Bluetooth -->
  <uses-permission android:name="android.permission.BLUETOOTH" />
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
  <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
  <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
  
  <!-- Permission localisation (requise pour BLE) -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  
  <!-- Déclarer que l'app utilise BLE -->
  <uses-feature
    android:name="android.hardware.bluetooth_le"
    android:required="true" />
  
  <application>
    <!-- ... -->
  </application>
</manifest>
7.5 Permissions iOS
<!-- ios/SionohmairInsight/Info.plist -->
<dict>
  <!-- Bluetooth -->
  <key>NSBluetoothAlwaysUsageDescription</key>
  <string>Sionohmair Insight utilise Bluetooth pour se connecter aux casques EEG</string>
  
  <key>NSBluetoothPeripheralUsageDescription</key>
  <string>L'application nécessite l'accès Bluetooth pour communiquer avec les dispositifs EEG</string>
  
  <!-- Localisation (iOS 13+) -->
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>La localisation est requise pour scanner les dispositifs Bluetooth</string>
</dict>
8. Sécurité et authentification
8.1 Flux d'authentification JWT
1. INSCRIPTION/CONNEXION
   Client → POST /api/auth/login
   ↓
   Server vérifie credentials
   ↓
   Server génère JWT token (expire: 30j)
   ↓
   Client stocke token dans AsyncStorage
   
2. REQUÊTES AUTHENTIFIÉES
   Client ajoute header: Authorization: Bearer <token>
   ↓
   Middleware vérifie token
   ↓
   Si valide: req.user = decoded
   ↓
   Route handler traite la requête
   
3. RENOUVELLEMENT
   Si token expire → Client redirige vers login
8.2 Configuration JWT sécurisée
// backend/config/jwt.js
const jwt = require('jsonwebtoken');

module.exports = {
  // Générer un token
  generateToken: (payload) => {
    return jwt.sign(
      payload,
      process.env.JWT_SECRET,
      {
        expiresIn: process.env.JWT_EXPIRES_IN || '30d',
        issuer: 'sionohmair-api',
        audience: 'sionohmair-client'
      }
    );
  },

  // Vérifier un token
  verifyToken: (token) => {
    try {
      return jwt.verify(token, process.env.JWT_SECRET, {
        issuer: 'sionohmair-api',
        audience: 'sionohmair-client'
      });
    } catch (error) {
      throw new Error('Token invalide');
    }
  },

  // Décoder sans vérifier (pour debug)
  decodeToken: (token) => {
    return jwt.decode(token);
  }
};
8.3 Hachage des mots de passe
// backend/utils/password.js
const bcrypt = require('bcryptjs');

const SALT_ROUNDS = 10;

module.exports = {
  // Hasher un mot de passe
  hash: async (password) => {
    const salt = await bcrypt.genSalt(SALT_ROUNDS);
    return bcrypt.hash(password, salt);
  },

  // Comparer un mot de passe
  compare: async (password, hash) => {
    return bcrypt.compare(password, hash);
  },

  // Valider la force du mot de passe
  validateStrength: (password) => {
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

    const errors = [];

    if (password.length < minLength) {
      errors.push(`Minimum ${minLength} caractères requis`);
    }
    if (!hasUpperCase) {
      errors.push('Au moins une majuscule requise');
    }
    if (!hasLowerCase) {
      errors.push('Au moins une minuscule requise');
    }
    if (!hasNumbers) {
      errors.push('Au moins un chiffre requis');
    }
    if (!hasSpecialChar) {
      errors.push('Au moins un caractère spécial requis');
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }
};
8.4 Rate Limiting
// backend/middlewares/rateLimiter.js
const rateLimit = require('express-rate-limit');

// Limiter les tentatives de connexion
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 tentatives
  message: {
    success: false,
    message: 'Trop de tentatives de connexion. Réessayez dans 15 minutes.'
  },
  standardHeaders: true,
  legacyHeaders: false
});

// Limiter les appels API généraux
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requêtes par minute
  message: {
    success: false,
    message: 'Trop de requêtes. Veuillez ralentir.'
  }
});

// Limiter les uploads de données EEG
const eegDataLimiter = rateLimit({
  windowMs: 1000, // 1 seconde
  max: 50, // 50 points de données par seconde max
  message: {
    success: false,
    message: 'Flux de données trop rapide'
  }
});

module.exports = {
  loginLimiter,
  apiLimiter,
  eegDataLimiter
};
8.5 CORS Configuration
// backend/config/cors.js
const cors = require('cors');

const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [
      'http://localhost:3000',
      'http://localhost:8081',
      'http://192.168.1.100:3000'
    ];

    // Permettre les requêtes sans origin (mobile apps)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Non autorisé par CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  maxAge: 86400 // 24 heures
};

module.exports = cors(corsOptions);
8.6 Validation des entrées
// backend/middlewares/validator.js
const { body, param, query, validationResult } = require('express-validator');

// Middleware de vérification
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      message: 'Erreurs de validation',
      errors: errors.array()
    });
  }
  next();
};

// Validations communes
const validators = {
  // Inscription
  register: [
    body('email')
      .isEmail()
      .withMessage('Email invalide')
      .normalizeEmail(),
    body('password')
      .isLength({ min: 8 })
      .withMessage('Mot de passe minimum 8 caractères')
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
      .withMessage('Mot de passe doit contenir majuscule, minuscule et chiffre'),
    body('name')
      .trim()
      .isLength({ min: 2, max: 100 })
      .withMessage('Nom entre 2 et 100 caractères'),
    validate
  ],

  // Session EEG
  createEEGSession: [
    body('deviceType')
      .isIn(['muse', 'emotiv', 'openbci', 'neurosky', 'other'])
      .withMessage('Type de dispositif invalide'),
    body('deviceId')
      .trim()
      .notEmpty()
      .withMessage('ID dispositif requis'),
    validate
  ],

  // Données EEG
  sendEEGData: [
    param('sessionId')
      .isMongoId()
      .withMessage('ID session invalide'),
    body('timestamp')
      .isISO8601()
      .withMessage('Timestamp invalide'),
    body('channels')
      .isObject()
      .withMessage('Canaux requis'),
    validate
  ],

  // Pagination
  pagination: [
    query('limit')
      .optional()
      .isInt({ min: 1, max: 100 })
      .withMessage('Limite entre 1 et 100'),
    query('skip')
      .optional()
      .isInt({ min: 0 })
      .withMessage('Skip doit être >= 0'),
    validate
  ]
};

module.exports = validators;
9. Déploiement
9.1 Docker - Backend
# backend/Dockerfile
FROM node:18-alpine

WORKDIR /app

# Installer les dépendances
COPY package*.json ./
RUN npm ci --only=production

# Copier le code
COPY . .

# Exposer le port
EXPOSE 4000

# Variables d'environnement
ENV NODE_ENV=production

# Démarrer l'application
CMD ["node", "server.js"]
9.2 Docker Compose
# docker-compose.yml
version: '3.8'

services:
  # Backend API
  backend:
    build: ./backend
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/sionohmair
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/sionohmair_db
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - mongo
      - postgres
      - redis
    restart: unless-stopped

  # MongoDB
  mongo:
    image: mongo:5.0
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped

  # PostgreSQL
  postgres:
    image: postgres:14
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=sionohmair_db
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    restart: unless-stopped

volumes:
  mongo_data:
  postgres_data:
9.3 PM2 Ecosystem
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'sionohmair-api',
    script: './backend/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 4000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    merge_logs: true,
    max_memory_restart: '1G',
    autorestart: true,
    watch: false
  }]
};
9.4 CI/CD - GitHub Actions
# .github/workflows/deploy.yml
name: Deploy Sionohmair Insight

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: |
        cd backend
        npm ci
    
    - name: Run tests
      run: |
        cd backend
        npm test
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/sionohmair
          git pull origin main
          cd backend
          npm install
          pm2 restart sionohmair-api
10. Tests et validation
10.1 Tests Backend (Jest)
// backend/tests/auth.test.js
const request = require('supertest');
const app = require('../server');
const User = require('../models/User');
const mongoose = require('mongoose');

describe('Authentication API', () => {
  beforeAll(async () => {
