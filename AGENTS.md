This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **granola-mcp**: The Granola MCP server. The agent uses it to list recent meetings, fetch summaries and transcripts for newly-finished calls, and search across past meetings when answering Slack questions. Authenticated with a personal access token from your Granola account.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts each call recap to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires once a day (`every: 24h`, UTC). Polls Granola for newly-finished meetings and recaps any it hasn't seen before. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

- **GRANOLA_ACCESS_TOKEN**: A personal access token from Granola. Get one at granola.ai → Settings → Developer / API → create a new access token. Add it as a slot value when you connect the `granola-mcp` connector during the dashboard setup flow.

### External Setup

1. Mint a personal access token in Granola (Settings → Developer / API). Paste it into the `GRANOLA_ACCESS_TOKEN` slot during deploy.
2. After deploy, install the agent's Slack bot in your workspace and invite it to whichever deal channel(s) you want call recaps in. The agent posts each recap to every channel it's a member of — invite it to one focused channel, or several. If the bot has not been invited anywhere, recaps are sent as a DM to the workspace install user with a one-line nudge to invite it somewhere.
3. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc questions like *"what happened on the Acme call?"*.
4. The first heartbeat fire after deploy seeds MEMORY.md with the most recent existing meeting — the agent does NOT back-fill old recaps. The first real recap will land 5 minutes after the next Granola call ends. To smoke-test sooner, @mention the bot in Slack with a question like *"summarize my last meeting"* — that exercises the Slack + Granola path without waiting for a new call.

## Customizing

- **Change the heartbeat interval**: edit the `every` value on the `heartbeat` channel in `valet.yaml`, then redeploy. Common values: `24h` (default, once-a-day digest), `1h` (hourly), `15m` (near real-time, more Granola API traffic).
- **Change what counts as "moments that mattered"**: edit the *Phase 2: Distill the moments that mattered* section in `SOUL.md`. The current default is decisions / blockers / next step / risks, capped at 5 bullets total. Adjust the categories or the cap to match how your team thinks about a call.
- **Change the recap format**: the Slack `mrkdwn` template lives in `SOUL.md` under *Phase 3: Post the recap*. Edit the structure, emoji, or character cap there.
- **Control where recaps post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
- **Public-channel email redaction**: SOUL.md redacts participant emails from any channel where `is_private: false`. If your team is comfortable with emails in public channels, remove that rule from the *Always* guardrails.
