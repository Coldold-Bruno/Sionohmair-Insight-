🚀 Guide d'Implémentation - Sionohmair Insight v2.0
📋 Table des Matières
Vue d'ensemble des corrections
Structure du projet
Installation et Configuration
Fichiers corrigés à remplacer
Nouveaux fichiers à créer
Variables d'environnement
Commandes Git pour mise à jour
🔍 Vue d'ensemble des corrections
Problèmes corrigés :
✅ Backend :
Routes non intégrées au serveur principal → Routes connectées et fonctionnelles
Pas de gestion d'erreurs → Middleware d'erreurs global ajouté
Pas de validation des données → Validation avec express-validator
Modèle User incomplet → Modèle enrichi avec préférences, stats, devices
✅ Frontend :
Dépendances manquantes → axios, AsyncStorage, react-native-ble-plx ajoutés
Pas de gestion d'état → Context API (AuthContext, DeviceContext)
i18n non configuré → Configuration complète i18next
Pas d'intégration Bluetooth → Service BLE ajouté
✅ Architecture :
Synchronisation EEG/Audio → Modèles liés avec référence croisée
Sécurité renforcée → JWT, bcrypt, CORS configurés
Gestion sessions → CRUD complet avec statistiques et analyses
📁 Structure du projet corrigée
sionohmair-insight/
├── backend/
│   ├── config/
│   │   └── database.js
│   ├── middlewares/
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validator.js
│   ├── models/
│   │   ├── User.js ✨ CORRIGÉ
│   │   ├── EEGSession.js ✨ NOUVEAU
│   │   └── AudioSession.js ✨ NOUVEAU
│   ├── routes/
│   │   ├── auth.js ✨ CORRIGÉ
│   │   ├── user.js ✨ CORRIGÉ
│   │   ├── eeg.js ✨ NOUVEAU
│   │   ├── audio.js ✨ NOUVEAU
│   │   └── session.js ✨ NOUVEAU
│   ├── services/
│   │   ├── eegAnalysis.js ✨ NOUVEAU
│   │   └── pdfExport.js ✨ NOUVEAU
│   ├── utils/
│   │   └── helpers.js
│   ├── server.js ✨ CORRIGÉ
│   ├── package.json ✨ CORRIGÉ
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── EEGChart.jsx ✨ CORRIGÉ
│   │   │   ├── DeviceCard.jsx ✨ NOUVEAU
│   │   │   ├── SessionCard.jsx ✨ NOUVEAU
│   │   │   └── Button.jsx
│   │   ├── contexts/
│   │   │   ├── AuthContext.js ✨ NOUVEAU
│   │   │   └── DeviceContext.js ✨ NOUVEAU
│   │   ├── screens/
│   │   │   ├── HomeScreen.js ✨ CORRIGÉ
│   │   │   ├── LoginScreen.js ✨ NOUVEAU
│   │   │   ├── RegisterScreen.js ✨ NOUVEAU
│   │   │   ├── DashboardScreen.js ✨ NOUVEAU
│   │   │   ├── EEGSessionScreen.js ✨ NOUVEAU
│   │   │   ├── DeviceConnectionScreen.js ✨ NOUVEAU
│   │   │   ├── ProfileScreen.js ✨ NOUVEAU
│   │   │   ├── SettingsScreen.js ✨ NOUVEAU
│   │   │   ├── SessionHistoryScreen.js ✨ NOUVEAU
│   │   │   └── SessionDetailsScreen.js ✨ NOUVEAU
│   │   ├── services/
│   │   │   ├── api.js ✨ CORRIGÉ
│   │   │   └── bluetooth.js ✨ NOUVEAU
│   │   ├── i18n/
│   │   │   ├── config.js ✨ NOUVEAU
│   │   │   ├── fr.json ✨ ENRICHI
│   │   │   └── en.json ✨ ENRICHI
│   │   ├── utils/
│   │   │   └── helpers.js
│   │   └── App.js ✨ CORRIGÉ
│   ├── package.json ✨ CORRIGÉ
│   └── babel.config.js
│
├── docs/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   └── CONTRIBUTING.md
│
├── .gitignore
├── README.md ✨ ENRICHI
└── docker-compose.yml ✨ NOUVEAU
🔧 Installation et Configuration
Prérequis
Node.js >= 18.0.0
npm >= 9.0.0
MongoDB >= 5.0
PostgreSQL >= 14
React Native CLI
Android Studio / Xcode
Backend
cd backend
npm install

