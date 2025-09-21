# CemSmartOps AI - Complete Setup Guide

## 🚀 Quick Start

This guide will help you set up the complete CemSmartOps AI development environment with all required services.

## 📋 Prerequisites

- **Flutter SDK**: 3.32.8 or higher
- **Dart SDK**: 3.0.0 or higher
- **Android Studio**: Latest version with Android SDK
- **VS Code**: With Flutter and Dart extensions
- **Git**: For version control
- **Node.js**: 18+ (for Firebase CLI)
- **Python**: 3.8+ (for Google Cloud SDK)

## 🔧 Development Environment Setup

### 1. Flutter Installation

```bash
# Download Flutter SDK
git clone https://github.com/flutter/flutter.git -b stable
export PATH="$PATH:`pwd`/flutter/bin"

# Verify installation
flutter doctor
```

### 2. Android Setup

```bash
# Install Android SDK via Android Studio
# Set environment variables
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Accept Android licenses
flutter doctor --android-licenses
```

### 3. Project Dependencies

```bash
# Install Flutter dependencies
flutter pub get

# Install additional tools
flutter pub global activate flutterfire_cli
```

## 🔥 Firebase Setup

### 1. Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Create a project"
3. Enter project name: `cemsmartops-ai`
4. Enable Google Analytics (recommended)
5. Choose or create a Google Analytics account

### 2. Enable Firebase Services

#### Authentication
```bash
# Enable Authentication
# Go to Authentication > Sign-in method
# Enable: Email/Password, Google, Microsoft, Twitter
```

#### Firestore Database
```bash
# Create Firestore database
# Go to Firestore Database > Create database
# Choose: Start in test mode (for development)
# Select location: us-central1 (or your preferred region)
```

#### Cloud Storage
```bash
# Enable Cloud Storage
# Go to Storage > Get started
# Choose: Start in test mode
# Select location: us-central1
```

#### Cloud Messaging
```bash
# Enable Cloud Messaging
# Go to Cloud Messaging
# No additional setup required for basic functionality
```

### 3. Configure Firebase for Flutter

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login to Firebase
firebase login

# Configure FlutterFire
flutterfire configure --project=cemsmartops-ai

