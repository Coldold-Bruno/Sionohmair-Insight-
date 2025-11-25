// backend/models/User.js - Modèle utilisateur corrigé
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Le nom est requis'],
    trim: true,
    minlength: [2, 'Le nom doit contenir au moins 2 caractères'],
    maxlength: [100, 'Le nom ne peut pas dépasser 100 caractères']
  },
  
  email: {
    type: String,
    required: [true, 'L\'email est requis'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Email invalide']
  },
  
  passwordHash: {
    type: String,
    required: [true, 'Le mot de passe est requis']
  },
  
  language: {
    type: String,
    enum: ['fr', 'en', 'es', 'de'],
    default: 'fr'
  },
  
  preferences: {
    notifications: {
      type: Boolean,
      default: true
    },
    darkMode: {
      type: Boolean,
      default: false
    },
    audioVolume: {
      type: Number,
      default: 70,
      min: 0,
      max: 100
    },
    language: {
      type: String,
      default: 'fr'
    }
  },
  
  profile: {
    avatar: String,
    bio: String,
    birthDate: Date,
    gender: {
      type: String,
      enum: ['male', 'female', 'other', 'prefer_not_to_say']
    }
  },
  
  devices: [{
    type: {
      type: String,
      enum: ['muse', 'emotiv', 'openbci', 'neurosky', 'other']
    },
    deviceId: String,
    name: String,
    lastConnected: Date,
    addedAt: {
      type: Date,
      default: Date.now
    }
  }],
  
  subscription: {
    plan: {
      type: String,
      enum: ['free', 'premium', 'pro'],
      default: 'free'
    },
    startDate: Date,
    endDate: Date,
    autoRenew: {
      type: Boolean,
      default: false
    }
  },
  
  stats: {
    totalSessions: {
      type: Number,
      default: 0
    },
    totalDuration: {
      type: Number,
      default: 0
    },
    lastSessionDate: Date
  },
  
  lastLogin: {
    type: Date,
    default: Date.now
  },
  
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  updatedAt: {
    type: Date,
    default: Date.now
  },
  
  isActive: {
    type: Boolean,
    default: true
  },
  
  isVerified: {
    type: Boolean,
    default: false
  },
  
  verificationToken: String,
  resetPasswordToken: String,
  resetPasswordExpires: Date

}, {
  timestamps: true
});

// Index pour améliorer les performances
UserSchema.index({ email: 1 });
UserSchema.index({ createdAt: -1 });

// Méthode pour nettoyer l'objet utilisateur avant de l'envoyer au client
UserSchema.methods.toJSON = function() {
  const user = this.toObject();
  delete user.passwordHash;
  delete user.verificationToken;
  delete user.resetPasswordToken;
  delete user.resetPasswordExpires;
  return user;
};

// Hook pre-save pour mettre à jour updatedAt
UserSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  next();
});

module.exports = mongoose.model('User', UserSchema);
