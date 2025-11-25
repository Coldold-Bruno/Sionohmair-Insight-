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
    await mongoose.connect(process.env.MONGODB_URI_TEST);
  });

  afterAll(async () => {
    await User.deleteMany({});
    await mongoose.connection.close();
  });

  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test@example.com',
          password: 'Test1234!',
          language: 'fr'
        });

      expect(res.statusCode).toBe(201);
      expect(res.body.success).toBe(true);
      expect(res.body.token).toBeDefined();
      expect(res.body.user.email).toBe('test@example.com');
    });

    it('should reject duplicate email', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test@example.com',
          password: 'Test1234!',
          language: 'fr'
        });

      expect(res.statusCode).toBe(400);
      expect(res.body.success).toBe(false);
    });

    it('should reject weak password', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test2@example.com',
          password: 'weak',
          language: 'fr'
        });

      expect(res.statusCode).toBe(400);
      expect(res.body.errors).toBeDefined();
    });
  });

  describe('POST /api/auth/login', () => {
    it('should login existing user', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'Test1234!'
        });

      expect(res.statusCode).toBe(200);
      expect(res.body.success).toBe(true);
      expect(res.body.token).toBeDefined();
    });

    it('should reject wrong password', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'WrongPass123!'
        });

      expect(res.statusCode).toBe(401);
      expect(res.body.success).toBe(false);
    });
  });
});
10.2 Tests d'intégration EEG
// backend/tests/eeg.test.js
const request = require('supertest');
const app = require('../server');
const EEGSession = require('../models/EEGSession');

describe('EEG API', () => {
  let authToken;
  let sessionId;

  beforeAll(async () => {
    // S'authentifier pour obtenir un token
    const res = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: 'Test1234!'
      });
    
    authToken = res.body.token;
  });

  describe('POST /api/eeg/session/start', () => {
    it('should create a new EEG session', async () => {
      const res = await request(app)
        .post('/api/eeg/session/start')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          deviceType: 'muse',
          deviceId: 'MUSE-TEST-001'
        });

      expect(res.statusCode).toBe(201);
      expect(res.body.success).toBe(true);
      expect(res.body.sessionId).toBeDefined();
      
      sessionId = res.body.sessionId;
    });

    it('should reject without authentication', async () => {
      const res = await request(app)
        .post('/api/eeg/session/start')
        .send({
          deviceType: 'muse',
          deviceId: 'MUSE-TEST-001'
        });

      expect(res.statusCode).toBe(401);
    });
  });

  describe('POST /api/eeg/session/:id/data', () => {
    it('should accept EEG data', async () => {
      const res = await request(app)
        .post(`/api/eeg/session/${sessionId}/data`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          timestamp: new Date().toISOString(),
          channels: {
            TP9: 850.5,
            AF7: 820.3,
            AF8: 815.7,
            TP10: 840.2
          },
          quality: 'good'
        });

      expect(res.statusCode).toBe(200);
      expect(res.body.success).toBe(true);
    });
  });

  describe('POST /api/eeg/session/:id/stop', () => {
    it('should stop session and return stats', async () => {
      const res = await request(app)
        .post(`/api/eeg/session/${sessionId}/stop`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.statusCode).toBe(200);
      expect(res.body.success).toBe(true);
      expect(res.body.duration).toBeDefined();
      expect(res.body.statistics).toBeDefined();
    });
  });
});
10.3 Tests Frontend (React Native Testing Library)
// frontend/src/__tests__/LoginScreen.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import LoginScreen from '../screens/LoginScreen';
import { AuthProvider } from '../contexts/AuthContext';

