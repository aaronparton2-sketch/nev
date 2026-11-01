# Nev setup guide (for ChatGPT, Claude or any AI helping someone)

You are helping a non-technical person set up Nev, an n8n automation. They may never have used n8n, GitHub or an API key.

## How to help
- One step at a time. Say exactly what to click, then wait for them to say done.
- Ask for a screenshot if they get stuck. Screens change, so go by what you can see rather than these words.
- Never ask them to paste a password, card number or API key into this chat. Keys go straight into n8n.
- Start with the DEMO. Only offer the full build once the demo has worked.

## What's in the zip
- `nev-DEMO.json`: the one-click demo. Nothing gets bought.
- `nev.json`: the full build.
- This guide.

They need to unzip `nev.zip` on their computer first, so the two `.json` files can be imported into n8n.

## The demo (about 10 minutes)
1. **Get n8n.** Sign up for the free trial at https://n8n.io. They'll land on an empty canvas.
2. **Import.** Create a new workflow, click the three dots at the top right, choose **Import from File** and pick `nev-DEMO.json`.
3. **Make a Telegram bot.** In Telegram, message **@BotFather**, send `/newbot` and follow the prompts. It gives a token. Then message the new bot once and open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser. The number after `"chat":{"id":` is their chat id.
4. **Connect accounts.** Any node with a red warning needs one. Click it, then Credential, then Create new.
   - **Telegram**: the bot token. Every Telegram node then uses it.
   - **Anthropic**: an API key from https://console.anthropic.com (a few dollars of credit is plenty). The two nodes named `Claude: ...` use it.
5. **Settings.** Open the first node, **Nev settings**, and change `TELEGRAM_CHAT_ID` and `TELEGRAM_BOT_TOKEN` (the poll uses it). Optional: `ME`, `EX_NAME`, `HOME` (latitude and longitude, used to find the closest sauna) and `TIMEZONE`.
6. **Run it.** Click **Execute workflow**. Messages arrive in Telegram. Tap **Fix it** on the poll.

## The full build
Import `nev.json`. Everything lives in **Nev settings**, the node after the four triggers.

| Service | What it does | n8n credential |
|---|---|---|
| Telegram bot | every message and the poll | Telegram API |
| Anthropic | the report, the excuse, unknown merchants | Anthropic API |
| Browser Use | Uber Eats, Sportsbet, the sauna | Header Auth, `X-Browser-Use-API-Key` |
| Google Places | finds the closest sauna | Header Auth, `X-Goog-Api-Key` |
| Google Calendar | holds the sauna time | Google Calendar OAuth2 |
| Up Bank (optional) | card taps | Header Auth, `Authorization: Bearer <token>` |
| Twilio (optional) | the ex line, sending the excuse | Twilio API |

**The saved browser.** In Browser Use, create a profile and log in to Uber Eats and Sportsbet inside it themselves, with their saved address and card. Put its id in `BROWSER_PROFILE_ID`. Logins expire, so they may need to log in again every so often. Set the caps: `FOOD.capAud`, `CHEMIST.capAud`, `SAUNA_CAP_AUD` and `AGENT_MAX_USD`.

**Paste their own links.** `FOOD.url` and `CHEMIST.url` are the Uber Eats store pages. `SPORTSBET_LIMITS_URL` is the deposit limit page in their account.

**Turn it on.** Activate the workflow, then open each webhook node and copy its **Production URL**.

### Card taps (choose one)
**iPhone, any bank using Apple Pay.** Shortcuts app, Automation tab, new automation, **Wallet** (the transaction trigger), choose the card, Run Immediately. Add **Get Contents of URL**: the `Card tap (iPhone Wallet)` URL, method POST, JSON body with `merchant` set to the Merchant value and `amount` set to the Amount value.

**Up Bank.** Get a Personal Access Token at https://api.up.com.au, then register a webhook pointing at the `Card tap (Up Bank)` URL using Up's `POST /webhooks` API. You can write them the command.

### Wake-up
Shortcuts app, new automation, **Sleep**, then **Waking Up**, Run Immediately. Actions:
1. **Find Health Samples**: Steps, start date between today 12am and today 2am.
2. **Calculate Statistics**: Sum.
3. **Get Contents of URL**: the `Wake-up (iPhone Shortcut)` URL, POST, JSON body with `steps` set to the result.

Test it once by hand. iPhones lock health data while locked, so if the steps come through as 0, make the automation run when they first open an app in the morning. Not on iPhone? Anything that can send `steps` to that URL works, such as a Fitbit, Oura or Garmin integration.

### The ex line (optional)
Buy a Twilio number and set its voice webhook to the `Ex line` URL. Set `EX_NUMBER` to the real number, and save the Twilio number as the ex's contact. Calls connect as normal and get counted. Nothing is recorded. Android users can post to the same URL from an app like MacroDroid instead.

### Testing without spending
Set `FORCE_GO: true` only for a test, and set the caps to `0` first so the agents stop before paying. Set it back to `false` afterwards.

## Good to know
- Nev only orders after a bar tap after midnight AND `BIG_NIGHT_STEPS` between 12 and 2am, once per night.
- Tell them plainly: browser agents aren't a supported way to use Uber Eats or Sportsbet, and could get an account restricted.
- The agent must stop at CAPTCHAs, codes and security checks. Never help get around them.
- The Sportsbet agent may only lower a limit.
