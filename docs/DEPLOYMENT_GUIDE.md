# 🚀 Deployment Guide - GitHub Pages

Complete step-by-step instructions for deploying Maple Hospital Academy to GitHub Pages.

---

## 📋 Prerequisites

- Repository with GitHub Pages enabled
- Node.js 16.x or higher installed locally
- `gh-pages` npm package
- Firebase credentials configured in environment variables
- Git command line tools

---

## ⚙️ Step 1: Configure package.json

Add the following fields to your `package.json`:

```json
{
  "name": "maple-hospital-academy",
  "version": "1.0.0",
  "private": false,
  "homepage": "https://andrewholden822-cyber.github.io/Maple-Hospital-Academy",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "firebase": "^10.0.0",
    "lucide-react": "^0.263.1",
    "tailwindcss": "^3.3.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build",
    "eject": "react-scripts eject"
  },
  "devDependencies": {
    "gh-pages": "^5.0.0",
    "react-scripts": "5.0.1"
  }
}
```

**Key points:**
- Replace `https://andrewholden822-cyber.github.io/Maple-Hospital-Academy` with your actual GitHub Pages URL
- `predeploy` script runs automatically before `deploy`
- The `build` directory is deployed to GitHub Pages

---

## 🔑 Step 2: Environment Variables Setup

### For Local Development

Create a `.env.local` file in the project root:

```env
REACT_APP_FIREBASE_API_KEY=your_firebase_api_key
REACT_APP_FIREBASE_AUTH_DOMAIN=your_firebase_auth_domain
REACT_APP_FIREBASE_PROJECT_ID=your_firebase_project_id
REACT_APP_FIREBASE_STORAGE_BUCKET=your_firebase_storage_bucket
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=your_firebase_messaging_sender_id
REACT_APP_FIREBASE_APP_ID=your_firebase_app_id
```

### For GitHub Pages Deployment

Create a `.env.production` file with the same Firebase credentials:

```env
REACT_APP_FIREBASE_API_KEY=your_firebase_api_key
REACT_APP_FIREBASE_AUTH_DOMAIN=your_firebase_auth_domain
REACT_APP_FIREBASE_PROJECT_ID=your_firebase_project_id
REACT_APP_FIREBASE_STORAGE_BUCKET=your_firebase_storage_bucket
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=your_firebase_messaging_sender_id
REACT_APP_FIREBASE_APP_ID=your_firebase_app_id
```

⚠️ **Important:** Add `.env.local` and `.env` files to `.gitignore` to prevent exposing credentials:

```gitignore
# Environment variables
.env
.env.local
.env.production.local
.env.development.local
.env.test.local

# Build output
/build
/dist
node_modules/

# IDE
.vscode/
.idea/
*.swp
```

---

## 🔧 Step 3: Configure GitHub Repository

### 3.1 Enable GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under "Build and deployment", select:
   - **Source:** `Deploy from a branch`
   - **Branch:** `gh-pages` (this will be created automatically)
   - **Folder:** `/ (root)`
4. Click **Save**

### 3.2 Add GitHub Secrets (Optional - for Automated Deployment)

If using GitHub Actions for CI/CD:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add Firebase credentials as secrets:
   - `FIREBASE_API_KEY`
   - `FIREBASE_AUTH_DOMAIN`
   - `FIREBASE_PROJECT_ID`
   - etc.

---

## 🔨 Step 4: Build and Deploy Locally

### 4.1 Install Dependencies

```bash
npm install
```

### 4.2 Build for Production

```bash
npm run build
```

This creates an optimized build in the `/build` directory.

### 4.3 Deploy to GitHub Pages

```bash
npm run deploy
```

The `gh-pages` package will:
1. Create or update the `gh-pages` branch
2. Push build files to that branch
3. GitHub automatically deploys from that branch

---

## ✅ Step 5: Verify Deployment

1. Wait 2-3 minutes for GitHub Pages to build and deploy
2. Visit: `https://andrewholden822-cyber.github.io/Maple-Hospital-Academy`
3. Confirm the course loads and Firebase authentication works
4. Test user onboarding and progress saving

**Check deployment status:**
- Go to **Settings** → **Pages**
- Look for the deployment status badge
- Click on deployment details for logs

---

## 🔄 Step 6: Automated Deployment with GitHub Actions

For automatic deployment on every push to `main`:

