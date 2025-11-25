// backend/routes/eeg.js - Gestion des données EEG
const express = require('express');
const router = express.Router();
const authMiddleware = require('../middlewares/auth');
const EEGSession = require('../models/EEGSession');

// ============================================
// CRÉER UNE SESSION EEG
// ============================================
router.post('/session/start', authMiddleware, async (req, res) => {
  try {
    const { deviceType, deviceId } = req.body;

    const session = new EEGSession({
      userId: req.user.id,
      deviceType,
      deviceId,
      startTime: new Date(),
      status: 'active',
      data: []
    });

    await session.save();

    res.status(201).json({
      success: true,
      message: 'Session EEG démarrée',
      sessionId: session._id,
      startTime: session.startTime
    });

  } catch (err) {
    console.error('Erreur création session EEG:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors du démarrage de la session' 
    });
  }
});

// ============================================
// ENVOYER DONNÉES EEG EN TEMPS RÉEL
// ============================================
router.post('/session/:sessionId/data', authMiddleware, async (req, res) => {
  try {
    const { sessionId } = req.params;
    const { timestamp, channels, quality } = req.body;

    const session = await EEGSession.findById(sessionId);
    
    if (!session) {
      return res.status(404).json({ 
        success: false, 
        message: 'Session non trouvée' 
      });
    }

    if (session.userId.toString() !== req.user.id) {
      return res.status(403).json({ 
        success: false, 
        message: 'Accès non autorisé' 
      });
    }

    // Ajouter les données
    session.data.push({
      timestamp: new Date(timestamp),
      channels: channels, // {TP9, AF7, AF8, TP10, ...}
      quality: quality || 'good'
    });

    // Limiter la taille du buffer (garder les 10000 derniers points)
    if (session.data.length > 10000) {
      session.data = session.data.slice(-10000);
    }

    await session.save();

    res.json({
      success: true,
      message: 'Données enregistrées',
      dataPoints: session.data.length
    });

  } catch (err) {
    console.error('Erreur enregistrement données EEG:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors de l\'enregistrement' 
    });
  }
});

// ============================================
// TERMINER SESSION EEG
// ============================================
router.post('/session/:sessionId/stop', authMiddleware, async (req, res) => {
  try {
    const { sessionId } = req.params;

    const session = await EEGSession.findById(sessionId);
    
    if (!session) {
      return res.status(404).json({ 
        success: false, 
        message: 'Session non trouvée' 
      });
    }

    if (session.userId.toString() !== req.user.id) {
      return res.status(403).json({ 
        success: false, 
        message: 'Accès non autorisé' 
      });
    }

    session.endTime = new Date();
    session.status = 'completed';
    session.duration = (session.endTime - session.startTime) / 1000; // en secondes

    // Calculer les statistiques de base
    session.statistics = calculateStatistics(session.data);

    await session.save();

    res.json({
      success: true,
      message: 'Session terminée',
      duration: session.duration,
      dataPoints: session.data.length,
      statistics: session.statistics
    });

  } catch (err) {
    console.error('Erreur arrêt session EEG:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors de l\'arrêt de la session' 
    });
  }
});

// ============================================
// RÉCUPÉRER HISTORIQUE DES SESSIONS
// ============================================
router.get('/sessions', authMiddleware, async (req, res) => {
  try {
    const { limit = 20, skip = 0 } = req.query;

    const sessions = await EEGSession.find({ 
      userId: req.user.id 
    })
      .sort({ startTime: -1 })
      .limit(parseInt(limit))
      .skip(parseInt(skip))
      .select('-data'); // Ne pas inclure les données brutes

    const total = await EEGSession.countDocuments({ userId: req.user.id });

    res.json({
      success: true,
      sessions,
      pagination: {
        total,
        limit: parseInt(limit),
        skip: parseInt(skip),
        hasMore: skip + limit < total
      }
    });

  } catch (err) {
    console.error('Erreur récupération sessions:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors de la récupération' 
    });
  }
});

