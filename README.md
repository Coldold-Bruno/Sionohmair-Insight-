// frontend/src/screens/EEGSessionScreen.js - COMPLET
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  ScrollView,
  Dimensions
} from 'react-native';
import { LineChart } from 'react-native-chart-kit';
import BluetoothService from '../services/bluetooth';
import { eegAPI } from '../services/api';
import { useDevice } from '../contexts/DeviceContext';

const SCREEN_WIDTH = Dimensions.get('window').width;

export default function EEGSessionScreen({ navigation }) {
  const { connectedDevice } = useDevice();
  const [isRecording, setIsRecording] = useState(false);
  const [sessionId, setSessionId] = useState(null);
  const [eegData, setEegData] = useState({
    TP9: [],
    AF7: [],
    AF8: [],
    TP10: []
  });
  const [duration, setDuration] = useState(0);
  const [dataPoints, setDataPoints] = useState(0);
  const intervalRef = useRef(null);
  const bufferRef = useRef([]);

  useEffect(() => {
    if (!connectedDevice) {
      navigation.navigate('DeviceConnection');
    }
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [connectedDevice]);

  // Démarrer l'enregistrement
  const startRecording = async () => {
    try {
      // 1. Créer session backend
      const response = await eegAPI.startSession(
        connectedDevice.type,
        connectedDevice.id
      );
      
      setSessionId(response.data.sessionId);
      setIsRecording(true);

      // 2. Démarrer le timer
      intervalRef.current = setInterval(() => {
        setDuration(d => d + 1);
      }, 1000);

      // 3. Démarrer le streaming Bluetooth
      await BluetoothService.startDataStream((data) => {
        // Ajouter au buffer local
        bufferRef.current.push({
          timestamp: data.timestamp,
          channels: {
            [data.channel]: data.value
          },
          quality: 'good'
        });

        // Mettre à jour le graphique
        setEegData(prev => ({
          ...prev,
          [data.channel]: [...prev[data.channel].slice(-100), data.value]
        }));

        setDataPoints(p => p + 1);

        // Envoyer au backend toutes les 10 valeurs
        if (bufferRef.current.length >= 10) {
          sendBufferToBackend();
        }
      });

    } catch (error) {
      console.error('Erreur démarrage:', error);
      alert('Impossible de démarrer l\'enregistrement');
    }
  };

  // Envoyer le buffer au backend
  const sendBufferToBackend = async () => {
    if (bufferRef.current.length === 0) return;

    try {
      // Agréger les données par timestamp
      const aggregated = bufferRef.current.reduce((acc, item) => {
        const existing = acc.find(d => d.timestamp === item.timestamp);
        if (existing) {
          Object.assign(existing.channels, item.channels);
        } else {
          acc.push(item);
        }
        return acc;
      }, []);

      // Envoyer chaque point
      for (const dataPoint of aggregated) {
        await eegAPI.sendData(sessionId, {
          timestamp: dataPoint.timestamp,
          channels: dataPoint.channels,
          quality: dataPoint.quality
        });
      }

      // Vider le buffer
      bufferRef.current = [];

    } catch (error) {
      console.error('Erreur envoi données:', error);
    }
  };

  // Arrêter l'enregistrement
  const stopRecording = async () => {
    try {
      // 1. Arrêter le streaming
      BluetoothService.stopDataStream();

      // 2. Envoyer les dernières données
      await sendBufferToBackend();

      // 3. Terminer la session backend
      const response = await eegAPI.stopSession(sessionId);

      // 4. Nettoyer
      clearInterval(intervalRef.current);
      setIsRecording(false);

      // 5. Naviguer vers les détails
      navigation.navigate('SessionDetails', {
        sessionId: sessionId,
        stats: response.data
      });

    } catch (error) {
      console.error('Erreur arrêt:', error);
      alert('Erreur lors de l\'arrêt de la session');
    }
  };

  // Formater la durée
  const formatDuration = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  };

  // Configuration des graphiques
  const chartConfig = {
    backgroundGradientFrom: '#1E2923',
    backgroundGradientTo: '#08130D',
    color: (opacity = 1) => `rgba(26, 255, 146, ${opacity})`,
    strokeWidth: 2,
    barPercentage: 0.5,
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Session EEG en cours</Text>
        <Text style={styles.deviceName}>{connectedDevice?.name}</Text>
      </View>
      
      {/* Timer et stats */}
      <View style={styles.statsContainer}>
        <View style={styles.statBox}>
          <Text style={styles.statLabel}>Durée</Text>
          <Text style={styles.statValue}>{formatDuration(duration)}</Text>
        </View>
        <View style={styles.statBox}>
          <Text style={styles.statLabel}>Points de données</Text>
          <Text style={styles.statValue}>{dataPoints}</Text>
        </View>
      </View>

      {/* Graphiques des canaux EEG */}
      <View style={styles.chartsContainer}>
        {Object.entries(eegData).map(([channel, data]) => (
          <View key={channel} style={styles.chartWrapper}>
            <Text style={styles.channelLabel}>{channel}</Text>
            {data.length > 0 ? (
              <LineChart
                data={{
                  labels: [],
                  datasets: [{
                    data: data.length > 0 ? data : [0]
                  }]
                }}
                width={SCREEN_WIDTH - 40}
                height={180}
                chartConfig={{
                  ...chartConfig,
                  color: (opacity = 1) => {
                    const colors = {
                      TP9: `rgba(255, 99, 132, ${opacity})`,
                      AF7: `rgba(54, 162, 235, ${opacity})`,
                      AF8: `rgba(255, 206, 86, ${opacity})`,
                      TP10: `rgba(75, 192, 192, ${opacity})`
                    };
                    return colors[channel] || `rgba(255, 255, 255, ${opacity})`;
                  }
                }}
                bezier
                style={styles.chart}
                withDots={false}
                withInnerLines={false}
                withOuterLines={false}
              />
            ) : (
              <View style={styles.noDataContainer}>
                <Text style={styles.noDataText}>En attente de données...</Text>
              </View>
            )}
          </View>
        ))}
      </View>

      {/* Boutons de contrôle */}
      <View style={styles.controlsContainer}>
        {!isRecording ? (
          <TouchableOpacity
            style={[styles.button, styles.startButton]}
            onPress={startRecording}
          >
            <Text style={styles.buttonText}>Démarrer l'enregistrement</Text>
          </TouchableOpacity>
        ) : (
          <TouchableOpacity
            style={[styles.button, styles.stopButton]}
            onPress={stopRecording}
          >
            <Text style={styles.buttonText}>Arrêter l'enregistrement</Text>
          </TouchableOpacity>
        )}
      </View>

      {/* Indicateur de qualité du signal */}
      <View style={styles.qualityContainer}>
        <Text style={styles.qualityLabel}>Qualité du signal</Text>
        <View style={styles.qualityBars}>
          {[1, 2, 3, 4, 5].map(i => (
            <View
              key={i}
              style={[
                styles.qualityBar,
                i <= 4 && styles.qualityBarActive
              ]}
            />
          ))}
        </View>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0D1117',
  },
  header: {
    padding: 20,
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#FFFFFF',
    marginBottom: 8,
  },
  deviceName: {
    fontSize: 16,
    color: '#8B949E',
  },
  statsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    padding: 20,
  },
  statBox: {
    alignItems: 'center',
    backgroundColor: '#161B22',
    padding: 15,
    borderRadius: 10,
    flex: 1,
    marginHorizontal: 5,
  },
  statLabel: {
    fontSize: 12,
    color: '#8B949E',
    marginBottom: 5,
  },
  statValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#58A6FF',
  },
  chartsContainer: {
    padding: 20,
  },
  chartWrapper: {
    marginBottom: 25,
    backgroundColor: '#161B22',
    borderRadius: 10,
    padding: 15,
  },
  channelLabel: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#FFFFFF',
    marginBottom: 10,
  },
  chart: {
    borderRadius: 10,
  },
  noDataContainer: {
    height: 180,
    justifyContent: 'center',
    alignItems: 'center',
  },
  noDataText: {
    color: '#8B949E',
    fontSize: 14,
  },
  controlsContainer: {
    padding: 20,
  },
  button: {
    padding: 18,
    borderRadius: 10,
    alignItems: 'center',
  },
  startButton: {
    backgroundColor: '#238636',
  },
  stopButton: {
    backgroundColor: '#DA3633',
  },
  buttonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: 'bold',
  },
  qualityContainer: {
    padding: 20,
    alignItems: 'center',
  },
  qualityLabel: {
    color: '#8B949E',
    fontSize: 14,
    marginBottom: 10,
  },
  qualityBars: {
    flexDirection: 'row',
    gap: 5,
  },
  qualityBar: {
    width: 8,
    height: 30,
    backgroundColor: '#21262D',
    borderRadius: 4,
  },
  qualityBarActive: {
    backgroundColor: '#238636',
  },
}); 
