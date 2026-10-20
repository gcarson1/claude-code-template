# Gemini MCP setup (API-key based)

Server: [`@houtini/gemini-mcp`](https://www.npmjs.com/package/@houtini/gemini-mcp) (Apache-2.0, runs via `npx`, uses a Google AI Studio key, not Vertex AI/GCP). Needs Node 20.19+.

## 1. Get and check an API key
1. Go to https://aistudio.google.com/apikey and sign in with your Google account.
2. Click **Create API key**. Copy it once; treat it like a password.
3. Check it works (prints a short reply if the key is valid):
   ```bash
   curl -s "https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY" | head -c 300
   ```
   A JSON list of models means it works. An `API_KEY_INVALID` error means it doesn't.

## 2. Store it as an environment variable (not in the repo)
Put it in your shell profile so it is never in a project file:

```bash
# macOS (zsh): ~/.zshrc    Linux (bash): ~/.bashrc
echo 'export GEMINI_API_KEY="paste-key-here"' >> ~/.zshrc
chmod 600 ~/.zshrc
source ~/.zshrc
```
Then restart your terminal (and the Claude desktop app if you launch Claude Code from it) so it inherits the variable.

The project's `.mcp.json` only contains `${GEMINI_API_KEY}`, a reference, never the key, so it is safe to commit. `.gitignore` also blocks `.env` files as a backstop.

## 3. Register the server
**Option A (recommended): project config.** Copy `project/.mcp.json` from this template into the repo root. Start `claude` in the project and approve the server when prompted. Confirm with `/mcp`.

**Option B: one command, available in all your projects:**
```bash
claude mcp add -s user -e GEMINI_API_KEY="$GEMINI_API_KEY" -e GEMINI_DEFAULT_MODEL=gemini-3-flash-preview gemini -- npx -y @houtini/gemini-mcp
```
Trade-off: this writes the key's literal value into `~/.claude.json` in your home folder (outside any repo, but plaintext). Option A avoids that.

Check: `claude mcp list` should show `gemini` as connected.

## 4. Cost note
The server's default chat model is a paid-tier model, and a free-tier key will fail on it. The config above sets `GEMINI_DEFAULT_MODEL=gemini-3-flash-preview`, which has a free tier per the server's README. Check https://ai.google.dev/gemini-api/docs/pricing before relying on any model, since tiers change. Heavy long-context use on paid models costs real money; watch usage in AI Studio.

## 5. Using it in a session
Just ask in plain language and name Gemini:
- "Ask Gemini for a second opinion on this architecture explainer. What would a skeptical customer question?"
- "Use Gemini to summarize this 200-page RFP at `docs/rfp.pdf`'s extracted text and list the technical requirements."
- "Have Gemini search-ground a check on whether this product claim is current."

Claude calls the server's `gemini_chat` (or other) tool and shows you the reply. Don't paste customer-confidential material into Gemini unless your company's data policy allows sending it to Google.