// ============================================
// RÉCUPÉRER DÉTAILS D'UNE SESSION
// ============================================
router.get('/session/:sessionId', authMiddleware, async (req, res) => {
  try {
    const { sessionId } = req.params;

    const session = await EEGSession.findById(sessionId);
    
    if (!session) {
      return res.status(404).json({ 
        success: false, 
        message: 'Session non trouvée' 
      });
    }

    if (session.userId.toString() !== req.user.id) {
      return res.status(403).json({ 
        success: false, 
        message: 'Accès non autorisé' 
      });
    }

    res.json({
      success: true,
      session
    });

  } catch (err) {
    console.error('Erreur récupération session:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors de la récupération' 
    });
  }
});

// ============================================
// ANALYSER SESSION EEG
// ============================================
router.post('/session/:sessionId/analyze', authMiddleware, async (req, res) => {
  try {
    const { sessionId } = req.params;

    const session = await EEGSession.findById(sessionId);
    
    if (!session) {
      return res.status(404).json({ 
        success: false, 
        message: 'Session non trouvée' 
      });
    }

    if (session.userId.toString() !== req.user.id) {
      return res.status(403).json({ 
        success: false, 
        message: 'Accès non autorisé' 
      });
    }

    // Analyse avancée des données EEG
    const analysis = performAdvancedAnalysis(session.data);

    res.json({
      success: true,
      analysis: {
        brainwaves: analysis.brainwaves,
        meditation: analysis.meditation,
        focus: analysis.focus,
        relaxation: analysis.relaxation,
        insights: analysis.insights
      }
    });

  } catch (err) {
    console.error('Erreur analyse session:', err);
    res.status(500).json({ 
      success: false, 
      message: 'Erreur lors de l\'analyse' 
    });
  }
});

// ============================================
// FONCTIONS AUXILIAIRES
// ============================================
function calculateStatistics(data) {
  if (!data || data.length === 0) return {};

  // Calculer moyennes, écarts-types, etc.
  const channelStats = {};
  
  ['TP9', 'AF7', 'AF8', 'TP10'].forEach(channel => {
    const values = data
      .map(d => d.channels && d.channels[channel])
      .filter(v => v !== undefined && v !== null);
    
    if (values.length > 0) {
      const sum = values.reduce((a, b) => a + b, 0);
      const mean = sum / values.length;
      
      channelStats[channel] = {
        mean,
        min: Math.min(...values),
        max: Math.max(...values),
        samples: values.length
      };
    }
  });

  return {
    totalSamples: data.length,
    channels: channelStats,
    averageQuality: calculateAverageQuality(data)
  };
}

function calculateAverageQuality(data) {
  const qualityMap = { excellent: 4, good: 3, fair: 2, poor: 1 };
  const qualities = data
    .map(d => qualityMap[d.quality] || 0)
    .filter(q => q > 0);
  
  if (qualities.length === 0) return 'unknown';
  
  const avg = qualities.reduce((a, b) => a + b, 0) / qualities.length;
  
  if (avg >= 3.5) return 'excellent';
  if (avg >= 2.5) return 'good';
  if (avg >= 1.5) return 'fair';
  return 'poor';
}

function performAdvancedAnalysis(data) {
  // TODO: Implémenter l'analyse FFT et détection des ondes cérébrales
  // Pour l'instant, retourner des valeurs simulées
  
  return {
    brainwaves: {
      delta: Math.random() * 100,
      theta: Math.random() * 100,
      alpha: Math.random() * 100,
      beta: Math.random() * 100,
      gamma: Math.random() * 100
    },
    meditation: Math.floor(Math.random() * 100),
    focus: Math.floor(Math.random() * 100),
    relaxation: Math.floor(Math.random() * 100),
    insights: [
      'État de relaxation détecté',
      'Bon niveau de concentration'
    ]
  };
}

module.exports = router; 
