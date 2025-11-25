// backend/server.js - Version corrigée et améliorée
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const { Sequelize } = require('sequelize');

const app = express();

// ============================================
// CONFIGURATION MIDDLEWARE
// ============================================
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true
}));
app.use(bodyParser.json({ limit: '50mb' }));
app.use(bodyParser.urlencoded({ extended: true, limit: '50mb' }));

// Logging middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
  next();
});

// ============================================
// CONNEXION BASE DE DONNÉES
// ============================================
// MongoDB pour données non-structurées (sessions EEG, audio)
const connectMongoDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/sionohmair', {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });
    console.log('✓ MongoDB connecté avec succès');
  } catch (err) {
    console.error('✗ Erreur connexion MongoDB:', err.message);
  }
};

// PostgreSQL pour données structurées (utilisateurs, métadonnées)
const sequelize = new Sequelize(
  process.env.DATABASE_URL || 'postgres://localhost:5432/sionohmair_db',
  {
    logging: false,
    dialect: 'postgres'
  }
);

const connectPostgreSQL = async () => {
  try {
    await sequelize.authenticate();
    console.log('✓ PostgreSQL connecté avec succès');
  } catch (err) {
    console.error('✗ Erreur connexion PostgreSQL:', err.message);
  }
};

// ============================================
// ROUTES
// ============================================
const authRoutes = require('./routes/auth');
const userRoutes = require('./routes/user');
const eegRoutes = require('./routes/eeg');
const audioRoutes = require('./routes/audio');
const sessionRoutes = require('./routes/session');

app.use('/api/auth', authRoutes);
app.use('/api/user', userRoutes);
app.use('/api/eeg', eegRoutes);
app.use('/api/audio', audioRoutes);
app.use('/api/session', sessionRoutes);

// Route de test
app.get('/', (req, res) => {
  res.json({
    message: 'API Sionohmair Insight - Backend opérationnel',
    version: '2.0.0',
    status: 'healthy',
    timestamp: new Date().toISOString()
  });
});

// Route de santé
app.get('/health', (req, res) => {
  res.json({
    status: 'UP',
    mongodb: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected',
    postgresql: sequelize.authenticate().then(() => true).catch(() => false)
  });
});

// ============================================
// GESTION DES ERREURS
// ============================================
// 404 - Route non trouvée
app.use((req, res, next) => {
  res.status(404).json({
    success: false,
    message: 'Route non trouvée',
    path: req.path
  });
});

// Gestionnaire d'erreurs global
app.use((err, req, res, next) => {
  console.error('Erreur serveur:', err);
  res.status(err.status || 500).json({
    success: false,
    message: err.message || 'Erreur interne du serveur',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});

// ============================================
// DÉMARRAGE DU SERVEUR
// ============================================
const PORT = process.env.PORT || 4000;

const startServer = async () => {
  try {
    await connectMongoDB();
    await connectPostgreSQL();
    
    app.listen(PORT, () => {
      console.log(`
╔════════════════════════════════════════════════╗
║   🚀 Sionohmair Insight Backend démarré       ║
║   📡 Port: ${PORT}                             ║
║   🌍 Environment: ${process.env.NODE_ENV || 'development'}              ║
║   📅 ${new Date().toLocaleString('fr-FR')}    ║
╚════════════════════════════════════════════════╝
      `);
    });
  } catch (err) {
    console.error('Erreur au démarrage du serveur:', err);
    process.exit(1);
  }
};

// Gestion propre de l'arrêt
process.on('SIGINT', async () => {
  console.log('\n🛑 Arrêt du serveur...');
  await mongoose.connection.close();
  await sequelize.close();
  process.exit(0);
});

startServer();

module.exports = app;
