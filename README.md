{
  "name": "sionohmair-insight-frontend",
  "version": "2.0.0",
  "description": "Application mobile Sionohmair Insight - Développement de conscience spirituelle via EEG",
  "private": true,
  "main": "index.js",
  "scripts": {
    "start": "react-native start",
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "test": "jest",
    "lint": "eslint .",
    "clean": "cd android && ./gradlew clean && cd ../ios && pod deintegrate && pod install"
  },
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.72.7",
    "@react-navigation/native": "^6.1.9",
    "@react-navigation/native-stack": "^6.9.17",
    "react-native-screens": "^3.27.0",
    "react-native-safe-area-context": "^4.7.4",
    "react-native-gesture-handler": "^2.14.0",
    "react-native-reanimated": "^3.6.0",
    "@react-native-async-storage/async-storage": "^1.19.5",
    "axios": "^1.6.2",
    "i18next": "^23.7.6",
    "react-i18next": "^13.5.0",
    "react-native-chart-kit": "^6.12.0",
    "react-native-svg": "^14.0.0",
    "react-native-ble-plx": "^3.1.2",
    "react-native-permissions": "^3.10.1",
    "react-native-vector-icons": "^10.0.2",
    "react-native-modal": "^13.0.1",
    "date-fns": "^3.0.0",
    "zustand": "^4.4.7"
  },
  "devDependencies": {
    "@babel/core": "^7.23.5",
    "@babel/preset-env": "^7.23.5",
    "@babel/runtime": "^7.23.5",
    "@react-native/eslint-config": "^0.73.1",
    "@react-native/metro-config": "^0.73.2",
    "@testing-library/react-native": "^12.4.2",
    "babel-jest": "^29.7.0",
    "babel-plugin-module-resolver": "^5.0.0",
    "eslint": "^8.55.0",
    "jest": "^29.7.0",
    "metro-react-native-babel-preset": "^0.77.0",
    "react-test-renderer": "18.2.0"
  },
  "jest": {
    "preset": "react-native",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js",
      "jsx",
      "json",
      "node"
    ]
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
