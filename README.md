# Claude Code Discord Channel Setup

Connect Claude Code to Discord so you can message your coding agent from your phone or desktop via a Discord bot.

## Prerequisites

- Claude Code v2.1.80+
- Claude Max or Pro subscription (claude.ai login, not API key)
- [Bun](https://bun.sh) installed (`bun --version` to check)
- A Discord account and server

## Step 1: Create a Discord Bot

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **New Application** and name it (e.g. "Javis")
3. Go to **Bot** in the left sidebar
4. Click **Reset Token** and copy it immediately (only shown once)
5. Save the token to your `.env` file (e.g. `DISCORD_BOT_TOKEN=your-token-here`)

### Enable Message Content Intent

Still on the Bot page:

1. Scroll to **Privileged Gateway Intents**
2. Toggle **Message Content Intent** ON
3. Click **Save Changes** at the bottom

> Without this, the bot receives empty messages and won't respond.

### Bot Permissions (on the same page)

You do NOT need to tick anything in the Bot Permissions section on this page. Permissions are set in Step 2 via the OAuth2 URL.

### Public Bot Toggle

- **Leave Public Bot ON** if you want to use the OAuth2 URL Generator for inviting (required for the invite flow).
- The real security gate is the access policy (Step 5), not this toggle.

## Step 2: Invite the Bot to Your Server

1. Go to **OAuth2 > URL Generator** in the Developer Portal
2. Under **Scopes**, tick `bot`
3. Under **Bot Permissions**, tick:
   - **General:** View Channels
   - **Text:** Send Messages, Send Messages in Threads, Attach Files, Read Message History, Add Reactions
4. Set **Integration Type** to `Guild Install`
5. Copy the **Generated URL** and open it in your browser
6. Select your Discord server and authorize

## Step 3: Install the Discord Plugin in Claude Code

Run these commands inside Claude Code:

```
/plugin marketplace add anthropics/claude-plugins-official
/plugin install discord@claude-plugins-official
/reload-plugins
```

## Step 4: Configure the Bot Token

### Option A: Via the skill command

```
/discord:configure <your-bot-token>
```

### Option B: Manually

```bash
mkdir -p ~/.claude/channels/discord
echo "DISCORD_BOT_TOKEN=your-token-here" > ~/.claude/channels/discord/.env
chmod 600 ~/.claude/channels/discord/.env
```

If loading from a `.env` file:

```bash
source /path/to/your/.env
echo "DISCORD_BOT_TOKEN=$YOUR_ENV_VAR_NAME" > ~/.claude/channels/discord/.env
chmod 600 ~/.claude/channels/discord/.env
```

> Never print tokens in terminal output. The `.env` file should be `chmod 600` (owner-only).

## Step 5: Launch, Pair, and Lock Down

### Launch with channels enabled

```bash
claude --channels plugin:discord@claude-plugins-official
```

### Pair your Discord account

1. DM your bot on Discord — it replies with a pairing code
2. In Claude Code, run:
   ```
   /discord:access pair <pairing-code>
   ```

### Lock to allowlist (important!)

Once you're paired, lock access so only you can use the bot:

```
/discord:access policy allowlist
```

> The default `pairing` policy is temporary. Always lock to `allowlist` once your account is paired.

## Step 6: Enable Guild Channel Auto-Reply (Optional)

By default, the bot only responds to DMs. To make it respond to **every message** in a specific server channel (without needing `@mention`):

```
/discord:access group add <channel-id> --no-mention
```

To get a channel ID: enable **Developer Mode** in Discord (User Settings > Advanced), then right-click the channel > **Copy Channel ID**.

Options:

```bash
# Auto-reply to all messages in the channel (no @mention needed)
/discord:access group add <channel-id> --no-mention

# Only respond when @mentioned or replied to (default)
/discord:access group add <channel-id>

# Restrict to specific users in the channel
/discord:access group add <channel-id> --no-mention --allow user_id1,user_id2
```

> Changes take effect immediately — no restart needed.

## Step 7: Name Your Session

Name the session so you can resume it later:

```
/rename javis-discord
```

Now you can resume with:

```bash
claude --resume javis-discord --channels plugin:discord@claude-plugins-official
```

## Step 8: Auto-Start on Login (Optional)

To keep the bot alive across reboots, add a `.command` script as a macOS Login Item.

See [`javis-discord-start.command`](../javis-discord-start.command) for the script.

### Setup

1. Open **System Settings > General > Login Items & Extensions**
2. Under "Open at Login", click **+**
3. Navigate to and select the `.command` file

On login, Terminal.app opens and launches the Discord bot session automatically.

### How the script works

```zsh
# Resume existing session, or create new named session if not found
claude --resume javis-discord --channels plugin:discord@claude-plugins-official 2>/dev/null \
  || claude --name javis-discord --channels plugin:discord@claude-plugins-official
```

The fallback (`||`) handles the case where the session doesn't exist yet (first boot or after session cleanup).

## Security Summary

| Layer | What it does |
|-------|-------------|
| Private Bot (optional) | Only you can invite the bot to servers |
| Shared server requirement | Users must share a server with the bot to DM it |
| Pairing mode | Requires manual approval via `/discord:access pair` |
| Allowlist policy | Only approved Discord user IDs can interact |
| Token file permissions | `chmod 600` — owner-only read/write |

## Troubleshooting

### Bot is online but not replying

- Did you **save** Message Content Intent in the Developer Portal? The toggle alone isn't enough — click Save Changes.
- Check the Claude Code terminal for errors.
- Verify `bun --version` returns a version.

### `/discord:configure` not found

Run `/reload-plugins` first. If still not found, invoke via the Skill tool in the session.

### Bot goes offline

The bot only runs while the `claude --channels` session is open. Use the auto-start script (Step 7) to persist across reboots.

### Wrong token

Update the token:

```bash
source /path/to/your/.env
echo "DISCORD_BOT_TOKEN=$YOUR_NEW_TOKEN_VAR" > ~/.claude/channels/discord/.env
chmod 600 ~/.claude/channels/discord/.env
```

Then restart the `claude --channels` session.

## Limitations

- Bot only works while the Claude Code session is running
- Messages sent while the session is closed are lost
- No Slack support yet (Telegram and iMessage are also available)
- Channels is a research preview feature (as of March 2026)
