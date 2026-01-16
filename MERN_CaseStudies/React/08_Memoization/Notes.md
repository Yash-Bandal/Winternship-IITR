# 8. Memoization 
## StreamVision Case Study

<br>

## 1. Problem Statement

**StreamVision – Video Analytics Dashboard**

StreamVision is a video analytics dashboard that displays:
* Live video feeds (simulated)
* Analytics charts
* User comments and tags


As the dashboard grows:
* Some components become expensive to render
* User interactions trigger unnecessary re-renders
* Performance degrades without optimization


**Goal:**
Build the dashboard from scratch and optimize it using React memoization techniques.

<br>

## 2. Learning Objectives

By the end of this case study, you will:

* Understand memoization in React
* Use `useMemo` for expensive computations
* Use `useCallback` for stable function references
* Use `React.memo` to avoid unnecessary re-renders
* Combine all three correctly in a real app
* Understand when NOT to memoize

<br>

## 3. Project Setup

### 3.1 Create Project (Vite + React + TypeScript)

```bash
npm create vite@latest streamvision -- --template react-ts
cd streamvision
npm install
npm run dev
```

> Memoization uses built-in React hooks. No extra libraries required.

<br>

## 4. Folder Structure

```txt
src/
├── components/
│   ├── AnalyticsChart.tsx
│   ├── CommentList.tsx
│   ├── FilterInput.tsx
│   ├── VideoControls.tsx
│   ├── VideoOverlay.tsx
│   ├── TagList.tsx
│   └── TagInput.tsx
├── data/
│   ├── analytics.ts
│   └── comments.ts
├── App.tsx
└── main.tsx
```

<br>

## 5. Entry Files

### `src/main.tsx`

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

<br>

## 6. Mock Data & Heavy Computation

### `src/data/analytics.ts`

```ts
export type AnalyticsItem = {
  value: number;
};

export function computeAnalytics(data: AnalyticsItem[]) {
  console.log('Heavy analytics computation running');

  let total = 0;
  for (let i = 0; i < 50_000_000; i++) {
    total += i % 2;
  }

  return data.reduce((acc, item) => acc + item.value, total);
}
```

### `src/data/comments.ts`

```ts
export const comments = [
  { id: 1, text: 'Great video!' },
  { id: 2, text: 'Amazing quality' },
  { id: 3, text: 'Needs better lighting' },
  { id: 4, text: 'Love the overlay stats' },
];
```

<br>

## 7. useMemo – Analytics Chart

### `src/components/AnalyticsChart.tsx`

```tsx
import { useMemo } from 'react';
import { computeAnalytics } from '../data/analytics';
import type { AnalyticsItem } from '../data/analytics';  //type is imp

type Props = {
    data: AnalyticsItem[];
};

export function AnalyticsChart({ data }: Props) {
    console.log('📊 Rendering AnalyticsChart');

    const analyticsValue = useMemo(() => {
        return computeAnalytics(data);
    }, [data]);

    return <div>Analytics Value: {analyticsValue}</div>;
}

```

<br>

## 8. React.memo – Comment List

### `src/components/CommentList.tsx`

```tsx
import React from 'react';

type Comment = {
  id: number;
  text: string;
};

type Props = {
  comments: Comment[];
};

function CommentListComponent({ comments }: Props) {
  console.log('Rendering CommentList');

  return (
    <ul>
      {comments.map((c) => (
        <li key={c.id}>{c.text}</li>
      ))}
    </ul>
  );
}

export const CommentList = React.memo(CommentListComponent);
```

<br>

## 9. useCallback – Filter Input

### `src/components/FilterInput.tsx`

```tsx
import React from 'react';

type Props = {
  onFilter: (value: string) => void;
};

function FilterInputComponent({ onFilter }: Props) {
  console.log('Rendering FilterInput');

  return (
    <input
      placeholder="Filter comments..."
      onChange={(e) => onFilter(e.target.value)}
    />
  );
}

export const FilterInput = React.memo(FilterInputComponent);
```

<br>

## 10. Video Controls & Overlay

### `src/components/VideoControls.tsx`

