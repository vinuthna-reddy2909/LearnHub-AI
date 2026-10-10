# 🚀 LearnHub AI — Deployment Guide

---

## 🗄️ STEP 1 — Set Up MongoDB Atlas (Free Cloud Database)

> **Why Atlas?** Render does not include a built-in database. MongoDB Atlas is the standard free-tier cloud database for MERN apps.

1. Go to [https://www.mongodb.com/atlas](https://www.mongodb.com/atlas) → Sign up free
2. Create a new **Project** → Create a **Free Cluster (M0)**
3. Under **Security → Database Access**: Create a DB user (save username + password)
4. Under **Security → Network Access**: Click **"Allow Access from Anywhere"** (`0.0.0.0/0`) for Render compatibility
5. Click **Connect → Drivers** → Copy the connection string. It looks like:
   ```
   mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
6. Add your database name: change `/?retryWrites` → `/learnhubai?retryWrites`

---

## ⚙️ STEP 2 — Deploy Backend on Render (Node.js — No Docker needed!)

> Render **natively supports Node.js**. You do NOT need Docker.

1. Push your code to **GitHub** (make sure `.env` is in `.gitignore` ✅)
2. Go to [https://render.com](https://render.com) → New → **Web Service**
3. Connect your GitHub repo
4. Configure:
   - **Name**: `learnhub-ai-backend`
   - **Root Directory**: `backend`
   - **Runtime**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `node server.js`
   - **Plan**: Free
5. Under **Environment Variables**, add:

   | Key | Value |
   |-----|-------|
   | `NODE_ENV` | `production` |
   | `MONGO_URI` | *(your Atlas connection string from Step 1)* |
   | `JWT_SECRET` | *(any long random string, e.g. 64 char hex)* |
   | `GEMINI_API_KEY` | *(from https://aistudio.google.com/app/apikey)* |
   | `FRONTEND_URL` | *(your Vercel URL — add after step 3, then redeploy)* |

6. Click **Deploy** → Wait ~3 minutes
7. Copy your Render URL: `https://learnhub-ai-backend.onrender.com`

> ⚠️ **Free Render services sleep after 15 min inactivity.** The first request after sleep takes ~30s. Consider [UptimeRobot](https://uptimerobot.com) (free) to ping your `/` health check every 5 min.

---

## 🌐 STEP 3 — Deploy Frontend on Vercel

1. Go to [https://vercel.com](https://vercel.com) → New Project → Import GitHub repo
2. Configure:
   - **Root Directory**: `frontend`
   - **Framework Preset**: `Vite`
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
3. Under **Environment Variables**, add:

   | Key | Value |
   |-----|-------|
   | `VITE_API_URL` | `https://learnhub-ai-backend.onrender.com` |

4. Click **Deploy** → Wait ~1 minute
5. Copy your Vercel URL: `https://learnhub-ai.vercel.app`

---

## 🔄 STEP 4 — Link Frontend ↔ Backend (CORS Fix)

1. Go back to **Render** → Your backend service → **Environment**
2. Update `FRONTEND_URL` = `https://learnhub-ai.vercel.app` *(your actual Vercel URL)*
3. Click **Manual Deploy** → Deploy latest commit

---

## ✅ Verify Everything Works

- Visit your Vercel URL — homepage should load
- Visit `https://your-backend.onrender.com/` — should return `{"status":"ok","message":"LearnHub AI API is running ✅"}`
- Try registering an account → should store in MongoDB Atlas

---

## 📋 Files Created for Deployment

| File | Purpose |
|------|---------|
| `frontend/vercel.json` | Fixes React Router 404 on page refresh |
| `frontend/.env.production` | Template for Vercel env vars |
| `backend/.env.example` | Template for Render env vars |
| `render.yaml` | Render infrastructure-as-code config |
| `backend/.gitignore` | Keeps `.env` secrets out of GitHub |
| `frontend/.gitignore` | Keeps `node_modules` and secrets out |

---

## 🆘 Troubleshooting

| Problem | Fix |
|---------|-----|
| Login fails on production | Check `VITE_API_URL` in Vercel env vars points to Render URL |
| CORS error in browser console | Add your Vercel URL to `FRONTEND_URL` in Render env vars |
| Backend crashes on Render | Check `MONGO_URI` is correct and Atlas allows `0.0.0.0/0` |
| Page 404 on refresh | `vercel.json` rewrite rule handles this ✅ |
| Gemini AI returns mock data | Check `GEMINI_API_KEY` is set correctly in Render |