describe('LoginScreen', () => {
  it('should render login form', () => {
    const { getByPlaceholderText, getByText } = render(
      <AuthProvider>
        <LoginScreen />
      </AuthProvider>
    );

    expect(getByPlaceholderText('Email')).toBeTruthy();
    expect(getByPlaceholderText('Mot de passe')).toBeTruthy();
    expect(getByText('Se connecter')).toBeTruthy();
  });

  it('should validate email format', async () => {
    const { getByPlaceholderText, getByText, findByText } = render(
      <AuthProvider>
        <LoginScreen />
      </AuthProvider>
    );

    const emailInput = getByPlaceholderText('Email');
    const submitButton = getByText('Se connecter');

    fireEvent.changeText(emailInput, 'invalid-email');
    fireEvent.press(submitButton);

    const errorMessage = await findByText(/Email invalide/i);
    expect(errorMessage).toBeTruthy();
  });

  it('should login successfully', async () => {
    const mockNavigate = jest.fn();
    const { getByPlaceholderText, getByText } = render(
      <AuthProvider>
        <LoginScreen navigation={{ navigate: mockNavigate }} />
      </AuthProvider>
    );

    const emailInput = getByPlaceholderText('Email');
    const passwordInput = getByPlaceholderText('Mot de passe');
    const submitButton = getByText('Se connecter');

    fireEvent.changeText(emailInput, 'test@example.com');
    fireEvent.changeText(passwordInput, 'Test1234!');
    fireEvent.press(submitButton);

    await waitFor(() => {
      expect(mockNavigate).toHaveBeenCalledWith('Dashboard');
    });
  });
});
10.4 Tests Bluetooth Service
// frontend/src/__tests__/BluetoothService.test.js
import BluetoothService from '../services/bluetooth';
import { BleManager } from 'react-native-ble-plx';

jest.mock('react-native-ble-plx');

describe('BluetoothService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should scan for devices', async () => {
    const mockStartDeviceScan = jest.fn((_, __, callback) => {
      // Simuler la découverte d'un device
      setTimeout(() => {
        callback(null, {
          id: '123',
          name: 'Muse-A1B2',
          rssi: -60
        });
      }, 100);
    });

    BleManager.mockImplementation(() => ({
      startDeviceScan: mockStartDeviceScan,
      stopDeviceScan: jest.fn()
    }));

    const devices = await BluetoothService.scanDevices(200);
    
    expect(devices.length).toBeGreaterThan(0);
    expect(devices[0].name).toContain('Muse');
  });

  it('should connect to device', async () => {
    const mockConnect = jest.fn().mockResolvedValue({
      id: '123',
      name: 'Muse-A1B2',
      discoverAllServicesAndCharacteristics: jest.fn()
    });

    BleManager.mockImplementation(() => ({
      connectToDevice: mockConnect
    }));

    const device = await BluetoothService.connectToDevice('123');
    
    expect(mockConnect).toHaveBeenCalledWith('123');
    expect(device).toBeDefined();
  });
});
10.5 Script de tests complet
#!/bin/bash
# backend/scripts/run-tests.sh

echo "🧪 Démarrage des tests Sionohmair Insight"
echo "=========================================="

# Variables d'environnement de test
export NODE_ENV=test
export MONGODB_URI_TEST=mongodb://localhost:27017/sionohmair_test
export JWT_SECRET=test_secret_key

# Couleurs
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

# Nettoyer la base de test
echo "🗑️  Nettoyage de la base de données de test..."
mongo sionohmair_test --eval "db.dropDatabase()"

# Tests Backend
echo ""
echo "📦 Tests Backend..."
cd backend
npm test -- --coverage --verbose

if [ $? -eq 0 ]; then
  echo -e "${GREEN}✓ Tests backend réussis${NC}"
else
  echo -e "${RED}✗ Tests backend échoués${NC}"
  exit 1
fi

# Tests Frontend
echo ""
echo "📱 Tests Frontend..."
cd ../frontend
npm test -- --coverage

if [ $? -eq 0 ]; then
  echo -e "${GREEN}✓ Tests frontend réussis${NC}"
else
  echo -e "${RED}✗ Tests frontend échoués${NC}"
  exit 1
fi

echo ""
echo -e "${GREEN}✓ Tous les tests sont passés avec succès!${NC}"
11. Maintenance et monitoring
11.1 Logging avec Winston
// backend/utils/logger.js
const winston = require('winston');
const path = require('path');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp({
      format: 'YYYY-MM-DD HH:mm:ss'
    }),
    winston.format.errors({ stack: true }),
    winston.format.splat(),
    winston.format.json()
  ),
  defaultMeta: { service: 'sionohmair-api' },
  transports: [
    // Fichier pour les erreurs
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/error.log'),
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    // Fichier pour tous les logs
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/combined.log'),
      maxsize: 5242880,
      maxFiles: 5
    })
  ]
});

// En développement, logger dans la console
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

// Fonctions helper
logger.logRequest = (req, res, next) => {
  logger.info('HTTP Request', {
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.get('user-agent')
  });
  next();
};

logger.logError = (err, context = {}) => {
  logger.error(err.message, {
    stack: err.stack,
    ...context
  });
};

module.exports = logger;
Utilisation dans le code :
// Dans server.js
const logger = require('./utils/logger');

