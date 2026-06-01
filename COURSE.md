# Maple Hospital Academy - Complete Course Guide

## Course Overview
This course teaches you how to build a **full-stack educational dashboard for Roblox games** using React, Firebase, and Tailwind CSS. You'll learn to create an interactive learning platform with user authentication, cloud synchronization, and interactive educational content.

---

## Unit 1: Project Setup & Architecture (Beginner)

### 1.1 Understanding the Application Structure
**Duration:** 30 minutes

The Maple Hospital Academy dashboard is built with:
- **Frontend Framework:** React with Hooks
- **Backend Services:** Firebase (Authentication + Firestore)
- **Styling:** Tailwind CSS with custom utilities
- **Icons:** Lucide React

**Key Components:**
1. Authentication layer (Firebase Auth)
2. Cloud data persistence (Firestore)
3. Tab-based navigation system
4. Real-time progress tracking

**Environment Setup:**
```
Required environment variables:
- __firebase_config: Firebase configuration JSON
- __app_id: Unique application identifier
- __initial_auth_token: Optional custom auth token
```

### 1.2 Firebase Configuration & Authentication Flow
**Duration:** 45 minutes

The app uses a two-tier authentication system:

**Step 1: Initialize Firebase Services**
```javascript
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
```

**Step 2: Authentication Methods**
- Custom token authentication (for verified users)
- Anonymous authentication (fallback for new users)
- Session management with `onAuthStateChanged`

**Step 3: Implement Sign-Out with Fresh Session**
The logout function clears user data and creates a new anonymous session.

### 1.3 Setting Up Tailwind CSS & Responsive Design
**Duration:** 30 minutes

**Design System:**
- Color Palette: Slate-900 base with teal/emerald accents
- Breakpoints: Mobile-first (sm, md, lg)
- Component Scale: From xs (text) to 2xl (headers)
- Animations: Pulse, transitions, gradient overlays

**Key CSS Utilities Used:**
- `bg-slate-900`, `border-slate-800` (dark theme)
- `text-teal-400`, `text-amber-400` (accent colors)
- `rounded-xl`, `rounded-2xl` (border radius)
- `flex`, `grid grid-cols-1 md:grid-cols-12` (layout)

**Assignment:**
✓ Set up a new React project with Firebase and Tailwind CSS
✓ Create your Firebase project and configure credentials
✓ Implement the authentication hook structure

---

## Unit 2: User State Management & Cloud Persistence (Intermediate)

### 2.1 Managing Complex Application State
**Duration:** 60 minutes

**Core State Variables:**
```javascript
// Navigation states
const [activeTab, setActiveTab] = useState("syllabus");
const [selectedUnit, setSelectedUnit] = useState(0);
const [selectedLesson, setSelectedLesson] = useState(0);

// User profile states
const [userName, setUserName] = useState("");
const [activePlayerLevel, setActivePlayerLevel] = useState(1);

// Progress tracking states
const [completedLessons, setCompletedLessons] = useState({});
const [completedQuizzes, setCompletedQuizzes] = useState({});
const [assignmentsChecked, setAssignmentsChecked] = useState({});
const [finalExamAttempted, setFinalExamAttempted] = useState(false);
```

**State Management Pattern:**
- Local state for UI interactions (activeTab, quizActive)
- Persistent state for user progress (completedLessons, scores)
- Temporary state for form inputs (tempUserName, tempLevel)

### 2.2 Real-Time Firestore Synchronization
**Duration:** 75 minutes

**Database Architecture:**
```
Firestore Path: artifacts/{appId}/users/{uid}/courseProgress/profile

Document Structure:
{
  userName: string,
  activePlayerLevel: number (1-9999),
  completedLessons: Object<lessonId, boolean>,
  completedQuizzes: Object<unitId, score>,
  assignmentsChecked: Object<assignmentId, boolean>,
  finalExamAttempted: boolean,
  finalExamScore: number,
  showCertificate: boolean,
  acknowledged: boolean
}
```

**Firestore Hooks Implementation:**

**Hook 1: Authentication Setup (Must run first)**
```javascript
useEffect(() => {
  const setupAuth = async () => {
    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
      await signInWithCustomToken(auth, __initial_auth_token);
    } else {
      await signInAnonymously(auth);
    }

    const unsubscribe = onAuthStateChanged(auth, (authUser) => {
      setUser(authUser);
      if (!authUser) resetAllStates();
    });
    
    return unsubscribe;
  };
  
  setupAuth();
}, []);
```

