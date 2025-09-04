# https-github.com-your-username-secure-social-app
Social media app
const fs = require('fs');
const path = require('path');
const archiver = require('archiver');

const output = fs.createWriteStream('secure-social-app-final.zip');
const archive = archiver('zip', { zlib: { level: 9 } });

output.on('close', function () {
  console.log('✅ secure-social-app-final.zip created, total bytes: ' + archive.pointer());
});

archive.on('error', function (err) {
  throw err;
});

archive.pipe(output);

// Helper to add file
function addFile(filePath, content) {
  archive.append(content, { name: filePath });
}

// ------------------------
// Root files
// ------------------------
addFile('App.tsx', `import React from 'react';
import { AuthProvider } from './src/auth/AuthProvider';
import RootNavigator from './src/navigation/RootNavigator';
export default function App() { return (<AuthProvider><RootNavigator /></AuthProvider>); }`);

addFile('package.json', `{
  "name": "secure-social-app",
  "version": "1.0.0",
  "private": true,
  "scripts": { "start": "expo start" },
  "dependencies": {
    "expo": "~48.0.0",
    "react": "18.2.0",
    "react-native": "0.71.0",
    "react-native-keychain": "^8.1.1",
    "react-native-sodium": "^0.10.0",
    "firebase": "^10.0.0",
    "@react-navigation/native": "^6.1.6",
    "@react-navigation/native-stack": "^6.9.12"
  }
}`);

addFile('.env.template', `FIREBASE_API_KEY=YOUR_FIREBASE_API_KEY
FIREBASE_AUTH_DOMAIN=YOUR_PROJECT.firebaseapp.com
FIREBASE_PROJECT_ID=YOUR_PROJECT_ID
FIREBASE_STORAGE_BUCKET=YOUR_PROJECT.appspot.com
FIREBASE_MESSAGING_SENDER_ID=YOUR_SENDER_ID
FIREBASE_APP_ID=YOUR_APP_ID`);

addFile('app.config.js', `import 'dotenv/config';
export default {
  expo: {
    name: "SecureSocialApp",
    slug: "secure-social-app",
    version: "1.0.0",
    extra: {
      FIREBASE_API_KEY: process.env.FIREBASE_API_KEY,
      FIREBASE_AUTH_DOMAIN: process.env.FIREBASE_AUTH_DOMAIN,
      FIREBASE_PROJECT_ID: process.env.FIREBASE_PROJECT_ID,
      FIREBASE_STORAGE_BUCKET: process.env.FIREBASE_STORAGE_BUCKET,
      FIREBASE_MESSAGING_SENDER_ID: process.env.FIREBASE_MESSAGING_SENDER_ID,
      FIREBASE_APP_ID: process.env.FIREBASE_APP_ID,
    },
  },
};`);

addFile('README.md', `<p align="center">
  <a href="https://expo.dev/@<your-username>/secure-social-app" target="_blank">
    <img src="https://img.shields.io/badge/Open%20in-Expo-4CAF50?style=for-the-badge&logo=expo" alt="Open in Expo"/>
  </a>
</p>
# Secure Social App Skeleton
- AEAD encrypted messages
- Group key rotation
- Key backup & restore
- Firebase placeholders
- Expo ready`);

// ------------------------
// src/utils
// ------------------------
addFile('src/utils/crypto.ts', `import sodium from 'react-native-sodium';
export async function encryptMessage(message, passphrase) {
  const key = await sodium.crypto_generichash(32, passphrase);
  const nonce = await sodium.randombytes_buf(24);
  const ciphertext = await sodium.crypto_aead_xchacha20poly1305_ietf_encrypt(message, null, null, nonce, key);
  return { ciphertext, nonce };
}
export async function decryptMessage(ciphertext, nonce, passphrase) {
  const key = await sodium.crypto_generichash(32, passphrase);
  return await sodium.crypto_aead_xchacha20poly1305_ietf_decrypt(ciphertext, null, nonce, key);
}`);

addFile('src/utils/firestore.ts', `import { initializeApp } from 'firebase/app';
import { getFunctions } from 'firebase/functions';
import { getAuth } from 'firebase/auth';
import Constants from 'expo-constants';
const firebaseConfig = {
  apiKey: Constants.manifest?.extra?.FIREBASE_API_KEY,
  authDomain: Constants.manifest?.extra?.FIREBASE_AUTH_DOMAIN,
  projectId: Constants.manifest?.extra?.FIREBASE_PROJECT_ID,
  storageBucket: Constants.manifest?.extra?.FIREBASE_STORAGE_BUCKET,
  messagingSenderId: Constants.manifest?.extra?.FIREBASE_MESSAGING_SENDER_ID,
  appId: Constants.manifest?.extra?.FIREBASE_APP_ID,
};
const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const functions = getFunctions(app);
`);

