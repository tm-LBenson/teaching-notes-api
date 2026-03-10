# North Star API

North Star API is a lightweight REST API built for teaching modern API fundamentals with a professional feel. It gives students a realistic platform for learning authentication, protected routes, CRUD operations, JSON request and response handling, and API testing with Postman or browser-based `fetch()` calls.

This project is intentionally small enough for instructors to manage, but polished enough to feel like a real developer platform instead of a classroom mock API.

## What this project includes

- JWT authentication
- User registration and login
- Protected routes with bearer tokens
- Full CRUD for a `notes` resource
- Per-user data isolation
- JSON-only responses
- CORS enabled for browser clients
- Basic rate limiting
- Firebase Firestore for storage
- Render-ready deployment setup

Each student account only sees its own notes. That makes it useful for multi-user classroom exercises without students reading or overwriting one another's data.

## Tech stack

- Node.js
- Express
- Firebase Firestore
- Firebase Admin SDK
- JSON Web Tokens
- bcrypt
- Render.com

## Project structure

```text
north-star-api/
├─ src/
│  ├─ app.js
│  ├─ server.js
│  ├─ config/
│  │  └─ firebase.js
│  ├─ middleware/
│  │  ├─ auth.js
│  │  └─ error.js
│  ├─ routes/
│  │  ├─ auth.routes.js
│  │  └─ notes.routes.js
│  └─ utils/
│     └─ jwt.js
├─ .env.example
├─ .gitignore
├─ package.json
├─ render.yaml
├─ README.md
└─ README.student.md
```

## API features

### Authentication endpoints

- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`

### Notes endpoints

- `GET /notes`
- `POST /notes`
- `GET /notes/:id`
- `PUT /notes/:id`
- `DELETE /notes/:id`

### Behavior

- All responses are JSON
- All `/notes` routes require a valid bearer token
- Passwords are hashed with bcrypt
- JWT secret is stored in environment variables
- Firebase Admin credentials are stored in environment variables
- Requests return clear status codes and simple error messages

## Data model

### `users` collection

```json
{
  "email": "student@example.com",
  "passwordHash": "$2b$10$...",
  "createdAt": "2026-03-10T12:00:00.000Z"
}
```

### `notes` collection

```json
{
  "userId": "firebase-user-doc-id",
  "title": "My first note",
  "body": "Hello from the API",
  "createdAt": "2026-03-10T12:00:00.000Z",
  "updatedAt": "2026-03-10T12:00:00.000Z"
}
```

## Instructor setup

### 1. Install dependencies

```bash
npm install
```

### 2. Create Firebase project

In Firebase:

1. Create a new project
2. Create a Firestore database
3. Open Project Settings
4. Go to Service Accounts
5. Generate a new private key
6. Download the service account JSON

You will use values from that JSON in your `.env` file.

### 3. Create `.env`

Copy the example file:

```bash
cp .env.example .env
```

Then set values like this:

```env
PORT=3000
NODE_ENV=development
JWT_SECRET=replace-with-a-long-random-string
JWT_EXPIRES_IN=7d
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_CLIENT_EMAIL=your-service-account-email
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY_HERE\n-----END PRIVATE KEY-----\n"
```

### Important note

Do not use the Firebase browser config for this backend.

This API uses the Firebase Admin SDK, so it needs these values from the service account JSON:

- `project_id`
- `client_email`
- `private_key`

### 4. Run locally

```bash
npm run dev
```

Expected startup output:

```text
Server is running on port 3000
```

### 5. Health check

```http
GET http://localhost:3000/health
```

Expected response:

```json
{
  "status": "ok"
}
```

## Quick local test flow

### Register

```http
POST http://localhost:3000/auth/register
Content-Type: application/json
```

```json
{
  "email": "student@example.com",
  "password": "password123"
}
```

### Login

```http
POST http://localhost:3000/auth/login
Content-Type: application/json
```

```json
{
  "email": "student@example.com",
  "password": "password123"
}
```

### Get current user

```http
GET http://localhost:3000/auth/me
Authorization: Bearer YOUR_JWT_TOKEN
```

### Create note

```http
POST http://localhost:3000/notes
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

```json
{
  "title": "My first note",
  "body": "Hello from the API"
}
```

### List notes

```http
GET http://localhost:3000/notes
Authorization: Bearer YOUR_JWT_TOKEN
```

## Deploy to Render

### Option 1

1. Push the project to GitHub
2. Create a new Web Service in Render
3. Connect the repository
4. Use these settings:
   - Build Command: `npm install`
   - Start Command: `npm start`
5. Add your environment variables
6. Deploy

### Option 2

Use the included `render.yaml` for a Blueprint deploy.

## Required Render environment variables

```env
NODE_ENV=production
JWT_SECRET=your-long-random-secret
JWT_EXPIRES_IN=7d
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_CLIENT_EMAIL=your-service-account-email
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY_HERE\n-----END PRIVATE KEY-----\n"
```

## Recommended teaching flow

A sequence that works well in class:

1. Students register their own account
2. Students log in and copy their JWT
3. Students test `GET /auth/me`
4. Students create a note
5. Students list notes
6. Students fetch a single note by ID
7. Students update a note
8. Students delete a note
9. Students build a static frontend using `fetch()`

## Postman workflow

Recommended order for students:

1. `POST /auth/register`
2. `POST /auth/login`
3. `GET /auth/me`
4. `POST /notes`
5. `GET /notes`
6. `GET /notes/:id`
7. `PUT /notes/:id`
8. `DELETE /notes/:id`

Remind students to:

- set `Body -> raw -> JSON`
- send `Content-Type: application/json`
- send `Authorization: Bearer YOUR_JWT_TOKEN` on protected routes

## Example fetch request

```js
const token = "YOUR_JWT_TOKEN";

const response = await fetch("http://localhost:3000/notes", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    title: "My first note",
    body: "Hello from the API",
  }),
});

const data = await response.json();
console.log(data);
```

## Example curl request

```bash
curl -X POST http://localhost:3000/notes \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"My first note","body":"Hello from the API"}'
```

## Troubleshooting

### Firebase credentials fail

Make sure you used the Firebase service account JSON values, not the frontend Firebase config.

### `title and body are required`

The request body was likely not sent as JSON. In Postman, use raw JSON.

### `Invalid or expired token`

Check that:

- the token is current
- the header starts with `Bearer `
- the server has the correct `JWT_SECRET`

### Browser client cannot reach the API

Check:

- the base URL
- CORS
- the authorization header
- the JSON body

### Render deployment fails after startup

Verify all environment variables are set and that the private key preserves line breaks using `\n`.

## Notes for instructors

North Star API is intentionally simple and optimized for teaching clarity, not enterprise-scale production use. It uses sound fundamentals such as hashed passwords, JWT auth, environment-based secrets, and protected routes, while staying small enough to explain and support in a classroom.

The naming, docs, and workflow are designed to feel like a real developer platform so students can learn against something that feels professional.