**Hook 2: Data Fetching (Waits for auth)**
```javascript
useEffect(() => {
  if (!user) return; // AUTH RULE: Must authenticate first

  const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'courseProgress', 'profile');
  
  const unsubscribe = onSnapshot(docRef, (docSnap) => {
    if (docSnap.exists()) {
      // Populate all states from Firestore
      const data = docSnap.data();
      setUserName(data.userName || "");
      setActivePlayerLevel(data.activePlayerLevel || 1);
      // ... continue for all fields
    }
    setLoading(false);
  });

  return () => unsubscribe();
}, [user]); // Only re-run when user changes
```

### 2.3 Implementing Cloud Sync Helper Function
**Duration:** 45 minutes

**saveProgressToCloud Pattern:**
```javascript
const saveProgressToCloud = async (updates) => {
  if (!user) return; // Safety check

  const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'courseProgress', 'profile');
  
  // Create merged state snapshot
  const stateSnapshot = {
    userName: updates.userName !== undefined ? updates.userName : userName,
    activePlayerLevel: updates.activePlayerLevel !== undefined ? updates.activePlayerLevel : activePlayerLevel,
    // ... merge pattern for all fields
  };

  try {
    await setDoc(docRef, stateSnapshot, { merge: true });
  } catch (err) {
    console.error("Failed to save progress:", err);
  }
};
```

**Usage Pattern:**
```javascript
const handleLessonComplete = (lessonId) => {
  const nextLessons = { ...completedLessons, [lessonId]: true };
  setCompletedLessons(nextLessons);
  saveProgressToCloud({ completedLessons: nextLessons }); // Sync immediately
};
```

**Assignment:**
✓ Set up Firestore rules to protect user data
✓ Implement the full authentication flow
✓ Create saveProgressToCloud for all state updates
✓ Test that data persists across page refreshes

---

## Unit 3: Building Interactive Learning Components (Intermediate)

### 3.1 Creating the Onboarding Screen
**Duration:** 45 minutes

**Purpose:** Collect user info before accessing course content

**Key Elements:**
- Username input (required)
- In-game level input (1-9999, required)
- Educational disclaimer banner
- Form validation and submission

**Component Logic:**
```javascript
const handleOnboardingSubmit = (e) => {
  e.preventDefault();
  const finalName = tempUserName.trim() ? tempUserName : "Maple Veteran";
  let finalLvl = parseInt(tempLevel, 10);
  
  if (isNaN(finalLvl) || finalLvl < 1) finalLvl = 1;
  if (finalLvl > 9999) finalLvl = 9999;
  
  setUserName(finalName);
  setActivePlayerLevel(finalLvl);
  setAcknowledged(true);
  
  saveProgressToCloud({
    userName: finalName,
    activePlayerLevel: finalLvl,
    acknowledged: true
  });
};
```

### 3.2 Building the Navigation System
**Duration:** 60 minutes

**Tab Structure:**
- Syllabus (course overview)
- Classroom (lecture + quiz)
- Assignments (task tracking)
- Final Exam (capstone test)
- Certificate (completion proof)

**Navigation Logic:**
```javascript
const isUnitLocked = (unitIndex) => {
  if (unitIndex === 0) return false; // Unit 1 always unlocked
  const previousUnit = COURSE_DATA.units[unitIndex - 1];
  return !completedQuizzes[previousUnit.id] || completedQuizzes[previousUnit.id] < 2;
};
```

**Access Rules:**
- Unit 1 always accessible
- Each subsequent unit locks until previous unit quiz is passed (score ≥ 2/3)
- Final Exam only unlocks after all 4 unit quizzes completed

### 3.3 Implementing the Lesson Player
**Duration:** 90 minutes

**Features:**
- Display lesson content with formatting
- Navigation between lessons in a unit
- "Mark as Read" button to track completion
- Quiz mode toggle

**Lesson Display:**
```javascript
{!quizActive ? (
  <div>
    <div className="flex items-center justify-between border-b border-slate-800 pb-4 mb-4">
      <h3 className="text-xl font-bold text-slate-100">
        {COURSE_DATA.units[selectedUnit].lessons[selectedLesson].title}
      </h3>
      <button 
        onClick={() => handleLessonComplete(lessonId)}
        className="bg-teal-500 hover:bg-teal-600 text-slate-950 text-xs font-black py-1.5 px-3 rounded-lg"
      >
        Mark as Read (+XP)
      </button>
    </div>
    <div className="prose prose-invert max-w-none text-slate-300 text-sm whitespace-pre-line">
      {COURSE_DATA.units[selectedUnit].lessons[selectedLesson].content}
    </div>
  </div>
) : (
  // Quiz mode UI
)}
```

**Assignment:**
✓ Create custom lesson content for 2 units
✓ Implement responsive sidebar for lesson navigation
✓ Test mark-as-read functionality
✓ Verify completion tracking works across sessions

