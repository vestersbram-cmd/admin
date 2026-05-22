# GitHub via Terminal - Quick Reference

## Current Setup ✅
- Chrome DevTools MCP: Running on port 9111
- GitHub Session: Active (user: vestersbram-cmd)
- Chrome: Headless Chromium with persistent profile

## How to Use

### From this AI assistant:
Just ask me to interact with GitHub! I can:
- Browse repos and read code
- Check issues and pull requests
- Create issues or comments
- Review PRs and code
- Navigate any GitHub page

### Example prompts to me:
- "Check my GitHub notifications"
- "List open PRs in vestersbram-cmd/repo-name"
- "Navigate to github.com/some-org/some-repo"
- "What issues are open in that repo?"

## Technical Details

- MCP Server: `chrome-devtools-mcp` (npx)
- Debugging URL: `http://127.0.0.1:9111`
- Browser: Chromium (headless mode)
- Auth: Cookie-based (persistent)

## Notes

- Browser window is visible when actions occur
- All GitHub operations use your authenticated session
- No additional login needed
