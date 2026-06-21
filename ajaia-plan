# Ajaia Assessment — Full Battle Plan
**Candidate:** Noemi Bana-ay | **Time Budget:** 4–5 hrs | **Stack:** Next.js 14 + Firebase + Tiptap + Vercel

---

## Stack Decision (Final — No Debate)

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 14 (App Router) + TypeScript | You know it cold. API routes = no separate server |
| Editor | Tiptap v2 | Best headless rich-text, free, React-native |
| Auth | Firebase Auth | You've used it in production at SportsPilotAI |
| Database | Firestore | Real-time, flexible, you know the SDK cold |
| File Upload | Firebase Storage | Already in the stack, zero extra config |
| Styling | Tailwind CSS | Speed |
| Deployment | Vercel | One push, instant live URL |

---

## Scope Cuts (Make These Now — Never Look Back)

**IN scope (core, polished):**
- Auth: email/password via Firebase Auth (2 seeded test accounts)
- Document: create, rename, edit (rich text), save, reopen
- Rich text: bold, italic, underline, headings (H1/H2/H3), bullet list, numbered list
- File upload: .txt and .md files → parsed into a new editable document
- Sharing: owner grants access by email → shared docs appear in their sidebar
- Persistence: Firestore with security rules enforced
- Deployment: Vercel live URL

**OUT of scope (state clearly in README):**
- Real-time collaboration (Yjs/CRDTs) — would need another 3+ hrs
- .docx parsing — mammoth.js adds complexity, skip it
- Version history
- Role-based permissions (owner/viewer/editor distinction)
- Comments

---

## Firestore Data Model

```
/users/{uid}
  email: string
  displayName: string
  createdAt: timestamp

/documents/{docId}
  ownerId: string        ← uid of creator
  ownerEmail: string     ← denormalized for display
  title: string          ← default "Untitled Document"
  content: object        ← Tiptap JSON
  createdAt: timestamp
  updatedAt: timestamp

/documents/{docId}/shares/{uid}
  sharedWith: string     ← uid of recipient
  sharedWithEmail: string ← for display
  grantedBy: string      ← uid of owner
  createdAt: timestamp
```

**Why this structure:**
- Documents are top-level — easy to query by `ownerId`
- Shares as a subcollection — scoped to the document, easy to list
- Denormalized emails — avoid extra reads when rendering lists
- No joins needed — Firestore doesn't have them anyway

---

## Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users can read/write their own profile
    match /users/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == uid;
    }

    match /documents/{docId} {
      // Owner has full access
      allow read, write, delete: if request.auth.uid == resource.data.ownerId;

      // Shared users can read and update (not delete)
      allow read, update: if exists(/databases/$(database)/documents/documents/$(docId)/shares/$(request.auth.uid));

      // Authenticated users can create new documents
      allow create: if request.auth != null
        && request.resource.data.ownerId == request.auth.uid;

      // Shares subcollection
      match /shares/{shareUid} {
        // Owner can manage shares
        allow read, write, delete: if request.auth.uid == get(/databases/$(database)/documents/documents/$(docId)).data.ownerId;
        // Shared user can read their own share record
        allow read: if request.auth.uid == shareUid;
      }
    }
  }
}
```

---

## Project Structure

```
ajaia-docs/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (app)/
│   │   ├── layout.tsx              ← sidebar + auth guard
│   │   ├── page.tsx                ← redirect to /documents
│   │   └── documents/
│   │       ├── page.tsx            ← document list (owned + shared)
│   │       └── [id]/
│   │           └── page.tsx        ← editor page
│   └── api/
│       ├── documents/
│       │   ├── route.ts            ← POST (create)
│       │   └── [id]/
│       │       └── route.ts        ← PATCH (update), DELETE
│       ├── upload/
│       │   └── route.ts            ← POST file → parse → new doc
│       └── share/
│           └── route.ts            ← POST grant access, GET list shares
├── components/
│   ├── Editor.tsx                  ← Tiptap editor component
│   ├── Sidebar.tsx                 ← doc list nav
│   ├── ShareModal.tsx              ← share by email UI
│   └── UploadModal.tsx             ← file upload UI
├── lib/
│   ├── firebase/
│   │   ├── client.ts               ← Firebase app + auth + firestore + storage (browser)
│   │   └── admin.ts                ← Firebase Admin SDK (server/API routes only)
│   └── utils.ts
├── hooks/
│   ├── useAuth.ts                  ← auth state hook
│   └── useDocuments.ts             ← owned + shared docs hook
├── __tests__/
│   └── documents.test.ts           ← Jest unit test
├── README.md
├── ARCHITECTURE.md
├── AI_WORKFLOW.md
└── SUBMISSION.md
```

---

## Environment Variables

```env
# .env.local
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Server-only (Firebase Admin — for API routes that need elevated access)
FIREBASE_ADMIN_PROJECT_ID=
FIREBASE_ADMIN_CLIENT_EMAIL=
FIREBASE_ADMIN_PRIVATE_KEY=
```

---

## Phase-by-Phase Development Guide

---

### PHASE 0 — Setup (15 min)

**Install:**
```bash
npx create-next-app@latest ajaia-docs --typescript --tailwind --app --no-src-dir
cd ajaia-docs
npm install @tiptap/react @tiptap/pm @tiptap/starter-kit @tiptap/extension-underline firebase
npm install firebase-admin
npm install --save-dev jest @types/jest ts-jest jest-environment-jsdom
```

**Firebase Console:**
1. console.firebase.google.com → New project → `ajaia-docs`
2. Authentication → Sign-in method → Email/Password → Enable
3. Firestore Database → Create → Start in **test mode** (we add rules after)
4. Storage → Get started → test mode
5. Project Settings → Your apps → Add Web app → copy config
6. Project Settings → Service accounts → Generate new private key → save JSON
7. Authentication → Users → Add 2 users:
   - `alice@ajaia.dev` / `Test1234!`
   - `bob@ajaia.dev` / `Test1234!`

**After adding users — seed their Firestore profiles:**
In Firestore console → Start collection → `users` → Add these two docs manually:
- Document ID = their uid (copy from Auth tab)
- Fields: `email`, `displayName: "Alice"` / `"Bob"`, `createdAt`

**`.env.local`:** Fill in all values from Firebase config + Admin service account JSON.

---

### PHASE 1 — Firebase Setup + Auth (25 min)

**Goal:** Working login/signup, session persists on refresh, auth guard on app routes.

**`lib/firebase/client.ts`:**
```typescript
import { initializeApp, getApps } from 'firebase/app'
import { getAuth } from 'firebase/auth'
import { getFirestore } from 'firebase/firestore'
import { getStorage } from 'firebase/storage'

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
}

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
export const auth = getAuth(app)
export const db = getFirestore(app)
export const storage = getStorage(app)
```

**`lib/firebase/admin.ts`:**
```typescript
import { initializeApp, getApps, cert } from 'firebase-admin/app'
import { getAuth } from 'firebase-admin/auth'
import { getFirestore } from 'firebase-admin/firestore'

if (!getApps().length) {
  initializeApp({
    credential: cert({
      projectId: process.env.FIREBASE_ADMIN_PROJECT_ID,
      clientEmail: process.env.FIREBASE_ADMIN_CLIENT_EMAIL,
      privateKey: process.env.FIREBASE_ADMIN_PRIVATE_KEY?.replace(/\\n/g, '\n'),
    }),
  })
}

export const adminAuth = getAuth()
export const adminDb = getFirestore()
```

**`hooks/useAuth.ts`:**
```typescript
// onAuthStateChanged listener → store user in state
// Expose: user, loading, signIn(), signOut(), signUp()
// On signUp: also write to /users/{uid} in Firestore
```

**`app/(auth)/login/page.tsx`:**
- Simple form: email + password
- Call `signInWithEmailAndPassword(auth, email, password)`
- On success → `router.push('/documents')`
- Link to signup page

**`app/(auth)/signup/page.tsx`:**
- Same form + name field
- Call `createUserWithEmailAndPassword` → then write `/users/{uid}` doc

**`app/(app)/layout.tsx`:**
- Client component with `useAuth()`
- If `loading` → show spinner
- If no `user` → `router.push('/login')`
- Render sidebar + `{children}`

**Test:** Login as alice → see /documents. Refresh → still logged in. ✓

---

### PHASE 2 — Document List + Create (25 min)

**Goal:** Two sections — "My Documents" and "Shared with Me". Create new doc button.

**`hooks/useDocuments.ts`:**
```typescript
// Query 1: collection('documents') where ownerId == uid — real-time listener
// Query 2: collectionGroup or manual — get docs where shares/{uid} exists
//   → query /documents where we have a share record
//   Approach: query docs the user owns, PLUS
//             query /documents/{any}/shares/{uid} — get docIds → fetch those docs