// Logger les requêtes
app.use(logger.logRequest);

// Logger les erreurs
app.use((err, req, res, next) => {
  logger.logError(err, {
    url: req.url,
    method: req.method,
    userId: req.user?.id
  });
  next(err);
});
11.2 Monitoring avec PM2
# Commandes PM2 essentielles

# Démarrer l'application
pm2 start ecosystem.config.js

# Voir les logs en temps réel
pm2 logs sionohmair-api

# Voir le statut
pm2 status

# Monitorer les ressources
pm2 monit

# Redémarrer sans downtime
pm2 reload sionohmair-api

# Voir les métriques
pm2 describe sionohmair-api

# Sauvegarder la configuration
pm2 save

# Setup au démarrage du serveur
pm2 startup
11.3 Health Check Endpoint
// backend/routes/health.js
const express = require('express');
const router = express.Router();
const mongoose = require('mongoose');
const { Sequelize } = require('sequelize');
const os = require('os');

router.get('/health', async (req, res) => {
  const healthCheck = {
    status: 'UP',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV,
    services: {}
  };

  // Vérifier MongoDB
  try {
    healthCheck.services.mongodb = {
      status: mongoose.connection.readyState === 1 ? 'UP' : 'DOWN',
      responseTime: await checkMongoDBLatency()
    };
  } catch (error) {
    healthCheck.services.mongodb = {
      status: 'DOWN',
      error: error.message
    };
    healthCheck.status = 'DEGRADED';
  }

  // Vérifier PostgreSQL
  try {
    const sequelize = req.app.get('sequelize');
    await sequelize.authenticate();
    healthCheck.services.postgresql = {
      status: 'UP',
      responseTime: await checkPostgreSQLLatency(sequelize)
    };
  } catch (error) {
    healthCheck.services.postgresql = {
      status: 'DOWN',
      error: error.message
    };
    healthCheck.status = 'DEGRADED';
  }

  // Métriques système
  healthCheck.system = {
    memory: {
      total: Math.round(os.totalmem() / 1024 / 1024),
      free: Math.round(os.freemem() / 1024 / 1024),
      used: Math.round((os.totalmem() - os.freemem()) / 1024 / 1024)
    },
    cpu: {
      cores: os.cpus().length,
      loadAverage: os.loadavg()
    },
    process: {
      memory: Math.round(process.memoryUsage().heapUsed / 1024 / 1024),
      pid: process.pid
    }
  };

  const statusCode = healthCheck.status === 'UP' ? 200 : 503;
  res.status(statusCode).json(healthCheck);
});

async function checkMongoDBLatency() {
  const start = Date.now();
  await mongoose.connection.db.admin().ping();
  return Date.now() - start;
}

async function checkPostgreSQLLatency(sequelize) {
  const start = Date.now();
  await sequelize.query('SELECT 1');
  return Date.now() - start;
}

module.exports = router;
11.4 Métriques et Analytics
// backend/services/analytics.js
const { Sequelize } = require('sequelize');

class AnalyticsService {
  constructor(sequelize) {
    this.sequelize = sequelize;
  }

  // Enregistrer un événement
  async trackEvent(userId, eventType, eventData = {}) {
    try {
      await this.sequelize.query(
        `INSERT INTO analytics_events (user_id, event_type, event_data, timestamp)
         VALUES ($1, $2, $3, NOW())`,
        {
          bind: [userId, eventType, JSON.stringify(eventData)],
          type: Sequelize.QueryTypes.INSERT
        }
      );
    } catch (error) {
      console.error('Error tracking event:', error);
    }
  }

  // Obtenir les statistiques d'utilisation
  async getUserStats(userId, days = 30) {
    const [results] = await this.sequelize.query(
      `SELECT 
        COUNT(DISTINCT session_id) as total_sessions,
        SUM(duration) as total_duration,
        AVG(duration) as avg_duration,
        AVG(meditation_score) as avg_meditation,
        AVG(focus_score) as avg_focus
       FROM sessions_metadata
       WHERE user_id = $1 
       AND start_time >= NOW() - INTERVAL '${days} days'`,
      {
        bind: [userId],
        type: Sequelize.QueryTypes.SELECT
      }
    );

    return results[0];
  }

  // Obtenir les tendances globales
  async getGlobalStats() {
    const [results] = await this.sequelize.query(
      `SELECT 
        COUNT(DISTINCT user_id) as total_users,
        COUNT(*) as total_sessions,
        SUM(duration) as total_duration,
        AVG(meditation_score) as avg_meditation
       FROM sessions_metadata
       WHERE start_time >= NOW() - INTERVAL '7 days'`,
      {
        type: Sequelize.QueryTypes.SELECT
      }
    );

    return results[0];
  }

