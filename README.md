# Exam Toilet Monitor (Map View)

A secure, real-time exam toilet monitoring application built for invigilation management. It allows invigilators across examination clusters to coordinate student toilet visits in real-time, preventing corridor bottlenecks and maintaining exam integrity.

---

## 🔒 Security Architecture

This repository has been hardened for safe deployment to **GitHub** and **GitHub Pages**:

| Layer | Security Measure | Details |
|---|---|---|
| **Database Rules** | Strict Realtime Database Rules | Restricts reads and writes strictly to authenticated sessions (`auth != null`). Validates payload types, length limits (max 50 chars), timestamp validity (`<= now`), and rejects unexpected keys (`$other: false`). |
| **Authentication** | Firebase Anonymous Sign-In | Automatic client authentication so requests carry valid credentials without requiring invigilators to remember usernames. |
| **Admin Access** | SHA-256 Hashing + Rate Limiter | Admin passcode (`1756`) is stored exclusively as a SHA-256 hash. Enforces a **30-second lockout after 5 consecutive failed attempts** to stop automated brute-force attacks. |
| **DOM / XSS Protection** | HTML Entity Sanitization | All room names, block titles, and user inputs are escaped via `escapeHtml()` and safe DOM node creation to prevent Cross-Site Scripting (XSS). |
| **Headers & Referrer** | Strict Referrer Policy | `strict-origin-when-cross-origin` meta tag and strict response headers prevent leaking internal URLs. |

---

## 🚀 Deployment Guide (GitHub Pages)

### 1. Push Code to GitHub
1. Create a new repository on [GitHub](https://github.com/new) (e.g. `exam-toilet-monitor`).
2. Upload the core files:
   - `index.html`
   - `database.rules.json`
   - `firebase.json`
   - `.gitignore`
   - `README.md`
   *(Scratch `.txt` and backup files are automatically ignored by `.gitignore`)*

### 2. Enable GitHub Pages
1. In your GitHub repository, go to **Settings** > **Pages** (in the left sidebar).
2. Under **Build and deployment** > **Source**, select **Deploy from a branch**.
3. Under **Branch**, choose `main` (or `master`) and folder `/(root)`.
4. Click **Save**.
5. Your live link will be available at:
   ```
   https://<your-username>.github.io/<repo-name>/
   ```

---

## 🛡️ Critical: Lock Down Your Firebase API Key

Because client-side Firebase web apps include the configuration in JavaScript, follow these standard Google Cloud steps to guarantee **nobody can misuse your API key outside your domain**:

1. Go to the [Google Cloud Console Credentials Page](https://console.cloud.google.com/apis/credentials) and select your project (`toilet-movement`).
2. Click on the API key used by this app (e.g., `Browser key (auto created by Firebase)`).
3. Under **Application restrictions**:
   - Select **Websites**.
   - Under **Website restrictions**, click **ADD** and add:
     - `https://<your-username>.github.io/*` (your production GitHub Pages URL)
     - `http://localhost:*` (for local testing, if needed)
4. Under **API restrictions**:
   - Select **Restrict key**.
   - Check **only**:
     - *Firebase Realtime Database API*
     - *Identity Toolkit API* (Firebase Authentication)
5. Click **Save**.

> **Result**: Even if your repository is public, your API key will only function when requests originate from your authorized GitHub Pages URL, and cannot be used for any paid Google Cloud APIs.

---

## 🗄️ Deploying Firebase Security Rules

Make sure your Firebase Realtime Database has the rules applied:

### Option A: Via Firebase CLI
```bash
firebase deploy --only database
```

### Option B: Via Firebase Console
1. Open the [Firebase Console](https://console.firebase.google.com/) > Select **toilet-movement**.
2. In the left menu, select **Build** > **Realtime Database** > **Rules** tab.
3. Paste the contents of [`database.rules.json`](database.rules.json).
4. Click **Publish**.

### Enable Anonymous Authentication
1. In Firebase Console, go to **Build** > **Authentication** > **Sign-in method** tab.
2. Select **Anonymous**, toggle **Enable**, and click **Save**.

---

## ⚙️ Admin Configuration

- **Default Admin Passcode**: `1756`
- **To change the admin passcode**:
  1. Generate a SHA-256 hash of your desired passcode (e.g. via browser console `crypto.subtle.digest` or any online SHA-256 tool).
  2. Update `ADMIN_PASSCODE_HASH` in `index.html`:
     ```javascript
     const ADMIN_PASSCODE_HASH = "your_new_sha256_hex_hash_here";
     ```
