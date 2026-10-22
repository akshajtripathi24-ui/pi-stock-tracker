# 7. Using the website

A tour of every page and button.

> Prices come from Nasdaq, Yahoo Finance and Finnhub and may be delayed. Everything shows **past** performance only. It's not investment advice.

---

## The top bar

| Item | What it does |
|---|---|
| **📈 Pi Stock Tracker** | Goes back to the Market page |
| **Market** | Overview of today's market |
| **Screener** | All US stocks in one table |
| **★ Favorites** | The stocks you've starred |
| **🔔 Alerts** | Phone notification setup and your alerts |
| **Search box** | Find any stock by name, ticker or CEO |
| **● LIVE badge** | Whether live prices are on. See [3. Live prices](03-live-prices.md). |
| **◐** | Switches between light and dark mode. Each device remembers its own choice. |

### The grey status line

Under the top bar, a line like this:

```
6,999 US stocks · prices updated 2026-10-22 11:03 · loading price history 6,305/6,999
```

- **US stocks:** how many stocks the Pi knows about
- **prices updated:** when the full list of prices was last downloaded (every minute)
- **loading price history x/y:** the first time, the Pi downloads a year of history for each stock. When finished it says **price history ready**.
- **company profiles unavailable:** shown if Yahoo blocks the company info download. Everything else still works.

---

## Market page

### Index cards

Four cards at the top: **S&P 500**, **Nasdaq 100**, **Dow Jones** and **Russell 2000**. Each shows:
- the current value and today's % change (green up, red down)
- a small chart of the last month
- a **pulsing green dot** when it's moving live (needs a Finnhub key)

They refresh every 3 seconds.

### Top & worst performers

The 10 biggest **gainers** (left, green) and **losers** (right, red).

- **1D · 1W · 1M · 6M · YTD · 1Y** buttons choose the time period. YTD means "year to date", since the start of this year.
- The **size** dropdown filters by company size (market cap):

  | Option | Companies worth… |
  |---|---|
  | All sizes | anything |
  | Small+ | more than $300 million |
  | Mid+ (default) | more than $2 billion |
  | Large | more than $10 billion |
  | Mega | more than $200 billion |

  Tiny companies often jump 50% or more on very little news, so **Mid+** keeps the lists more meaningful.
- With live prices on, the 1D lists re-sort themselves as prices change.
- Click any stock to open its page.

### Most active today

The 10 stocks with the most shares traded today (companies over $300 million).

### Sectors today

How each sector (Technology, Health Care, Finance…) did today. Bigger companies count more, the same way the big indexes work. Green bars go right, red bars go left. **Click a sector** to open the Screener filtered to it.

---

## Screener

Every US-listed stock in one table.

**Filters (at the top):**
- **Filter by name or ticker…:** type part of a name or ticker
- **All sectors:** pick one sector
- **Size:** same options as on the Market page

**Columns:**

| Column | Meaning |
|---|---|
| ☆ | Click to add to or remove from Favorites |
| Stock | Ticker and company name |
| Price | Latest price (live if streaming) |
| 1D | Change today |
| 1W, 1M, 6M, YTD, 1Y | Change over the last week, month, 6 months, this year, and year |
| Mkt cap | Company value (B = billion, T = trillion) |
| Volume | Shares traded today |
| Sector | Industry group |

- **Click a column name** to sort by it. Click again to reverse the order. An arrow (↓ or ↑) shows which column is sorted.
- **← Prev / Next →** move through the results 50 at a time.
- **Click a row** to open that stock.
- The screener remembers your filters and sorting on each device.

`–` means there's no number yet, for example while the Pi is still downloading history.

---

## Search

Type in the search box at the top of any page:
- a **ticker**: `AAPL`
- part of a **company name**: `coca`
- part of a **CEO's name**: `cook` (3 or more letters; works once the Pi has downloaded that company's profile)

Up to 15 results appear, biggest companies first, with price and today's change. Click one, or use the **↑ ↓** arrow keys and **Enter**. **Esc** closes the list.

---

## Stock page

Opens when you click any stock.

| Part | What it shows |
|---|---|
| **Header** | Company name, ☆ to favorite, ticker · sector · industry |
| **Big price** | Latest price, flashing green or red when it changes live, plus today's change in $ and % |
| **Chart** | Pick **1D · 5D · 1M · 6M · YTD · 1Y · 5Y · Max**. The default is 1Y. Hover or touch the chart to see the price at any point. Green if it went up over that range, red if down. On **1D** with live prices, the chart grows as trades come in. |
| **🔔 Price alerts** | Add alerts for this stock. See [4. Price alerts](04-price-alerts.md). |
| **Performance** | Change over 1D, 1W, 1M, 6M, YTD and 1Y |
| **Key stats** | Market cap, volume today, 52-week high and low (from daily closing prices), country, IPO year |
| **About** | CEO, employees, headquarters, website and a short description. If it says "not loaded yet", the Pi hasn't downloaded it; try again later. |
| **← Back** | Back to where you were |

---

## Favorites

Shows every stock you've starred (☆ → ★), in the same table as the Screener. Click a column name to sort.

Favorites are saved **in this browser** on this device. Other devices have their own favorites list. See [6. Use it on more devices](06-more-devices.md).

---

## Alerts

Set up phone notifications, see all your alerts, and see recent notifications. See [4. Price alerts](04-price-alerts.md) for the full guide.

---

## Tips

- **Ctrl + F5** (or pull down to refresh on a phone) reloads the page fully, for example after an update.
- Prices only move during US market hours: 9:30am–4pm Eastern, Monday–Friday.
- If a stock's numbers look odd, compare with another site like Google Finance. Data sources sometimes disagree, especially for small stocks.

Next: [8. Cooling fan →](08-cooling-fan.md)
