# Sionohmair-Insight-
Application logiciel Web qui développe l'état de conscience spirituelle Document 1 – Spécifications techniques pour intégration des capteurs


---

1. Objectif

Permettre à l’application Sionohmair de se connecter, récupérer, traiter et exploiter les données issues de capteurs EEG, électromagnétiques et audio vocaux.

2. Dispositifs cibles

Montres connectées (Apple Watch, Fitbit, Muse, Emotiv)

Casques EEG (Muse, OpenBCI, NeuroSky)

Microphones haute fidélité (intégrés ou externes)


3. Interfaces & Protocoles

Bluetooth Low Energy (BLE) pour la plupart des montres et casques

USB / Lightning pour certains casques

API propriétaires ou SDK (ex : Muse SDK)

Formats de données : JSON, binaire ou streams temps réel


4. Fonctionnalités à implémenter

Scanning et appairage des dispositifs

Gestion des connexions simultanées

Synchronisation temporelle entre sources multiples (EEG + audio)

Lecture et décodage des flux de données brutes

Bufferisation et gestion des pertes de paquets

Transmission sécurisée vers le serveur backend


5. Contraintes

Latence minimale (<100 ms) pour temps réel

Respect de la vie privée et conformité RGPD

Compatibilité multiplateforme (Android, iOS)



---

Document 2 – Plan de tests d’intégration matériel/logiciel


---

1. Pré-requis

Jeux de données simulées et réelles

Dispositifs physiques à tester

Environnement de test (smartphones, tablettes)


2. Tests fonctionnels

Découverte et appairage des capteurs

Qualité et continuité des flux reçus

Synchronisation des données EEG et audio

Réaction en cas de perte de signal ou coupure

Sécurité des communications


3. Tests de performance

Mesure de latence de transmission

Consommation batterie en mode connecté

Stabilité lors d’une session prolongée


4. Tests utilisateur

Facilité d’appairage et d’utilisation

Compréhension des messages d’état

Feedbacks sur l’expérience globale


