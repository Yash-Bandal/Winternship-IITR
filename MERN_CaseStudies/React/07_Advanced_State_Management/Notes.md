# 7. Advanced State Management with Zustand 
## CollabNotes Case Study

<br>

## 1. Problem Statement

**CollabNotes – Real-Time Collaborative Notes App**

A team-based note-taking application with the following requirements:

* Users can create, edit, and delete notes
* Notes sync with a cloud API
* State persists across reloads
* All actions are logged for audit / undo-redo
* Minimal boilerplate, high performance, strict TypeScript

<br>

## 2. Tech Stack

* React + TypeScript (Vite)
* Zustand (state management)
* Zustand Middleware:

  * devtools
  * immer
  * persist
* React Query v5 (server state)
* Strict TypeScript (`noImplicitAny`, `verbatimModuleSyntax`)

<br>

## 3. Project Initialization

### 3.1 Create Project

```bash
npm create vite@latest collabnotes -- --template react-ts
cd collabnotes
npm install
npm run dev
```

<br>

## 4. Install Libraries

```bash
npm install zustand
npm install immer
npm install @tanstack/react-query
```

<br>

## 5. Folder Structure

```txt
src/
├── api/
│   ├── notes.api.ts
│   └── collaborators.api.ts
├── store/
│   ├── note.store.ts
│   ├── preferences.store.ts
│   ├── session.store.ts
│   └── history.store.ts
├── components/
│   ├── NotesList.tsx
│   └── Collaborators.tsx
├── providers/
│   └── QueryProvider.tsx
├── App.tsx
└── main.tsx
```

<br>

## 6. React Query Provider Setup

### `src/providers/QueryProvider.tsx`

```ts
import type { ReactNode } from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

export function QueryProvider({ children }: { children: ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### `src/main.tsx`

```ts
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { QueryProvider } from './providers/QueryProvider';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryProvider>
      <App />
    </QueryProvider>
  </React.StrictMode>
);
```

<br>

## 7. API Layer (Mock Cloud)

### `src/api/notes.api.ts`

```ts
export type Note = {
  id: string;
  text: string;
};

export async function fetchNotesFromAPI(): Promise<Note[]> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: '1', text: 'Team sync at 10 AM' },
        { id: '2', text: 'Deploy Zustand demo' },
      ]);
    }, 800);
  });
}
```

### `src/api/collaborators.api.ts`

```ts
export type Collaborator = {
  id: string;
  name: string;
};

export async function fetchCollaborators(): Promise<Collaborator[]> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 'u1', name: 'Alice' },
        { id: 'u2', name: 'Bob' },
      ]);
    }, 600);
  });
}
```

<br>

## 8. Notes Store (Zustand + Immer + Devtools + Logging)

### `src/store/note.store.ts`

```ts
import { create, type StateCreator } from 'zustand';
import { devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import type { Note } from '../api/notes.api';

type NoteState = {
  notes: Note[];
  setNotes: (notes: Note[]) => void;
  addNote: (note: Note) => void;
  updateNote: (id: string, text: string) => void;
  deleteNote: (id: string) => void;
};

type NoteStateCreator = StateCreator<
  NoteState,
  [['zustand/immer', never], ['zustand/devtools', never]]
>;

const logMiddleware =
  (config: NoteStateCreator): NoteStateCreator =>
  (set, get, api) =>
    config(
      (args) => {
        console.log('Before:', get());
        set(args);
        console.log('After:', get());
      },
      get,
      api
    );

const noteStoreCreator: NoteStateCreator = (set) => ({
  notes: [],

  setNotes: (notes) =>
    set((state) => {
      state.notes = notes;
    }),

  addNote: (note) =>
    set((state) => {
      state.notes.push(note);
    }),

  updateNote: (id, text) =>
    set((state) => {
      const note = state.notes.find((n: Note) => n.id === id);
      if (note) note.text = text;
    }),

  deleteNote: (id) =>
    set((state) => {
      state.notes = state.notes.filter((n: Note) => n.id !== id);
    }),
});

export const useNoteStore = create<NoteState>()(
  devtools(
    immer(logMiddleware(noteStoreCreator)),
    { name: 'NoteStore' }
  )
);
```

<br>

## 9. Preferences Store (Persist + Migration)

### `src/store/preferences.store.ts`

```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

type PreferencesState = {
  theme: 'light' | 'dark';
  fontSize: number;
  setTheme: (theme: 'light' | 'dark') => void;
  setFontSize: (size: number) => void;
};

export const usePreferencesStore = create<PreferencesState>()(
  persist(
    (set) => ({
      theme: 'light',
      fontSize: 14,
      setTheme: (theme) => set({ theme }),
      setFontSize: (size) => set({ fontSize: size }),
    }),
    {
      name: 'collabnotes-preferences',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ theme: state.theme, fontSize: state.fontSize }),
      version: 2,
      migrate: (persisted: any, version) => {
        if (version < 2) return { ...persisted, fontSize: 14 };
        return persisted;
      },
    }
  )
);
```

<br>

## 10. Session Store (Partial Persist + Migration)

### `src/store/session.store.ts`

```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

