//
// backend/models/EEGSession.js - Modèle de session EEG
const mongoose = require('mongoose');

const EEGDataPointSchema = new mongoose.Schema({
  timestamp: {
    type: Date,
    required: true
  },
  channels: {
    TP9: Number,  // Lobe temporal gauche
    AF7: Number,  // Front gauche
    AF8: Number,  // Front droit
    TP10: Number, // Lobe temporal droit
    AUX: Number   // Canal auxiliaire
  },
  quality: {
    type: String,
    enum: ['excellent', 'good', 'fair', 'poor'],
    default: 'good'
  },
  battery: Number // Niveau de batterie du dispositif (0-100)
}, { _id: false });

const EEGSessionSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    index: true
  },
  
  deviceType: {
    type: String,
    enum: ['muse', 'emotiv', 'openbci', 'neurosky', 'other'],
    required: true
  },
  
  deviceId: {
    type: String,
    required: true
  },
  
  startTime: {
    type: Date,
    required: true,
    default: Date.now
  },
  
  endTime: {
    type: Date
  },
  
  duration: {
    type: Number, // en secondes
    default: 0
  },
  
  status: {
    type: String,
    enum: ['active', 'paused', 'completed', 'error'],
    default: 'active'
  },
  
  data: [EEGDataPointSchema],
  
  statistics: {
    totalSamples: Number,
    averageQuality: String,
    channels: {
      TP9: {
        mean: Number,
        min: Number,
        max: Number,
        samples: Number
      },
      AF7: {
        mean: Number,
        min: Number,
        max: Number,
        samples: Number
      },
      AF8: {
        mean: Number,
        min: Number,
        max: Number,
        samples: Number
      },
      TP10: {
        mean: Number,
        min: Number,
        max: Number,
        samples: Number
      }
    }
  },
  
  analysis: {
    brainwaves: {
      delta: Number,   // 0.5-4 Hz - Sommeil profond
      theta: Number,   // 4-8 Hz - Méditation, créativité
      alpha: Number,   // 8-13 Hz - Relaxation, yeux fermés
      beta: Number,    // 13-30 Hz - Concentration, éveil
      gamma: Number    // 30-100 Hz - Haute cognition
    },
    meditation: Number,    // Score 0-100
    focus: Number,         // Score 0-100
    relaxation: Number,    // Score 0-100
    stress: Number,        // Score 0-100
    insights: [String],
    performedAt: Date
  },
  
  notes: {
    type: String,
    maxlength: 1000
  },
  
  tags: [String],
  
  environment: {
    location: String,
    ambient: {
      type: String,
      enum: ['quiet', 'moderate', 'noisy']
    },
    activity: {
      type: String,
      enum: ['meditation', 'work', 'rest', 'exercise', 'other']
    }
  },
  
  audioSync: {
    enabled: Boolean,
    audioSessionId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'AudioSession'
    }
  },
  
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  updatedAt: {
    type: Date,
    default: Date.now
  }

}, {
  timestamps: true
});

// Index pour améliorer les performances
EEGSessionSchema.index({ userId: 1, startTime: -1 });
EEGSessionSchema.index({ status: 1 });
EEGSessionSchema.index({ createdAt: -1 });

// Méthode virtuelle pour obtenir la durée en minutes
EEGSessionSchema.virtual('durationMinutes').get(function() {
  return this.duration ? Math.round(this.duration / 60) : 0;
});

// Méthode pour calculer les statistiques
EEGSessionSchema.methods.calculateStatistics = function() {
  if (!this.data || this.data.length === 0) {
    return null;
  }

  const channelStats = {};
  const channels = ['TP9', 'AF7', 'AF8', 'TP10'];
  
  channels.forEach(channel => {
    const values = this.data
      .map(d => d.channels && d.channels[channel])
      .filter(v => v !== undefined && v !== null);
    
    if (values.length > 0) {
      const sum = values.reduce((a, b) => a + b, 0);
      const mean = sum / values.length;
      
      channelStats[channel] = {
        mean: Math.round(mean * 100) / 100,
        min: Math.min(...values),
        max: Math.max(...values),
        samples: values.length
      };
    }
  });

  // Calculer la qualité moyenne
  const qualityMap = { excellent: 4, good: 3, fair: 2, poor: 1 };
  const qualities = this.data
    .map(d => qualityMap[d.quality] || 0)
    .filter(q => q > 0);
  
  let averageQuality = 'unknown';
  if (qualities.length > 0) {
    const avg = qualities.reduce((a, b) => a + b, 0) / qualities.length;
    if (avg >= 3.5) averageQuality = 'excellent';
    else if (avg >= 2.5) averageQuality = 'good';
    else if (avg >= 1.5) averageQuality = 'fair';
    else averageQuality = 'poor';
  }

  this.statistics = {
    totalSamples: this.data.length,
    averageQuality,
    channels: channelStats
  };

  return this.statistics;
};

// Hook pre-save
EEGSessionSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  
  // Calculer la durée si la session est terminée
  if (this.endTime && this.startTime) {
    this.duration = (this.endTime - this.startTime) / 1000;
  }
  
  next();
});

module.exports = mongoose.model('EEGSession', EEGSessionSchema);