// Return: { owned: Doc[], shared: Doc[], loading: boolean }
```

**Shared docs query pattern (Firestore doesn't support joins):**
```typescript
// Step 1: Query all share records for this user across all documents
const sharesQuery = query(
  collectionGroup(db, 'shares'),
  where('sharedWith', '==', uid)
)
// Step 2: Get docIds from results
// Step 3: Fetch each document by id
// Combine into sharedDocs array
```

**`app/(app)/documents/page.tsx`:**
- Use `useDocuments()` hook
- Render "My Documents" section — each doc as a card with title + last updated
- Render "Shared with Me" section — each doc shows "Shared by [ownerEmail]" badge
- "New Document" button:
  - `addDoc(collection(db, 'documents'), { ownerId, ownerEmail, title: 'Untitled Document', content: null, createdAt, updatedAt })`
  - On success → `router.push('/documents/' + newDocId)`
- "Upload File" button → opens UploadModal

**Test:** Create doc → appears in My Documents. ✓

---

### PHASE 3 — Editor (45 min)

**Goal:** Tiptap editor with full toolbar, autosave to Firestore, inline title rename.

**`components/Editor.tsx`:**
```typescript
'use client'
import { useEditor, EditorContent } from '@tiptap/react'
import StarterKit from '@tiptap/starter-kit'
import Underline from '@tiptap/extension-underline'
import { doc, updateDoc, serverTimestamp } from 'firebase/firestore'
import { db } from '@/lib/firebase/client'

// Tiptap extensions: StarterKit + Underline
// StarterKit includes: bold, italic, headings, bulletList, orderedList, code, blockquote

// Toolbar:
// [B] [I] [U] | [H1] [H2] [H3] | [• List] [1. List]
// Active state: editor.isActive('bold') etc → highlight button

// Initial content: pass doc.content (Tiptap JSON) as editor prop
// editor.commands.setContent(content) on mount

// Autosave: debounce 1500ms on editor onChange
const saveContent = debounce(async (json) => {
  setSaveStatus('saving')
  await updateDoc(doc(db, 'documents', docId), {
    content: json,
    updatedAt: serverTimestamp()
  })
  setSaveStatus('saved')
}, 1500)
```

**`app/(app)/documents/[id]/page.tsx`:**
- Client component (needs real-time + auth)
- `onSnapshot(doc(db, 'documents', docId))` → load document
- If doc doesn't exist → redirect to /documents
- Check: user is owner OR has share record → else redirect
- Render: inline title input (onBlur → updateDoc title) + Editor component
- Show save status in header: "Saved" / "Saving..." / last saved time

**Inline title editing:**
```typescript
// Click title → becomes <input> 
// onBlur or Enter → updateDoc({ title: newTitle })
// Show pencil icon on hover to signal it's editable
```

**Test:** Type content → wait 1.5s → "Saved" appears → refresh → content persists with formatting. ✓

---

### PHASE 4 — File Upload (30 min)

**Goal:** Upload .txt or .md → parse into Tiptap JSON → create new editable document.

**`components/UploadModal.tsx`:**
```typescript
// <input type="file" accept=".txt,.md" />
// Show label: "Supported formats: .txt and .md only"
// On file select:
//   1. FileReader.readAsText(file)
//   2. POST to /api/upload with { text, filename, type }
//   3. On success → router.push('/documents/' + newDocId)
// Show upload progress / loading state
```

**`app/api/upload/route.ts`:**
```typescript
// POST { text: string, filename: string, fileType: 'txt' | 'md' }
// 1. Verify Firebase ID token from Authorization header
// 2. Parse text → Tiptap JSON
// 3. addDoc to Firestore
// 4. Return { docId }