```tsx
type Props = {
  onPlay: () => void;
  onPause: () => void;
};

export function VideoControls({ onPlay, onPause }: Props) {
  console.log('Rendering VideoControls');

  return (
    <div>
      <button onClick={onPlay}>Play</button>
      <button onClick={onPause}>Pause</button>
    </div>
  );
}
```

### `src/components/VideoOverlay.tsx`

```tsx
import React from 'react';

type Overlay = {
  id: number;
  label: string;
};

type Props = {
  overlays: Overlay[];
};

function VideoOverlayComponent({ overlays }: Props) {
  console.log('Rendering VideoOverlay');

  return (
    <div>
      {overlays.map((o) => (
        <span key={o.id}>{o.label} </span>
      ))}
    </div>
  );
}

export const VideoOverlay = React.memo(VideoOverlayComponent);
```

<br>

## 11. Mini Project – Tags

### `src/components/TagList.tsx`

```tsx
import React, { useMemo } from 'react';

type Props = {
  tags: string[];
  filter: string;
};

function TagListComponent({ tags, filter }: Props) {
  console.log('Rendering TagList');

  const filteredTags = useMemo(() => {
    return tags.filter((t) => t.includes(filter));
  }, [tags, filter]);

  return (
    <ul>
      {filteredTags.map((t) => (
        <li key={t}>{t}</li>
      ))}
    </ul>
  );
}

export const TagList = React.memo(TagListComponent);
```

### `src/components/TagInput.tsx`

```tsx
import React, { useState } from 'react';

type Props = {
  onAddTag: (tag: string) => void;
};

function TagInputComponent({ onAddTag }: Props) {
  console.log('Rendering TagInput');

  const [value, setValue] = useState('');

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <button
        onClick={() => {
          onAddTag(value);
          setValue('');
        }}
      >
        Add Tag
      </button>
    </div>
  );
}

export const TagInput = React.memo(TagInputComponent);
```

<br>

## 12. App Integration

### `src/App.tsx`

```tsx
import { useCallback, useMemo, useState } from 'react';
import { AnalyticsChart } from './components/AnalyticsChart';
import { CommentList } from './components/CommentList';
import { FilterInput } from './components/FilterInput';
import { VideoControls } from './components/VideoControls';
import { VideoOverlay } from './components/VideoOverlay';
import { TagList } from './components/TagList';
import { TagInput } from './components/TagInput';
import { comments } from './data/comments';

function App() {
  const [playing, setPlaying] = useState(false);
  const [filter, setFilter] = useState('');
  const [tags, setTags] = useState(['react', 'video', 'dashboard']);
  const [unrelated, setUnrelated] = useState(0);

  const analyticsData = useMemo(() => [{ value: 10 }, { value: 20 }], []);

  const handlePlay = useCallback(() => setPlaying(true), []);
  const handlePause = useCallback(() => setPlaying(false), []);
  const handleFilter = useCallback((v: string) => setFilter(v), []);
  const handleAddTag = useCallback(
    (tag: string) => setTags((t) => [...t, tag]),
    []
  );

  return (
    <div style={{ padding: 20 }}>
      <h1>StreamVision Dashboard</h1>

      <VideoControls onPlay={handlePlay} onPause={handlePause} />
      <div>Playing: {playing ? 'Yes' : 'No'}</div>

      <AnalyticsChart data={analyticsData} />

      <FilterInput onFilter={handleFilter} />
      <CommentList comments={comments.filter((c) => c.text.includes(filter))} />

      <VideoOverlay overlays={[{ id: 1, label: 'LIVE' }]} />

      <TagInput onAddTag={handleAddTag} />
      <TagList tags={tags} filter={filter} />

      <hr />
      <button onClick={() => setUnrelated((n) => n + 1)}>
        Update unrelated state ({unrelated})
      </button>
    </div>
  );
}

export default App;
```

<br>

## 13. Output & Observations

* Heavy analytics computation runs only when data changes
* Memoized components do not re-render on unrelated state updates
* Console logs prove memoization effectiveness


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7579d2f3-567a-459a-8749-c94027fa7ebe" />


<br>

## 14. Final Takeaway

* `useMemo` → cache expensive values
* `useCallback` → stabilize function references
* `React.memo` → prevent unnecessary re-renders

Together, they are essential for performance optimization in real-world React applications.

<br>


