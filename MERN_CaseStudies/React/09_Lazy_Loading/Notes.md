# 9. Lazy Loading & Code Splitting 

## 1. Problem Statement – EduStream Learning Platform

EduStream is a large online education platform with multiple features:
* Dashboard
* Courses
* Video Lectures
* Quizzes
* Forums
* Admin Panel
* Profile Settings

Most users only use a few features per session. Loading the entire app upfront increases initial load time, especially on slow networks.

**Goal:**
Split the React application into smaller bundles and load code only when required, improving performance and user experience.

<br>

## 2. Learning Objectives

By completing this case study, we achieve:
* Understanding code splitting and lazy loading
* Using `React.lazy()` and dynamic `import()`
* Handling loading states with `Suspense`
* Implementing route-based and component-based lazy loading
* Handling lazy load failures using Error Boundaries
* Building an interview-ready, production-style React app

<br>

## 3. Conceptual Analogy – Modular Classroom

* Each feature = a classroom
* Main app = school hall
* Classrooms unlock only when needed

This avoids opening all rooms at once, saving time and resources.

<br>

## 4. Technical Concepts

### 4.1 Code Splitting

Breaking the app into smaller JS bundles so users download only what they need.

### 4.2 Lazy Loading

Loading components only when they are rendered for the first time.

### 4.3 Tools Used

* `import()` – dynamic import
* `React.lazy()` – lazy-loaded components
* `Suspense` – fallback UI while loading

<br>

## 5. Project Setup

### 5.1 Create Project

```bash
npm create vite@latest edustream-lazy -- --template react-ts
cd edustream-lazy
npm install
```

### 5.2 Install Dependencies

```bash
npm install react@18.2.0 react-dom@18.2.0 react-router-dom@6.22.3
```

### 5.3 Run Project

```bash
npm run dev
```

<br>

## 6. Folder Structure

```
src/
│
├─ main.tsx
├─ App.tsx
│
├─ components/
│   ├─ Loader.tsx
│   ├─ ErrorBoundary.tsx
│
├─ pages/
│   ├─ Dashboard.tsx
│   ├─ Courses.tsx
│   ├─ VideoLecture.tsx
│   ├─ Forum.tsx
│   ├─ AdminPanel.tsx
│   ├─ ProfileSettings.tsx
│   ├─ ProfileSettingsButton.tsx
│
└─ features/
    └─ Quiz.tsx
```

<br>

## 7. Entry Point

### 7.1 `main.tsx`

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

<br>

## 8. App-Level Route-Based Lazy Loading

### 8.1 `App.tsx`

```tsx
import { Routes, Route, Link } from "react-router-dom";
import { lazy, Suspense } from "react";
import Loader from "./components/Loader";
import ErrorBoundary from "./components/ErrorBoundary";

const Dashboard = lazy(() => import("./pages/Dashboard"));
const Courses = lazy(() => import("./pages/Courses"));
const Forum = lazy(() => import("./pages/Forum"));
const VideoLecture = lazy(() => import("./pages/VideoLecture"));
const AdminPanel = lazy(() => import("./pages/AdminPanel"));

export default function App() {
  return (
    <div>
      <nav style={{ display: "flex", gap: "12px" }}>
        <Link to="/">Dashboard</Link>
        <Link to="/courses">Courses</Link>
        <Link to="/forum">Forum</Link>
        <Link to="/admin">Admin</Link>
      </nav>

      <ErrorBoundary>
        <Suspense fallback={<Loader />}>
          <Routes>
            <Route path="/" element={<Dashboard />} />
            <Route path="/courses" element={<Courses />} />
            <Route path="/forum" element={<Forum />} />
            <Route path="/lecture/:id" element={<VideoLecture />} />
            <Route path="/admin" element={<AdminPanel />} />
          </Routes>
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}
```

<br>

## 9. Shared Components

### 9.1 Loader

```tsx
export default function Loader() {
  return <div style={{ padding: 20 }}>Loading...</div>;
}
```

### 9.2 Error Boundary

```tsx
import { Component, ReactNode } from "react";

type Props = { children: ReactNode };
type State = { hasError: boolean };

export default class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong while loading.</div>;
    }
    return this.props.children;
  }
}
```

<br>

## 10. Pages

### 10.1 Dashboard

```tsx
import ProfileSettingsButton from "./ProfileSettingsButton";

export default function Dashboard() {
  return (
    <div>
      <h1>EduStream Dashboard</h1>
      <ProfileSettingsButton />
    </div>
  );
}
```

### 10.2 Profile Settings (Component-Based Lazy Loading)

```tsx
export default function ProfileSettings() {
  return <div>User Profile Settings Loaded</div>;
}
```

```tsx
import { lazy, Suspense, useState } from "react";
import Loader from "../components/Loader";

const ProfileSettings = lazy(() => import("./ProfileSettings"));

export default function ProfileSettingsButton() {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setOpen(true)}>Open Settings</button>
      {open && (
        <Suspense fallback={<Loader />}>
          <ProfileSettings />
        </Suspense>
      )}
    </div>
  );
}
```

<br>

### 10.3 Courses + Quiz (Conditional Loading)

```tsx
import { lazy, Suspense, useState } from "react";
import Loader from "../components/Loader";

const Quiz = lazy(() => import("../features/Quiz"));

export default function Courses() {
  const [showQuiz, setShowQuiz] = useState(false);

  return (
    <div>
      <h2>Courses</h2>
      <button onClick={() => setShowQuiz(true)}>Take Quiz</button>

      {showQuiz && (
        <Suspense fallback={<Loader />}>
          <Quiz />
        </Suspense>
      )}
    </div>
  );
}
```

```tsx
export default function Quiz() {
  return <div>Quiz Loaded Lazily</div>;
}
```

<br>

### 10.4 Other Pages

```tsx
export default function Forum() {
  return <div>Forum Page</div>;
}
```

```tsx
import { useParams } from "react-router-dom";

export default function VideoLecture() {
  const { id } = useParams();
  return <div>Video Lecture {id}</div>;
}
```

```tsx
export default function AdminPanel() {
  return <div>Admin Panel (Rarely Used)</div>;
}
```

<br>

## 11. What We Implemented

* Route-based lazy loading using `React.lazy`
* Component-based lazy loading (Settings, Quiz)
* `Suspense` fallback for loading states
* Error Boundary for failed imports
* Clean modular folder structure


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/52e72060-40ab-4d88-90f4-a31f6a7ed364" />

<br>

## 12. Key Takeaways

* Start with route-level splitting
* Lazy load only heavy or rarely-used features
* Always wrap lazy components with `Suspense`
* Keep critical UI in the main bundle
* Use React 18 for ecosystem stability

<br>

## 13. Outcome

* Faster initial load
* Smaller JS bundle
* Better scalability
* Production-ready and interview-ready React architecture