  // Obtenir les dispositifs les plus utilisés
  async getDeviceStats() {
    const results = await this.sequelize.query(
      `SELECT 
        device_type,
        COUNT(*) as usage_count,
        AVG(duration) as avg_duration
       FROM sessions_metadata
       WHERE start_time >= NOW() - INTERVAL '30 days'
       GROUP BY device_type
       ORDER BY usage_count DESC`,
      {
        type: Sequelize.QueryTypes.SELECT
      }
    );

    return results;
  }
}

module.exports = AnalyticsService;
11.5 Backup automatique
#!/bin/bash
# backend/scripts/backup.sh

# Configuration
BACKUP_DIR="/var/backups/sionohmair"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Créer le dossier de backup
mkdir -p $BACKUP_DIR

echo "🔄 Démarrage du backup - $DATE"

# Backup MongoDB
echo "📦 Backup MongoDB..."
mongodump --uri="$MONGODB_URI" --out="$BACKUP_DIR/mongo_$DATE"
tar -czf "$BACKUP_DIR/mongo_$DATE.tar.gz" "$BACKUP_DIR/mongo_$DATE"
rm -rf "$BACKUP_DIR/mongo_$DATE"

# Backup PostgreSQL
echo "🐘 Backup PostgreSQL..."
pg_dump "$DATABASE_URL" > "$BACKUP_DIR/postgres_$DATE.sql"
gzip "$BACKUP_DIR/postgres_$DATE.sql"

# Supprimer les anciens backups
echo "🗑️  Nettoyage des anciens backups..."
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "✅ Backup terminé avec succès!"

# Optionnel: Upload vers S3 ou autre cloud storage
# aws s3 sync $BACKUP_DIR s3://sionohmair-backups/
Ajouter au crontab :
# Backup quotidien à 2h du matin
0 2 * * * /var/www/sionohmair/backend/scripts/backup.sh >> /var/log/sionohmair-backup.log 2>&1
11.6 Alertes et notifications
// backend/services/alerting.js
const nodemailer = require('nodemailer');
const logger = require('../utils/logger');

class AlertingService {
  constructor() {
    this.transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: process.env.SMTP_PORT,
      secure: true,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS
      }
    });

    this.adminEmail = process.env.ADMIN_EMAIL;
  }

  // Envoyer une alerte critique
  async sendCriticalAlert(title, message, details = {}) {
    try {
      await this.transporter.sendMail({
        from: process.env.SMTP_USER,
        to: this.adminEmail,
        subject: `🚨 ALERTE CRITIQUE - ${title}`,
        html: `
          <h2>Alerte Critique - Sionohmair Insight</h2>
          <p><strong>${title}</strong></p>
          <p>${message}</p>
          <hr>
          <h3>Détails:</h3>
          <pre>${JSON.stringify(details, null, 2)}</pre>
          <hr>
          <p>Timestamp: ${new Date().toISOString()}</p>
          <p>Environnement: ${process.env.NODE_ENV}</p>
        `
      });

      logger.warn('Critical alert sent', { title, message });
    } catch (error) {
      logger.error('Failed to send alert email', error);
    }
  }

  // Alerte si erreur BD
  async alertDatabaseError(dbType, error) {
    await this.sendCriticalAlert(
      `Erreur Base de données ${dbType}`,
      `La connexion à ${dbType} a échoué`,
      {
        error: error.message,
        stack: error.stack
      }
    );
  }

  // Alerte si trop d'erreurs
  async alertHighErrorRate(errorCount, timeWindow) {
    await this.sendCriticalAlert(
      'Taux d\'erreurs élevé',
      `${errorCount} erreurs détectées en ${timeWindow} minutes`,
      {
        errorCount,
        timeWindow,
        timestamp: new Date().toISOString()
      }
    );
  }

  // Alerte si CPU/Mémoire élevés
  async alertHighResourceUsage(type, percentage) {
    if (percentage > 90) {
      await this.sendCriticalAlert(
        `Usage ${type} critique`,
        `${type} à ${percentage}%`,
        {
          type,
          percentage,
          pid: process.pid
        }
      );
    }
  }
}

