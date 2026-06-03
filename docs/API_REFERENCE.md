# 🔌 API Reference - Maple Hospital Academy

Technical documentation for developers working with Maple Hospital Academy's architecture, components, and Firebase integration.

---

## 📚 Table of Contents

1. [React Components](#react-components)
2. [Firebase Firestore Operations](#firebase-firestore-operations)
3. [State Management](#state-management)
4. [Authentication Flow](#authentication-flow)
5. [Error Handling](#error-handling)
6. [Data Structures](#data-structures)

---

## 🏗️ React Components

### App Component

**Main application component** - `src/App.js`

**Responsibilities:**
- Firebase initialization and authentication
- User progress state management
- Tab navigation between course sections
- Quiz and exam logic
- Cloud data synchronization

**Props:** None (root component)

**Key State Variables:**

```javascript
// Authentication
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(true);

// User Profile
const [userName, setUserName] = useState("");
const [activePlayerLevel, setActivePlayerLevel] = useState(1);
const [acknowledged, setAcknowledged] = useState(false);

// Progress Tracking
const [completedLessons, setCompletedLessons] = useState({});
const [completedQuizzes, setCompletedQuizzes] = useState({});
const [assignmentsChecked, setAssignmentsChecked] = useState({});

// Exam Progress
const [finalExamAttempted, setFinalExamAttempted] = useState(false);
const [finalExamScore, setFinalExamScore] = useState(0);
const [showCertificate, setShowCertificate] = useState(false);
```

---

## 🔥 Firebase Firestore Operations

### Authentication Setup

**Function:** `setupAuth()` (in useEffect)

**Purpose:** Initialize Firebase authentication

**Process:**
1. Check for custom token in `__initial_auth_token` global
2. If present, sign in with custom token
3. If not, sign in anonymously
4. Set up auth state listener

**Code:**
```javascript
const setupAuth = async () => {
  try {
    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
      await signInWithCustomToken(auth, __initial_auth_token);
    } else {
      await signInAnonymously(auth);
    }
  } catch (err) {
    console.error("Authentication setup failed", err);
  }

  unsubscribeAuth = onAuthStateChanged(auth, (authUser) => {
    setUser(authUser);
    // Handle logout by resetting states
  });
};
```

### Load Progress from Cloud

**Function:** `useEffect` with Firestore real-time listener

**Purpose:** Fetch user progress data from Firestore

**Path:** `/artifacts/{appId}/users/{uid}/courseProgress/profile`

**Code:**
```javascript
useEffect(() => {
  if (!user) return;

  const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'courseProgress', 'profile');
  
  const unsubscribe = onSnapshot(docRef, (docSnap) => {
    if (docSnap.exists()) {
      const data = docSnap.data();
      setUserName(data.userName || "");
      setActivePlayerLevel(data.activePlayerLevel || 1);
      setCompletedLessons(data.completedLessons || {});
      // ... set other states
    }
    setLoading(false);
  }, (error) => {
    console.error("Error reading progress document:", error);
    setLoading(false);
  });

  return () => unsubscribe();
}, [user]);
```

### Save Progress to Cloud

**Function:** `saveProgressToCloud(updates)`

**Purpose:** Update user progress in Firestore

**Parameters:**
- `updates` (object) - Fields to update

**Code:**
```javascript
const saveProgressToCloud = async (updates) => {
  if (!user) return;
  const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'courseProgress', 'profile');
  
  const stateSnapshot = {
    userName: updates.userName !== undefined ? updates.userName : userName,
    activePlayerLevel: updates.activePlayerLevel !== undefined ? updates.activePlayerLevel : activePlayerLevel,
    completedLessons: updates.completedLessons !== undefined ? updates.completedLessons : completedLessons,
    completedQuizzes: updates.completedQuizzes !== undefined ? updates.completedQuizzes : completedQuizzes,
    assignmentsChecked: updates.assignmentsChecked !== undefined ? updates.assignmentsChecked : assignmentsChecked,
    finalExamAttempted: updates.finalExamAttempted !== undefined ? updates.finalExamAttempted : finalExamAttempted,
    finalExamScore: updates.finalExamScore !== undefined ? updates.finalExamScore : finalExamScore,
    showCertificate: updates.showCertificate !== undefined ? updates.showCertificate : showCertificate,
    acknowledged: updates.acknowledged !== undefined ? updates.acknowledged : acknowledged
  };

  try {
    await setDoc(docRef, stateSnapshot, { merge: true });
  } catch (err) {
    console.error("Failed to save progress to cloud storage:", err);
  }
};
```

**Usage Examples:**
```javascript
// Update single field
await saveProgressToCloud({ userName: "NewName" });

// Update multiple fields
await saveProgressToCloud({
  completedQuizzes: newQuizzes,
  finalExamScore: 9
});

// Merge with existing data (merge: true in setDoc)
await saveProgressToCloud({ activePlayerLevel: 100 });
```

---

## 🧠 State Management

### User Profile State

```javascript
interface UserProfile {
  userName: string;                    // Player's Roblox username
  activePlayerLevel: number;           // Current in-game level (1-9999)
  acknowledged: boolean;               // Accepted disclaimer
}
```

### Progress Tracking State

```javascript
interface ProgressState {
  completedLessons: {
    [lessonId: string]: boolean;       // Lesson viewed (true/false)
  };
  completedQuizzes: {
    [unitId: string]: number;          // Quiz score (0-3)
  };
  assignmentsChecked: {
    [assignmentId: string]: boolean;   // Assignment completed (true/false)
  };
}
```

### Exam State

```javascript
interface ExamState {
  finalExamAttempted: boolean;         // Has taken exam
  finalExamScore: number;              // Score (0-10)
  showCertificate: boolean;            // Eligible for certificate
}
```

---

## 🔐 Authentication Flow

### Sequence Diagram

```
User Opens App
    ↓
[useEffect: setupAuth()]
    ↓
Check for __initial_auth_token
    ↓
  ├─ YES → signInWithCustomToken()
  └─ NO → signInAnonymously()
    ↓
onAuthStateChanged triggered
    ↓
setUser(authUser)
    ↓
[useEffect: Load Progress]
    ↓
Query Firestore: /artifacts/{appId}/users/{uid}/courseProgress/profile
    ↓
Update React state with user data
    ↓
setLoading(false) → Component renders
```

### Sign Out Flow

```javascript
const handleLogOut = async () => {
  setLoading(true);
  try {
    await signOut(auth);
    // Re-sign in anonymously for fresh session
    await signInAnonymously(auth);
  } catch (err) {
    console.error("Logout process error:", err);
    setLoading(false);
  }
};
```

---

## 📊 Quiz Logic

### Start Quiz

```javascript
const handleStartQuiz = () => {
  setQuizActive(true);
  setCurrentQuizAnswers({});
  setQuizSubmitted(false);
};
```

### Record Answer

```javascript
const handleQuizAnswer = (qIdx, optIdx) => {
  setCurrentQuizAnswers(prev => ({
    ...prev,
    [qIdx]: optIdx
  }));
};
```

### Submit Quiz

```javascript
const handleSubmitQuiz = (unit) => {
  let score = 0;
  
  // Calculate score
  unit.quiz.questions.forEach((q, idx) => {
    if (currentQuizAnswers[idx] === q.answer) {
      score += 1;
    }
  });
  
  // Save to state and Firebase
  const nextQuizzes = { ...completedQuizzes, [unit.id]: score };
  setCompletedQuizzes(nextQuizzes);
  setQuizSubmitted(true);
  
  await saveProgressToCloud({ completedQuizzes: nextQuizzes });
};
```

---

## 🏆 Final Exam Logic

### Start Exam

```javascript
const handleStartExam = () => {
  setExamAnswers({});
  setExamSubmitted(false);
};
```

### Submit Exam & Generate Certificate

```javascript
const handleSubmitExam = () => {
  let correct = 0;
  
  // Count correct answers
  FINAL_EXAM_QUESTIONS.forEach((q, idx) => {
    if (examAnswers[idx] === q.answer) {
      correct++;
    }
  });
  
  // Determine pass/fail (8/10 required)
  const pass = correct >= 8;
  
  // Update states
  setFinalExamScore(correct);
  setFinalExamAttempted(true);
  setExamSubmitted(true);
  
  if (pass) {
    setShowCertificate(true);
  }
  
  // Save to Firebase
  await saveProgressToCloud({
    finalExamScore: correct,
    finalExamAttempted: true,
    showCertificate: pass
  });
};
```

---

## 🔒 Error Handling

### Firebase Authentication Errors

```javascript
try {
  await signInAnonymously(auth);
} catch (err) {
  if (err.code === 'auth/operation-not-allowed') {
    console.error('Anonymous authentication not enabled');
  } else if (err.code === 'auth/network-request-failed') {
    console.error('Network error - check internet connection');
  } else {
    console.error('Authentication error:', err.message);
  }
}
```

### Firestore Errors

```javascript
try {
  await setDoc(docRef, data, { merge: true });
} catch (err) {
  if (err.code === 'permission-denied') {
    console.error('Insufficient permissions - check security rules');
  } else if (err.code === 'not-found') {
    console.error('Document not found');
  } else {
    console.error('Firestore error:', err.message);
  }
}
```

---

## 🏅 Level Progression Helper

### Get Progress Percentage

```javascript
const getFunctionalProgressPercentage = () => {
  const lvl = activePlayerLevel || 1;
  
  // Cap at 100 for percentage display
  if (lvl >= 100) return 100;
  
  // Calculate percentage (max 100 for UI purposes)
  return Math.round((lvl / 100) * 100);
};
```

### Handle Level Change

```javascript
const handleLevelChange = (newVal) => {
  if (newVal === "") {
    setActivePlayerLevel("");
    return;
  }
  
  const lvl = parseInt(newVal, 10);
  
  // Validate: 1-9999 range
  if (!isNaN(lvl) && lvl >= 1 && lvl <= 9999) {
    setActivePlayerLevel(lvl);
    saveProgressToCloud({ activePlayerLevel: lvl });
  }
};
```

---

## 🔓 Unit Locking Logic

### Check if Unit is Locked

```javascript
const isUnitLocked = (unitIndex) => {
  // Unit 0 (Unit 1) is always unlocked
  if (unitIndex === 0) return false;
  
  // Check if previous unit's quiz is passed
  const previousUnit = COURSE_DATA.units[unitIndex - 1];
  
  // Locked if quiz not taken OR score < 2/3
  return !completedQuizzes[previousUnit.id] || 
         completedQuizzes[previousUnit.id] < 2;
};
```

---

## 📋 Data Structures

### Course Data Structure

```javascript
const COURSE_DATA = {
  units: [
    {
      id: "unit-1",
      title: "Unit 1: Hospital Onboarding & Core Mechanics",
      difficulty: "Beginner",
      description: "...",
      lessons: [
        {
          id: "u1-l1",
          title: "1.1 Navigating Maple Hospital",
          icon: "Activity",
          content: "..."
        }
      ],
      assignments: [
        {
          id: "u1-a1",
          text: "Do a full lap around Maple Hospital..."
        }
      ],
      quiz: {
        title: "Unit 1 Certification Quiz",
        questions: [
          {
            q: "On which floor is the Lab and Pharmacy located?",
            options: ["Floor 1", "Floor 2", "Floor 3", "The Basement"],
            answer: 2  // Index of correct answer
          }
        ]
      }
    }
  ]
};
```

### Firestore Document Schema

```javascript
// Path: /artifacts/maple-hospital-academy/users/{uid}/courseProgress/profile

{
  userName: "PlayerName",
  activePlayerLevel: 50,
  completedLessons: {
    "u1-l1": true,
    "u1-l2": true,
    "u2-l1": false
  },
  completedQuizzes: {
    "unit-1": 3,  // Score: 3/3
    "unit-2": 2   // Score: 2/3
  },
  assignmentsChecked: {
    "u1-a1": true,
    "u1-a2": false,
    "u1-a3": true
  },
  finalExamAttempted: false,
  finalExamScore: 0,
  showCertificate: false,
  acknowledged: true
}
```

---

## 🔧 Environment Variables

### Required Variables

```env
REACT_APP_FIREBASE_API_KEY=<your-api-key>
REACT_APP_FIREBASE_AUTH_DOMAIN=<your-project>.firebaseapp.com
REACT_APP_FIREBASE_PROJECT_ID=<your-project-id>
REACT_APP_FIREBASE_STORAGE_BUCKET=<your-bucket>.appspot.com
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=<your-sender-id>
REACT_APP_FIREBASE_APP_ID=<your-app-id>
```

### Optional Global Variables

```javascript
// In HTML head or injected at runtime
window.__firebase_config = JSON.stringify({...});
window.__initial_auth_token = "jwt-token";
window.__app_id = "maple-hospital-academy";
```

---

## 📡 Network Requests

### Real-Time Listener Behavior

- Connects to Firestore WebSocket
- Auto-reconnects on network loss
- Queues updates while offline
- Syncs on reconnection

### Debouncing Saves

Currently no debouncing - each state change triggers a save. For high-frequency updates, consider:

```javascript
const debouncedSave = useMemo(
  () => debounce((updates) => saveProgressToCloud(updates), 1000),
  []
);
```

---

## 🎨 UI/UX Constants

### Icons (from lucide-react)

```javascript
import {
  BookOpen,
  CheckSquare,
  Award,
  HeartPulse,
  // ... 20+ icons
} from 'lucide-react';
```

### Color Scheme

```
Primary: teal (#14b8a6, #0d9488)
Secondary: emerald (#10b981)
Accent: amber (#f59e0b), purple (#a855f7)
Background: slate-950 (#030712), slate-900 (#0f172a)
Text: slate-100 (#f1f5f9), slate-300 (#cbd5e1)
```

---

## 📚 Additional Resources

- [Firebase JavaScript SDK](https://firebase.google.com/docs/web/setup)
- [React Hooks Documentation](https://react.dev/reference/react)
- [Firestore Real-time Listeners](https://firebase.google.com/docs/firestore/query-data/listen)
- [Firebase Security Rules](https://firebase.google.com/docs/firestore/security/rules-query)

---

**Last Updated:** June 2026 | Maple Hospital Academy API Reference v1.0