---

## Unit 4: Quiz & Exam System (Advanced)

### 4.1 Building Interactive Quizzes
**Duration:** 75 minutes

**Quiz Structure:**
```javascript
{
  title: "Unit 1 Certification Quiz",
  questions: [
    {
      q: "Question text?",
      options: ["Option A", "Option B", "Option C", "Option D"],
      answer: 1 // Index of correct answer
    }
  ]
}
```

**Quiz Flow:**
1. User clicks "Take Unit Quiz"
2. Questions display with selectable options
3. User selects answers (stored in `currentQuizAnswers`)
4. User submits quiz
5. Score calculated (correct answers / total questions)
6. Score saved to `completedQuizzes[unitId]`

**Implementation:**
```javascript
const handleSubmitQuiz = (unit) => {
  let score = 0;
  unit.quiz.questions.forEach((q, idx) => {
    if (currentQuizAnswers[idx] === q.answer) score += 1;
  });
  
  const nextQuizzes = { ...completedQuizzes, [unit.id]: score };
  setCompletedQuizzes(nextQuizzes);
  setQuizSubmitted(true);
  saveProgressToCloud({ completedQuizzes: nextQuizzes });
};
```

### 4.2 Creating the Final Examination
**Duration:** 90 minutes

**Final Exam Purpose:**
- Comprehensive test covering all 4 units
- 10 questions total
- Passing score: 8/10 (80%)
- Unlocks certificate upon passing

**Exam Logic:**
```javascript
const handleSubmitExam = () => {
  let correct = 0;
  FINAL_EXAM_QUESTIONS.forEach((q, idx) => {
    if (examAnswers[idx] === q.answer) correct++;
  });
  
  const pass = correct >= 8;
  
  setFinalExamScore(correct);
  setFinalExamAttempted(true);
  setExamSubmitted(true);
  
  if (pass) {
    setShowCertificate(true); // Unlock certificate tab
  }
  
  saveProgressToCloud({
    finalExamScore: correct,
    finalExamAttempted: true,
    showCertificate: pass
  });
};
```

### 4.3 Generating Digital Certificates
**Duration:** 45 minutes

**Certificate Features:**
- Displays student name
- Shows completion level
- Includes current date
- Professional styling with borders
- Only visible after passing exam

**Certificate UI:**
```javascript
{activeTab === 'certificate' && showCertificate && (
  <div className="bg-slate-900 border-2 border-dashed border-purple-500/40 rounded-2xl p-8">
    <div className="w-full max-w-2xl border-4 border-double border-slate-800 p-8 rounded-xl bg-slate-950">
      <h1 className="text-3xl font-black text-slate-100">CERTIFICATE OF GRADUATION</h1>
      <p className="text-xs text-slate-400">Virtual In-Game Clinical Certification</p>
      <div className="my-6">
        <span className="text-[10px] uppercase text-slate-500">Graduating Professional</span>
        <strong className="text-2xl font-serif text-teal-400">{userName}</strong>
      </div>
      <p className="text-xs text-slate-400">Completed all objectives up to Level {activePlayerLevel}</p>
    </div>
  </div>
)}
```

**Assignment:**
✓ Write 10 final exam questions covering all units
✓ Implement scoring logic for exam
✓ Create certificate display component
✓ Test certificate unlock after passing

---

## Unit 5: Advanced Features & Optimization (Expert)

### 5.1 Level System with Infinite Cap
**Duration:** 60 minutes

**Infinite Level Tracking:**
```javascript
const handleLevelChange = (newVal) => {
  if (newVal === "") {
    setActivePlayerLevel("");
    return;
  }
  const lvl = parseInt(newVal, 10);
  if (!isNaN(lvl) && lvl >= 1 && lvl <= 9999) {
    setActivePlayerLevel(lvl);
    saveProgressToCloud({ activePlayerLevel: lvl });
  }
};
```

**Progress Bar Calculation:**
```javascript
const getFunctionalProgressPercentage = () => {
  const lvl = activePlayerLevel || 1;
  if (lvl >= 100) return 100; // Cap display at 100%
  return Math.round((lvl / 100) * 100);
};
```

**Milestone System:**
- Level 5: Nurse Role ✓
- Level 25: Charge Nurse ✓
- Level 50: Senior Doctor ✓
- Level 80: Pharmacist ✓
- Level 100: Prestige Badge ✓

### 5.2 Implementing Real-Time Progress Dashboard
**Duration:** 45 minutes

