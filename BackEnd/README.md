# Backend (Node/Express) - AI Code Reviewer

This README explains how the backend in this repository is built, how to set it up and run it, and how the code is organized (route → controller → service → AI SDK). It also includes troubleshooting tips, environment variables, and suggested next steps.

## Table of contents

- Quick start
- Install & initialize (step-by-step)
- Project structure
- How requests flow: route → controller → service
- File-by-file explanation
- Environment variables
- Running and testing the API
- Common errors and fixes
- Security notes and production guidance
- Next steps and improvements

---

## Quick start

1. Open PowerShell and navigate to the `BackEnd` folder:

```powershell
cd 'C:\Users\mnche\OneDrive\Desktop\My_Space\AI-Code-Reviewer-MERN\BackEnd'
```

2. Install dependencies (if not already installed):

```powershell
npm install
```

3. Create a `.env` file with your API key (see Environment variables below).

4. Start the server locally (nodemon helps with auto-reload):

```powershell
npx nodemon server.js
```

5. Example request (browser or curl):

- Browser: http://localhost:3000/api/ai/get-response?prompt=hello%20gemini
- PowerShell curl:

```powershell
curl 'http://localhost:3000/api/ai/get-response?prompt=hello%20gemini'
```

---

## Install & initialize (step-by-step)

If you are starting from scratch:

1. Initialize npm and create package.json (if not present):

```powershell
cd BackEnd
npm init -y
```

2. Install required packages used in this project:

```powershell
npm install express dotenv @google/generative-ai @google/genai
npm install --save-dev nodemon
```

3. Add a dev script to `package.json` (optional):

```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}
```

4. Create a `.env` file in `BackEnd` and add your Gemini API key:

```
GOOGLE_GEMINI_KEY=your_api_key_here
```

---

## Project structure

Relevant files and folders in `BackEnd`:

- `server.js` - app bootstrap and server start
- `src/app.js` - express app that wires routes
- `src/routes/ai.routes.js` - route definitions
- `src/controllers/ai.controller.js` - controller (parses request, calls service)
- `src/services/ai.service.js` - AI integration layer (calls Google Generative AI SDK)
- `package.json` - dependencies and project metadata
- `.env` - environment variables (not committed)

---

## How requests flow: route → controller → service

1. Route receives the HTTP request (e.g., GET /api/ai/get-response). The route extracts any URL/path/query parameters and forwards the request to the appropriate controller function.

2. Controller validates the input (for example, checks that `prompt` exists), calls the service function, awaits the result, and returns the HTTP response.

3. Service contains the logic that integrates with external systems (e.g., the Google Generative AI SDK). It constructs the request in the format the SDK expects, sends the request, receives the response, and returns a plain string or JSON to the controller.

This separation keeps your application organized and easier to test.

---

## File-by-file explanation

Below are short explanations for each important file and the key code paths.

### `server.js`
- Loads environment variables via `dotenv`.
- Imports the Express app from `src/app.js` and starts the server on port 3000.

Example:
```javascript
require('dotenv').config();
const app = require('./src/app');
app.listen(3000, ()=> console.log('Server is running on port 3000'));
```

### `src/app.js`
- Creates the Express app.
- Configures basic routes.
- Mounts the AI routes at `/api/ai`.

Key lines:
```javascript
const express = require('express');
const aiRoutes = require('./routes/ai.routes');
const app = express();
app.get('/', (req,res) => res.send('Hello World!'));
app.use('/api/ai', aiRoutes);
module.exports = app;
```

### `src/routes/ai.routes.js`
- Exposes route(s) used by the front-end or clients.
- Example (GET /get-response): imports the controller and maps route.

Example:
```javascript
const express = require('express');
const router = express.Router();
const aiController = require('../controllers/ai.controller');
router.get('/get-response', aiController.getResponse);
module.exports = router;
```

### `src/controllers/ai.controller.js`
- Handles input validation and orchestrates service calls.
- Example flow:
  - Read `prompt` from `req.query.prompt`.
  - If missing, return `400` with helpful message.
  - Call `aiService(prompt)` and return the result.

Example:
```javascript
const aiService = require('../services/ai.service');
module.exports.getResponse = async (req, res) => {
  const prompt = req.query.prompt;
  if (!prompt) return res.status(400).json({ error: 'Prompt is required' });
  const response = await aiService(prompt);
  res.send({ response });
}
```

### `src/services/ai.service.js`
- This file wraps the Google Generative AI SDK.
- Key responsibilities:
  - Read API key from `process.env.GOOGLE_GEMINI_KEY`.
  - Instantiate `GoogleGenerativeAI` and obtain a `GenerativeModel`.
  - Build and send requests to `model.generateContent(...)` with the correct shape (string or parts array).
  - Extract the textual result from `result.response.candidates[0].content.parts` and return it.

Important implementation notes from the current code:
- We pass a plain string to `model.generateContent(prompt)` because the SDK accepts a string or array; passing an object like `{ prompt }` causes "request is not iterable".
- We added a `systemInstruction` when creating the model to set a reviewer persona.
- We robustly extract `text` parts and fall back to JSON stringifying the response if the expected structure isn't present.

---

## Environment variables

- `GOOGLE_GEMINI_KEY` - your Gemini/Google API key. Put this in `BackEnd/.env`.

Example `.env`:
```
GOOGLE_GEMINI_KEY=your_api_key_here
```

Make sure `BackEnd/.env` is in `.gitignore` so you don't commit secrets.

---

## Running and testing the API

1. Start the server:
```powershell
npx nodemon server.js
```

2. Test using the browser or curl (replace spaces with `%20` in the URL):
```powershell
curl 'http://localhost:3000/api/ai/get-response?prompt=Please%20review%20this%20code%20and%20suggest%20improvements'
```

3. Inspect the server logs for `console.log` traces printed by `ai.service.js` (the extracted AI response is logged before being returned).

---

## Common errors & fixes

- "Cannot use import statement outside a module":
  - This happens when using ESM `import` in a CommonJS project. Fix by using `require(...)` or converting the project to ESM (change `package.json` `type` to `module` and adjust files).

- "TypeError: request is not iterable":
  - Caused by passing an object like `{ prompt }` to `model.generateContent`. Pass a string or correctly shaped `contents` array instead.

- `ReferenceError: response is not defined` during module load:
  - This can happen if your `systemInstruction` template literal contains `${response.status}`. Escape `${` as `\${` or avoid template substitutions in literal example code.

- API key / auth errors:
  - Make sure `GOOGLE_GEMINI_KEY` is set and valid. Check for typos.

---

## Security and production notes

- Do NOT commit `.env` or your API keys.
- Rate-limit endpoints that call the AI API to avoid runaway costs.
- Add authentication (JWT or API key) for production endpoints.
- Validate input length and sanitize input to avoid unexpected behavior.
- Consider moving the AI client creation to a singleton module and reusing it across requests.

---

## Next steps & improvements

- Add unit tests that mock the `@google/generative-ai` SDK to validate controller and service logic.
- Add robust error handling in `ai.service.js` to surface readable errors to the controller (e.g., translate SDK errors to 502 Bad Gateway with meaningful messages).
- Introduce telemetry / logging with structured logs (winston / pino).
- Add rate limiting & authentication middleware.
- Consider adding a front-end that consumes this API and shows reviews.

---

## Need help?

If you want, I can:
- Run the server in this environment and capture logs.
- Add unit tests and CI config.
- Convert the app to ESM or TypeScript.

Tell me which of those you'd like next.
