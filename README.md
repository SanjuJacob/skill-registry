# Skill Registry

A shared, real-time registry for the robot capability model. It organizes skills in a five-level hierarchy, from hardware resources up to complete task plans. Approved team members can browse, add, edit and export skills from the browser, and every change appears for everyone straight away.

**Live site:** https://sanjujacob.github.io/skill-registry/

---

## Contents

- [Features](#features)
- [How it works](#how-it-works)
- [Repository structure](#repository-structure)
- [Access model](#access-model)
- [Using the registry](#using-the-registry)
- [Administration](#administration)
- [Deploying your own instance](#deploying-your-own-instance)
- [Updating the site](#updating-the-site)
- [Running locally](#running-locally)
- [Backups](#backups)
- [Security](#security)
- [Costs and limits](#costs-and-limits)
- [Troubleshooting](#troubleshooting)

---

## Features

- **Five-level hierarchy:** L0 Resources, L1 Elementary, L2 Basic Skills, L3 Composite and L4 Task Plans, with sections, filters and search.
- **Real-time collaboration:** changes sync to every open browser within about a second, with a notification showing who changed what.
- **Private by default:** only approved accounts can view or edit. Everyone else sees a lock screen.
- **Approval workflow:** users sign in with Google and request access, and the admin approves or denies them from inside the app.
- **Edit history:** each skill records who last edited it and when.
- **Import and export:** load or download the full registry as YAML or JSON.
- **Folder auto-save (optional):** in Chrome, Edge or Brave, link a local folder to write YAML and JSON copies on every change.
- **Light and dark themes.**

## How it works

The app is a single static HTML page with no build step and no server to maintain.

```
 Browser (index.html)
   │
   ├── Hosting ............ GitHub Pages            serves the page
   ├── Sign-in ............ Firebase Authentication Google accounts
   └── Data ............... Cloud Firestore         skills, editors, access requests
                              └── security rules    enforce who can read and write
```

| Component | Role |
|---|---|
| **GitHub Pages** | Hosts `index.html` as a public website. |
| **Firebase Authentication** | Handles Google sign-in. The app never sees or stores passwords. |
| **Cloud Firestore** | Stores the registry and pushes live updates to connected browsers. |
| **Security rules** | Enforced on Google's servers, so access control can't be bypassed by changing the page. |

### Data layout

| Collection | Contents | Who can access |
|---|---|---|
| `skills` | One document per skill | Read and write: admin and approved editors |
| `editors` | One document per approved user, keyed by email | Admin manages; users can read their own entry |
| `requests` | One document per pending access request, keyed by email | Users create their own; admin reviews |

## Repository structure

```
skill-registry/
├── index.html   The complete application (UI, logic, Firebase connection)
└── README.md    This file
```

> The Firestore security rules aren't stored in this repository. They're maintained by the admin and published directly in the **Firebase console → Firestore Database → Rules**.

## Access model

| User | Can view | Can edit | What they see |
|---|:-:|:-:|---|
| Not signed in | ✗ | ✗ | Lock screen with **Sign in with Google** |
| Signed in, not approved | ✗ | ✗ | **Request access** button |
| Request pending | ✗ | ✗ | "Access requested". They get in automatically when approved. |
| Approved editor | ✓ | ✓ | Full registry |
| Admin | ✓ | ✓ | Full registry plus **👥 Manage access** |

Approval and removal take effect immediately, with no page reload.

## Using the registry

1. Open the [live site](https://sanjujacob.github.io/skill-registry/).
2. Click **Sign in with Google** and choose your account.
3. Click **Request access**. You'll be let in automatically once the admin approves.
4. Once approved, you can:
   - browse by level using the sidebar, the summary cards or the filter bar
   - search by skill ID or description
   - click a skill card to expand its details
   - add, edit or delete skills (changes save instantly for everyone)
   - **Download YAML / JSON** for an offline snapshot

> **Import** and **Reset to defaults** replace the entire shared registry for all users. Download a backup first.

## Administration

The admin account is defined in two places, which must match:

- `ADMIN_EMAIL` in `index.html`
- `isAdmin()` in the Firestore security rules (Firebase console)

### Managing access

Sign in as the admin and open **👥 Manage access** in the sidebar.

| Task | How |
|---|---|
| Approve a request | **✓ Approve** next to the request |
| Deny a request | **✗ Deny** |
| Grant access without a request | Type the email under **Add an editor directly** |
| Revoke access | **Remove** next to the editor |

New requests trigger a notification while the admin has the site open.

### Firebase console actions

[Authentication → Users](https://console.firebase.google.com/project/skill-registry-c64d3/authentication/users) lists every account that has signed in.

| Action | Effect |
|---|---|
| **Disable account** | Blocks the user from signing in. Recommended for permanently removing someone; combine with **Remove** in Manage access. |
| **Delete account** | Only removes the sign-in record. The user can sign in again, and any editor approval (stored by email) stays. It does not affect their Google account. |
| **Reset password** | Not applicable. The app uses Google sign-in only. |

## Deploying your own instance

Follow these steps to run a separate copy of the registry under your own accounts. Everything runs on free plans.

### 1. Create a Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com) and create a project. Google Analytics is not needed.
2. Add a **Web app** (`</>` icon). Leave Firebase Hosting unticked.
3. Copy the `firebaseConfig` values shown.

### 2. Create the database

1. Open **Firestore Database** and click **Create database**.
2. Choose a location near your users (this can't be changed later) and **production mode**.

### 3. Publish the security rules

1. Open **Firestore Database → Rules**.
2. Paste in the security rules, with your admin email in `isAdmin()`. Ask the current admin for a copy; the rules aren't part of this repository.
3. Click **Publish**.

### 4. Enable Google sign-in

1. Open **Authentication → Sign-in method** and enable **Google**.
2. Under **Authentication → Settings → Authorized domains**, add `<your-github-username>.github.io`.

### 5. Configure the app

In `index.html`, update:

- the `firebaseConfig` object with your project's values
- `ADMIN_EMAIL` with your admin's Google account

> The Firebase web config is designed to be public. Access is controlled by the security rules, not by keeping the config secret.

### 6. Publish on GitHub Pages

1. Create a **public** GitHub repository and upload `index.html` and `README.md`.
2. Go to **Settings → Pages** and set **Source** to **Deploy from a branch**, with branch **`main`** and folder **`/ (root)`**.
3. After about a minute the site is live at `https://<username>.github.io/<repository>/`.

### 7. Load initial data

Sign in as the admin, then either **Import** a YAML/JSON export or use **Reset to defaults**.

## Updating the site

| Change | What to do |
|---|---|
| Page layout or behaviour (`index.html`) | Upload the new file to the repository. GitHub Pages redeploys within a minute or two. |
| Access rules | Publish in **Firebase console → Firestore Database → Rules**. Keep a copy of the current rules outside this repository. |
| Registry content | Edit through the site. No deployment needed. |

Browsers may cache the page for up to 10 minutes. Press **Ctrl + Shift + R** to force a refresh.

When changing both rules and page in a way that depends on each other, **publish the rules first**.

## Running locally

Google sign-in needs the page served over `http://localhost`. Opening the file directly (`file://`) won't work.

```bash
python -m http.server 8080
```

Then open <http://localhost:8080>. `localhost` is authorized for sign-in by default, and the local copy uses the same live database.

## Backups

The registry lives in Firestore. Bulk operations (**Import**, **Reset to defaults**) and mistaken deletions affect all users and can't be undone from the app.

- Download a **JSON** export regularly, and always before importing or resetting.
- To restore, sign in as an editor and **Import** the JSON file.

## Security

- **No passwords handled:** authentication is delegated to Google. The app only receives the user's verified email and name.
- **Server-side enforcement:** all read and write permissions are enforced by Firestore security rules, so modifying the page in a browser grants no extra access.
- **Tamper-resistant audit fields:** each write must carry the editor's own email and the server timestamp.
- **Input escaping:** user-entered text is rendered as plain text, preventing script injection between users.
- **Encrypted transport:** all traffic is over HTTPS.
- **Data already viewed can be kept:** approved users can download or save what they have seen. Revoking access stops future access only.

## Costs and limits

The project runs on the free tiers of Firebase (Spark plan, no billing account) and GitHub Pages. It can't incur charges: if a daily quota is reached, the service pauses until the quota resets.

| Resource | Free allowance |
|---|---|
| Firestore reads | 50,000 / day |
| Firestore writes | 20,000 / day |
| Firestore storage | 1 GiB |
| Google sign-ins | No practical limit |
| GitHub Pages bandwidth | ~100 GB / month |

This is ample for team use. Current figures are on the [Firebase pricing page](https://firebase.google.com/pricing). Usage is visible under **Firestore Database → Usage**.

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Old version of the page appears | Browser cache. Press **Ctrl + Shift + R**. |
| "Cannot reach the shared registry" | Network or an extension blocking Google/Firebase domains. Check the connection or try another browser. |
| Sign-in popup closes or fails | The site's domain is missing from **Authentication → Authorized domains**, or the browser is blocking popups. |
| "Not allowed — your account cannot edit" | The account hasn't been approved, or was removed. Request access again. |
| **Manage access** not visible | You're not signed in as the admin account, or the page is cached. |
| **Request access** fails, or approved users can't see the registry | The security rules in the Firebase console are missing or outdated. Republish the current rules. |
| Folder auto-save option missing | Only available in Chromium-based browsers (Chrome, Edge, Brave) and only for approved users. |
