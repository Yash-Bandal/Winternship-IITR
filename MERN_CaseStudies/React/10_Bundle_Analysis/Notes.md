# Bundle Analysis in React + TypeScript

## 1. Problem Statement – ShopEase E-Commerce Platform

ShopEase is an e-commerce platform targeting global users, including slow mobile networks.

Over time:

* Many third-party libraries were added (charts, date handling, utilities)
* TypeScript was adopted for safety
* Bundle size increased significantly

This caused:

* Slow initial page loads
* Poor Core Web Vitals
* SEO impact

**Goal:**
Analyze what contributes to bundle size and apply practical optimizations to reduce it.

<br>

## 2. Learning Objectives

By the end of this case study:

* Understand what contributes to bundle size
* Analyze bundles using Vite tools
* Understand the real impact of TypeScript on bundles
* Identify heavy libraries and duplicate code
* Apply tree-shaking, selective imports, and code splitting
* Build an interview-ready optimization workflow

<br>

## 3. Core Concepts

### 3.1 What Affects Bundle Size

* Runtime JavaScript code
* Third-party libraries
* Duplicate dependencies
* Initial vs lazy-loaded chunks
* Imported assets (CSS, images, fonts)

### 3.2 TypeScript and Bundle Size

* Types and interfaces are erased at build time
* Types do NOT increase bundle size
* Some TS features generate JS:

  * enums
  * decorators
* TS helps tree-shaking by making unused exports clearer

<br>

## 4. Tooling Choice (Vite)

For Vite-based projects, we use:

* `npx vite-bundle-visualizer`

This tool analyzes production build output and generates a visual treemap.

<br>

## 5. Project Setup (From Scratch)

### 5.1 Create Project

```bash
npm create vite@latest shopease-bundle -- --template react-ts
cd shopease-bundle
npm install
```

### 5.2 Install Example Heavy Libraries

```bash
npm install lodash moment chart.js
```

These libraries are intentionally added to demonstrate bundle growth.

<br>

## 6. Folder Structure

```
src/
│
├─ main.tsx
├─ App.tsx
│
├─ pages/
│   ├─ Home.tsx
│   ├─ Analytics.tsx
│   └─ Admin.tsx
│
└─ utils/
    └─ date.ts
```

<br>

## 7. Entry Point

### 7.1 `main.tsx`

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

<br>

## 8. Initial (Unoptimized) Implementation

### 8.1 `App.tsx`

```tsx
import Home from "./pages/Home";
import Analytics from "./pages/Analytics";

export default function App() {
  return (
    <>
      <Home />
      <Analytics />
    </>
  );
}
```

<br>

### 8.2 `pages/Home.tsx` (Heavy Date Library)

```tsx
import moment from "moment";

export default function Home() {
  const today = moment().format("YYYY-MM-DD");
  return <div>Today: {today}</div>;
}
```

<br>

### 8.3 `pages/Analytics.tsx` (Heavy Chart Library)

```tsx
import Chart from "chart.js/auto";

export default function Analytics() {
  console.log(Chart);
  return <div>Analytics Page</div>;
}
```

<br>

### 8.4 `utils/date.ts` (Bad Lodash Import)

```ts
import _ from "lodash";

export function debounceFn(fn: (...args: any[]) => any) {
  return _.debounce(fn, 300);
}
```

<br>

## 9. Build and Analyze Bundle

### 9.1 Build Project

```bash
npm run build
```

### 9.2 Run Bundle Analyzer

```bash
npx vite-bundle-visualizer
```

### 9.3 What the Report Shows

* Large blocks for `moment`, `chart.js`, and `lodash`
* All heavy libraries included in the initial bundle
* Clear visualization of bundle weight distribution

<br>

## 10. Optimization Techniques Applied

### 10.1 Replace Moment.js

#### Before

```ts
import moment from "moment";
```

#### After

```ts
import { format } from "date-fns";

format(new Date(), "yyyy-MM-dd");
```

<br>

### 10.2 Optimize Lodash Import (Tree-Shaking)

#### Before

```ts
import _ from "lodash";
```

#### After (Final Correct Version)

```ts
import debounce from "lodash/debounce";

export function debounceFn<T extends (...args: any[]) => any>(fn: T) {
  return debounce(fn, 300);
}
```

<br>

### 10.3 Lazy Load Heavy Analytics Page

```tsx
import { lazy, Suspense } from "react";

const Analytics = lazy(() => import("./pages/Analytics"));

<Suspense fallback={<div>Loading analytics...</div>}>
  <Analytics />
</Suspense>
```

This moves `chart.js` out of the initial bundle.

<br>

## 11. TypeScript Configuration for Better Bundling

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "ESNext",
    "moduleResolution": "node",
    "isolatedModules": true,
    "declaration": false,
    "removeComments": true
  }
}
```

<br>

## 12. Rebuild and Re-Analyze

```bash
npm run build
npx vite-bundle-visualizer
```



<img width="1881" height="1022" alt="image" src="https://github.com/user-attachments/assets/71623933-f9f8-4856-bbde-269dbccafe71" />

### Observed Improvements

* Smaller initial JS chunk
* Reduced vendor bundle size
* Heavy libraries loaded only when needed

<br>

## 13. Key Takeaways

* Bundle size comes from runtime JavaScript, not TypeScript types
* Library choice has the biggest impact
* Import style matters for tree-shaking
* Analyze before optimizing
* Measure → optimize → re-measure

<br>

## 14. Final Outcome

* Faster initial load
* Improved performance on slow networks
* Reduced bundle size
* Production-ready and interview-ready bundle optimization workflow
