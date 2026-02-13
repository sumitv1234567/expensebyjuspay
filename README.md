# Project Documentation

## 📖 1. Project Overview
**Expense Native App** is a comprehensive expense management mobile application built for enterprise use. It enables users to submit, track, and manage reimbursement claims efficiently. The app focuses on a seamless user experience with features like biometric authentication, offline support, bulk invoice uploading, and granular policy compliance checks.

The application is built using **React Native** and **Expo**, leveraging modern mobile development practices.

---

## 🏗 2. Architecture & Tech Stack

### 2.1 Technology Stack
- **Framework:** [React Native](https://reactnative.dev/) (v0.81.5) with [Expo](https://expo.dev/) (SDK 54)
- **Language:** TypeScript
- **Navigation:** [Expo Router](https://docs.expo.dev/router/introduction/) (File-based routing, strict typed routes)
- **State Management:**
    - **Global Client State:** [Zustand](https://github.com/pmndrs/zustand) (Simple, unopinionated state management)
    - **Server State:** [TanStack React Query](https://tanstack.com/query/latest) (Caching, optimistic updates, background fetching)
- **Networking:** [Axios](https://axios-http.com/) with a custom interceptor layer for auth headers.
- **Form Handling:** [React Hook Form](https://react-hook-form.com/)
- **UI & Styling:**
    - Custom Design System (Found in `constants/colors.ts`, `fonts.ts`)
    - [SolarIcons](components/SolarIcons) (Custom SVG icon set)
    - [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/) & [Gesture Handler](https://docs.swmansion.com/react-native-gesture-handler/) for interactions.
    - [@gorhom/bottom-sheet](https://gorhom.github.io/react-native-bottom-sheet/) for modal interactions.

### 2.2 Application Structure (Navigation)
The app uses a filesystem-based routing structure via Expo Router:

- **`app/_layout.tsx`**: The robust Root Layout. It handles:
    - Global Providers (`QueryProvider`, `AuthProvider`, `ApiDataProvider`, `ErrorModalProvider`).
    - Notification permissions and listeners.
    - Biometric data cleanup on first launch.
    - System UI configuration (Status Bar, Safe Areas).

- **`app/(auth)`**: Authentication screens.
    - `login/`: Login flow, likely including "Forgot Password".

- **`app/(protected)`**: Authenticated application area.
    - **`app/(protected)/(tabs)`**: Main Tab Navigator.
        - **Overview**: Dashboard/Summary (`index.tsx`).
        - **Expense**: Main list of claims/invoices (`(expense)/index.tsx`).
        - **Cards**: Corporate card management (`cards.tsx`).
        - **Genius**: AI Assistant features (Conditional, `genius.tsx`).
        - **Profile**: User settings (`profile.tsx`).

---

## ⚡ 3. Key Features & Workflows

### 3.1 Authentication & Security
- **Biometric Login:**
    - Supports FaceID and TouchID via `expo-local-authentication`.
    - **Security:** Uses RSA key pairs. Private key stored in `SecureStore`, Public key sent to backend.
    - **Strict Mode:** Keys are invalidated if the device's biometric enrollment changes.
    - **Documentation:** [BIOMETRIC_FLOW.md](docs/BIOMETRIC_FLOW.md)

### 3.2 Expense Management
The core module of the application. It handles the lifecycle of an expense claim.

- **Creating Expenses (`components/expense/addExpenseModal`):**
    - **Multi-modal Input:**
        - **Scan Receipt:** Uses `expo-camera` via `ScanView.tsx` to capture receipts.
        - **Manual Entry:** Uses `ManualEntryView.tsx` with extensive form validation logic.
        - **File Upload:** detailed in "Bulk Upload".
    - **Drafting:** Expenses start as `DRAFTED` status.

- **Bulk Upload:**
    - Users can upload multiple files (PDFs, Images) simultaneously.
    - **Optimistic UI:** Uploads appear immediately as "Processing..." items using a local optimistic state before server confirmation.
    - **Sync:** Automatically syncs with backend upon completion notification.
    - **Documentation:** [BULK_UPLOAD_IMPLEMENTATION.md](docs/BULK_UPLOAD_IMPLEMENTATION.md)

- **Viewing & Managing:**
    - **`ClaimsList`**: A complex list component handling grouped expenses (by date) and folders.
    - **Filtering:** Status-based tabs (Drafted, Submitted, Approved, etc.).
    - **Details View:** `invoice-detail.tsx` provides deep-dive view into a specific invoice, including approval history timeline (`ActivityTimeline.tsx`).

### 3.3 Data Model
Key entities defined in `types/expense.ts`:
- **`InvoiceDetail`**: The central object representing an expense line item. Contains amount, merchant, status, approval history, and dynamic fields (`expenseData`).
- **`FolderDetail`**: Represents a collection/report of expenses submitted together.
- **`InvoiceStatus`**: State machine states like `DRAFTED`, `SUBMITTED`, `APPROVED`, `REIMBURSED`.

---

## 📂 4. Detailed Folder Structure

```
/
├── apis/                 # API definitions. Separated by domain (auth, expense, cards).
│   ├── expense.ts        # Expense-related endpoints.
│   └── auth.ts           # Authentication endpoints.
├── app/                  # Application screens & routes (Expo Router).
│   ├── (auth)/           # Unauthenticated routes.
│   ├── (protected)/      # Authenticated routes.
│   └── _layout.tsx       # Root configuration.
├── assets/               # Static assets (fonts, images).
├── components/           # Reusable UI components.
│   ├── expense/          # Expense-specific components (Domain Driven Design).
│   └── ...               # Shared atoms/molecules.
├── constants/            # Design tokens (Colors.ts, Fonts.ts, Enums).
├── contexts/             # React Context Providers.
│   ├── AuthProvider.tsx  # User session management.
│   └── ...               # ApiData, ErrorModal, etc.
├── docs/                 # Detailed technical documentation.
├── hooks/                # Custom hooks.
│   ├── useBiometric.ts   # Biometric logic encapsulation.
│   ├── useFileUpload.ts  # Bulk upload logic.
│   └── ...
├── services/             # External services (Notifications, etc.).
├── store/                # Zustand stores (Global state).
│   └── feature.ts        # Feature flagging store.
├── types/                # TypeScript type definitions.
│   ├── expense.ts        # Data models for expenses.
│   └── ...
└── utils/                # Helper functions (Formatting, Validation).
```

---

## 🚀 5. Setup & Development

### 5.1 Prerequisites
- Node.js (Latest Stable)
- Package Manager: `pnpm` (Inferred from `package.json` lockfile) or `npm`.
- XCode (iOS) or Android Studio (Android) for native builds.

### 5.2 Running Locally
1.  **Install Dependencies:**
    ```bash
    npm install
    ```
2.  **Start Development Server:**
    ```bash
    npm start
    # or
    npx expo start
    ```
3.  **Run on Simulator/Emulator:**
    - Press `i` for iOS Simulator.
    - Press `a` for Android Emulator.

### 5.3 Environment Variables
The app uses `.env` files for configuration.
- `development`: `EXPO_PUBLIC_ENV=development`
- `sandbox`: `EXPO_PUBLIC_ENV=SANDBOX`
- `production`: `EXPO_PUBLIC_ENV=PROD`
See `package.json` scripts for how these are injected during build time.

### 5.4 Linting & Formatting
- **Lint:** `npm run lint` (ESLint)
- **Format:** `npm run format` (Prettier)
- **Type Check:** `tsc` (TypeScript Compiler)

---

## 🔧 6. API & State Management
- **API Layer**: `apis/` folder contains pure functions that return Promises. They are typically wrapped in React Query hooks (e.g., `useAuthInfoQuery`).
- **React Query**: Used for almost all server state. Key configuration is in `contexts/QueryProvider.tsx`.
- **Background Sync**: The app relies on React Query's `refetchOnWindowFocus` and explicit invalidation (e.g., after upload) to keep data in sync.

---

## 🐛 7. Troubleshooting
- **Biometrics Issues**: If biometrics fail repeatedly, the app enters a "lockout" state. Clear app data or reinstall to reset keys (handled in `_layout.tsx`).
- **Upload Stalls**: Check `upload-utils.ts` logic. The app uses an opportunistic approach; ensure network connectivity is stable.