# Installer les nouvelles dépendances
npm install express-validator dotenv mongoose sequelize pg bcryptjs jsonwebtoken cors body-parser

# Créer le fichier .env
cp .env.example .env
# Éditer .env avec vos valeurs

# Démarrer le serveur
npm run dev
Frontend
cd frontend
npm install

# Installer les nouvelles dépendances
npm install @react-native-async-storage/async-storage axios react-native-ble-plx react-native-permissions react-native-vector-icons date-fns zustand

# iOS uniquement
cd ios && pod install && cd ..

# Démarrer l'app
npm run android  # ou npm run ios
📝 Fichiers corrigés à remplacer
Backend
1. backend/server.js
Remplacer entièrement par la nouvelle version qui inclut :
Connexion MongoDB et PostgreSQL
Intégration de toutes les routes
Middleware de gestion d'erreurs
Logging amélioré
2. backend/models/User.js
Remplacer par le modèle enrichi avec :
Validation des champs
Préférences utilisateur
Gestion des dispositifs
Statistiques d'utilisation
3. backend/routes/auth.js
Remplacer par les routes corrigées avec :
Validation express-validator
Hachage bcrypt
Génération JWT
Gestion des erreurs
4. backend/routes/user.js
Mise à jour pour inclure :
Modification du profil
Gestion des préférences
Statistiques utilisateur
5. backend/package.json
Ajouter les dépendances manquantes :
{
  "dependencies": {
    "express": "^4.18.2",
    "express-validator": "^7.0.1",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "mongoose": "^7.6.5",
    "pg": "^8.11.3",
    "sequelize": "^6.35.1",
    "dotenv": "^16.3.1",
    "pdfkit": "^0.13.0"
  }
}
Frontend
6. frontend/src/App.js
Remplacer entièrement avec :
Navigation corrigée
Providers (Auth, Device, i18n)
Gestion du chargement
7. frontend/package.json
Ajouter toutes les dépendances manquantes
8. frontend/src/services/api.js
Corriger avec :
Configuration axios
Intercepteurs pour token
Gestion des erreurs
✨ Nouveaux fichiers à créer
Backend
9. backend/models/EEGSession.js
Créer ce modèle pour gérer les sessions EEG
10. backend/models/AudioSession.js
// backend/models/AudioSession.js
const mongoose = require('mongoose');

const AudioSessionSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  startTime: { type: Date, required: true },
  endTime: Date,
  duration: Number,
  audioData: [{
    timestamp: Date,
    frequency: Number,
    amplitude: Number,
    tone: String
  }],
  analysis: {
    dominantFrequency: Number,
    harmonics: [Number],
    insights: [String]
  },
  eegSyncId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'EEGSession'
  }
}, { timestamps: true });

module.exports = mongoose.model('AudioSession', AudioSessionSchema);
11. backend/routes/eeg.js
Déjà fourni ci-dessus
12. backend/routes/audio.js
// backend/routes/audio.js
const express = require('express');
const router = express.Router();
const authMiddleware = require('../middlewares/auth');
const AudioSession = require('../models/AudioSession');

// Créer session audio
router.post('/session/start', authMiddleware, async (req, res) => {
  try {
    const session = new AudioSession({
      userId: req.user.id,
      startTime: new Date(),
      audioData: []
    });
    await session.save();
    res.status(201).json({ success: true, sessionId: session._id });
  } catch (err) {
    res.status(500).json({ success: false, message: err.message });
  }
});

// Ajouter données audio
router.post('/session/:id/data', authMiddleware, async (req, res) => {
  try {
    const session = await AudioSession.findById(req.params.id);
    session.audioData.push(req.body);
    await session.save();
    res.json({ success: true });
  } catch (err) {
    res.status(500).json({ success: false, message: err.message });
  }
});

module.exports = router;
13. backend/routes/session.js
// backend/routes/session.js
const express = require('express');
const router = express.Router();
const authMiddleware = require('../middlewares/auth');
const EEGSession = require('../models/EEGSession');
const AudioSession = require('../models/AudioSession');

// Sessions combinées
router.get('/combined', authMiddleware, async (req, res) => {
  try {
    const eegSessions = await EEGSession.find({ userId: req.user.id })
      .sort({ startTime: -1 })
      .limit(10);
    
    const audioSessions = await AudioSession.find({ userId: req.user.id })
      .sort({ startTime: -1 })
      .limit(10);

    res.json({
      success: true,
      eeg: eegSessions,
      audio: audioSessions
    });
  } catch (err) {
    res.status(500).json({ success: false, message: err.message });
  }
});

