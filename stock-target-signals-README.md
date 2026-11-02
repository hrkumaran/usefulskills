# Stock Target Signals

A skill that reads your stock list (with target prices) and tells you, for each Indian (NSE/BSE) stock, whether the signal is **Buy**, **Hold** or **Sell**, with a one-line reason.

> This gives data-based signals to help your own decision. It is not financial advice. Markets move, so check with a registered advisor for big decisions.

---

## What it does

1. Reads your Excel (.xlsx) or CSV file.
2. Gets the current price of each stock.
3. Checks recent news for each stock (results, rating changes, regulatory news, and so on).
4. Compares the current price with your target price and adjusts for the news.
5. Shows a table: Stock, Current price, Target, Gap to target, Signal, Reason.

---

## Input file format

Use a CSV or Excel file with these columns. Names can vary, but the meaning must match.

| Column | Required | Meaning |
|---|---|---|
| Stock | Yes | Company name or NSE symbol (e.g. TCS, INFY) |
| Target Price | Yes | Price at which you want to sell |
| Current Price | Recommended | Today's price. Add this for the most reliable result |
| Buy Price | Optional | What you paid |
| Quantity | Optional | Number of shares |

Example (`my_stocks.csv`):

```csv
Stock,Buy Price,Target Price,Current Price,Quantity
Reliance Industries,1250,1700,1450,10
TCS,3400,4500,3900,5
Infosys,1500,1900,1650,20
```

**Why Current Price is recommended:** live prices cannot always be fetched reliably, and the skill will never guess a price. If it cannot confirm a fresh, dated price, it marks the row "Price not found" and asks you for today's price.

---

## How the signal is decided

Gap % = (Target - Current) / Current x 100

| Gap to target | Starting signal |
|---|---|
| Target reached or price above target | **Sell** |
| Within about 5% below target | **Hold / Near target** |
| 5% to 25% below target | **Hold** |
| More than 25% below target | **Buy / Accumulate** |

Then news adjusts it:
- Strong positive news can soften a Sell to Hold, or strengthen a Buy.
- Serious negative news (fraud, big profit fall, downgrades) can downgrade a Buy to "Hold / Review".

You can change these numbers. Edit the table in `SKILL.md`, or state your own rule (for example, "sell when 10% below target").

---

## Install in IntelliJ

These steps assume you use Claude Code inside IntelliJ (the Claude Code plugin or terminal). Skills for Claude Code are folders containing a `SKILL.md` file.

1. In your IntelliJ project, create this folder structure:

   ```
   your-project/
   └── .claude/
       └── skills/
           └── stock-target-signals/
               └── SKILL.md
   ```

   (To use it in all projects instead, put the folder in `~/.claude/skills/stock-target-signals/`.)

2. Copy the full text from the **SKILL.md content** section below into `SKILL.md`. Keep the `---` lines at the top; they are required.
3. Put your stock file (for example `my_stocks.csv`) inside the project.
4. Restart the Claude Code session so it picks up the new skill.

If you use a different AI assistant in IntelliJ that does not support skills, paste the SKILL.md instructions into your chat first, then attach or paste your stock list.

---

## How to use

Open Claude in IntelliJ and say something like:

- "Check my stocks in my_stocks.csv against my targets"
- "Which of my stocks should I buy, sell or hold? File: my_stocks.csv"

You get a table with a signal for each stock. If any price is missing, reply with today's prices and it will finish the table.

---

## Known limits

- Live price lookup depends on websites that can be reached from your environment. Some are blocked or show old data. Providing a Current Price column avoids this.
- Signals use a simple rule plus recent news. They do not model your tax, holding period, or full portfolio.
- It covers Indian (NSE/BSE) stocks.

---

## SKILL.md content

Copy everything inside the box below into `.claude/skills/stock-target-signals/SKILL.md`.

````markdown
---
name: stock-target-signals
description: Reads the user's uploaded stock list (Excel or CSV with stock names and target prices) and gives a Buy / Hold / Sell signal for each Indian (NSE/BSE) stock by comparing the current price with the target price and checking recent news. Use this whenever the user shares their stock holdings, portfolio, watchlist or target prices and asks whether to buy, sell or hold, whether targets are hit, or how their stocks are doing, even if they don't say "skill" or "signal".
---

