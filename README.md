// frontend/src/contexts/AuthContext.js - Gestion de l'authentification
import React, { createContext, useContext, useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';
import api from '../services/api';

const AuthContext = createContext({});

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth doit être utilisé dans un AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(null);
  const [loading, setLoading] = useState(true);

  // Vérifier si l'utilisateur est déjà connecté au démarrage
  useEffect(() => {
    checkAuthStatus();
  }, []);

  const checkAuthStatus = async () => {
    try {
      const storedToken = await AsyncStorage.getItem('auth_token');
      const storedUser = await AsyncStorage.getItem('user_data');

      if (storedToken && storedUser) {
        setToken(storedToken);
        setUser(JSON.parse(storedUser));
        
        // Configurer le token dans l'API
        api.defaults.headers.common['Authorization'] = `Bearer ${storedToken}`;
        
        // Vérifier si le token est toujours valide
        try {
          const response = await api.get('/auth/verify');
          if (response.data.success) {
            setUser(response.data.user);
          } else {
            await logout();
          }
        } catch (error) {
          console.log('Token invalide, déconnexion');
          await logout();
        }
      }
    } catch (error) {
      console.error('Erreur vérification auth:', error);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email, password) => {
    try {
      const response = await api.post('/auth/login', {
        email,
        password
      });

      if (response.data.success) {
        const { token: newToken, user: userData } = response.data;
        
        // Sauvegarder les données
        await AsyncStorage.setItem('auth_token', newToken);
        await AsyncStorage.setItem('user_data', JSON.stringify(userData));
        
        // Mettre à jour l'état
        setToken(newToken);
        setUser(userData);
        
        // Configurer le token dans l'API
        api.defaults.headers.common['Authorization'] = `Bearer ${newToken}`;
        
        return { success: true, user: userData };
      }
      
      return { 
        success: false, 
        message: response.data.message || 'Erreur de connexion' 
      };
    } catch (error) {
      console.error('Erreur login:', error);
      return {
        success: false,
        message: error.response?.data?.message || 'Erreur de connexion'
      };
    }
  };

  const register = async (name, email, password, language = 'fr') => {
    try {
      const response = await api.post('/auth/register', {
        name,
        email,
        password,
        language
      });

      if (response.data.success) {
        const { token: newToken, user: userData } = response.data;
        
        // Sauvegarder les données
        await AsyncStorage.setItem('auth_token', newToken);
        await AsyncStorage.setItem('user_data', JSON.stringify(userData));
        
        // Mettre à jour l'état
        setToken(newToken);
        setUser(userData);
        
        // Configurer le token dans l'API
        api.defaults.headers.common['Authorization'] = `Bearer ${newToken}`;
        
        return { success: true, user: userData };
      }
      
      return { 
        success: false, 
        message: response.data.message || 'Erreur d\'inscription' 
      };
    } catch (error) {
      console.error('Erreur register:', error);
      return {
        success: false,
        message: error.response?.data?.message || 'Erreur d\'inscription'
      };
    }
  };

  const logout = async () => {
    try {
      // Supprimer les données stockées
      await AsyncStorage.removeItem('auth_token');
      await AsyncStorage.removeItem('user_data');
      
      // Retirer le token de l'API
      delete api.defaults.headers.common['Authorization'];
      
      // Réinitialiser l'état
      setToken(null);
      setUser(null);
      
      return { success: true };
    } catch (error) {
      console.error('Erreur logout:', error);
      return { success: false, message: 'Erreur de déconnexion' };
    }
  };

  const updateUser = async (updates) => {
    try {
      const response = await api.put('/user/profile', updates);
      
      if (response.data.success) {
        const updatedUser = { ...user, ...response.data.user };
        setUser(updatedUser);
        await AsyncStorage.setItem('user_data', JSON.stringify(updatedUser));
        return { success: true, user: updatedUser };
      }
      
      return { 
        success: false, 
        message: response.data.message || 'Erreur de mise à jour' 
      };
    } catch (error) {
      console.error('Erreur update user:', error);
      return {
        success: false,
        message: error.response?.data?.message || 'Erreur de mise à jour'
      };
    }
  };

  const refreshUser = async () => {
    try {
      const response = await api.get('/user/profile');
      
      if (response.data.success) {
        const updatedUser = response.data.user;
        setUser(updatedUser);
        await AsyncStorage.setItem('user_data', JSON.stringify(updatedUser));
        return { success: true, user: updatedUser };
      }
      
      return { success: false };
    } catch (error) {
      console.error('Erreur refresh user:', error);
      return { success: false };
    }
  };

  const value = {
    user,
    token,
    loading,
    isAuthenticated: !!user,
    login,
    register,
    logout,
    updateUser,
    refreshUser
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
