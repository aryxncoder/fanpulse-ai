# FanPulse AI

A small MERN stack cricket companion that helps new fans understand scores and match statistics. It shows a live-style scoreboard, turns the numbers into plain-English insights, and answers beginner questions with predefined explanations. No paid AI API is used.

## Features

- Responsive dark sports dashboard with a scoreboard and match insights (run rate, required rate, projected score, wickets in hand, match phase)
- Beginner explainer: tap a topic or type a question such as "what is the required run rate?"
- Register and log in with JWT authentication and bcrypt password hashing
- Create, read, update and delete saved matches (each user only sees their own)
- MongoDB persistence through Mongoose
- Validation on both client and server, with clear error messages

## Technologies

| Layer    | Tools                                              |
| -------- | -------------------------------------------------- |
| Frontend | React 18, Vite, plain CSS                          |
| Backend  | Node.js, Express 4                                 |
| Database | MongoDB with Mongoose                              |
| Auth     | jsonwebtoken, bcryptjs (pure-JS bcrypt, no build tools needed) |
| Tests    | Node's built-in test runner, supertest             |

## Project structure

```
fanpulse-ai/
├── .gitignore
├── README.md
├── client/
│   ├── .env.example
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── api.js
│       ├── AuthContext.jsx
│       ├── styles.css
│       └── components/
│           ├── Scoreboard.jsx
│           ├── Insights.jsx
│           ├── Explainer.jsx
│           ├── AuthPanel.jsx
│           ├── MatchForm.jsx
│           └── MatchList.jsx
└── server/
    ├── .env.example
    ├── package.json
    ├── server.js          (starts the server and connects to MongoDB)
    ├── app.js             (Express app, used by server.js and tests)
    ├── config/db.js
    ├── middleware/        (auth.js, errorHandler.js)
    ├── models/            (User.js, Match.js)
    ├── routes/            (auth.js, matches.js, insights.js, explain.js)
    ├── utils/             (cricket.js, validate.js, explanations.js, asyncHandler.js)
    └── tests/             (cricket.test.js, validate.test.js, api.test.js)
```

## Setup

**Prerequisites:** Node.js 18 or newer, npm, and MongoDB (either installed locally and running, or a free MongoDB Atlas connection string).

### 1. Backend

```bash
cd server
npm install
cp .env.example .env
```

Open `server/.env` and set `JWT_SECRET` to a long random string. If you use Atlas, replace `MONGO_URI` with your Atlas connection string. On Windows Command Prompt use `copy .env.example .env`.

```bash
npm run dev
```

You should see `MongoDB connected` and `FanPulse API running on http://localhost:5000`.

### 2. Frontend (in a second terminal)

```bash
cd client
npm install
cp .env.example .env
npm run dev
```

Open http://localhost:5173.

### Production-style run (optional)

```bash
cd server && npm start
cd client && npm run build && npm run preview
```

## Usage

1. Open the dashboard. A demo match is shown right away, with insights and the explainer, no login needed.
2. Click **New here? Create an account**, register, and you are logged in.
3. Fill in **Save a match** (teams, who is batting, runs, wickets, overs, optional target) and press **Save match**.
4. Click a saved match to show it on the scoreboard. The insights update automatically.
5. Use **Edit** to change the score as the game moves, or **Delete** to remove a match.
6. Ask the explainer about terms like overs, powerplay, DRS or strike rate.

Notes on the numbers: overs use cricket notation, so `12.3` means 12 overs and 3 balls (the part after the dot goes from 0 to 5). **Target** is the number of runs the chasing team needs to win (first innings score plus one). Leave it empty for the first innings.

## API endpoints

Base URL: `http://localhost:5000/api`. Protected routes need the header `Authorization: Bearer <token>`.

