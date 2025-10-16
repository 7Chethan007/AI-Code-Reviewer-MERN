Frontend (React + Vite) — AI Code Reviewer
This README explains how the frontend is built, how to set it up and run it, how it connects to the backend, and explains the important code (components, state variables, external libraries).

Quick start
Open PowerShell and navigate to the Frontend folder:
Install dependencies (if not already installed):
Start the dev server:
Open the dev URL printed by Vite (usually http://localhost:5173) and try the app.
How the project was created
This frontend was initially scaffolded with Vite using the React template:

Vite provides a fast development server and a modern build pipeline.

Project structure (relevant files)
index.html — entry HTML with a div#root where React mounts.
main.jsx — React entry that mounts the App component.
App.jsx — The main UI component: code editor, review button, and review output.
App.css — Styles for the components.
index.css — Global styles and theme.
package.json — dependencies and scripts.
How frontend and backend connect
The frontend performs a POST request to the backend endpoint /ai/get-review with the source code in the request body:

URL used in the app: http://localhost:3000/ai/get-review
Request body: { code: "...source code..." }
The backend returns an object like { response: "AI review text" }.

The frontend extracts that text and renders it in the right panel using react-markdown.

Note: CORS must be enabled on the backend (the backend's app.js already uses cors()), otherwise the browser will block requests.

Files explained (detailed)
Below is a line-by-line style explanation of the frontend's important files and code patterns.

main.jsx
Bootstraps the React app and mounts the App component into the DOM.
Uses StrictMode for highlighting potential problems during development.
Example:

App.jsx — the main component (walkthrough)
Imports used:

react hooks: useState, useEffect — manage local component state and side effects.
prismjs + prismjs/themes/prism-tomorrow.css — syntax highlighting for code.
react-simple-code-editor — a lightweight code editor component.
react-markdown + rehype-highlight + highlight.js — render the AI review as formatted markdown with highlighted code blocks.
axios — HTTP client for making requests to the backend.
App.css — component-specific styles.
Key state variables in App:

code (string) — contains the code being edited. Initially set to:
review (string) — stores the AI's review. Rendered using react-markdown.
loading (boolean) — indicates a pending API request.
Side effects:

useEffect(() => { prism.highlightAll() }, []) — calls prism to highlight any code blocks on mount. This is useful if you pre-render markup that should be highlighted. The editor itself uses prism.highlight for inline highlighting.
The editor:

react-simple-code-editor is used as a controlled input where value={code} and onValueChange updates the code state.
highlight prop uses prism.highlight(code, prism.languages.javascript, 'javascript') to return highlighted HTML for the editor.
The Review action:

reviewCode() sends a POST request to http://localhost:3000/ai/get-review with body { code }.
It safely extracts response.data.response and sets review to a string. If the backend returns an object, the UI converts it to JSON string to avoid crashing the Markdown renderer.
Loading state toggles the button text between Review and Reviewing....
Rendering the review:

react-markdown renders the review string as Markdown. rehype-highlight adds language-aware highlighting for code blocks.
Potential pitfalls covered in code:

If the backend returns an object instead of a string, the renderer could break — we guard by converting non-strings to JSON text.
The app uses CORS; ensure the backend allows cross-origin requests.
External libraries and why they were chosen
react / react-dom — UI library.
vite — development server & build tool.
prismjs — client-side syntax highlighting (works in editor preview and Markdown blocks).
react-simple-code-editor — minimal, easy-to-style code editor that integrates with Prism.
react-markdown — render Markdown safely in React.
rehype-highlight + highlight.js — add code block highlighting to Markdown.
axios — simple HTTP client for browser requests.
Troubleshooting & debugging tips
Blank UI after clicking Review:

Open DevTools (F12) → Console to see any errors.
Check Network tab for the POST request — ensure it returns 200 and a valid JSON body.
Make sure the backend returns a string (or the front-end converts non-strings to text).
Prism CSS not found:

Confirm node_modules/prismjs/themes/prism-tomorrow.css exists.
If missing, run npm install prismjs.
CORS errors:

Ensure backend includes app.use(cors()) and it’s running on port 3000.
Testing the UI manually
Start the backend:
Start the frontend (in another terminal):
Open the app (http://localhost:5173). Paste code into the editor and click Review.

Watch the backend logs for console.log entries from ai.service.js and frontend console for network errors.

Next improvements you can make
Add input validation and limit code size to avoid expensive AI calls.
Add authentication and rate-limiting.
Add client-side error banners and nicer spinner components.
Add unit tests (Jest + React Testing Library) for the UI and integration tests for the full stack.
Replace react-simple-code-editor with a full-featured editor (Monaco) if you need intellisense.
