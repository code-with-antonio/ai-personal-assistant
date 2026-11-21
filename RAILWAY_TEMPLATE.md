# Deploy and Host Personal AI Assistant on Railway

**The open-source personal AI assistant you can text.** Message it on Telegram or iMessage. It searches the web, works on a computer of its own and sends the files back.

Sign-in, a paid plan and observability are already in place. You start from a working product and change whatever you like.

🎬 [Watch it being built on YouTube](https://www.youtube.com/watch?v=5B1FGk3CMyg)

## ✨ Features

- 💬 **Text it where you already are** — one-to-one chats on Telegram and on iMessage, with the same agent behind both.
- 🌐 **Live web access** — searches the web, reads pages and pulls structured data from marketplaces, travel sites, listings and more.
- 🖥️ **A computer of its own** — a persistent Linux sandbox per user, with a shell and a filesystem that stay there between chats and platforms.
- 📎 **Files both ways** — send it a PDF, photo or document to work on, and it sends files and screenshots back in the chat.
- 🔗 **Account linking from the chat** — the first message gets a sign-in link, and the chat is tied to that account once it's approved.
- 💳 **A paid plan** — only users on the Pro plan reach the agent; everyone else gets an upgrade link.
- ♾️ **Durable turns** — long tasks run as background jobs with retries, up to 30 minutes per turn.

## ⚠️ Before you deploy

One click gets you the webhook server, not the whole product. The agent runs on Trigger.dev, and the server forwards messages to two Trigger.dev webhook endpoints whose URLs and secrets Railway asks for. So **deploy the tasks to Trigger.dev first**, then deploy this template:

```bash
git clone https://github.com/code-with-antonio/ai-personal-assistant.git
cd ai-personal-assistant
npm install
```

Set `project` in `trigger.config.ts` to your own Trigger.dev project ref, then:

```bash
npx trigger.dev@4.8.3 deploy
```

In the Trigger.dev dashboard, open the production `telegram` and `linq` webhook endpoints and copy each one's URL and secret. Railway asks for them below.

## About Hosting Personal AI Assistant

This template deploys one service: the Hono server that receives the Telegram and iMessage webhooks, built from the [public repository](https://github.com/code-with-antonio/ai-personal-assistant). It checks each message, links the sender to an account, checks their plan and hands the message to the agent on Trigger.dev. Everything else is a managed service you connect with a key.

| Service         | What it does here                                                   | Needed   |
| --------------- | ------------------------------------------------------------------- | -------- |
| **Clerk**       | Sign-in through an OAuth device flow, plus Billing for the Pro plan | Yes      |
| **Neon**        | Postgres for linked accounts, and the AI Gateway the model runs on  | Yes      |
| **Trigger.dev** | Runs the chat agent and the account-linking task                    | Yes      |
| **Telegram**    | A bot from @BotFather                                               | Yes      |
| **Daytona**     | One cloud sandbox per user: shell and files                         | Yes      |
| **Firecrawl**   | Web search, scraping and structured data sources                    | Yes      |
| **Linq**        | iMessage                                                            | Optional |
| **Sentry**      | Errors, logs, traces and AI monitoring                              | Optional |

## 🔑 Variables

Railway asks for these when you deploy.

| Variable                                                          | Where it comes from                                       |
| ----------------------------------------------------------------- | --------------------------------------------------------- |
| `CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`                       | Clerk dashboard, under API keys                           |
| `DATABASE_URL`                                                    | Neon, the pooled connection string                        |
| `TRIGGER_SECRET_KEY`                                              | Trigger.dev, your project's **production** secret key     |
| `TELEGRAM_WEBHOOK_SECRET`                                         | A random string you choose. You give it to Telegram later |
| `TRIGGER_TELEGRAM_WEBHOOK_URL`, `TRIGGER_TELEGRAM_WEBHOOK_SECRET` | Trigger.dev, the production `telegram` webhook endpoint   |
| `TRIGGER_LINQ_WEBHOOK_URL`, `TRIGGER_LINQ_WEBHOOK_SECRET`         | Trigger.dev, the production `linq` webhook endpoint       |
| `LINQ_API_V3_API_KEY`, `LINQ_WEBHOOK_SECRET`                      | Optional. Linq dashboard. Leave empty to skip iMessage    |
| `SENTRY_DSN`                                                      | Optional. Leave it empty and nothing is reported          |

## 🚀 After you deploy

**1. Create the database tables**

Put your Neon direct connection string in `.env` as `DATABASE_URL_UNPOOLED`, then:

```bash
npm run db:push
```

**2. Give the tasks their variables**

In Trigger.dev, add these to the project's production environment: `DATABASE_URL`, `CLERK_OAUTH_ISSUER`, `CLERK_OAUTH_CLIENT_ID`, `CLERK_OAUTH_CLIENT_SECRET`, `NEON_AI_GATEWAY_BASE_URL`, `NEON_AI_GATEWAY_TOKEN`, `TELEGRAM_BOT_TOKEN`, `DAYTONA_API_KEY` and `FIRECRAWL_API_KEY`. Add `LINQ_API_V3_API_KEY` for iMessage and `TRIGGER_SENTRY_DSN` for Sentry.

The three `CLERK_OAUTH_*` values come from a confidential OAuth app in Clerk with the Device Authorization Grant enabled.

**3. Set up billing in Clerk**

Enable Billing and create a user plan slugged `pro`. Without it every message gets an upgrade link.

**4. Point Telegram at your Railway domain**

```bash
curl "https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/setWebhook" \
  -d "url=https://<your-railway-domain>/telegram/webhook" \
  -d "secret_token=<TELEGRAM_WEBHOOK_SECRET>"
```

For iMessage, set `https://<your-railway-domain>/linq/webhook` as the `message.received` webhook in Linq.

Then message your bot, follow the sign-in link, subscribe to Pro and start chatting.

## Common Use Cases

- A starting point for your own assistant product that people text instead of opening an app, with sign-in and a paid plan already working
- An assistant that runs errands on the web: comparing prices, checking availability, reading and summarising pages
- Learning how webhook gating, account linking over chat, durable agent turns and per-user sandboxes fit together in a real codebase

## Dependencies for Personal AI Assistant Hosting

- A [Clerk](https://clerk.com) application with Billing enabled and an OAuth app with the Device Authorization Grant
- A [Neon](https://neon.com) project with Postgres and an AI Gateway
- A [Trigger.dev](https://trigger.dev) project with the tasks deployed
- A Telegram bot from [@BotFather](https://t.me/BotFather)
- A [Daytona](https://www.daytona.io) API key and a [Firecrawl](https://firecrawl.dev) API key
- Optionally, a [Linq](https://linqapp.com) account for iMessage and a [Sentry](https://sentry.io) project

### Deployment Dependencies

- [Source code and local setup](https://github.com/code-with-antonio/ai-personal-assistant)
- [Clerk Billing](https://clerk.com/docs/billing/overview)
- [Trigger.dev deployment](https://trigger.dev/docs/deployment/overview)
- [Telegram setWebhook](https://core.telegram.org/bots/api#setwebhook)
- [Neon connection strings](https://neon.com/docs/connect/connect-from-any-app)

## Why Deploy Personal AI Assistant on Railway?

Railway runs the one part of the assistant that needs a server. It builds the webhook server from GitHub and gives it the public HTTPS domain Telegram and Linq deliver messages to. There is nothing else to host.

---

Built by [Code With Antonio](https://www.codewithantonio.com).