| Method | Endpoint          | Auth | Description                                    |
| ------ | ----------------- | ---- | ---------------------------------------------- |
| GET    | `/health`         | No   | Server health check                            |
| POST   | `/auth/register`  | No   | Create account. Body: `name`, `email`, `password` |
| POST   | `/auth/login`     | No   | Log in. Body: `email`, `password`. Returns `token` |
| GET    | `/auth/me`        | Yes  | Current user                                   |
| GET    | `/matches`        | Yes  | List your saved matches                        |
| GET    | `/matches/:id`    | Yes  | Get one match                                  |
| POST   | `/matches`        | Yes  | Create a match                                 |
| PUT    | `/matches/:id`    | Yes  | Update a match (send all fields)               |
| DELETE | `/matches/:id`    | Yes  | Delete a match                                 |
| POST   | `/insights`       | No   | Insights for the match numbers in the body     |
| GET    | `/explain`        | No   | List explainer topics                          |
| POST   | `/explain`        | No   | Body: `question`. Returns a predefined answer  |

Match body fields: `title`, `team1`, `team2`, `battingTeam` (must equal team1 or team2), `runs`, `wickets`, `overs`, `totalOvers` (default 20), `target` (optional), `venue` (optional), `status` (`upcoming`, `live` or `completed`).

Errors always look like `{ "message": "..." }` with a suitable status code (400 invalid input, 401 not logged in, 404 not found, 409 email already used).

## Testing

### Automated tests

```bash
cd server
npm test
```

These cover the cricket calculations, match validation, auth guards, the insights route and the explainer. They do not need MongoDB.

### Manual test steps (needs MongoDB and the server running)

```bash
# 1. Register (copy the token from the response)
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Asha","email":"asha@example.com","password":"secret123"}'

# 2. Log in
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"asha@example.com","password":"secret123"}'

# 3. Create a match (replace TOKEN)
curl -X POST http://localhost:5000/api/matches \
  -H "Content-Type: application/json" -H "Authorization: Bearer TOKEN" \
  -d '{"title":"Final","team1":"Lions","team2":"Tigers","battingTeam":"Tigers","runs":112,"wickets":3,"overs":13.2,"totalOvers":20,"target":178}'

# 4. List matches
curl http://localhost:5000/api/matches -H "Authorization: Bearer TOKEN"

# 5. Update (replace MATCH_ID) and delete
curl -X PUT http://localhost:5000/api/matches/MATCH_ID \
  -H "Content-Type: application/json" -H "Authorization: Bearer TOKEN" \
  -d '{"title":"Final","team1":"Lions","team2":"Tigers","battingTeam":"Tigers","runs":130,"wickets":4,"overs":15.0,"totalOvers":20,"target":178}'
curl -X DELETE http://localhost:5000/api/matches/MATCH_ID -H "Authorization: Bearer TOKEN"
```

### Manual checks in the browser

- Register with a short password (expect an error), then with valid details (expect to be logged in).
- Refresh the page: you should stay logged in. Click **Log out**: the saved matches disappear.
- Try saving a match with overs `12.6` or wickets `11` (expect clear error messages).
- Save, edit and delete a match, and confirm the list and scoreboard update.
- Stop the backend and reload: the page should show a "Cannot reach the server" message instead of crashing.
- Resize the window to phone width: the layout should stack into one column.

## Troubleshooting

- **`Could not connect to MongoDB`**: start MongoDB, or check `MONGO_URI` in `server/.env`.
- **`Missing MONGO_URI or JWT_SECRET`**: you have not created `server/.env` yet.
- **Browser shows a CORS error**: `CLIENT_URL` in `server/.env` must match the address of the React app (`http://localhost:5173`).
- **Port already in use**: change `PORT` in `server/.env` and update `VITE_API_URL` in `client/.env` to match.

## Publishing to GitHub

```bash
git init
git add .
git commit -m "Initial commit: FanPulse AI MERN project"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/fanpulse-ai.git
git push -u origin main
```

Create the empty public repository named `fanpulse-ai` on github.com first (do not add a README there). Check that `.env` is not listed by `git status` before committing.

## License

MIT
