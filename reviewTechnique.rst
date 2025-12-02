Reviewing complex React Native project code effectively requires a structured, multi-layered approach. Here’s a practical, battle-tested workflow you can use solo or with a team:

✅ 1. Start With High-Level Context

Before diving into files:

Understand the architecture

Is it Expo or bare React Native?

What state management is used? (Redux, Zustand, Recoil, Jotai, MobX)

Navigation system? (React Navigation, Native Navigation)

API layer & data fetching? (REST, GraphQL, React Query)

Documents to look for

README

Architecture diagram

Folder structure documentation

.env.example / config files

✅ 2. Review the Folder & Module Structure

A complex project should have a clear and consistent layout. Check for:

✔️ Clear separation of concerns

/components

/screens

/hooks

/services or /api

/context or /store

/utils

/assets

✔️ Things to flag

Deeply nested imports that indicate poor structure

Components doing too many things (UI + logic + network requests)

✅ 3. Analyze the State Management Approach

This is where complexity usually hides.

Check:

Where state lives

How global state is updated

Logical separation of local vs global state

Actions/selectors/reducers (if Redux)

Side effects (Thunk, Saga, React Query)

Red flags

Global state used when local state is enough

Excessive prop drilling

Async logic scattered all over the code

✅ 4. Review Component Structure

For each major component:

Look for:

Clear separation of UI vs business logic

Overly large components (300+ lines)

Reusable components vs duplicated UI

Hooks extracted from components where possible

Apply “The 3 Questions Test”

Is this component doing too much?

Can logic be extracted into a hook?

Can UI be extracted into smaller components?

✅ 5. Inspect the API / Networking Layer

Check for:

✔️ API consistency

Centralized API client (Axios / Fetch wrapper)

Error handling strategy

Token management & refresh logic

React Query usage (if applicable)

❌ Red flags

API logic inside components

Repeated API calls due to missing caching

No retry/backoff strategies

✅ 6. Review Styling & Theming

Look for:

Consistent use of StyleSheet, tailwind, or styled-components

A unified theme (colors, spacing, typography)

Shared design tokens

Red flags

Hardcoded styles everywhere

Inconsistent padding/margins

No dark mode strategy (if required)

✅ 7. Evaluate Type Safety

Even if not using TypeScript, check:

PropTypes or TS interfaces

Correct navigation types (TS types for route params)

Clear function signatures

✅ 8. Performance Review

Look for:

🔍 Performance pitfalls:

Unnecessary re-renders

Inline functions without memoization

Large lists without FlatList optimizations (keyExtractor, getItemLayout, initialNumToRender)

Heavy computations on the UI thread

Tools:

React DevTools Profiler

Flipper for React Native

✅ 9. Test Coverage

Evaluate:

Unit tests (Jest)

Component tests (React Native Testing Library)

E2E tests (Detox)

Check if tests cover:

Critical flows

Major screens

High-risk logic

✅ 10. Run & Interact With the App

Final step before providing feedback:

Build and run on both iOS and Android

Log errors & warnings

Test edge cases: offline mode, slow network, invalid user input

🔥 Bonus: Code Review Checklist (Copy/Paste)
Architecture

 Is the folder structure logical and clean?

 Is there unnecessary complexity?

Components

 Are components small and focused?

 Is logic extracted into hooks?

State Management

 Is state placed at the correct level?

 Are async flows structured and predictable?

API Layer

 Is network logic centralized?

 Are errors handled properly?

Performance

 Any wasted renders?

 Are lists properly optimized?

Styling

 Are styles consistent and centralized?

Types/Tests

 Are types clear and consistent?

 Are critical paths tested?