# Submission

## Included

- Email/password authentication with persistent sessions
- Document create, rename, edit, save, and reopen
- Rich text toolbar: bold, italic, underline, H1/H2/H3, bullet list, numbered list
- `.txt` and `.md` upload into editable documents
- Share by email with shared docs appearing in "Shared with Me"
- Firestore persistence with security rules file included
- Jest unit test for markdown parsing
- README, ARCHITECTURE, and AI workflow documentation

## Missing / Manual Steps Required

- Firebase Admin service account values must be added to `.env.local` for upload/share API routes
- Firestore rules must be published in Firebase Console from `firestore.rules`
- Firestore composite index for `collectionGroup('shares')` may need to be created on first shared-doc query
- Vercel deployment and environment variable setup
- Add Vercel domain to Firebase Auth authorized domains

## Test Accounts

- `alice@ajaia.dev` / `Test1234!`
- `bob@ajaia.dev` / `Test1234!`

## What's Next

- Real-time collaboration with Yjs/CRDTs
- Richer markdown parsing
- Version history
- Role-based permissions (viewer/editor)
- Comments

## Verification Checklist

- [ ] Login as Alice and create a document
- [ ] Edit rich text and confirm autosave
- [ ] Upload a `.md` file and confirm heading formatting
- [ ] Share a document with Bob
- [ ] Login as Bob and confirm document appears under Shared with Me
- [ ] Run `npm run test`
