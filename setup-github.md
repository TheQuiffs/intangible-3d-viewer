# Quick GitHub + Netlify Setup Guide

## Step 1: Create GitHub Repository (2 minutes)

1. Go to: https://github.com/new
2. Repository name: `intangible-3d-viewer`
3. Make it **Public** or **Private** (your choice)
4. **DO NOT** check "Add a README file" (we already have one)
5. Click **"Create repository"**

## Step 2: Copy These Commands

After creating the repo, GitHub will show you commands. **Ignore those** and use these instead:

```bash
cd "C:\Users\coote\Documents\CURSOR\intangible\web3d"
git remote add origin https://github.com/YOUR-USERNAME/intangible-3d-viewer.git
git branch -M main
git push -u origin main
```

**Replace `YOUR-USERNAME`** with your actual GitHub username.

## Step 3: Connect Netlify (3 minutes)

1. Go to: https://app.netlify.com/
2. Click **"Add new site"** → **"Import an existing project"**
3. Click **"Deploy with GitHub"**
4. Authorize Netlify to access your GitHub (if first time)
5. Select `intangible-3d-viewer` from the list
6. Build settings:
   - Build command: *leave empty*
   - Publish directory: `.` (just a dot)
7. Click **"Deploy site"**

## Done! 🎉

From now on, to update your 3D viewer:

```bash
cd "C:\Users\coote\Documents\CURSOR\intangible\web3d"
# Make your changes to index.html or other files
git add .
git commit -m "Updated exposure/background/etc"
git push
```

Netlify will auto-deploy in ~30 seconds!

## Delete Old Sites

Go to Netlify → Sites → Click each old site → Site settings → Delete site

This will clean up those 25 empty projects.
