# firebase-widgets

A toolkit of embeddable, real-time widgets for static sites — powered by Firebase Hosting and Firestore. Fork this repo, point it at your own Firebase project, and drop the widgets into any static site via `<iframe>`.

No build step. No framework. Just HTML, vanilla JS, and Firebase.

Everything here runs on the **Firebase free tier (Spark plan)** — no credit card required. The only services used are Firebase Hosting and Firestore, both of which have generous free quotas well beyond what a personal site will consume. Deployment is handled by GitHub Actions, which is also free for public repos.

---

## Widgets

| Widget | Path | Description |
|---|---|---|
| Visitor Counter | `/counter/` | Counts unique session visits, displays a 6-digit retro counter |
| Chat | `/chat/` | Real-time chat, stores last 150 messages |
| News | `/news/` | Top 2 Hacker News stories, live from the HN API |
| Guestbook | `/guestbook/` | Persistent guestbook, one entry per browser via localStorage |

---

## Getting started

### 1. Fork this repo

Click **Fork** on GitHub. Clone your fork locally.

### 2. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a new project
2. Add a **Web app** to the project (the `</>` button in Project Settings)
3. Copy the `firebaseConfig` object — you'll need it in the next step
4. Enable **Firestore Database** (Build → Firestore Database → Create database → Production mode)

### 3. Configure your project

**`.firebaserc`** — replace the project ID:
```json
{
  "projects": {
    "default": "your-firebase-project-id"
  }
}
```

**`public/*/index.html`** — replace the `firebaseConfig` block in each widget file with your own:
```js
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

> **Note:** Firebase web API keys are safe to commit — they are public identifiers, not secrets. Security is enforced by Firestore rules. See [Firebase docs](https://firebase.google.com/docs/projects/api-keys) for details.
>
> **Important:** Restrict your API key in [Google Cloud Console](https://console.cloud.google.com) → APIs & Services → Credentials. Allow only **Firebase Installations API** and **Cloud Firestore API**. Make sure the **Generative Language API (Gemini)** is not included — if left unrestricted, an exposed key could be used to run Gemini requests billed to your account.

### 4. Deploy via GitHub Actions (no CLI required)

1. Go to Firebase Console → Project Settings → Service accounts → **Generate new private key** and download the JSON
2. In your GitHub repo → Settings → Secrets and variables → Actions → add a secret:
   - Name: `FIREBASE_SERVICE_ACCOUNT`
   - Value: paste the full contents of the downloaded JSON
3. In Google Cloud Console → IAM & Admin → IAM, find the `firebase-adminsdk` service account and add these roles:
   - **Service Usage Consumer**
   - **Firebase Rules Admin**
4. Push to `main` — GitHub Actions will deploy hosting and Firestore rules automatically

Your widgets will be live at `https://your-project-id.web.app/`.

---

## Themes

All widgets support four themes via the `?theme=` URL parameter:

| Theme | Description |
|---|---|
| `cyber` | Dark, blue-tinted, VT323 monospace — default |
| `paper` | Light parchment, Georgia serif, warm brown accent |
| `dusk` | Deep purple, lavender text, violet accent |
| `moss` | Dark green, sage text, green accent |

Append `?theme=paper` (or `dusk`, `moss`) to any widget URL to switch themes. No param defaults to `cyber`.

---

## Embedding

Each widget is a self-contained page designed to be embedded via `<iframe>`:

```html
<!-- Visitor counter -->
<iframe src="https://your-project-id.web.app/counter/?theme=cyber" width="60" height="20" frameborder="0"></iframe>

<!-- Chat -->
<iframe src="https://your-project-id.web.app/chat/?theme=cyber" width="400" height="300" frameborder="0"></iframe>

<!-- News -->
<iframe src="https://your-project-id.web.app/news/?theme=cyber" width="400" height="80" frameborder="0"></iframe>

<!-- Guestbook -->
<iframe src="https://your-project-id.web.app/guestbook/?theme=cyber" width="400" height="300" frameborder="0"></iframe>
```

All widgets have a transparent or semi-transparent background and use the [VT323](https://fonts.google.com/specimen/VT323) monospace font. They're designed to sit naturally over dark or textured site backgrounds.

---

## Firestore rules

Rules are in `firestore.rules` and deployed automatically. Each widget only has access to its own collection. Writes are validated for field presence and length limits.

---

## Adding your own widgets

1. Create a new folder under `public/` (e.g. `public/my-widget/`)
2. Add an `index.html` using the same `firebaseConfig` pattern
3. Add a `frame-ancestors *` header for the new path in `firebase.json`
4. Add any required Firestore rules to `firestore.rules`
5. Push to deploy