module.exports = new AlertingService();
12. Documentation API complète
12.1 Format de réponse standardisé
Toutes les réponses API suivent ce format :
// Succès
{
  "success": true,
  "message": "Description de l'action",
  "data": { ... },        // Données retournées
  "meta": {               // Métadonnées (optionnel)
    "timestamp": "2024-11-25T10:30:00Z",
    "requestId": "req_abc123"
  }
}

// Erreur
{
  "success": false,
  "message": "Description de l'erreur",
  "errors": [             // Détails des erreurs (optionnel)
    {
      "field": "email",
      "message": "Email invalide"
    }
  ],
  "code": "VALIDATION_ERROR"
}
12.2 Codes d'erreur personnalisés
// backend/utils/errorCodes.js
module.exports = {
  // Authentication
  AUTH_INVALID_CREDENTIALS: 'AUTH_001',
  AUTH_TOKEN_EXPIRED: 'AUTH_002',
  AUTH_TOKEN_INVALID: 'AUTH_003',
  AUTH_EMAIL_EXISTS: 'AUTH_004',
  
  // Validation
  VALIDATION_ERROR: 'VAL_001',
  VALIDATION_WEAK_PASSWORD: 'VAL_002',
  
  // EEG
  EEG_SESSION_NOT_FOUND: 'EEG_001',
  EEG_DEVICE_NOT_CONNECTED: 'EEG_002',
  EEG_INVALID_DATA: 'EEG_003',
  
  // Database
  DB_CONNECTION_ERROR: 'DB_001',
  DB_QUERY_ERROR: 'DB_002',
  
  // Rate Limiting
  RATE_LIMIT_EXCEEDED: 'RATE_001'
};
12.3 Postman Collection
{
  "info": {
    "name": "Sionohmair Insight API",
    "description": "Collection complète des endpoints API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{auth_token}}",
        "type": "string"
      }
    ]
  },
  "item": [
    {
      "name": "Authentication",
      "item": [
        {
          "name": "Register",
          "request": {
            "method": "POST",
            "header": [],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Test User\",\n  \"email\": \"test@example.com\",\n  \"password\": \"Test1234!\",\n  \"language\": \"fr\"\n}",
              "options": {
                "raw": {
                  "language": "json"
                }
              }
            },
            "url": {
              "raw": "{{base_url}}/api/auth/register",
              "host": ["{{base_url}}"],
              "path": ["api", "auth", "register"]
            }
          }
        },
        {
          "name": "Login",
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "const response = pm.response.json();",
                  "if (response.token) {",
                  "  pm.environment.set('auth_token', response.token);",
                  "}"
                ],
                "type": "text/javascript"
              }
            }
          ],
          "request": {
            "method": "POST",
            "header": [],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"email\": \"test@example.com\",\n  \"password\": \"Test1234!\"\n}",
              "options": {
                "raw": {
                  "language": "json"
                }
              }
            },
            "url": {
              "raw": "{{base_url}}/api/auth/login",
              "host": ["{{base_url}}"],
              "path": ["api", "auth", "login"]
            }
          }
        }
      ]
    },
    {
      "name": "EEG Sessions",
      "item": [
        {
          "name": "Start Session",
          "request": {
            "method": "POST",
            "header": [],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"deviceType\": \"muse\",\n  \"deviceId\": \"MUSE-A1B2\"\n}",
              "options": {
                "raw": {
                  "language": "json"
                }
              }
            },
            "url": {
              "raw": "{{base_url}}/api/eeg/session/start",
              "host": ["{{base_url}}"],
              "path": ["api", "eeg", "session", "start"]
            }
          }
        },
        {
          "name": "Send Data",
          "request": {
            "method": "POST",
            "header": [],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"timestamp\": \"{{$isoTimestamp}}\",\n  \"channels\": {\n    \"TP9\": 850.5,\n    \"AF7\": 820.3,\n    \"AF8\": 815.7,\n    \"TP10\": 840.2\n  },\n  \"quality\": \"good\"\n}",
              "options": {
                "raw": {
                  "language": "json"
                }
              }
            },
            "url": {
              "raw": "{{base_url}}/api/eeg/session/{{session_id}}/data",
              "host": ["{{base_url}}"],
              "path": ["api", "eeg", "session", "{{session_id}}", "data"]
            }
          }
        }
      ]
    }
  ],
  "variable": [
    {
      "key": "base_url",
      "value": "http://localhost:4000",
      "type": "string"
    }
  ]
}
13. Checklist finale de déploiement
✅ Configuration serveur
[ ] Node.js 18+ installé
[ ] MongoDB 5.0+ configuré et sécurisé
[ ] PostgreSQL 14+ configuré et sécurisé
[ ] Redis installé (optionnel mais recommandé)
[ ] Certificats SSL configurés
[ ] Firewall configuré (ports 80, 443, 4000)
[ ] Utilisateur système non-root créé
✅ Backend
[ ] Variables .env configurées
[ ] JWT_SECRET généré (32+ caractères aléatoires)
[ ] Connexions BD testées
[ ] Migrations de BD exécutées
[ ] PM2 configuré et testé
[ ] Logs configurés dans /var/log/sionohmair/
[ ] Backup automatique configuré
[ ] Health check endpoint testé
[ ] Rate limiting activé
[ ] CORS correctement configuré
✅ Frontend
[ ] Build de production généré
[ ] Variables d'environnement configurées
[ ] API_URL pointe vers production
[ ] Permissions Bluetooth configurées
[ ] Icônes et splash screens générés
[ ] Code signing (iOS) configuré
[ ] Keystore (Android) généré et sauvegardé
✅ Sécurité
[ ] HTTPS activé
[ ] Headers de sécurité configurés
[ ] SQL injection protection
[ ] XSS protection
[ ] CSRF protection
[ ] Mots de passe avec bcrypt (10+ rounds)
[ ] Tokens JWT avec expiration
[ ] Validation de toutes les entrées
✅ Monitoring
[ ] PM2 monitoring actif
[ ] Logs centralisés
[ ] Alertes email configurées
[ ] Health check automatique
[ ] Métriques de performance suivies
✅ Tests
[ ] Tests unitaires passés
[ ] Tests d'intégration passés
[ ] Tests E2E sur devices réels
[ ] Tests de charge effectués
[ ] Scénarios utilisateurs validés
✅ Documentation
[ ] README.md à jour
[ ] API documentation complète
[ ] Guide utilisateur créé
[ ] Commentaires de code
[ ] Architecture documentée
14. Commandes rapides de référence
Développement local
# Démarrer tout (avec Docker)
docker-compose up -d

