# Gemini Journal

A private, authenticated personal memory journal web application built with **Google AI Studio** and prepared for the **Google Cloud Run AI Challenge**. 

Gemini Journal provides a personal, reflective digital archive where users preserve authentic life memories in their own words. The application combines isolated cloud data persistence, multi-turn AI conversation grounded in personal journal entries, and security architecture designed around Google Cloud and Firebase standards.

---

## Overview

Gemini Journal was architected from the ground up using **Google AI Studio** to demonstrate a production-ready, full-stack AI-enabled application. The system guarantees that the user's original journal text remains the immutable source of truth while enabling an interactive Gemini conversational assistant to converse about, reflect upon, and explore recorded memories.

All user content is protected behind federated Google Authentication, ensuring that private memories are inaccessible until authenticated. Cloud Firestore security rules strictly isolate data by user identity, and server-side secret management ensures zero client-side credential exposure.

---

## Key Features

### 1. Firebase Authentication
- **Federated Google Sign-In**: Secure sign-in powered by Firebase Authentication (`GoogleAuthProvider`) with account selection prompts.
- **Session Persistence**: Authentication state persists across browser refreshes, automatically restoring the user's private session.
- **Immediate Session Teardown**: Clean sign-out routine that terminates real-time listeners and clears in-memory state.

### 2. User-Isolated Cloud Firestore Memories
- **Strict Data Segregation**: Every memory is stored under the authenticated user's isolated subcollection (`/users/{userId}/memories/{memoryId}`).
- **Real-Time Synchronization**: Changes made in one session or tab synchronize immediately via Firestore snapshot listeners (`onSnapshot`).
- **Offline & Reconnect Resilience**: Built-in network error handling and recovery banners.

### 3. Chronological Timeline
- **Chronological Visualization**: Memories are presented along a timeline with dates, titles, and original journal text.
- **Real-Time Search & Filtering**: Instant search across titles, memory narratives, and dates.
- **Sort Ordering**: One-click toggle between descending (newest first) and ascending (oldest first) chronological order.

### 4. Full Memory Management (Create, Edit, Delete)
- **Create**: Add new memories with dates, titles, and multi-paragraph journal entries.
- **Edit**: Update existing memories with instantaneous cloud synchronization.
- **Delete**: Safe deletion dialog preventing accidental data loss, immediately pruning the Firestore document.
- **Starter Template Option**: New users can populate curated starter entries to explore timeline features immediately.

### 5. Gemini Multi-Turn Conversation
- **Memory-Grounded Dialogue**: A conversational interface powered by the `@google/genai` SDK that references the user's journal entries as grounding context.
- **Multi-Turn Context**: Preserves ongoing chat history across the session for natural follow-up queries, emotional reflection, and thematic discovery.
- **Resilient Fallback Ladder**: Server-side helper handles API rate limits or transient errors through an automated availability fallback chain (`gemini-3.8-flash` primary).

### 6. Focused-Memory Conversation & Clear Focus
- **Timeline-to-Chat Focus**: Clicking **"Converse"** on any timeline memory card transitions directly to Gemini Chat with that specific memory highlighted and loaded as priority context.
- **Verbatim Text Inspector**: Expandable context viewer within the chat window showing the exact original journal text under discussion.
- **Clear Focus**: A one-click **"Clear focus"** control allows users to switch effortlessly between conversing about an individual memory and exploring all memories across their entire journal.

### 7. In-Memory Local Photo & Video Attachments
- **Client-Side Media Selection**: Users can attach photos (`.png`, `.jpg`, `.webp`) and videos (`.mp4`, `.webm`) directly from their device.
- **Instant Preview**: Previews load immediately using in-browser object URLs without external cloud storage costs or requiring paid Firebase Blaze plans.
- **Strict Zero-Base64 & Zero-Storage Hygiene**: Media is never uploaded to remote storage buckets or serialized as Base64 into Firestore documents.
- **Memory Leak Protection**: Comprehensive lifecycle management automatically calls `URL.revokeObjectURL()` upon attachment removal, modal cancellation, memory deletion, and component unmounting.

### 8. Google Cloud Secret Manager Integration
- **Server-Side API Key Resolution**: The application dynamically fetches the `GEMINI_API_KEY` from Google Cloud Secret Manager at runtime, with fallback to environment variables.
- **Zero Client-Side Exposure**: All Gemini AI interactions are routed through an Express backend proxy (`/api/chat`). API credentials never touch the browser.

---

## Technology Stack

| Layer | Technologies |
| :--- | :--- |
| **Development Platform** | **Google AI Studio** |
| **AI & LLM Engine** | **Gemini** via `@google/genai` TypeScript SDK |
| **Authentication** | **Firebase Authentication** (Google Sign-In) |
| **Database** | **Cloud Firestore** (User-isolated collections) |
| **Secret Management** | **Google Cloud Secret Manager** (`@google-cloud/secret-manager`) |
| **Backend Runtime** | **Node.js (v20)**, **Express (v4)**, **TypeScript** |
| **Frontend Framework** | **React (v19)**, **Vite (v6)**, **Tailwind CSS (v4)**, **Lucide Icons** |
| **Deployment Target** | **Google Cloud Run** (Serverless container deployment) |

---

## Security Architecture

Gemini Journal enforces a zero-trust, defense-in-depth security model:

1. **Authentication Gate**: Unauthenticated visitors are restricted to the sign-in screen; no journal content or user metadata is rendered until authentication succeeds.
2. **Owner-Bound Firestore Rules**: Database security rules enforce that users can only read, write, or delete documents within their own subcollection (`request.auth.uid == userId`):
   ```javascript
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/memories/{memoryId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```
3. **Zero Hardcoded Secrets**: No API keys, passwords, or service account credentials exist in the source code or client bundle. Credentials are provided via Google Cloud Secret Manager or container environment injection.
4. **Data Sanitization & Injection Prevention**: Payloads are sanitized before writing to Firestore to strip `undefined` fields and reject invalid data schemas. User inputs to the Gemini proxy are passed as structured context rather than raw prompt concatenations.

---

## Local Development & Setup

### Prerequisites
- **Node.js**: Version 20 or higher
- **npm**: Version 10 or higher
- A Google Cloud Project with Cloud Firestore enabled
- A Gemini API Key from Google AI Studio

### 1. Clone the Repository
```bash
git clone <REPOSITORY_URL>
cd gemini-journal
```

### 2. Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```
Populate `.env` with your development credentials:
```env
GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
```

### 3. Firebase Configuration
Ensure your Firebase web configuration is present in `firebase-applet-config.json`:
```json
{
  "projectId": "YOUR_PROJECT_ID",
  "appId": "YOUR_APP_ID",
  "apiKey": "YOUR_FIREBASE_API_KEY",
  "authDomain": "YOUR_PROJECT_ID.firebaseapp.com",
  "firestoreDatabaseId": "(default)",
  "storageBucket": "YOUR_PROJECT_ID.firebasestorage.app",
  "messagingSenderId": "YOUR_MESSAGING_SENDER_ID"
}
```

### 4. Install Dependencies
```bash
npm install
```

### 5. Start the Full-Stack Development Server
```bash
npm run dev
```
The application will be accessible at `http://localhost:3000`.

### 6. Build and Type Check
```bash
# Run TypeScript compilation check
npm run lint

# Build client assets and backend server bundle for production
npm run build

# Test production build locally
npm run start
```

---

## Google Cloud Run Deployment

Gemini Journal includes a multi-stage `Dockerfile` optimized for container execution on Cloud Run.

### 1. Enable Required Google Cloud APIs
```bash
gcloud services enable \
  run.googleapis.com \
  secretmanager.googleapis.com \
  firestore.googleapis.com
```

### 2. Set Up Secret Manager for the Gemini API Key
```bash
# 1. Create the secret
gcloud secrets create GEMINI_API_KEY --replication-policy="automatic"

# 2. Add your Gemini API key as a secret version
echo -n "YOUR_GEMINI_API_KEY" | gcloud secrets versions add GEMINI_API_KEY --data-file=-

# 3. Grant the Cloud Run compute service account access to read the secret
PROJECT_NUMBER=$(gcloud projects describe $(gcloud config get-value project) --format='value(projectNumber)')

gcloud secrets add-iam-policy-binding GEMINI_API_KEY \
  --member="serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### 3. Deploy the Service to Cloud Run
```bash
gcloud run deploy gemini-journal \
  --source . \
  --region asia-southeast1 \
  --allow-unauthenticated \
  --set-secrets GEMINI_API_KEY=GEMINI_API_KEY:latest
```

### 4. Apply the Cloud Run AI Challenge Verification Label
Apply the mandatory challenge label to register the service for automated evaluation:
```bash
gcloud run services update gemini-journal \
  --update-labels=dev-tutorial=cloud-run-ai-challenge \
  --region asia-southeast1
```

---

## Testing & Functional Verification

Follow this walkthrough to verify every user workflow:

1. **Sign In / Sign Up**:
   - Navigate to the application. Verify the unauthenticated welcome screen appears.
   - Click **"Sign in with Google"**. Complete authentication via the Google popup.
   - Confirm you are redirected to the personal timeline view with your email displayed in the header.

2. **Create a Memory**:
   - Click **"+ New Memory"**.
   - Enter a date, title, and detailed journal entry.
   - Optionally attach a photo or video from your device to preview the attachment.
   - Click **"Save Memory"**. Verify the memory card appears on your timeline.

3. **Timeline Search & Sort**:
   - Type a keyword into the search bar. Verify memories filter dynamically in real time.
   - Toggle the chronological sort button (newest vs. oldest) and confirm the order updates.

4. **Edit & Delete Memory**:
   - Click the pencil icon on any memory card. Update the text and save. Confirm the change reflects immediately.
   - Click the trash icon on a memory card. Confirm deletion in the modal dialog. Verify the memory is removed from the timeline and Firestore.

5. **Focused & Multi-Turn Gemini Conversation**:
   - Click **"Converse"** on a memory card. Confirm navigation to the **Gemini Chat** screen with that memory focused.
   - Send a question about the memory (e.g., *"What emotions stand out in this memory?"*). Verify Gemini responds with accurate, grounded reflections.
   - Send a follow-up query to verify multi-turn context retention.
   - Click **"Clear focus"** and query across your broader journal history.

6. **Authentication Persistence & Sign Out**:
   - Refresh the browser page. Confirm your authentication session and timeline memories reload without requiring another sign-in.
   - Click **"Sign Out"** in the header. Confirm immediate session teardown and return to the login screen.

---

## Google AI Studio & Cloud Run AI Challenge Notes

This project was built iteratively using **Google AI Studio** as an end-to-end cloud-native engineering workflow:

- **Iterative Cloud Prototyping**: Developed with continuous verification across authentication, Firestore schema design, and Gemini model interactions.
- **Custom Security Instructions**: Implemented agentic threat modeling across all five threat zones (Input Surfaces, Planning & Reasoning, Tool Execution, Memory & State, and Inter-System Communication).
- **Expanded Capabilities**: Extended the baseline journal application with dynamic Google Cloud Secret Manager resolution, resilient model fallback pipelines, strict zero-leakage local media handling, and owner-bound database rules ready for production deployment on Google Cloud Run.
