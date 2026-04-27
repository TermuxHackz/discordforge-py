![DiscordForge](https://discordforge.org/images/logo.png)

# discordforge-py

Python SDK for the [DiscordForge](https://discordforge.org) bot listing platform.  
Fully async. Compatible with `discord.py` and other Python Discord libraries.

[![PyPI version](https://img.shields.io/pypi/v/discordforge.svg?style=flat-square&cacheSeconds=300)](https://pypi.org/project/discordforge)
[![PyPI downloads](https://img.shields.io/pypi/dm/discordforge.svg?style=flat-square&cacheSeconds=300)](https://pypi.org/project/discordforge)
[![CI](https://img.shields.io/github/actions/workflow/status/discordforge/discordforge-py/test.yml?style=flat-square&cacheSeconds=300)](https://github.com/discordforge/discordforge-py/actions)
[![license](https://img.shields.io/pypi/l/discordforge.svg?style=flat-square&cacheSeconds=300)](https://github.com/discordforge/discordforge-py/blob/main/LICENSE.txt)

---

## Installation

```bash
pip install discordforge
```

**Requirements:** Python 3.8 or higher.

## discord.py Compatibility

This package does not conflict with `discord.py` imports:

- `discord.py` uses `import discord`
- this package uses `import discordforge`

Use `AsyncDiscordForgeClient` in `discord.py` bots to avoid blocking the event loop.

## Quick Start

```python
from discordforge import DiscordForgeClient

client = DiscordForgeClient(api_key="YOUR_API_KEY")

# Post your bot stats
client.post_bot_stats(server_count=1500, shard_count=5, user_count=50000)

# Check if a user voted
vote = client.check_vote(bot_id="YOUR_BOT_ID", user_id="USER_ID")
if vote.has_voted:
    print("Thanks for voting!")

# Fetch your bot's public profile (no API key required)
bot = client.get_bot(bot_id="YOUR_BOT_ID")
print(f"{bot.name} — {bot.vote_count} votes")
```

## API Reference

### `DiscordForgeClient(api_key?)`

Synchronous client. Use for scripts and non-async bots.

### `AsyncDiscordForgeClient(api_key?)`

Async client. Use with `discord.py` and other async frameworks.

### Methods

| Method | Returns | Description | Rate Limit |
|--------|---------|-------------|------------|
| `post_bot_stats(server_count, shard_count?, user_count?, voice_connections?)` | `None` | Update your bot's stats | 1 req / 5 min |
| `check_vote(bot_id, user_id)` | `VoteResult` | Check if a user voted in the last 12h | 60 req / min |
| `get_bot(bot_id)` | `BotInfo` | Fetch your bot's public profile | — |
| `sync_commands(commands)` | `SyncResult` | Sync up to 200 slash commands | — |
| `sync_from_discordpy(command_source, category?, limit?, strict_limit?)` | `SyncResult` | Map and sync from a `discord.py` command tree | — |

### Error Handling

```python
from discordforge import DiscordForgeClient, DiscordForgeAPIError, DiscordForgeValidationError

client = DiscordForgeClient(api_key="YOUR_API_KEY")

try:
    client.post_bot_stats(server_count=1500)
except DiscordForgeValidationError as exc:
    print("Invalid input:", exc)
except DiscordForgeAPIError as exc:
    print(f"API error {exc.status_code}:", exc)
```

## Usage with discord.py

```python
import os
import discord
from discord.ext import tasks
from discordforge import AsyncDiscordForgeClient

bot = discord.Client(intents=discord.Intents.default())
forge = AsyncDiscordForgeClient(api_key=os.environ["DISCORDFORGE_API_KEY"])

@bot.event
async def on_ready():
    post_stats.start()

@tasks.loop(minutes=5)
async def post_stats():
    await forge.post_bot_stats(
        server_count=len(bot.guilds),
        shard_count=getattr(bot, "shard_count", None),
    )

@bot.event
async def setup_hook():
    await forge.sync_from_discordpy(command_source=bot.tree, category="General")

bot.run(os.environ["BOT_TOKEN"])
```

## Syncing Slash Commands

The SDK accepts both DiscordForge custom format and raw Discord API format — auto-detected per command.

```python
result = client.sync_commands(
    commands=[
        {
            "name": "ban",
            "description": "Ban a user from the server",
            "usage": "/ban <user> [reason]",
            "category": "Moderation",
        },
        {
            "name": "play",
            "description": "Play a song in your voice channel",
            "type": 1,
            "options": [{"name": "query", "type": 3, "required": True}],
        },
    ]
)
print(result.success, result.synced)
```

Each sync **replaces** all previously synced commands. Max 200 commands per request.

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting a PR.

Originally created by [Ram2](https://github.com/Antiscammer-Dev-team).

## Links

- [DiscordForge Dashboard](https://discordforge.org/dashboard)
- [API Documentation](https://discordforge.org/support/developers)
- [PyPI Package](https://pypi.org/project/discordforge)

## License

[MIT](LICENSE.txt) © DiscordForge