# Backend seulement
cd backend && npm run dev

# Frontend seulement
cd frontend && npm start

# Tests
npm test

# Logs
pm2 logs sionohmair-api
docker-compose logs -f backend
Production
# Déployer
git pull origin main
cd backend && npm install --production
pm2 reload sionohmair-api

# Vérifier le statut
pm2 status
curl http://localhost:4000/health

# Voir les logs
pm2 logs sionohmair-api --lines 100

# Backup manuel
./backend/scripts/backup.sh

# Redémarrer services
pm2 restart sionohmair-api
sudo systemctl restart mongodb
sudo systemctl restart postgresql
Debugging
# Vérifier les connexions BD
mongo sionohmair --eval "db.stats()"
psql -d sionohmair_db -c "SELECT version();"

# Tester l'API
curl -X POST http://localhost:4000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test1234!"}'

# Monitorer les ressources
htop
pm2 monit

# Analyser les logs
tail -f /var/log/sionohmair/error.log
grep "ERROR" /var/log/sionohmair/combined.log | tail -20
15. Support et ressources
Documentation technique
API Reference: https://docs.sionohmair.com/api
Architecture: https://docs.sionohmair.com/architecture
Guides: https://docs.sionohmair.com/guides
Ressources externes
Muse Developer
Emotiv API
React Native Docs
Node.js Best Practices
Contact support
Email: dev@sionohmair.com
Issues GitHub: https://github.com/Coldold-Bruno/Sionohmair-Insight-/issues
Slack: sionohmair-dev.slack.com
🎓 Conclusion
Vous disposez maintenant d'un livrable complet pour développer, déployer et maintenir Sionohmair Insight. Ce document couvre:
✅ Architecture complète (Backend Node.js + Frontend React Native)
✅ Intégration Bluetooth avec capteurs EEG
✅ Base de données MongoDB + PostgreSQL
✅ Authentification JWT sécurisée
✅ API REST documentée
✅ Tests automatisés
✅ Déploiement Docker + PM2
✅ Monitoring et alertes
✅ Scripts de maintenance
Prochaines étapes recommandées:
Configurer l'environnement de développement
Implémenter l'analyse FFT des ondes cérébrales
Ajouter le Machine Learning pour les insights
Développer l'export PDF
Créer les notifications push
Mettre en place le CI/CD complet
Bon développement ! 🚀 
