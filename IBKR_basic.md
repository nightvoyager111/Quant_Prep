1. What a broker does

A broker is the intermediary between you and an exchange (NYSE, NASDAQ, CME, etc.). Exchanges don't let individuals connect directly — brokers do four jobs:

- Route orders to exchanges (or internalize them).
- Clear & settle — make sure shares/cash actually change hands (typically T+1 for US stocks: settles one business day later).
- Custody your assets and cash.
- Provide market data, statements, tax forms, and margin (lending).

IBKR is unusual among retail brokers because it exposes a serious API and gives you direct market access across many asset classes. That's why it's the default for algo/RL projects.

2. Bid / Ask / Spread

At any instant, an exchange's order book has:

- Bid — highest price someone is willing to buy at.
- Ask (or offer) — lowest price someone is willing to sell at.
- Spread = ask − bid. Narrow = liquid (AAPL ~$0.01). Wide = illiquid.
- Last — price of the most recent trade (can be different from current bid/ask).

Example: AAPL bid $180.00 / ask $180.02.
- If you buy market now, you pay $180.02 (you "cross the spread").
- If you sell market now, you receive $180.00.
- That $0.02 is your implicit cost on a round trip. On illiquid names this dominates commissions.

Mid price = (bid+ask)/2, often used as a reference "fair" price in sims.

3. Order types

The three you'll use 95% of the time:

Market order

"Fill me right now at whatever price." Guarantees execution, not price. Dangerous in thin markets — can slip badly.

Limit order

"Buy at $180.00 or better (lower); sell at $180.00 or better (higher)." Guarantees price, not execution. If the market never reaches your limit, you don't trade.
- A buy limit above the ask → acts like a market order (fills immediately at ask).
- A buy limit below the bid → sits on the book as a "resting" order until matched or cancelled.

Stop order (aka stop-loss)

A trigger, not a live order. "If price touches $175, then send a market order to sell."
- Stop-market: when triggered, becomes a market order. Guaranteed exit, unknown price.
- Stop-limit: when triggered, becomes a limit order. Known price floor, but might not fill in a fast crash.

A few others worth knowing by name

- Marketable limit: a limit priced to cross immediately — common safer substitute for market orders.
- GTC / DAY: time-in-force. GTC = good-till-cancelled; DAY = cancelled at close.
- IOC / FOK: immediate-or-cancel / fill-or-kill (algo-friendly).

4. Positions

A position is what you currently hold in a given instrument.

- Long 100 AAPL @ avg $180 — you own 100 shares; profit if price rises.
- Short 100 AAPL @ avg $180 — you borrowed and sold 100 shares; profit if price falls. Requires margin + a borrow.
- Flat — no position.

Positions have an average entry price (weighted avg of your fills) and a current market value (shares × last price).

5. P&L (Profit & Loss)

Two flavors — this trips people up:

- Unrealized P&L = (current price − avg entry) × shares. Paper gain/loss on open positions; moves tick by tick.
- Realized P&L = locked in once you close (fully or partly). This is what gets taxed.

Total P&L = realized + unrealized − fees − financing costs.

In an RL/sim context, your reward function usually tracks mark-to-market equity (= cash + unrealized value of positions), because it updates every step, not only on closes.

6. Margin basics

Margin = buying with borrowed money (or shorting, which requires borrowing shares).

Key concepts:
- Cash account: no borrowing. You can only buy with settled cash. Simplest.
- Margin account: broker lends you money against your assets. Lets you leverage and short.
- Initial margin: % of a position's value you must put up to open it (US stocks: typically 50% under Reg T; futures: much smaller, like 5-15%).
- Maintenance margin: minimum equity % you must keep. Falling below triggers a margin call — broker demands cash or liquidates positions.
- Leverage = position size / your equity. 2× leverage doubles both gains and losses.
- Financing cost: borrowed cash accrues interest daily. Short positions pay a borrow fee.

IBKR offers two margin types: Reg T (traditional) and Portfolio Margin (risk-based, higher leverage, requires $110K+ equity on live). Paper accounts simulate both.

For paper trading / learning: stick to cash-like behavior at first — small positions, no shorting, no leverage. Once you understand fills and P&L, enable margin to see how it changes things.

---