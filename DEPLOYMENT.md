# LearnHub AI deployment

The backend uses Java 21, Spring Boot, and PostgreSQL. Render can create the
backend and a managed PostgreSQL database from the root `render.yaml`.
The frontend is a Vite app deployed separately to Vercel.

## Push the project to GitHub

The repository remote is already configured as
`https://github.com/vinuthna-reddy2909/LearnHub-AI.git`.

Before pushing, review `git status` and make sure no `.env` files, passwords,
API keys, or local build output are staged. Push the reviewed changes to the
branch you intend to deploy.

## Deploy the backend and database on Render

1. In Render, create a new Blueprint and connect the GitHub repository.
2. Render reads the root `render.yaml`, creates a PostgreSQL database and
   deploys the Java backend. The backend is built from `backend-java/Dockerfile`.
3. Provide `GEMINI_API_KEY` when prompted (or set it in the backend service's
   Environment settings later). Render generates `JWT_SECRET`.
4. After deployment, copy the backend URL, for example
   `https://learnhub-ai-backend.onrender.com`.
5. Set the backend's `FRONTEND_URL` to the Vercel frontend origin after
   deploying the frontend, then redeploy the backend.

`backend-java/src/main/resources/schema.sql` creates the PostgreSQL tables on
startup if they do not already exist. Render's `DB_HOST`, `DB_PORT`, `DB_NAME`,
`DB_USER`, and `DB_PASSWORD` values are mapped to the managed database by
`render.yaml`; do not add those credentials to GitHub.

## Deploy the frontend on Vercel

1. Import the GitHub repository as a Vercel project.
2. Set the project **Root Directory** to `frontend` and use the Vite framework
   preset. The build command is `npm run build`; the output directory is `dist`.
3. Add the environment variable `VITE_API_URL` with the Render backend URL,
   without a trailing slash, such as
   `https://learnhub-ai-backend.onrender.com`.
4. Deploy. Vercel uses `frontend/vercel.json` to route client-side app URLs
   back to `index.html`.
5. Copy the Vercel origin into Render's `FRONTEND_URL` and redeploy the backend
   so CORS allows requests from the deployed frontend.

## Local development

Install Java 21, Maven, Node.js, and PostgreSQL. Create a local database named
`learnhubai`, then set the variables listed in `backend-java/.env.example` in
your shell environment. Spring Boot does not automatically load `.env` files.
For PowerShell, for example:

```sh
$env:DB_HOST = "localhost"
$env:DB_PORT = "5432"
$env:DB_NAME = "learnhubai"
$env:DB_USER = "postgres"
$env:DB_PASSWORD = "your-local-postgres-password"
$env:JWT_SECRET = "replace-this-with-at-least-32-random-characters"
$env:FRONTEND_URL = "http://localhost:5173"
Set-Location backend-java
mvn spring-boot:run
```

The backend listens on `http://localhost:5000`. The frontend's
`frontend/.env.example` points to that local API URL; copy it to
`frontend/.env.local` if needed, then start the Vite development server:

```sh
cd frontend
npm install
npm run dev
```

The PostgreSQL schema is initialized automatically on backend startup. It
creates the tables but does not populate sample course or quiz data.

## Troubleshooting

| Problem | Check |
| --- | --- |
| Backend fails to start | Check the Render database environment variables and database status. |
| Login fails in production | Confirm `VITE_API_URL` points to the deployed Render service. |
| Browser reports a CORS error | Set Render `FRONTEND_URL` to the exact Vercel origin (no path) and redeploy. |
| Frontend route returns 404 on refresh | Confirm `frontend/vercel.json` is included in the Vercel deployment. |
| AI recommendations are unavailable | Set a valid `GEMINI_API_KEY` in the Render backend environment. |