# This will:
# - Download google-services.json for Android
# - Update firebase_options.dart
# - Configure platform-specific settings
```

### 4. Firebase Security Rules

#### Firestore Rules
```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read/write their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Operations data - authenticated users only
    match /operations/{document} {
      allow read, write: if request.auth != null;
    }
    
    // Maintenance records - authenticated users only
    match /maintenance/{document} {
      allow read, write: if request.auth != null;
    }
    
    // Chat sessions - users can only access their own
    match /chat_sessions/{sessionId} {
      allow read, write: if request.auth != null && 
        resource.data.userId == request.auth.uid;
    }
    
    // Reports - authenticated users only
    match /reports/{document} {
      allow read, write: if request.auth != null;
    }
  }
}
```

#### Storage Rules
```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    // Users can upload their own profile images
    match /users/{userId}/profile/{allPaths=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Operations documents - authenticated users only
    match /operations/{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### 5. Firebase Configuration Files

#### Android Configuration
```json
// android/app/google-services.json
{
  "project_info": {
    "project_number": "YOUR_PROJECT_NUMBER",
    "project_id": "cemsmartops-ai",
    "storage_bucket": "cemsmartops-ai.appspot.com"
  },
  "client": [
    {
      "client_info": {
        "mobilesdk_app_id": "YOUR_APP_ID",
        "android_client_info": {
          "package_name": "com.cemsmartops.ai"
        }
      }
    }
  ]
}
```

## 🤖 Google AI SDK Setup

### 1. Enable Google AI API

1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Create a new project or select existing
3. Enable the Generative AI API
4. Generate an API key

### 2. Configure API Key

```dart
// lib/config/ai_config.dart
class AIConfig {
  static const String apiKey = 'YOUR_GOOGLE_AI_API_KEY';
  static const String model = 'gemini-pro';
  static const String version = 'v1beta';
}
```

### 3. Environment Variables

```bash
# Create .env file
echo "GOOGLE_AI_API_KEY=your_api_key_here" > .env

# Add to .gitignore
echo ".env" >> .gitignore
```

### 4. AI Service Implementation

```dart
// lib/services/ai_service.dart
import 'package:google_generative_ai/google_generative_ai.dart';

class AIService {
  static final GenerativeModel _model = GenerativeModel(
    model: 'gemini-pro',
    apiKey: AIConfig.apiKey,
  );

  static Future<String> generateResponse(String prompt) async {
    try {
      final content = [Content.text(prompt)];
      final response = await _model.generateContent(content);
      return response.text ?? 'No response generated';
    } catch (e) {
      return 'Error: ${e.toString()}';
    }
  }
}
```

## ☁️ Google Cloud Vertex AI Setup

### 1. Enable Vertex AI API

```bash
# Install Google Cloud SDK
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Initialize gcloud
gcloud init

# Enable Vertex AI API
gcloud services enable aiplatform.googleapis.com
```

### 2. Create Service Account

```bash
# Create service account
gcloud iam service-accounts create cemsmartops-ai-sa \
    --description="Service account for CemSmartOps AI" \
    --display-name="CemSmartOps AI Service Account"

# Grant necessary permissions
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:cemsmartops-ai-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"

# Create and download key
gcloud iam service-accounts keys create key.json \
    --iam-account=cemsmartops-ai-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com
```

### 3. Vertex AI Configuration

```dart
// lib/config/vertex_ai_config.dart
class VertexAIConfig {
  static const String projectId = 'YOUR_PROJECT_ID';
  static const String location = 'us-central1';
  static const String model = 'gemini-pro';
  static const String serviceAccountKeyPath = 'assets/key.json';
}
```

### 4. Vertex AI Service Implementation

```dart
// lib/services/vertex_ai_service.dart
import 'package:googleapis/aiplatform/v1.dart';
import 'package:googleapis_auth/auth_io.dart';

class VertexAIService {
  static Future<String> generateContent(String prompt) async {
    try {
      // Initialize authentication
      final credentials = await _getCredentials();
      final client = await clientViaServiceAccount(credentials, [
        'https://www.googleapis.com/auth/cloud-platform',
      ]);

      // Create AI Platform client
      final aiPlatform = AiPlatformApi(client);
      
      // Generate content using Vertex AI
      final request = GoogleCloudAiplatformV1PredictRequest(
        instances: [
          {
            'prompt': prompt,
          }
        ],
      );

      final response = await aiPlatform.projects.locations.endpoints.predict(
        request,
        'projects/${VertexAIConfig.projectId}/locations/${VertexAIConfig.location}/endpoints/${VertexAIConfig.model}',
      );

      return response.predictions?.first['content'] ?? 'No response';
    } catch (e) {
      return 'Error: ${e.toString()}';
    }
  }

  static Future<ServiceAccountCredentials> _getCredentials() async {
    // Load service account key
    final keyFile = await rootBundle.loadString(VertexAIConfig.serviceAccountKeyPath);
    final keyJson = json.decode(keyFile);
    return ServiceAccountCredentials.fromJson(keyJson);
  }
}
```

## 🔐 Authentication Setup

### 1. Google Sign-In Configuration

```bash
# Get SHA-1 fingerprint for Android
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android

# Add SHA-1 to Firebase Console
# Go to Project Settings > General > Your apps > Android app
# Add SHA-1 fingerprint
```

### 2. Microsoft SSO Setup

```dart
// lib/config/oauth_config.dart
class OAuthConfig {
  static const String microsoftClientId = 'YOUR_MICROSOFT_CLIENT_ID';
  static const String microsoftTenantId = 'YOUR_TENANT_ID';
  static const String microsoftRedirectUri = 'YOUR_REDIRECT_URI';
}
```

### 3. Twitter SSO Setup

```dart
class OAuthConfig {
  static const String twitterApiKey = 'YOUR_TWITTER_API_KEY';
  static const String twitterApiSecret = 'YOUR_TWITTER_API_SECRET';
  static const String twitterRedirectUri = 'YOUR_REDIRECT_URI';
}
```

## 📱 Build Configuration

### 1. Android Build Setup

```gradle
// android/app/build.gradle
android {
    compileSdkVersion 34
    defaultConfig {
        applicationId "com.cemsmartops.ai"
        minSdkVersion 23
        targetSdkVersion 34
        versionCode 1
        versionName "1.0.0"
    }
    
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### 2. Release Keystore Setup

```bash
# Generate release keystore
keytool -genkey -v -keystore cemsmartops-ai-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias cemsmartops-ai

# Create key.properties
echo "storePassword=YOUR_STORE_PASSWORD" > android/key.properties
echo "keyPassword=YOUR_KEY_PASSWORD" >> android/key.properties
echo "keyAlias=cemsmartops-ai" >> android/key.properties
echo "storeFile=cemsmartops-ai-release-key.jks" >> android/key.properties
```

## 🚀 Deployment

### 1. Build Release APK

```bash
# Build release APK
flutter build apk --release

# Build App Bundle for Play Store
flutter build appbundle --release
```

### 2. Firebase Hosting (for web version)

```bash
# Initialize Firebase hosting
firebase init hosting

# Build web version
flutter build web

# Deploy to Firebase hosting
firebase deploy
```

## 🔍 Testing

### 1. Unit Tests

```bash
# Run unit tests
flutter test
```

### 2. Integration Tests

```bash
# Run integration tests
flutter test integration_test/
```

### 3. Firebase Test Lab

```bash
# Run tests on Firebase Test Lab
gcloud firebase test android run \
    --app build/app/outputs/flutter-apk/app-debug.apk \
    --test build/app/outputs/flutter-apk/app-debug-androidTest.apk
```

## 📊 Monitoring & Analytics

### 1. Firebase Analytics

```dart
// lib/services/analytics_service.dart
import 'package:firebase_analytics/firebase_analytics.dart';

class AnalyticsService {
  static final FirebaseAnalytics _analytics = FirebaseAnalytics.instance;

  static Future<void> logEvent(String name, Map<String, dynamic> parameters) async {
    await _analytics.logEvent(name: name, parameters: parameters);
  }
}
```

### 2. Crashlytics

```bash
# Enable Crashlytics in Firebase Console
# Add to pubspec.yaml
dependencies:
  firebase_crashlytics: ^3.4.9
```

## 🔧 Troubleshooting

### Common Issues

1. **Firebase Configuration Error**
   ```bash
   # Regenerate firebase_options.dart
   flutterfire configure --project=cemsmartops-ai
   ```

2. **Google AI API Key Error**
   ```bash
   # Verify API key in Google AI Studio
   # Check quota and billing
   ```

3. **Build Errors**
   ```bash
   # Clean and rebuild
   flutter clean
   flutter pub get
   flutter build apk --release
   ```

4. **Authentication Issues**
   ```bash
   # Verify SHA-1 fingerprints
   # Check OAuth configuration
   # Verify redirect URIs
   ```

## 📚 Additional Resources

- [Flutter Documentation](https://docs.flutter.dev/)
- [Firebase Documentation](https://firebase.google.com/docs)
- [Google AI SDK Documentation](https://ai.google.dev/docs)
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [Google Cloud SDK Documentation](https://cloud.google.com/sdk/docs)

## 🆘 Support

For technical support and questions:
- **Email**: support@cemsmartops.ai
- **Documentation**: [Internal Wiki]
- **Issues**: [GitHub Issues]

---

**CemSmartOps AI** - Complete Setup Guide
*Last Updated: September 2024*
