# HL7 Field Inspector — GitHub + Cloudflare Pages Setup

This repository contains the reviewed single-file HL7 Field Inspector.

## Repository layout

```text
hl7-field-inspector/
├── public/
│   └── index.html
└── README.md
```

`public/index.html` is the deployable application. It requires no server, database, package manager, or framework.

---

## 1. Create a private GitHub repository

Create a new GitHub repository named:

```text
hl7-field-inspector
```

Set **Visibility = Private**.

Do not initialize it with a README if you are going to upload this prepared folder directly.

### Easiest method — GitHub web upload

1. Open the new private repository.
2. Choose **Add file > Upload files**.
3. Upload the contents of this package so the repository ends up with:

```text
public/index.html
README.md
```

4. Commit to the `main` branch.

### Alternative — Git command line

From inside the `hl7-field-inspector-github` folder:

```bash
git init
git add .
git commit -m "Initial HL7 Field Inspector"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/hl7-field-inspector.git
git push -u origin main
```

---

## 2. Create the Cloudflare Pages project

In Cloudflare:

1. Go to **Workers & Pages**.
2. Select **Create application**.
3. Select **Pages**.
4. Select **Import an existing Git repository / Connect to Git**.
5. Connect GitHub.
6. When GitHub asks which repositories Cloudflare can access, choose **Only select repositories** and select `hl7-field-inspector`.
7. Select the private `hl7-field-inspector` repository.

Use these deployment settings:

```text
Project name:           hl7-field-inspector
Production branch:      main
Framework preset:       None
Build command:          exit 0
Build output directory: public
```

Then select **Save and Deploy**.

After deployment you should receive a URL similar to:

```text
https://hl7-field-inspector.pages.dev
```

At this stage, assume the deployment is public until Cloudflare Access is configured.

---

## 3. Protect the site with Cloudflare Access

The goal for now is:

```text
Internet
   ↓
Cloudflare Access
   ↓
Only your identity/email
   ↓
HL7 Field Inspector
```

### Enable Access for Pages

1. Open **Workers & Pages > hl7-field-inspector**.
2. Go to **Settings**.
3. Select **Enable access policy**.

Cloudflare initially creates an Access application for preview deployments.

### Protect the main pages.dev production URL

Cloudflare currently requires an additional step to protect the production `pages.dev` hostname:

1. From the Pages Access policy, select **Manage**.
2. Go to **Zero Trust > Access controls > Applications**.
3. Open the Access application created for this Pages project.
4. Select **Configure**.
5. Under **Public hostname**, remove the wildcard `*` from the Subdomain field so it protects the main production hostname.
6. Save. If Cloudflare reports a duplicate-name error, give the application a slightly different name such as `HL7 Inspector Production`.
7. Return to **Workers & Pages > hl7-field-inspector > Settings > General**.
8. Select **Enable access policy** again.

You should now have two Access applications/policies:

```text
hl7-field-inspector.pages.dev
*.hl7-field-inspector.pages.dev
```

The first protects production. The wildcard protects branch and preview deployments.

---

## 4. Allow only yourself

In **Zero Trust > Access controls > Policies**, create an Allow policy such as:

```text
Policy name: Only Me
Action:      Allow
Include:     Email
Value:       YOUR_EXACT_EMAIL_ADDRESS
```

Do not use **Everyone** and do not allow all users who can obtain an email one-time PIN.

Apply the `Only Me` policy to both Access applications:

```text
hl7-field-inspector.pages.dev
*.hl7-field-inspector.pages.dev
```

Use your normal Cloudflare/identity-provider login, or configure email one-time PIN if you prefer.

---

## 5. Verify that it is actually private

Do not test only in a browser where you are already logged in.

1. Open an Incognito/Private window.
2. Visit your `hl7-field-inspector.pages.dev` URL.
3. Confirm the HL7 Inspector does **not** appear immediately.
4. You should be sent to Cloudflare Access authentication.
5. Authenticate using the email/identity allowed by the `Only Me` policy.
6. Confirm the application loads.
7. If possible, try an unauthorized account and confirm access is denied.

Do the same with a preview URL once you create a second branch.

---

## 6. Recommended Git workflow

Once the initial deployment is working, create a `dev` branch:

```text
main  = stable / production
dev   = testing
```

Cloudflare Pages treats `main` as production and other branches as preview deployments.

Suggested workflow:

```text
make changes
    ↓
push to dev
    ↓
test Cloudflare preview privately
    ↓
merge dev into main
    ↓
production deploys automatically
```

---

## 7. Later: custom domain and tools portal

You do not need these for the first deployment.

Later the recommended structure is:

```text
tools.example.com          → tools portal
hl7.tools.example.com      → HL7 Field Inspector
barcode.tools.example.com  → another app
```

Each app should remain:

- its own GitHub repository
- its own Cloudflare Pages project
- independently public or Access-protected

If you add a custom domain to this Pages project, create a separate Cloudflare Access application for the custom hostname as well. Keep the original `pages.dev` hostname protected so it cannot be used to bypass the custom-domain login.

---

## Important privacy note

The HL7 Field Inspector is designed to process pasted messages locally in the browser. Keep it self-contained and avoid adding analytics, remote JavaScript libraries, tracking, external fonts, or telemetry if it may be used with clinical HL7 containing patient information.

Do not commit real patient HL7 examples to GitHub, even if the repository is private.
