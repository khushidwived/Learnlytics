# ⚡ [Learnlytics](https://github.com/khushidwived/Learnlytics) — CampusFlow Performance Engine
 
> **Next-generation academic performance tracking for higher education.**  
> A single-file, real-time web application that helps students log study sessions, track productivity metrics, and visualize progress across customizable timeframes.
 
---
 
## 📋 Table of Contents
 
- [Overview](#overview)
- [Live Features](#live-features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Firebase Setup](#firebase-setup)
- [Authentication](#authentication)
- [Data Model](#data-model)
- [Core Modules](#core-modules)
  - [Auth & Session Management](#1-auth--session-management)
  - [Real-Time Sync Engine](#2-real-time-sync-engine)
  - [Time Engine](#3-time-engine)
  - [Task Management](#4-task-management)
  - [Render & Filter Pipeline](#5-render--filter-pipeline)
  - [Metrics Calculator](#6-metrics-calculator)
  - [Chart Visualizations](#7-chart-visualizations)
  - [Spatial UI Effects](#8-spatial-ui-effects)
- [UI Components](#ui-components)
- [CSS Design System](#css-design-system)
- [Event Listeners & Interactions](#event-listeners--interactions)
- [Error Handling](#error-handling)
- [Offline Support](#offline-support)
- [Known Limitations](#known-limitations)
- [Getting Started](#getting-started)
---
 
## Overview
 
**[Learnlytics](https://github.com/khushidwived/Learnlytics)** (internally titled *CampusFlow Pro | Performance Engine*) is a fully self-contained, single HTML-file web application. It requires no build step, no bundler, and no server — simply open it in a browser. The app uses Firebase for user authentication and cloud data storage, Chart.js for interactive analytics, and Tailwind CSS for a polished dark-mode UI.
 
Students can log academic tasks (study sessions, lectures, assignments), mark them complete, and view productivity trends over daily, weekly, monthly, and yearly timeframes.
 
---
 
## Live Features
 
| Feature | Description |
|---|---|
| 🔐 **Google Sign-In** | One-click OAuth via Firebase Authentication |
| 👤 **Guest Mode** | Anonymous sign-in for quick access without an account |
| ➕ **Task Logging** | Add tasks with category, duration (hours), and problems solved |
| ✅ **Task Completion** | Toggle tasks complete/incomplete with live UI feedback |
| 🗑️ **Task Deletion** | Delete individual tasks or bulk-clear today's completed ones |
| 📊 **Doughnut Chart** | Category distribution (Study / Lecture / Assignment) by hours |
| 📈 **Line Graph** | Historical hourly productivity waveform across selected timeframe |
| 🗓️ **Timeframe Filtering** | Filter all analytics by Day, Week, Month, or Year |
| 📡 **Real-Time Sync** | Firestore `onSnapshot` listener keeps data live across devices/tabs |
| 📴 **Offline Mode** | Persistent local Firestore cache keeps the app functional offline |
| 🕐 **Live Clock** | Header displays real-time date and time, updated every second |
| 🎯 **5 KPI Metrics** | Period Gross, Period Net hours, Period Problems, Grand Total Problems, Focus Rating % |
| 🌐 **3D Card Tilt** | Mouse-tracked perspective tilt on every dashboard card |
| 📱 **Responsive Design** | Mobile-first layout using Tailwind's responsive grid system |
 
---
 
## Tech Stack
 
| Technology | Version | Purpose |
|---|---|---|
| **HTML5** | — | Application shell and markup |
| **Tailwind CSS** | CDN | Utility-first styling and responsive layout |
| **Font Awesome** | 6.0.0 | Icon library |
| **Plus Jakarta Sans** | Google Fonts | Primary UI typeface |
| **Chart.js** | 4.4.1 | Doughnut and line chart visualizations |
| **Firebase App** | 11.6.1 | App initialization |
| **Firebase Auth** | 11.6.1 | Google OAuth + anonymous authentication |
| **Firebase Firestore** | 11.6.1 | NoSQL cloud database + real-time sync |
 
> All dependencies are loaded via CDN. No `npm install` or build step is required.
 
---
 
## Project Structure
 
The entire application is contained in a **single HTML file** with three logical sections:
 
```
Learnlytics.html
│
├── <head>
│   ├── Meta tags & viewport
│   ├── Tailwind CSS CDN
│   ├── Chart.js CDN
│   ├── Font Awesome CDN
│   └── <style> block — Custom CSS design system
│
├── <body>
│   ├── .bg-mesh            — Fixed gradient background layer
│   ├── #loadingOverlay     — Auth gate (spinner → login form)
│   ├── <nav>               — Top navigation bar
│   ├── <main>              — Dashboard layout
│   │   ├── <header>        — Title + timeframe tabs (Day/Week/Month/Year)
│   │   ├── #metrics        — 5 KPI stat cards (dynamically rendered)
│   │   └── .grid           — Two-column layout
│   │       ├── Left Panel  — Doughnut chart + Task input form
│   │       └── Right Panel — Line graph + Daily task queue
│   │
│   └── <script type="module">
│       ├── Firebase imports & config
│       ├── App state variables
│       ├── initRealtimeSync()
│       ├── onAuthStateChanged()
│       ├── runTimeEngine()
│       ├── applySpatialEffects()
│       ├── render()
│       ├── updateQueue()
│       ├── updateDisplayMetrics()
│       ├── window.toggleTask()
│       ├── window.deleteTask()
│       ├── window.changeTimeframe()
│       ├── updateDataVisuals()
│       ├── initializeCore()
│       └── Event listeners (buttons, inputs)
```
 
---
 
## Firebase Setup
 
The app is pre-configured with a Firebase project. The credentials are hardcoded in the `<script type="module">` block:
 
```javascript
const firebaseConfig = {
    apiKey: "AIzaSyA1nwks3NgRl3wi3Q674xOVM5KUJC4LtRc",
    authDomain: "tracker-a68ac.firebaseapp.com",
    projectId: "tracker-a68ac",
    storageBucket: "tracker-a68ac.firebasestorage.app",
    messagingSenderId: "404201696253",
    appId: "1:404201696253:web:f42eb6f7ce64576b10c794"
};
```
 
### Firestore Initialization
 
Firestore is initialized with **persistent multi-tab local caching** for offline support:
 
```javascript
const db = initializeFirestore(app, {
    localCache: persistentLocalCache({
        tabManager: persistentMultipleTabManager()
    })
});
```
 
### Auth Persistence
 
Authentication state is persisted across browser sessions using `browserLocalPersistence`:
 
```javascript
setPersistence(auth, browserLocalPersistence);
```
 
> ⚠️ **Security Note:** If you fork this project, replace `firebaseConfig` with your own Firebase project credentials and configure Firestore Security Rules to restrict access per user UID.
 
---
 
## Authentication
 
The app uses a two-step auth overlay:
 
1. **Loading Spinner** — Shown while Firebase checks the existing session.
2. **Auth Container** — Revealed if no active session is found.
### Sign-In Options
 
```javascript
// Google OAuth (popup-based)
document.getElementById('googleSignInBtn').onclick = () =>
    signInWithPopup(auth, new GoogleAuthProvider());
 
// Anonymous / Guest
document.getElementById('anonSignInBtn').onclick = () =>
    signInAnonymously(auth);
 
// Logout
document.getElementById('logoutBtn').onclick = () =>
    signOut(auth).then(() => location.reload());
```
 
### Auth State Observer
 
`onAuthStateChanged` drives the entire app lifecycle:
 
```javascript
onAuthStateChanged(auth, user => {
    if (user) {
        currentUser = user;
        // Populate user avatar and display name
        // Hide auth overlay
        // Start real-time data sync
        initRealtimeSync();
    } else {
        // Show auth overlay / login form
    }
});
```
 
---
 
## Data Model
 
Tasks are stored in Firestore under a per-user collection path:
 
```
/artifacts/{appId}/users/{uid}/tasks/{taskId}
```
 
Where `appId = 'performance-engine-v4'` and `taskId` is a Unix timestamp string (`Date.now().toString()`).
 
### Task Document Schema
 
```javascript
{
    text:      String,   // Task description, e.g. "Advanced Calculus Module 3"
    category:  String,   // "Study" | "Lectures" | "Assignment"
    hours:     Number,   // Duration in hours, e.g. 1.5
    questions: Number,   // Number of problems/questions solved, e.g. 20
    completed: Boolean,  // Whether the task is marked done
    createdAt: String    // ISO 8601 timestamp, e.g. "2025-01-15T14:30:00.000Z"
}
```
 
### Application-Level State
 
```javascript
let allTasks = [];            // All tasks fetched from Firestore for the current user
let currentTimeframe = 'week'; // Active filter: 'day' | 'week' | 'month' | 'year'
let categoryChart;            // Chart.js doughnut chart instance
let lineGraph;                // Chart.js line chart instance
let currentUser = null;       // Firebase User object
let lastDayCheck;             // Tracks day rollover for auto-refresh
 
const colors = {
    'Study':      '#6366f1',   // Indigo
    'Lectures':   '#f43f5e',   // Rose
    'Assignment': '#f59e0b'    // Amber
};
```
 
---
 
## Core Modules
 
### 1. Auth & Session Management
 
Handles sign-in, sign-out, and session persistence. On successful auth, populates the navbar with the user's name and avatar photo. On sign-out, reloads the page to reset all state.
 
---
 
### 2. Real-Time Sync Engine
 
```javascript
function initRealtimeSync() {
    const tasksRef = collection(db, 'artifacts', appId, 'users', currentUser.uid, 'tasks');
 
    onSnapshot(tasksRef, snap => {
        allTasks = snap.docs.map(d => ({ id: d.id, ...d.data() }));
        render(); // Re-render UI on every change
    });
 
    // Toggle Firestore network based on browser connectivity
    window.addEventListener('online',  () => enableNetwork(db));
    window.addEventListener('offline', () => disableNetwork(db));
}
```
 
- Uses Firestore's `onSnapshot` for push-based updates — no polling.
- The "Cloud Sync Active" indicator in the navbar shows/hides based on sync status.
- Network listeners gracefully handle going online/offline.
---
 
### 3. Time Engine
 
```javascript
function runTimeEngine() {
    const now = new Date();
    // Updates the live clock in the header every second
    // Detects day rollover and triggers a re-render for the "Today" view
}
setInterval(runTimeEngine, 1000);
```
 
Runs every second to keep the header clock live. Also detects when midnight passes and triggers `render()` to refresh the daily task queue.
 
---
 
### 4. Task Management
 
#### Adding a Task
 
```javascript
async function handleAddTask() {
    // Validates that task description is not empty (shakes UI if invalid)
    // Disables button and shows "Deploying..." spinner during write
    // Writes to Firestore with setDoc()
    // Shows success state briefly, then resets the form
}
```
 
- Triggered by the **Deploy Task** button click or pressing `Enter` in text/question inputs.
- Optimistic UI: button state changes immediately before Firestore confirms.
#### Toggling Completion
 
```javascript
window.toggleTask = async (id) => {
    const t = allTasks.find(x => x.id === id);
    await updateDoc(doc(db, ..., id), { completed: !t.completed });
};
```
 
#### Deleting a Task
 
```javascript
window.deleteTask = async (id) => {
    await deleteDoc(doc(db, ..., id));
};
```
 
#### Bulk Clear Completed
 
Uses a Firestore **batch write** to atomically delete all of today's completed tasks:
 
```javascript
const batch = writeBatch(db);
toDelete.forEach(t => batch.delete(doc(db, ..., t.id)));
await batch.commit();
```
 
---
 
### 5. Render & Filter Pipeline
 
```javascript
function render() {
    const now = new Date();
 
    // Filter allTasks to the selected timeframe
    const analyticalData = allTasks.filter(t => {
        if (currentTimeframe === 'day')   return createdAt.toDateString() === todayStr;
        if (currentTimeframe === 'week')  return diffDays <= 7;
        if (currentTimeframe === 'month') return diffDays <= 30;
        if (currentTimeframe === 'year')  return createdAt.getFullYear() === now.getFullYear();
    });
 
    // Daily queue always shows only today's tasks (regardless of timeframe)
    const dailyQueue = allTasks.filter(t =>
        new Date(t.createdAt).toDateString() === todayStr
    );
 
    updateDisplayMetrics(analyticalData);  // KPI cards
    updateQueue(dailyQueue);               // Task list
    updateDataVisuals(analyticalData);     // Charts
    applySpatialEffects();                 // 3D tilt re-bind
}
```
 
---
 
### 6. Metrics Calculator
 
```javascript
function updateDisplayMetrics(filtered) {
    const completedTasks = filtered.filter(t => t.completed);
 
    // Period Net = hours from completed tasks in the selected timeframe
    const focusHours = completedTasks.reduce((s, t) => s + (Number(t.hours) || 0), 0);
 
    // Period Gross = hours from ALL tasks (completed + pending) in the timeframe
    const targetHours = filtered.reduce((s, t) => s + (Number(t.hours) || 0), 0);
 
    // Focus Rating = (focusHours / targetHours) * 100
    const efficiency = targetHours > 0 ? Math.round((focusHours / targetHours) * 100) : 0;
 
    // Period Problems = questions solved in the selected timeframe
    const periodQuestions = filtered.reduce((s, t) => s + (Number(t.questions) || 0), 0);
 
    // Grand Total Problems = questions solved across ALL time (not filtered)
    const totalQuestionsEver = allTasks.reduce((s, t) => s + (Number(t.questions) || 0), 0);
}
```
 
The five KPI cards rendered are:
 
| Card | Metric |
|---|---|
| **Period Gross** | Total planned hours in the selected timeframe |
| **Period Net** | Actual completed hours in the selected timeframe |
| **Problems (Period)** | Questions/problems solved in the selected timeframe |
| **Grand Total Problems** | All-time questions solved across every session |
| **Focus Rating** | `(Net / Gross) * 100` — completion efficiency percentage |
 
---
 
### 7. Chart Visualizations
 
#### Doughnut Chart (Category Distribution)
 
```javascript
categoryChart = new Chart(catCtx, {
    type: 'doughnut',
    data: {
        datasets: [{
            data: [studyHours, lectureHours, assignmentHours],
            backgroundColor: ['#6366f1', '#f43f5e', '#f59e0b'],
            cutout: '82%',
            borderRadius: 8
        }]
    }
});
```
 
- Displays the proportion of completed hours by category.
- The center of the donut shows the **total net hours** value.
#### Line Graph (Productivity Waveform)
 
```javascript
lineGraph = new Chart(ctx, {
    type: 'line',
    data: { labels: [], datasets: [{ data: [], borderColor: '#6366f1', fill: true, tension: 0.4 }] }
});
```
 
**Label/data generation by timeframe:**
 
| Timeframe | X-Axis Labels | Y-Axis Data |
|---|---|---|
| `day` | Hour slots: `00:00`, `06:00`, `12:00`, `18:00`, `21:00`, `23:59` | Cumulative hours up to each time slot |
| `week` | Last 7 days (weekday short names) | Total completed hours per day |
| `month` | Last 30 days (date labels every 7th day) | Total completed hours per day |
| `year` | Jan–Dec | Total completed hours per month |
 
---
 
### 8. Spatial UI Effects
 
```javascript
function applySpatialEffects() {
    document.querySelectorAll('.tilt-target').forEach(card => {
        card.onmousemove = e => {
            const rect = card.getBoundingClientRect();
            const x = (e.clientX - rect.left - rect.width / 2) / 60;
            const y = -(e.clientY - rect.top - rect.height / 2) / 60;
            card.style.transform =
                `perspective(1000px) rotateY(${x}deg) rotateX(${y}deg) translateY(-2px)`;
        };
        card.onmouseleave = () => card.style.transform = 'none';
    });
}
```
 
Every card with the `.tilt-target` class gets a subtle 3D perspective tilt effect that follows the user's mouse cursor. It's re-applied on every `render()` call to cover dynamically created cards.
 
---
 
## UI Components
 
### Navigation Bar
- **Logo** — "[Learnlytics](https://github.com/khushidwived/Learnlytics)" brand mark; clicking reloads the page.
- **Cloud Sync Badge** — Pulsing green dot shows real-time Firestore connection status.
- **User Info** — Displays user name and avatar. Shows "Guest Operative" for anonymous users.
- **Logout Button** — Signs out and reloads the page.
### Auth Overlay (`#loadingOverlay`)
- Full-screen overlay shown before authentication.
- Transitions from a loading spinner to the login card.
- Fades out smoothly after successful sign-in.
### Timeframe Tabs
- Four tabs: **Today**, **Week**, **Month**, **Year**.
- Active tab is styled with an indigo background and ring highlight.
- Calls `window.changeTimeframe(time)` which updates `currentTimeframe` and re-renders.
### Task Input Form
- **Task Description** — Free-text input. Shakes with red border on empty submit.
- **Category Select** — Dropdown: `Deep Study`, `Lecture`, `Assignment`.
- **Duration** — Number input (step: 0.5, min: 0.5). Defaults to `1.5`.
- **Problems Done** — Number input (min: 0). Defaults to `0`.
- **Deploy Task Button** — 3D press-style button with spinner feedback.
### Daily Task Queue (`#taskList`)
- Shows only today's tasks, sorted: pending first (newest at top), completed last.
- Each item shows: task name, category tag, hours, problems badge (if > 0).
- Checkbox button toggles completion.
- Trash icon (visible on hover) deletes the task.
- Empty state shows a placeholder illustration and message.
---
 
## CSS Design System
 
### CSS Custom Properties
 
```css
:root {
    --brand-primary:   #6366f1;  /* Indigo-500 */
    --brand-secondary: #4f46e5;  /* Indigo-600 */
    --bg-dark:         #0f172a;  /* Slate-950 */
    --card-bg:         rgba(30, 41, 59, 0.7); /* Slate-800 with transparency */
}
```
 
### Key Utility Classes
 
| Class | Description |
|---|---|
| `.spatial-card` | Glassmorphism card with backdrop blur, subtle border, hover indigo glow |
| `.btn-3d` | Gradient button with 3D bottom-border press animation |
| `.btn-success` | Green variant of `.btn-3d` for success feedback |
| `.glass-input` | Dark semi-transparent input with indigo focus ring |
| `.pulse-dot` | Animated green dot indicator for live sync status |
| `.task-item` | Slide-in animated task row with hover translate effect |
| `.category-tag` | Micro badge for category labels with category-colored border |
| `.stat-badge` | Indigo pill badge for problem count display |
| `.custom-scrollbar` | Thin custom scrollbar for the task list panel |
 
### Animations
 
| Animation | Trigger | Effect |
|---|---|---|
| `slideIn` | Task item mount | Fade in + slide up from 12px below |
| `pulse` | `.pulse-dot` | Opacity and scale pulsing |
| `animate-shake` | Invalid submit | Horizontal shake (2 cycles, 200ms) |
| `animate-spin` | Loading states | Continuous 360° rotation |
 
---
 
## Event Listeners & Interactions
 
```javascript
// Add task on button click
addTaskBtn.addEventListener('click', handleAddTask);
 
// Add task on Enter key (task input)
taskInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') handleAddTask(); });
 
// Add task on Enter key (question input)
questionInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') handleAddTask(); });
 
// Bulk clear today's completed tasks
document.getElementById('clearCompletedBtn').onclick = async () => { ... };
 
// Timeframe tab switching (inline onclick in HTML)
// onclick="changeTimeframe('day'|'week'|'month'|'year')"
 
// Toggle task complete/incomplete (inline onclick in rendered HTML)
// onclick="window.toggleTask('${t.id}')"
 
// Delete task (inline onclick in rendered HTML)
// onclick="window.deleteTask('${t.id}')"
 
// Auth buttons
document.getElementById('googleSignInBtn').onclick = () => signInWithPopup(...);
document.getElementById('anonSignInBtn').onclick = () => signInAnonymously(auth);
document.getElementById('logoutBtn').onclick = () => signOut(auth).then(() => location.reload());
```
 
---
 
## Error Handling
 
| Scenario | Handling |
|---|---|
| Empty task submit | Shakes input and button; does not submit |
| Firestore write error | Shows "System Error" button state; logs to console |
| Firestore clear error | Logs to console; resets UI label |
| Firestore sync error | Hides "Cloud Sync Active" badge; logs to console |
| Task toggle/delete error | Logs to console silently |
| Missing DOM elements | All DOM queries are null-checked before use |
 
---
 
## Offline Support
 
The app continues to function without an internet connection thanks to:
 
1. **Persistent Local Cache** — Firestore caches all documents locally using `persistentLocalCache`.
2. **Multi-Tab Manager** — `persistentMultipleTabManager` coordinates cache across browser tabs.
3. **Network Listeners** — `enableNetwork` / `disableNetwork` are called on browser `online` / `offline` events to gracefully pause and resume Firestore sync.
Any tasks added while offline will be written to the local cache and automatically synced to Firestore when the connection is restored.
 
---
 
## Known Limitations
 
- **No Firebase Security Rules** documented — the app relies on the per-user path structure for isolation, but Firestore Security Rules should be configured in the Firebase Console to enforce this server-side.
- **No pagination** — All tasks for the user are loaded into memory at once. Performance may degrade with very large task histories.
- **Single category per task** — Tasks can only belong to one of three fixed categories.
- **No task editing** — Tasks cannot be modified after creation; only toggled or deleted.
- **Time-based task IDs** — Using `Date.now().toString()` as document IDs could theoretically cause collisions if two tasks are created in the same millisecond (extremely unlikely in normal use).
- **API key exposed** — The Firebase API key is client-side and visible. This is normal for Firebase web apps, but proper Firestore Security Rules and App Check should be used in production.
---
 
## Getting Started
 
1. **Clone or download** the HTML file.
2. **Open** `Learnlytics.html` directly in any modern browser — no server needed.
3. **Sign in** with your Google account or continue as a Guest.
4. **Start logging** your study sessions, lectures, and assignments.
5. Use the **timeframe tabs** to review your productivity over different periods.
> To use your own Firebase project, replace the `firebaseConfig` object with your own project's credentials from the [Firebase Console](https://console.firebase.google.com/).
 
---
 
*Built with ⚡ Firebase, Chart.js, and Tailwind CSS.*