// ------------------------
// src/auth
// ------------------------
addFile('src/auth/AuthProvider.tsx', `import React, { createContext, useState } from 'react';
export const AuthContext = createContext(null);
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return <AuthContext.Provider value={{ user }}>{children}</AuthContext.Provider>;
};`);

// ------------------------
// src/navigation
// ------------------------
addFile('src/navigation/RootNavigator.tsx', `import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import Feed from '../screens/Feed';
import Stories from '../screens/Stories';
import Messages from '../screens/Messages';
import Profile from '../screens/Profile';
import GroupManager from '../screens/GroupManager';
import KeyBackupRestore from '../screens/KeyBackupRestore';
import Fingerprints from '../screens/Fingerprints';
const Stack = createNativeStackNavigator();
export default function RootNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Feed">
        <Stack.Screen name="Feed" component={Feed} />
        <Stack.Screen name="Stories" component={Stories} />
        <Stack.Screen name="Messages" component={Messages} />
        <Stack.Screen name="Profile" component={Profile} />
        <Stack.Screen name="GroupManager" component={GroupManager} />
        <Stack.Screen name="KeyBackupRestore" component={KeyBackupRestore} />
        <Stack.Screen name="Fingerprints" component={Fingerprints} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}`);

// ------------------------
// src/screens (clickable)
// ------------------------
const screens = {
  'Feed.tsx': `import React from 'react'; import { View, Text } from 'react-native'; export default function Feed() { return <View><Text>Feed</Text></View>; }`,
  'Stories.tsx': `import React from 'react'; import { View, Text } from 'react-native'; export default function Stories() { return <View><Text>Stories</Text></View>; }`,
  'Profile.tsx': `import React from 'react'; import { View, Text } from 'react-native'; export default function Profile() { return <View><Text>Profile</Text></View>; }`,
  'Fingerprints.tsx': `import React from 'react'; import { View, Text } from 'react-native'; export default function Fingerprints() { return <View><Text>Fingerprints</Text></View>; }`,

  // Messages clickable
  'Messages.tsx': `import React, { useState } from 'react';
import { View, Text, Button, TextInput, Alert, ScrollView } from 'react-native';
import { encryptMessage, decryptMessage } from '../utils/crypto';
export default function Messages() {
  const [message, setMessage] = useState('');
  const [encrypted, setEncrypted] = useState('');
  const [decrypted, setDecrypted] = useState('');
  const handleEncrypt = async () => {
    if (!message) return Alert.alert('Enter a message');
    const { ciphertext, nonce } = await encryptMessage(message, 'test-pass');
    setEncrypted(ciphertext); const decryptedMsg = await decryptMessage(ciphertext, nonce, 'test-pass'); setDecrypted(decryptedMsg);
    Alert.alert('✅ Message encrypted and decrypted!');
  };
  return (<ScrollView contentContainerStyle={{ padding: 20 }}>
    <Text style={{ fontWeight: 'bold', marginBottom: 10 }}>Encrypted Messaging Test</Text>
    <TextInput placeholder="Enter message" value={message} onChangeText={setMessage} style={{ borderWidth: 1, marginBottom: 10, padding: 5 }} />
    <Button title="Encrypt & Decrypt Message" onPress={handleEncrypt} />
    {encrypted ? (<View style={{ marginTop: 20 }}>
      <Text style={{ fontWeight: 'bold' }}>Encrypted:</Text><Text selectable>{encrypted}</Text>
      <Text style={{ fontWeight: 'bold', marginTop: 10 }}>Decrypted:</Text><Text selectable>{decrypted}</Text>
    </View>) : null}
  </ScrollView>); }`,

  // GroupManager clickable
  'GroupManager.tsx': `import React, { useState } from 'react';
import { View, Text, Button, FlatList, Alert } from 'react-native';
import { encryptMessage } from '../utils/crypto';
export default function GroupManager() {
  const [members] = useState([{ id: 'user1', name: 'Alice' }, { id: 'user2', name: 'Bob' }]);
  const [groupKeyVersion, setGroupKeyVersion] = useState(1);
  const [encryptedKeys, setEncryptedKeys] = useState([]);
  const rotateGroupKey = async () => {
    const newGroupKey = 'newRandomGroupKey123';
    const keys = await Promise.all(members.map(async (m) => {
      const { ciphertext } = await encryptMessage(newGroupKey, m.id);
      return { memberId: m.id, encryptedKey: ciphertext };
    }));
    setEncryptedKeys(keys);
    setGroupKeyVersion(prev => prev + 1