### Create `.github/workflows/deploy.yml`

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build application
      run: npm run build
      env:
        REACT_APP_FIREBASE_API_KEY: ${{ secrets.FIREBASE_API_KEY }}
        REACT_APP_FIREBASE_AUTH_DOMAIN: ${{ secrets.FIREBASE_AUTH_DOMAIN }}
        REACT_APP_FIREBASE_PROJECT_ID: ${{ secrets.FIREBASE_PROJECT_ID }}
        REACT_APP_FIREBASE_STORAGE_BUCKET: ${{ secrets.FIREBASE_STORAGE_BUCKET }}
        REACT_APP_FIREBASE_MESSAGING_SENDER_ID: ${{ secrets.FIREBASE_MESSAGING_SENDER_ID }}
        REACT_APP_FIREBASE_APP_ID: ${{ secrets.FIREBASE_APP_ID }}

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./build
```

**Setup:**
1. Create the `.github/workflows/` directory in your repository
2. Add the `deploy.yml` file above
3. Add Firebase secrets to GitHub (Settings → Secrets and variables → Actions)
4. Push to `main` branch
5. GitHub Actions will automatically build and deploy

---

## 🌍 Firebase Configuration for GitHub Pages

### Allow GitHub Pages Origin

In Firebase Console:

1. Go to **Authentication** → **Settings** → **Authorized domains**
2. Add your GitHub Pages domain:
   - `andrewholden822-cyber.github.io`
3. Save changes

### CORS Configuration (if needed)

If API calls are blocked due to CORS, update your Firebase security rules to allow the GitHub Pages origin.

**Firestore Security Rules Example:**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /artifacts/{appId}/users/{uid}/courseProgress/{document=**} {
      allow read, write: if request.auth.uid == uid;
    }
  }
}
```

---

## 🧪 Testing the Deployment

### 1. Test Anonymous Authentication
- Load the deployed site
- Verify you can create an onboarding profile
- Confirm Firestore saves your progress

### 2. Test Quiz Functionality
- Take a unit quiz
- Verify answers save to database
- Check quiz results display correctly

### 3. Test Level Tracking
- Update your in-game level
- Refresh the page
- Confirm level persists from Firebase

### 4. Test Certificate Generation
- Complete the final exam with ≥8/10
- Verify certificate displays
- Check certificate includes username and date

---

## 🔒 Security Considerations

### Protect Firebase Credentials

✅ **DO:**
- Use GitHub Secrets for sensitive values
- Store `.env.local` in `.gitignore`
- Use Firebase security rules to restrict data access
- Enable Authentication requirements in Firebase

❌ **DON'T:**
- Commit `.env` files to Git
- Expose API keys in public code
- Use overly permissive Firestore rules
- Share GitHub Secrets with untrusted collaborators

### Firebase Security Rules Best Practice

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Only allow authenticated users to read/write their own progress
    match /artifacts/{appId}/users/{uid}/courseProgress/{document=**} {
      allow read, write: if request.auth.uid == uid;
    }
    
    // Prevent access to other users' data
    match /artifacts/{appId}/users/{otherUid=**} {
      allow read, write: if false;
    }
  }
}
```

---

## 📦 Update & Redeploy

To update your deployed course:

1. **Make changes locally:**
   ```bash
   git add .
   git commit -m "Update course content"
   git push origin main
   ```

2. **If using GitHub Actions:** Automatic deployment triggers
   
3. **If deploying manually:**
   ```bash
   npm run deploy
   ```

4. **Verify changes:** Wait 2-3 minutes, then refresh GitHub Pages URL

---

## 🐛 Troubleshooting

### Issue: "Cannot GET /Maple-Hospital-Academy/"

**Solution:**
- Ensure `homepage` in `package.json` is correct
- Check GitHub Pages is enabled in Settings → Pages
- Wait 5 minutes for deployment to complete

### Issue: Firebase authentication fails on deployed site

**Solution:**
1. Add `andrewholden822-cyber.github.io` to Firebase authorized domains
2. Verify `REACT_APP_FIREBASE_*` environment variables are set
3. Check Firebase security rules allow anonymous authentication

### Issue: Progress not saving to Firestore

**Solution:**
1. Verify Firebase credentials in `.env.production`
2. Check Firestore security rules permit write access
3. Review browser console for authentication errors
4. Confirm Firebase project has Firestore database enabled

### Issue: GitHub Pages shows old version

**Solution:**
```bash
# Hard refresh browser
Ctrl+Shift+R (Windows/Linux)
Cmd+Shift+R (Mac)

# Or clear browser cache
# Settings → Privacy → Clear browsing data → Cached images/files
```

---

## 📊 Performance Optimization

### Build Optimization

```bash
npm run build
# Check build size
ls -lh build/
```

### Enable Gzip Compression

GitHub Pages automatically serves gzipped content—no additional configuration needed.

### Optimize Bundle Size

Add to `package.json`:
```json
{
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

---

## 🎉 Next Steps

1. ✅ Deploy to GitHub Pages
2. ✅ Test all features work correctly
3. ✅ Configure custom domain (optional)
4. ✅ Share your course link with students
5. ✅ Monitor student progress in Firebase

---

## 📚 Additional Resources

- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [gh-pages npm Package](https://www.npmjs.com/package/gh-pages)
- [Create React App Deployment Guide](https://create-react-app.dev/docs/deployment/)
- [Firebase Hosting Alternatives](https://firebase.google.com/docs/hosting)

---

**Last Updated:** June 2026 | Maple Hospital Academy Deployment Guide
