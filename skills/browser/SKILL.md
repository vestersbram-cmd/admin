---
name: browser
description: Use Chrome DevTools browser automation to interact with web pages, extract information, and verify functionality through an existing browser session.
---

# Browser

Use the browser-use agent to interact with web pages through an existing Chrome DevTools session. The browser is already authenticated with common services (GitHub, Gmail, Supabase, etc.), so no manual login is required.

## When to Use

- Extract information from a web page (documentation, prices, data)
- Test web application functionality (click buttons, fill forms, check results)
- Verify that code changes render correctly in a browser
- Navigate websites and extract structured information
- Check for console errors, broken layouts, or missing elements
- Validate responsive design and accessibility
- Interact with services that require login (the browser profile is pre-authenticated)

## Prerequisites

- Chrome must be installed
- Chrome DevTools server must be running on port 9111
- The browser window is visible (non-headless) — you can watch actions happen
- Multiple sessions can collaborate on the same browser

## How to Use

1. **Spawn the browser-use agent** with a clear task description and starting URL
2. **Be specific** about what elements to interact with or what information to extract
3. **Check results** — review the agent's step-by-step outcomes, console errors, and lessons

## Example Tasks

- "Navigate to localhost:3000 and verify the login form works"
- "Go to the documentation page and extract the API endpoint list"
- "Check for JavaScript console errors on the dashboard"
- "Fill out the contact form and verify the success message"
- "Navigate to pull requests and show the open ones"
- "Check if the new feature renders correctly on mobile viewport"

## Output to Review

After the agent completes, check:

- **`results`** — step-by-step outcomes of each action
- **`consoleErrors`** — any JavaScript errors found during the session
- **`lessons`** — advice on improving future browser automation tasks

## Important Notes

- The browser profile is persistent — saved credentials, bookmarks, and extensions are available
- No need to log in to services — the browser is already authenticated
- If a session fails or times out, clean up lingering tmux sessions with `tmux kill-session`
- For tasks requiring 3+ steps, plan the navigation flow before spawning the agent
- Be precise about selectors and expected page states to avoid ambiguity
