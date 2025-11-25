
// frontend/src/App.js - Application React Native corrigée
import React, { useEffect, useState } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { StatusBar, ActivityIndicator, View } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { I18nextProvider } from 'react-i18next';
import i18n from './i18n/config';

// Screens
import HomeScreen from './screens/HomeScreen';
import LoginScreen from './screens/LoginScreen';
import RegisterScreen from './screens/RegisterScreen';
import DashboardScreen from './screens/DashboardScreen';
import EEGSessionScreen from './screens/EEGSessionScreen';
import DeviceConnectionScreen from './screens/DeviceConnectionScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';
import SessionHistoryScreen from './screens/SessionHistoryScreen';
import SessionDetailsScreen from './screens/SessionDetailsScreen';

// Context
import { AuthProvider, useAuth } from './contexts/AuthContext';
import { DeviceProvider } from './contexts/DeviceContext';

const Stack = createNativeStackNavigator();

// Navigation pour utilisateurs authentifiés
function AuthStack() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: true,
        headerStyle: {
          backgroundColor: '#3A7BD5',
        },
        headerTintColor: '#fff',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <Stack.Screen 
        name="Dashboard" 
        component={DashboardScreen}
        options={{ title: 'Tableau de bord' }}
      />
      <Stack.Screen 
        name="EEGSession" 
        component={EEGSessionScreen}
        options={{ title: 'Session EEG' }}
      />
      <Stack.Screen 
        name="DeviceConnection" 
        component={DeviceConnectionScreen}
        options={{ title: 'Connexion dispositif' }}
      />
      <Stack.Screen 
        name="Profile" 
        component={ProfileScreen}
        options={{ title: 'Profil' }}
      />
      <Stack.Screen 
        name="Settings" 
        component={SettingsScreen}
        options={{ title: 'Paramètres' }}
      />
      <Stack.Screen 
        name="SessionHistory" 
        component={SessionHistoryScreen}
        options={{ title: 'Historique' }}
      />
      <Stack.Screen 
        name="SessionDetails" 
        component={SessionDetailsScreen}
        options={{ title: 'Détails session' }}
      />
    </Stack.Navigator>
  );
}

// Navigation pour utilisateurs non authentifiés
function GuestStack() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
      }}
    >
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Login" component={LoginScreen} />
      <Stack.Screen name="Register" component={RegisterScreen} />
    </Stack.Navigator>
  );
}

// Composant de navigation principal
function RootNavigator() {
  const { user, loading } = useAuth();

  if (loading) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', backgroundColor: '#3A7BD5' }}>
        <ActivityIndicator size="large" color="#ffffff" />
      </View>
    );
  }

  return (
    <NavigationContainer>
      {user ? <AuthStack /> : <GuestStack />}
    </NavigationContainer>
  );
}

// Composant principal de l'application
export default function App() {
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    // Initialisation de l'application
    const initializeApp = async () => {
      try {
        // Charger les préférences utilisateur
        const savedLanguage = await AsyncStorage.getItem('user_language');
        if (savedLanguage) {
          i18n.changeLanguage(savedLanguage);
        }

        // Autres initialisations...
        await new Promise(resolve => setTimeout(resolve, 500));
        
        setIsReady(true);
      } catch (error) {
        console.error('Erreur initialisation:', error);
        setIsReady(true);
      }
    };

    initializeApp();
  }, []);

  if (!isReady) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', backgroundColor: '#3A7BD5' }}>
        <ActivityIndicator size="large" color="#ffffff" />
      </View>
    );
  }

  return (
    <I18nextProvider i18n={i18n}>
      <AuthProvider>
        <DeviceProvider>
          <StatusBar barStyle="light-content" backgroundColor="#3A7BD5" />
          <RootNavigator />
        </DeviceProvider>
      </AuthProvider>
    </I18nextProvider>
  );
}