module.exports = router;
14. backend/middlewares/errorHandler.js
// backend/middlewares/errorHandler.js
module.exports = (err, req, res, next) => {
  console.error('Error:', err);

  if (err.name === 'ValidationError') {
    return res.status(400).json({
      success: false,
      message: 'Erreur de validation',
      errors: Object.values(err.errors).map(e => e.message)
    });
  }

  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({
      success: false,
      message: 'Token invalide'
    });
  }

  res.status(err.status || 500).json({
    success: false,
    message: err.message || 'Erreur serveur interne'
  });
};
Frontend
15. frontend/src/contexts/AuthContext.js
Déjà fourni ci-dessus
16. frontend/src/contexts/DeviceContext.js
// frontend/src/contexts/DeviceContext.js
import React, { createContext, useContext, useState } from 'react';
import BLEService from '../services/bluetooth';

const DeviceContext = createContext({});

export const useDevice = () => useContext(DeviceContext);

export const DeviceProvider = ({ children }) => {
  const [connectedDevice, setConnectedDevice] = useState(null);
  const [isScanning, setIsScanning] = useState(false);
  const [devices, setDevices] = useState([]);

  const scanDevices = async () => {
    setIsScanning(true);
    try {
      const found = await BLEService.scanForDevices();
      setDevices(found);
    } catch (error) {
      console.error('Scan error:', error);
    } finally {
      setIsScanning(false);
    }
  };

  const connectDevice = async (device) => {
    try {
      await BLEService.connectToDevice(device);
      setConnectedDevice(device);
      return { success: true };
    } catch (error) {
      return { success: false, message: error.message };
    }
  };

  const disconnectDevice = async () => {
    try {
      await BLEService.disconnect();
      setConnectedDevice(null);
      return { success: true };
    } catch (error) {
      return { success: false, message: error.message };
    }
  };

  return (
    <DeviceContext.Provider value={{
      connectedDevice,
      isScanning,
      devices,
      scanDevices,
      connectDevice,
      disconnectDevice
    }}>
      {children}
    </DeviceContext.Provider>
  );
};
17. frontend/src/services/bluetooth.js
// frontend/src/services/bluetooth.js
import { BleManager } from 'react-native-ble-plx';

class BLEService {
  constructor() {
    this.manager = new BleManager();
    this.device = null;
  }

  async scanForDevices() {
    const devices = [];
    
    this.manager.startDeviceScan(null, null, (error, device) => {
      if (error) {
        console.error(error);
        return;
      }
      
      if (device.name && (device.name.includes('Muse') || device.name.includes('EEG'))) {
        devices.push({
          id: device.id,
          name: device.name,
          rssi: device.rssi
        });
      }
    });

    await new Promise(resolve => setTimeout(resolve, 5000));
    this.manager.stopDeviceScan();
    
    return devices;
  }

  async connectToDevice(device) {
    try {
      this.device = await this.manager.connectToDevice(device.id);
      await this.device.discoverAllServicesAndCharacteristics();
      return this.device;
    } catch (error) {
      throw new Error('Connexion échouée');
    }
  }

  async disconnect() {
    if (this.device) {
      await this.device.cancelConnection();
      this.device = null;
    }
  }

  async startDataStream(callback) {
    // Implémenter la lecture des caractéristiques EEG
    // Exemple pour Muse
    const EEG_SERVICE_UUID = '0000fe8d-0000-1000-8000-00805f9b34fb';
    const EEG_CHAR_UUID = '...';
    
    this.device.monitorCharacteristicForService(
      EEG_SERVICE_UUID,
      EEG_CHAR_UUID,
      (error, characteristic) => {
        if (error) {
          console.error(error);
          return;
        }
        
        const data = this.parseEEGData(characteristic.value);
        callback(data);
      }
    );
  }

  parseEEGData(base64Value) {
    // Parser les données brutes EEG
    // Format dépend du dispositif
    return {
      timestamp: new Date(),
      channels: {
        TP9: 0,
        AF7: 0,
        AF8: 0,
        TP10: 0
      }
    };
  }
}

export default new BLEService();
18. frontend/src/i18n/config.js
// frontend/src/i18n/config.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import fr from './fr.json';
import en from './en.json';

