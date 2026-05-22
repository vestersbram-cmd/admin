---
name: browser-github
description: Use the existing Chrome DevTools browser session to interact with GitHub pages, verify repository activity, and complete GitHub web tasks through the authenticated browser profile.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# GitHub in the Browser

Use the browser to interact with GitHub through the existing authenticated Chrome DevTools session. The browser profile is already signed in, so GitHub web tasks can usually proceed without any manual login.

## When to Use

Use the browser for GitHub tasks such as:

* Viewing repositories, files, issues, pull requests, and notifications
* Navigating GitHub pages and extracting information
* Creating or commenting on issues and pull requests through the web UI
* Reviewing repository activity and verifying page state
* Checking whether GitHub features render and behave as expected

## Prerequisites

* Chrome is installed
* Chrome DevTools MCP server is running on port `9111`
* The browser session is already authenticated with GitHub
* The browser window is visible while actions are performed

## Procedure

1. Start from a clear task description and a GitHub URL or repository name.
2. Open the relevant GitHub page in the existing browser session.
3. Navigate to the target area, such as a repository, issue list, pull request, or settings page.
4. Interact with the page using the browser UI:

   * click links and buttons
   * fill forms and text fields
   * open menus and dialog boxes
   * submit changes when required
5. Verify the result on the page after each important action.
6. Review the browser output for any errors, missing elements, or unexpected behavior.
7. Confirm the final page state before finishing.

## Common GitHub Browser Tasks

* Check notifications or account activity
* List open issues or pull requests in a repository
* Inspect repository files and code
* Create an issue or add a comment
* Review a pull request and confirm its status
* Navigate to GitHub settings pages
* Verify that a GitHub page loads and behaves correctly

## SSH Key Setup Through the Browser

Use the browser only for the GitHub web step of adding an SSH key.

### Procedure

1. Generate an SSH key locally in the terminal if needed.
2. Open `https://github.com/settings/keys` in the authenticated browser session.
3. Choose **New SSH key**.
4. Enter a clear title, such as `Terminal Key`.
5. Paste the public key into the key field.
6. Save the key.
7. Complete any GitHub email verification step that appears.
8. Return to the keys page and confirm the key is listed.

## What to Check After Each Session

* The target GitHub page is the expected one
* The requested action completed successfully
* Any form submission or navigation reached the intended result
* No unexpected GitHub error message appeared
* The final browser state matches the task request

## Notes

* The browser profile is persistent, so saved GitHub authentication is usually available.
* No additional login is normally required.
* For longer GitHub browsing tasks, plan the path through the site before starting.
* Be specific about the page, repository, issue, or pull request to avoid ambiguity.