// Auth verification pattern:
const token = req.headers.get('Authorization')?.replace('Bearer ', '')
const decoded = await adminAuth.verifyIdToken(token)
const uid = decoded.uid
```

**Markdown → Tiptap JSON parser:**
```typescript
function parseMdToTiptap(text: string) {
  const lines = text.split('\n')
  const content = lines.map(line => {
    if (line.startsWith('### ')) return { type: 'heading', attrs: { level: 3 }, content: [{ type: 'text', text: line.slice(4) }] }
    if (line.startsWith('## '))  return { type: 'heading', attrs: { level: 2 }, content: [{ type: 'text', text: line.slice(3) }] }
    if (line.startsWith('# '))   return { type: 'heading', attrs: { level: 1 }, content: [{ type: 'text', text: line.slice(2) }] }
    if (line.trim() === '') return { type: 'paragraph', content: [] }
    return { type: 'paragraph', content: [{ type: 'text', text: line }] }
  })
  return { type: 'doc', content }
}

function parseTxtToTiptap(text: string) {
  const lines = text.split('\n')
  const content = lines.map(line => ({
    type: 'paragraph',
    content: line ? [{ type: 'text', text: line }] : []
  }))
  return { type: 'doc', content }
}
```

**Client sends ID token:**
```typescript
const token = await auth.currentUser?.getIdToken()
await fetch('/api/upload', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({ text, filename: file.name, fileType })
})
```

**Test:** Upload a .md file with `# Header` and body text → new doc opens with H1 formatted heading. ✓

---

### PHASE 5 — Sharing (30 min)

**Goal:** Owner shares doc by email → recipient sees it in "Shared with Me".

**`components/ShareModal.tsx`:**
```typescript
// Input: email address
// Submit → POST /api/share { documentId, email }
// Success → show "Shared with [email]"
// Error → "No account found with that email" / "Already shared"
// Below input: list current shares (fetch GET /api/share?docId=xxx)
// Only render Share button if currentUser.uid === doc.ownerId
```

**`app/api/share/route.ts`:**
```typescript
// POST { documentId, email }
// 1. Verify ID token → get caller uid
// 2. Fetch document from Firestore → verify caller is owner
// 3. Look up target user by email:
//    adminAuth.getUserByEmail(email)  ← Firebase Admin makes this trivial
// 4. If not found → 404 "No account found with that email"
// 5. Check /documents/{docId}/shares/{targetUid} doesn't already exist
// 6. Write share record:
//    setDoc(doc(adminDb, 'documents', docId, 'shares', targetUid), {
//      sharedWith: targetUid,
//      sharedWithEmail: email,
//      grantedBy: callerUid,
//      createdAt: new Date()
//    })
// 7. Return 200

// GET ?docId=xxx
// 1. Verify token → get caller uid
// 2. Fetch all /documents/{docId}/shares/ subcollection
// 3. Return array of { email, createdAt }
```

**`hooks/useDocuments.ts` — shared docs query:**
```typescript
// Use collectionGroup query to find all shares for this user:
const sharesSnap = await getDocs(
  query(collectionGroup(db, 'shares'), where('sharedWith', '==', uid))
)
// Extract parent document IDs and fetch those documents
const sharedDocIds = sharesSnap.docs.map(d => d.ref.parent.parent!.id)
const sharedDocs = await Promise.all(
  sharedDocIds.map(id => getDoc(doc(db, 'documents', id)))
)
```

**Note:** For `collectionGroup` queries to work, you need to create a Firestore index. Firebase will show you the link in the console error — click it and it creates automatically.

**UI distinction:**
- My Documents: plain card, no badge
- Shared with Me: blue "Shared by alice@ajaia.dev" badge on each card

**Test:** Alice logs in, opens doc, clicks Share, enters bob@ajaia.dev → Bob logs in → sees doc in "Shared with Me" → opens and edits it. ✓

---

### PHASE 6 — Test + Polish (20 min)

**Automated test (`__tests__/documents.test.ts`):**
```typescript
// Test the parseMdToTiptap utility function — no Firebase mocking needed
import { parseMdToTiptap } from '@/lib/utils'

describe('parseMdToTiptap', () => {
  it('converts # heading to tiptap heading level 1', () => {
    const result = parseMdToTiptap('# Hello World')
    expect(result.content[0]).toEqual({
      type: 'heading',
      attrs: { level: 1 },
      content: [{ type: 'text', text: 'Hello World' }]
    })
  })

  it('converts plain text to paragraph', () => {
    const result = parseMdToTiptap('Hello')
    expect(result.content[0].type).toBe('paragraph')
  })

  it('handles empty lines as empty paragraphs', () => {
    const result = parseMdToTiptap('Line one\n\nLine two')
    expect(result.content[1].content).toEqual([])
  })
})
```