# Stock Target Signals

Turn a list of stocks with target prices into a clear table that says, for each stock, whether the signal is Buy, Hold or Sell, and why. The user owns Indian stocks (NSE/BSE, prices in rupees) and wants a quick, honest read on where each one stands against its target.

You are a helper for the user's own decision, not a financial advisor. Say so once, briefly, at the end. Do not state a signal as a guarantee; markets move, and the signal is only as good as the data behind it.

## Step 1: Read the file

The user uploads an Excel (.xlsx) or CSV file. Read it with the file tools (pandas works well for both).

Look for these columns. Names vary, so match by meaning, not exact spelling:
- **Stock**: company name or NSE/BSE symbol (e.g. "Reliance", "TCS", "INFY")
- **Target price**: the price at which the user wants to sell, or the price they believe it will reach
- **Buy price** (optional): what they paid
- **Quantity** (optional): number of shares
- **Current price** (optional): if the user's file already has today's price, use it and skip the price lookup in Step 2 for that stock

If the target price column is missing, or a row has no target, do not guess one. Mark that row "No target given" and ask the user at the end if they want to add targets.

If the stock name is ambiguous (two companies with similar names), pick the most likely listed company and mention the assumption in the Notes column.

## Step 2: Get current prices and recent news

**Prices.** If the file has no current price, search for each stock's latest price on NSE (BSE if NSE isn't available), one stock at a time. Search results usually give only page links and titles, not the price itself, so open the most promising quote page from the results (finance sites such as Upstox, ICICIdirect, Equitymaster or Business Standard) and read the price from it. Some sites are not reachable from this environment, and some pages are just landing pages without quote data. Do not retry the same blocked page; move to another result.

A price is usable only if you can see both the number and the date it belongs to, and the date is today or the last trading day. Search results often surface pages from weeks or months ago, and a signal built on an old price is worse than no signal, because the user may act on it. If you cannot confirm a fresh price for a stock, do not estimate it and do not fall back on memory. Instead, mark that row "Price not found" and, at the end, ask the user to reply with today's prices (or add a Current Price column to the file). Once they do, finish the signals using those prices. Prices the user gives you are trusted; note them as "price provided by you".

**News.** Then search for recent news on each stock (last few weeks): earnings results, management changes, regulatory action, big orders, analyst rating changes, sector trends. News matters because a stock near its target with strong positive news may deserve more patience, while a stock far below target with serious bad news may not be a bargain.

## Step 3: Decide the signal

Compute how far the current price is from the target:

`gap % = (target - current) / current * 100`

Use this as the starting point, then adjust for news:

| Gap to target | Starting signal |
|---|---|
| Target already reached or price above target | **Sell** (target hit; consider booking profit) |
| Within about 5% below target | **Hold / Near target** (close to target, decide whether to book profit) |
| 5% to 25% below target | **Hold** |
| More than 25% below target | **Buy / Accumulate** (large room to target) |

Then adjust with news:
- Strong positive news can move a "Sell" to "Hold" (target may be too low) or strengthen a "Buy".
- Serious negative news (fraud, big profit fall, regulatory trouble, rating downgrades) can downgrade a "Buy" to "Hold / Review" because a low price may be low for a reason.
- If news is mixed or there is none, keep the starting signal.

These bands are a simple rule of thumb. If the user gives their own rule (for example, "sell when 10% below target"), use theirs instead.

## Step 4: Present the result

Show a table in the chat, one row per stock, ordered with Sell first, then Buy, then Hold, so the most actionable ones are at the top:

| Stock | Current price | Target | Gap to target | Signal | Reason (one line) |
|---|---|---|---|---|---|

Rules for the Reason column: one short line combining the price gap and the key news point. Plain words, no jargon. If you used the buy price and quantity, mention profit or loss only if the user would find it useful (for example, a Sell signal with a large gain).

Below the table, add:
1. Price date and time ("Prices as of ..."), so the user knows how fresh the data is.
2. Any rows you could not complete and why.
3. One short sentence: this is a data-based signal to help your own decision, not financial advice, and checking with a registered advisor is wise for big decisions.

Keep the whole answer short and easy to scan. Do not write a long essay about each stock unless the user asks.

## Sources

End with a short "Sources:" list of the pages used for prices and news, as markdown links, so the user can verify.
````