type SessionState = {
  userId: string | null;
  token: string | null;
  expiresAt: number | null;
  role: 'admin' | 'user';
  setSession: (data: Partial<SessionState>) => void;
  clearSession: () => void;
};

export const useSessionStore = create<SessionState>()(
  persist(
    immer((set) => ({
      userId: null,
      token: null,
      expiresAt: null,
      role: 'user',
      setSession: (data) =>
        set((state) => {
          Object.assign(state, data);
        }),
      clearSession: () =>
        set((state) => {
          state.userId = null;
          state.token = null;
          state.expiresAt = null;
        }),
    })),
    {
      name: 'collabnotes-session',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ userId: state.userId, token: state.token }),
      version: 2,
      migrate: (persisted: any, version) => {
        if (version < 2) return { ...persisted, role: 'user' };
        return persisted;
      },
    }
  )
);
```

<br>

## 11. React Query + Zustand Sync

### `src/components/NotesList.tsx`

```tsx
import { useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { fetchNotesFromAPI } from '../api/notes.api';
import { useNoteStore } from '../store/note.store';

export function NotesList() {
  const notes = useNoteStore((s) => s.notes);
  const setNotes = useNoteStore((s) => s.setNotes);

  const { data, isLoading } = useQuery({
    queryKey: ['notes'],
    queryFn: fetchNotesFromAPI,
  });

  useEffect(() => {
    if (data) setNotes(data);
  }, [data, setNotes]);

  if (isLoading) return <div>Loading notes...</div>;

  return (
    <ul>
      {notes.map((n) => (
        <li key={n.id}>{n.text}</li>
      ))}
    </ul>
  );
}
```

<br>

## 12. Collaborators Component

### `src/components/Collaborators.tsx`

```tsx
import { useQuery } from '@tanstack/react-query';
import { fetchCollaborators } from '../api/collaborators.api';

export function Collaborators() {
  const { data, isLoading } = useQuery({
    queryKey: ['collaborators'],
    queryFn: fetchCollaborators,
  });

  if (isLoading) return <div>Loading collaborators...</div>;

  return (
    <ul>
      {data?.map((c) => (
        <li key={c.id}>{c.name}</li>
      ))}
    </ul>
  );
}
```

<br>

## 13. App Entry

### `src/App.tsx`

```tsx
import { NotesList } from './components/NotesList';
import { Collaborators } from './components/Collaborators';

function App() {
  return (
    <div style={{ padding: 20 }}>
      <h1>CollabNotes</h1>

      <h2>Notes</h2>
      <NotesList />

      <h2>Collaborators</h2>
      <Collaborators />
    </div>
  );
}

export default App;
```

<br>

## 14. Final Output

* App starts with `npm run dev`
* Notes fetched via React Query
* Synced into Zustand
* State mutations tracked in Redux DevTools
* Notes, preferences, and session persist across reloads
* Strict TypeScript with zero errors



<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/142e2368-ab2f-4780-b4e3-5f03690cd080" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/634db5f1-f4d3-417a-bf06-eb83c541fd22" />



<br>

## 15. Key Best Practices

* React Query for server state
* Zustand for UI/client state
* Immer for safe immutable updates
* Persist only what is necessary
* Always version persisted state
* Use selectors to avoid re-renders

<br>

**END OF CASE STUDY**