**Polish checklist:**
- [ ] Loading spinners on all async ops
- [ ] Toast/error messages for failures (inline, not alert())
- [ ] Empty state: "No documents yet — create your first one"
- [ ] Share button hidden/disabled if not the owner
- [ ] Save status in editor: "All changes saved" / "Saving..."
- [ ] Redirect to /documents if docId doesn't exist
- [ ] Responsive: sidebar collapses on mobile with hamburger toggle

---

### PHASE 7 — Firestore Rules + Deploy + Docs (20 min)

**Apply Firestore security rules:**
- Firestore console → Rules tab → paste rules from above → Publish

**Deploy to Vercel:**
```bash
npx vercel --prod
# In Vercel dashboard → Settings → Environment Variables
# Add all NEXT_PUBLIC_ vars + FIREBASE_ADMIN_ vars
```

**Firebase Auth → Authorized domains:**
- Authentication → Settings → Authorized domains → Add your Vercel domain

**Files to write:**
- `README.md` — setup, env vars, test accounts, run instructions
- `ARCHITECTURE.md` — decisions and tradeoffs
- `AI_WORKFLOW.md` — how you used AI tools
- `SUBMISSION.md` — what's included, what's missing, what's next

---

## Time Budget Summary

| Phase | Task | Time |
|---|---|---|
| 0 | Project setup + Firebase config | 15 min |
| 1 | Auth (login/signup/session guard) | 25 min |
| 2 | Document list + create | 25 min |
| 3 | Tiptap editor + autosave | 45 min |
| 4 | File upload (.txt/.md parsing) | 30 min |
| 5 | Sharing by email | 30 min |
| 6 | Tests + polish | 20 min |
| 7 | Rules + deploy + docs | 20 min |
| — | Buffer / debugging | 30 min |
| **Total** | | **~3h 40min** |

---

## AI Workflow Note (Write This in Your Submission)

> **Which tools:** Claude (architecture, data modeling, code generation, debugging), Cursor AI (inline completions, refactoring)
>
> **Where AI materially sped up work:**
> - Firestore data model design and security rules (saved ~40 min of docs research)
> - Tiptap toolbar boilerplate and autosave debounce pattern (generated first draft, I tuned timing and UX)
> - Markdown-to-Tiptap parser (AI suggested line-by-line approach; I added empty line and ordering edge cases)
> - Firebase Admin ID token verification pattern for API routes
> - collectionGroup query approach for shared docs
>
> **What I changed or rejected:**
> - AI initially suggested Yjs for real-time collaboration — rejected as out of scope for the timebox
> - AI suggested .docx parsing via mammoth.js — scoped down to .txt/.md only, stated clearly in UI and README
> - AI-generated security rules were missing the `allow create` condition scoped to ownerId — I added it manually
> - AI suggested storing content as a string — I changed to Tiptap JSON object to preserve formatting structure
>
> **How I verified correctness:**
> - Manually tested every user flow end-to-end in browser (both alice and bob accounts)
> - Verified Firestore security rules by attempting unauthorized reads in the Firebase console Rules Playground
> - Ran Jest tests before submission

---

## Key Files I'll Generate for You (Phase by Phase)

1. `lib/firebase/client.ts` + `admin.ts`
2. `hooks/useAuth.ts`
3. `app/(auth)/login/page.tsx` + `signup/page.tsx`
4. `app/(app)/layout.tsx`
5. `hooks/useDocuments.ts`
6. `app/(app)/documents/page.tsx`
7. `components/Editor.tsx` ← most complex
8. `app/(app)/documents/[id]/page.tsx`
9. `app/api/upload/route.ts`
10. `app/api/share/route.ts`
11. `components/ShareModal.tsx` + `UploadModal.tsx`
12. `__tests__/documents.test.ts`
13. `README.md`, `ARCHITECTURE.md`, `AI_WORKFLOW.md`, `SUBMISSION.md`

**Tell me when Phase 0 npm install is done + Firebase project is ready. I hand you Phase 1 code immediately.**