**Dashboard Metrics:**
```javascript
<div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
  <div className="bg-slate-950 p-4 rounded-lg">
    <span className="text-xs uppercase text-slate-500">Lessons Viewed</span>
    <strong className="text-2xl font-extrabold text-teal-400">
      {Object.keys(completedLessons).length} / 8
    </strong>
  </div>
  
  <div className="bg-slate-950 p-4 rounded-lg">
    <span className="text-xs uppercase text-slate-500">Quizzes Passed</span>
    <strong className="text-2xl font-extrabold text-amber-400">
      {Object.keys(completedQuizzes).filter(k => completedQuizzes[k] >= 2).length} / 4
    </strong>
  </div>
  
  <div className="bg-slate-950 p-4 rounded-lg">
    <span className="text-xs uppercase text-slate-500">Assignments Logged</span>
    <strong className="text-2xl font-extrabold text-purple-400">
      {Object.keys(assignmentsChecked).filter(k => assignmentsChecked[k]).length} / 12
    </strong>
  </div>
</div>
```

### 5.3 Error Handling & Loading States
**Duration:** 45 minutes

**Loading Screen:**
```javascript
if (loading) {
  return (
    <div className="min-h-screen bg-slate-950 flex items-center justify-center">
      <div className="text-center space-y-4">
        <HeartPulse className="w-16 h-16 mx-auto text-teal-400 animate-pulse" />
        <h2 className="text-xl font-bold">Syncing Academy Cloud Progress...</h2>
      </div>
    </div>
  );
}
```

**Error Handling:**
```javascript
const unsubscribe = onSnapshot(
  docRef,
  (docSnap) => { /* success */ },
  (error) => {
    console.error("Error reading progress document:", error);
    setLoading(false);
  }
);
```

**Logout with Fresh Session:**
```javascript
const handleLogOut = async () => {
  setLoading(true);
  try {
    await signOut(auth);
    await signInAnonymously(auth); // Fresh anonymous session
  } catch (err) {
    console.error("Logout error:", err);
    setLoading(false);
  }
};
```

**Assignment:**
✓ Add try-catch error handling to all Firebase calls
✓ Implement retry logic for failed saves
✓ Create error notification UI
✓ Test error scenarios (no internet, invalid token)

---

## Final Project Assignment

### Build Your Own Educational Course

**Requirements:**

1. **Content Creation** (20 points)
   - Write 4 units with 2 lessons each (8 total lessons)
   - Each lesson should be 200+ words
   - Create unit quizzes (3 questions each)
   - Design 10 final exam questions

2. **Code Implementation** (30 points)
   - Implement all authentication flows
   - Set up Firestore with proper security rules
   - Create responsive UI for all 5 tabs
   - Implement quiz/exam scoring logic

3. **Data Persistence** (20 points)
   - Set up Firestore structure
   - Implement all state sync functions
   - Test data persists across page reloads
   - Verify cloud storage works correctly

4. **Features** (20 points)
   - Unit locking system based on quiz completion
   - Progress tracking dashboard
   - Certificate generation
   - Level system (with at least 3 milestones)

5. **Polish** (10 points)
   - Professional styling with Tailwind
   - Responsive design (mobile, tablet, desktop)
   - Proper error handling
   - Loading states during sync

### Grading Rubric

| Criteria | Excellent (5) | Good (4) | Acceptable (3) | Poor (2) |
|----------|---------------|---------|----------------|---------|
| **Content Quality** | Engaging, clear, well-structured | Good content, minor issues | Adequate but basic | Incomplete/unclear |
| **Code Quality** | Clean, documented, follows patterns | Generally good, some issues | Works but messy | Broken/incomplete |
| **Functionality** | All features working perfectly | Most features work | Some features work | Core features missing |
| **Design** | Professional, polished | Good UI/UX | Acceptable appearance | Poor design |
| **Firebase Integration** | Flawless sync, proper security | Works well, minor issues | Basic implementation | Not working |

---

## Key Takeaways

1. **Authentication First:** Always set up auth before fetching data
2. **Firestore Structure:** Use clear paths and security rules
3. **State Management:** Separate UI state from persistent state
4. **Responsive Design:** Test on mobile, tablet, and desktop
5. **Error Handling:** Always wrap async operations in try-catch
6. **Cloud Sync Pattern:** Call saveProgressToCloud after each state change
7. **User Experience:** Provide loading states and clear feedback
8. **Security:** Never expose Firebase config to client; use environment variables

---

## Resources

- [Firebase Documentation](https://firebase.google.com/docs)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [React Hooks API](https://react.dev/reference/react)
- [Lucide Icons](https://lucide.dev)

---

**Course Created:** 2026
**Total Duration:** 12-15 hours
**Difficulty:** Beginner to Advanced
