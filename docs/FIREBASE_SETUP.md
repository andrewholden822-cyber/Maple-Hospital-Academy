# 🔑 Firebase Setup Guide - Maple Hospital Academy

Step-by-step instructions for configuring Firebase to power Maple Hospital Academy's cloud backend.

---

## 📋 Prerequisites

- Google account with Firebase access
- Active Firebase project (or ability to create one)
- Understanding of NoSQL database concepts
- Text editor for configuration files

---

## 🚀 Step 1: Create a Firebase Project

### 1.1 Go to Firebase Console

1. Navigate to [Firebase Console](https://console.firebase.google.com)
2. Sign in with your Google account
3. Click **"Add project"**

### 1.2 Configure Project Settings

**Project Name:** `Maple-Hospital-Academy`

**Settings:**
- ☑ Enable Google Analytics (optional)
- Accept terms and conditions
- Click **"Create project"**

### 1.3 Wait for Project Creation

Fire base will provision resources. This takes 2-3 minutes.

---

## 🔐 Step 2: Enable Authentication

### 2.1 Set Up Anonymous Authentication

1. In Firebase Console, go to **Build** → **Authentication**
2. Click **"Get started"** or **"Sign-in method"**
3. Select **"Anonymous"** provider
4. Toggle **"Enable"** to ON
5. Click **"Save"**

### 2.2 Enable Custom Token Authentication (Optional)

For advanced integrations with external identity systems:

1. Go to **Authentication** → **Sign-in method**
2. Scroll to **"Custom token"** provider
3. Toggle **"Enable"** to ON
4. Click **"Save"**

**Note:** Custom token generation requires backend implementation

---

## 💾 Step 3: Set Up Firestore Database

### 3.1 Create Firestore Database

1. In Firebase Console, go to **Build** → **Firestore Database**
2. Click **"Create database"**
3. Select region (closest to your users):
   - `us-east1` (US East)
   - `europe-west1` (Europe)
   - `asia-southeast1` (Asia)
4. Click **"Next"**

### 3.2 Configure Security Rules

1. **Security Rules Setup:**
   - Select **"Start in test mode"** (for development)
   - Click **"Create"**

2. **Update Rules for Production:**
   - Go to **Firestore Database** → **Rules**
   - Replace default rules with secure configuration (see below)
   - Click **"Publish"**

### 3.3 Secure Firestore Rules

Replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow authenticated users to access only their own progress data
    match /artifacts/{appId}/users/{uid}/courseProgress/{document=**} {
      allow read, write: if request.auth.uid == uid;
    }
    
    // Deny access to all other paths
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

**Rules Explanation:**
- Users can only read/write their own course progress
- Firestore document path: `artifacts/{appId}/users/{uid}/courseProgress/profile`
- All other paths are blocked (deny by default)
- Each user identified by their Firebase `uid`

---

## 🔑 Step 4: Get Firebase Credentials

### 4.1 Create Web App

1. In Firebase Console, go to **Project Overview**
2. Click **"Add app"** → **"Web"** icon
3. App nickname: `Maple Hospital Academy`
4. Check **"Also set up Firebase Hosting for this app"** (optional)
5. Click **"Register app"**

### 4.2 Copy Firebase Config

Firebase will display your config object:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

**Copy and save these values** - you'll need them in the next step.

### 4.3 Configure Environment Variables

1. Create `.env.local` in your project root
2. Add Firebase credentials:

```env
REACT_APP_FIREBASE_API_KEY=YOUR_API_KEY
REACT_APP_FIREBASE_AUTH_DOMAIN=YOUR_PROJECT.firebaseapp.com
REACT_APP_FIREBASE_PROJECT_ID=YOUR_PROJECT_ID
REACT_APP_FIREBASE_STORAGE_BUCKET=YOUR_PROJECT.appspot.com
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=YOUR_MESSAGING_SENDER_ID
REACT_APP_FIREBASE_APP_ID=YOUR_APP_ID
```

**⚠️ Important:** Add `.env.local` to `.gitignore` to prevent exposing credentials

---

## 📁 Step 5: Set Up Firestore Data Structure

### 5.1 Create Collections

Firestore will automatically create collections when you save the first document. Use this structure:

```
Firestore Root
└── artifacts/                                    [Collection]
    └── maple-hospital-academy/                  [Document]
        └── users/                               [Collection]
            └── {USER_UID}/                      [Document]
                └── courseProgress/              [Collection]
                    └── profile                  [Document]
                        ├── userName: string
                        ├── activePlayerLevel: number
                        ├── completedLessons: map
                        ├── completedQuizzes: map
                        ├── assignmentsChecked: map
                        ├── finalExamAttempted: boolean
                        ├── finalExamScore: number
                        ├── showCertificate: boolean
                        └── acknowledged: boolean
```

### 5.2 Automatic Document Creation

**The React application creates user documents automatically:**

- First login triggers `onAuthStateChanged`
- `saveProgressToCloud()` function creates initial document
- Subsequent updates use `merge: true` option
- No manual database setup required

### 5.3 Manual Test Document (Optional)

To verify setup, manually create a test document:

1. Go to **Firestore Database**
2. Click **"Start collection"**
3. Collection ID: `artifacts`
4. Click **"Next"**
5. Document ID: `maple-hospital-academy`
6. Add field:
   - Name: `_initialized`
   - Type: Boolean
   - Value: `true`
7. Click **"Save"**

This confirms your Firestore access is working.

---

## 🌍 Step 6: Configure Authorized Domains

### 6.1 Add Authentication Domains

1. Go to **Authentication** → **Settings** → **Authorized domains**
2. Add your deployment domains:

**For GitHub Pages:**
```
andrewholdden822-cyber.github.io
```

**For Vercel:**
```
maple-hospital-academy.vercel.app
```

**For Netlify:**
```
maple-hospital-academy.netlify.app
```

**For Local Development:**
```
localhost:3000
127.0.0.1:3000
```

3. Click **"Add domain"** for each URL

### 6.2 Verify Domain Configuration

- Firebase automatically verifies domains
- Check **"Domain verified"** status
- Wait 5-10 minutes for global propagation

---

## 🧪 Step 7: Test Firebase Connection

### 7.1 Test Authentication

```bash
# Start development server
npm start

# Open browser console (F12)
# You should see: "Firebase initialized successfully"
```

### 7.2 Test Firestore Write

1. Load the app at `http://localhost:3000`
2. Complete onboarding (enter username and level)
3. Open **browser DevTools** → **Console**
4. Go to Firebase Console → **Firestore Database**
5. Check if user document was created:
   - Navigate to: `artifacts/maple-hospital-academy/users/`
   - You should see a document with a long ID (the user's Firebase UID)

### 7.3 Verify Data Structure

```javascript
// Expected document structure:
{
  userName: "Your Username",
  activePlayerLevel: 1,
  completedLessons: {},
  completedQuizzes: {},
  assignmentsChecked: {},
  finalExamAttempted: false,
  finalExamScore: 0,
  showCertificate: false,
  acknowledged: true
}
```

---

## 🔒 Step 8: Security Best Practices

### 8.1 Enable API Key Restrictions

1. Go to **Build** → **Settings** → **Service accounts**
2. Click on **"User API Key Credentials"**
3. For each API key, click **"Edit"**
4. Under **"API restrictions":**
   - Select **"Restrict key"**
   - Enable only: `Cloud Firestore API` and `Firebase Authentication API`
5. Click **"Save"**

### 8.2 Enable Application Restrictions

1. In API key settings, find **"Application restrictions"**
2. Select **"HTTP referrers (web sites)"**
3. Add your domains:
   ```
   https://andrewholden822-cyber.github.io/Maple-Hospital-Academy
   https://localhost:3000
   ```
4. Click **"Save"**

### 8.3 Monitor Usage

1. Go to **Firestore Database** → **Usage**
2. Set up budget alerts:
   - Click **"Set up billing alerts"**
   - Enter alert threshold (e.g., $5 per month)
   - Add email address for notifications

---

## 📊 Step 9: Monitor and Debug

### 9.1 View Real-Time Logs

1. Go to **Firestore Database** → **Logs**
2. Filter by:
   - Time range
   - Operation type (read, write, delete)
   - Success/failure status

### 9.2 Common Issues and Solutions

**Issue: Permission denied on write**
```
Error: Missing or insufficient permissions
```

**Solution:**
1. Check Firestore security rules allow authenticated users
2. Verify user is authenticated (`request.auth.uid` is set)
3. Check document path matches rule patterns

**Issue: User document not creating**
```
Error: Failed to save progress to cloud storage
```

**Solution:**
1. Verify Firebase credentials in `.env.local`
2. Check Firebase is initialized before user operations
3. Review browser console for detailed error messages
4. Ensure Firestore database is created and active

**Issue: Slow database reads/writes**
```
Response times > 1000ms
```

**Solution:**
1. Check network connection
2. Verify Firestore region is geographically close
3. Monitor quota usage in Firebase Console
4. Add database indexes for complex queries (if needed)

### 9.3 Enable Debug Logging

Add to your React component:

```javascript
// Enable Firebase debug logging
import { enableLogging } from 'firebase/firestore';
enableLogging(true);
```

---

## 🚀 Step 10: Scale and Optimize

### 10.1 Database Indexing

For complex queries, Firebase suggests creating indexes:

1. Go to **Firestore Database** → **Indexes**
2. Create indexes for frequently queried fields
3. Example: `(userId, timestamp)` compound index

### 10.2 Backup and Recovery

1. Go to **Firestore Database** → **Backups**
2. Create manual backup:
   - Click **"Create backup"**
   - Select collections to backup
   - Choose retention period
3. Automatic backups enabled by default

### 10.3 Export Data

```bash
# Export Firestore data to JSON
gcloud firestore export gs://your-bucket/firestore-export
```

---

## 📝 Configuration Checklist

✅ Firebase project created
✅ Anonymous authentication enabled
✅ Firestore database created
✅ Security rules configured
✅ Firebase credentials obtained
✅ Environment variables set (`.env.local`)
✅ `.env.local` added to `.gitignore`
✅ Authorized domains configured
✅ Local connection tested
✅ User document creation verified
✅ API key restrictions applied
✅ Backup strategy configured

---

## 📚 Additional Resources

- [Firebase Documentation](https://firebase.google.com/docs)
- [Firestore Security Rules Guide](https://firebase.google.com/docs/firestore/security/get-started)
- [Firebase Authentication Guide](https://firebase.google.com/docs/auth)
- [Firebase Console](https://console.firebase.google.com)
- [Firebase Emulator Suite](https://firebase.google.com/docs/emulator-suite) (for local development)

---

**Last Updated:** June 2026 | Maple Hospital Academy Firebase Setup