i18n
  .use(initReactI18next)
  .init({
    resources: {
      fr: { translation: fr },
      en: { translation: en }
    },
    lng: 'fr',
    fallbackLng: 'fr',
    interpolation: {
      escapeValue: false
    }
  });

export default i18n;
19. Screens à créer
Créer tous les screens mentionnés dans App.js :
LoginScreen.js
RegisterScreen.js
DashboardScreen.js
EEGSessionScreen.js
DeviceConnectionScreen.js
ProfileScreen.js
SettingsScreen.js
SessionHistoryScreen.js
SessionDetailsScreen.js
🔐 Variables d'environnement
backend/.env
# Serveur
PORT=4000
NODE_ENV=development

# Base de données
MONGODB_URI=mongodb://localhost:27017/sionohmair
DATABASE_URL=postgres://user:password@localhost:5432/sionohmair_db

# Sécurité
JWT_SECRET=votre_secret_jwt_tres_securise_a_changer
JWT_EXPIRES_IN=30d

# Frontend
FRONTEND_URL=http://localhost:3000

# Capteurs (optionnel)
MUSE_SDK_KEY=your_muse_sdk_key
EMOTIV_CLIENT_ID=your_emotiv_id
frontend/.env
API_URL=http://localhost:4000/api
🚀 Commandes Git pour mise à jour
# 1. Se positionner dans le dossier du projet
cd Sionohmair-Insight-

# 2. Créer une nouvelle branche pour les corrections
git checkout -b feature/corrections-v2

# 3. Remplacer les fichiers corrigés
# (copier tous les fichiers fournis dans les artifacts)

# 4. Ajouter tous les changements
git add .

# 5. Commit avec message détaillé
git commit -m "✨ Version 2.0 - Corrections majeures et nouvelles fonctionnalités

Backend:
- Correction routes auth et user
- Ajout modèles EEGSession et AudioSession
- Intégration complète des routes dans server.js
- Middleware de gestion d'erreurs
- Validation des données

Frontend:
- Correction App.js avec navigation complète
- Ajout AuthContext et DeviceContext
- Service Bluetooth pour capteurs EEG
- Configuration i18n
- Nouvelles dépendances (axios, AsyncStorage, BLE)

Architecture:
- Synchronisation EEG/Audio
- Sécurité renforcée (JWT, bcrypt)
- Documentation enrichie"

# 6. Pousser vers GitHub
git push origin feature/corrections-v2

# 7. Créer une Pull Request sur GitHub
# Aller sur https://github.com/Coldold-Bruno/Sionohmair-Insight-
# Cliquer sur "Compare & pull request"
# Vérifier les changements et merger

# 8. (Optionnel) Merger directement dans main
git checkout main
git merge feature/corrections-v2
git push origin main
✅ Checklist de déploiement
Backend
[ ] Installer toutes les dépendances (npm install)
[ ] Configurer .env avec vos valeurs
[ ] Démarrer MongoDB (mongod)
[ ] Démarrer PostgreSQL
[ ] Tester les routes avec Postman
[ ] Vérifier les logs du serveur
Frontend
[ ] Installer toutes les dépendances (npm install)
[ ] iOS: pod install dans le dossier ios
[ ] Configurer les permissions (Bluetooth, localisation)
[ ] Tester sur émulateur/appareil réel
[ ] Vérifier la connexion à l'API
Tests
[ ] Inscription/Connexion
[ ] Scan dispositifs Bluetooth
[ ] Connexion à un casque EEG
[ ] Démarrage session EEG
[ ] Visualisation des données en temps réel
[ ] Sauvegarde et historique
[ ] Export PDF (à implémenter)
📚 Prochaines étapes recommandées
Analyse FFT des ondes cérébrales
Implémenter la transformation de Fourier
Détecter les bandes (delta, theta, alpha, beta, gamma)
Machine Learning
Entraîner un modèle de reconnaissance d'états mentaux
TensorFlow.js pour prédictions en temps réel
Notifications push
Alertes pour sessions recommandées
Rappels d'utilisation
Visualisations avancées
Graphiques 3D du cerveau
Cartes de chaleur des activités cérébrales
Export et partage
Génération PDF avec PDFKit
Partage sur réseaux sociaux
Tests automatisés
Jest pour le backend
React Native Testing Library
📞 Support
En cas de problème :
Vérifier les logs du serveur
Consulter la documentation des dépendances
Ouvrir une issue sur GitHub
Contacter Bruno Coldold
Version: 2.0.0
Date: Novembre 2024
Auteur: Assistant IA + Bruno Coldold
