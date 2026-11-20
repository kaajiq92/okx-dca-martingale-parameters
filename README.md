# OKX DCA Bot: Spot vs Futures Martingale, the nine parameters that set your risk, and what stopping the bot actually does (full BTC walkthrough)

OKX runs two DCA bots: Spot DCA (Martingale) for spot pairs and Futures DCA (Martingale) for perpetual contracts. Both scale into a losing position with safety orders and close the whole cycle at a take-profit target. The spot version buys coins you own outright; the futures version runs on margin, so it can also be liquidated.

What follows is the part that decides whether the bot helps or hurts you: which of the two fits your account, every field each setup screen asks for, the multiplier math that most people fill in wrong, a $25,000 BTCUSDT cycle step by step, and the one behavior almost everyone misreads when they press stop.

> **Quick fit check**
> - Buying spot, want lower entries, no liquidation risk → Spot DCA (Martingale)
> - Trading perpetuals, can manage margin, accept forced-liquidation risk → Futures DCA (Martingale)
>
> 👉 [Set up a DCA bot on OKX](https://okx.com/join/CASH20)

## No plan ladder, two product lines

There is no Starter/Pro/Enterprise split to compare here. Neither bot page lists a subscription price, and OKX's trading bot FAQ only notes that bot usage may incur fees on the underlying trades; it doesn't quote a rate for the bot itself. What changes your result is which bot you open and how you fill the fields, so that's the comparison worth making.

|  | Spot DCA (Martingale) | Futures DCA (Martingale) |
| --- | --- | --- |
| What it trades | Spot pairs you hold outright | Perpetual contracts with leverage |
| Add trigger | Price falls by your set percentage step | Price steps, optionally widened by a price steps multiplier |
| Size controls | Initial order and safety order amounts, max safety orders | Initial/safety order margin, amount multiplier, max safety orders |
| Take profit | Percentage gain per cycle | TP target per cycle, with the cycle ending once the order fully fills |
| Liquidation risk | None from the bot; you hold the asset | Yes, if margin falls below the maintenance requirement |
| Pressing stop | Cancels pending orders, sells at market, funds back to trading account | Same behavior, funds returned to your trading account |
| Setup entry | Trade → Trading bot → DCA Bots → Spot DCA (Martingale) | Web: Trade → Trading bots → Marketplace → Futures DCA. App: Trade → Trading bots → Futures DCA |
| Open it | [Start a Spot DCA bot](https://okx.com/join/CASH20) | [Start a Futures DCA bot](https://okx.com/join/CASH20) |

Both bots isolate your allocated funds once created. On futures this matters beyond bookkeeping: money committed to the bot stops counting toward the margin available for your other positions.

## DCA, recurring buy, Martingale: three terms, three different tools

OKX's own help center points out that DCA and recurring buys get used interchangeably and aren't the same thing:

> "Recurring buys imply buying a fixed amount of an asset at fixed intervals (daily, weekly or monthly), regardless of market movements."

A DCA bot does something else. It controls the buy price: orders trigger when price drops by a fixed percentage, and a sell triggers when the market recovers to your take-profit target. You are trading a price range, not a calendar.

Martingale is the sizing rule layered on top. After each loss, the next position gets bigger, so one profitable trade can cover the accumulated losses of the ones before it plus a margin. OKX builds both DCA bots on that rule, which is exactly where the risk lives.

## Spot DCA: how a cycle runs, and where it ends

A complete cycle has two required parts: an initial order and a take-profit order. The bot opens with the initial buy. If price drops by your set step, it adds a safety order sized as a multiple of the first. That repeats until it hits the maximum order count, the take-profit level, or the stop level. Reach the target and the cycle closes, then the bot starts a new one.

A concrete take-profit example from OKX's documentation: set a 10% target with an average position cost of 1,000 USDT. When price reaches 1,100 USDT, the cycle ends.

The stop loss is calculated as:

**Stop loss price = initial order average filled price × (1 − stop loss target)**

Once that price triggers, the entire strategy ends and the bot will not open a new cycle on its own. This is the difference between a stop and a pause, and it's worth reading twice if you're used to grid bots that keep running.

### Parameters you actually set

Manual setup asks for the price step percentage, the take-profit target per cycle, the initial and safety order amounts, and the maximum number of safety orders. Newer help pages describe the same two funding fields as initial order margin and safety order margin, so don't be surprised if the wording shifts between the web app and a help article.

OKX also reserves only the minimum at creation (initial order plus the first safety order) if you prefer not to lock up the whole budget upfront, with the rest transferred later as needed.

### Start conditions and the AI presets

Two ways to begin. Instant opens a new cycle immediately on creation and again after each completed cycle. The alternative is a technical trigger such as RSI, so entries wait for a signal instead of firing on schedule.

If you'd rather not choose numbers yourself, the AI Strategy path offers three risk profiles: conservative, moderate and aggressive. The recommended parameters combine backtested results with characteristics of the specific token, such as historical volatility, for each trading pair.

### Creating one

On web: hover Trade → Trading bot → DCA Bots → Spot DCA (Martingale). Choose AI Strategy or Manual, enter your amount or your parameters, pick Instant or an indicator trigger, confirm the order, then find the position under Bots → DCA on the Trading bot home screen.

## Futures DCA: the nine fields, explained

There are four ways in. Set myself for full manual control. Set myself - Auto fill to load the bot's backtested recommendation into editable fields. AI strategy for parameters based on a weekly backtest of that specific pair. Lead bots to copy a lead trader's running bot from the Marketplace in one click.

Then come the nine parameters:

1. **Price steps**: the percentage difference between two consecutive safety orders.
2. **TP target per cycle**: when the TP price triggers and the order fills completely, the cycle ends and the next one starts as configured.
3. **Initial order margin**: the first order of each cycle.
4. **Safety order margin**: the incremental adds that lower your average cost and pull the take-profit closer.
5. **Max safety orders**: the ceiling per cycle. The actual number placed can be lower, depending on your margin.
6. **Auto-transfer margin**: when margin level hits a high-risk threshold, funds move automatically from your trading account into the bot to avoid forced liquidation. You set the maximum transferable amount.
7. **Amount multiplier**: applied to the size of the next safety order.
8. **Price steps multiplier**: applied to the gap before the next safety order.
9. **Stop condition**: End of cycle stops the bot after the first cycle finishes; Price stops it once the last price hits your trigger.

### The two multipliers, with OKX's own numbers

These two fields are where setups go wrong, because both are exponential and they compound differently.

Amount multiplier, with a 100 USDT safety order and multiplier 2:

- 100 = 2⁰ × 100
- 200 = 2¹ × 100
- 400 = 2² × 100

Price steps multiplier, with a 1% price step and multiplier 2:

- 1% = (0 + 1 × 2⁰)%
- 3% = (1 + 1 × 2¹)%
- 7% = (3 + 1 × 2²)%

The first multiplies your money at risk. The second multiplies the distance the market has to travel before the next add. A multiplier of 2 on both means the third safety order is four times the first and sits 7% below the initial entry, not 3%.

### A $25,000 BTCUSDT cycle, step by step

Long BTCUSDT perpetual, current price $25,000, 5x leverage, price steps 2%, TP target 5% per cycle.

| Contract price | Step / target | Average price or result |
| --- | --- | --- |
| $25,000 | Initial order | $25,000 |
| $24,500 | −2% (Safety order #1) | $24,750 |
| $24,000 | −4% (Safety order #2) | $24,500 |
| $26,250 | +5% (Take profit) | $26,250 − $24,500 = **$1,750** |

The bot buys at $25,000. A 2% drop to $24,500 triggers a second buy, taking the average entry to $24,750. Another drop to $24,000 adds a third, average $24,500. At $26,250 the take-profit order sells all three and the bot opens a new cycle.

One detail worth checking on the screen before you confirm: in this futures example OKX calculates the TP price from the initial order price ($25,000 → $26,250), while the spot example measures the 10% target against average position cost. The two documents don't describe the reference point identically, so read which basis your order confirmation shows.

## What goes wrong

The math above looks tidy because the price came back. Run it forward instead.

Your safety orders grow geometrically. A run of adds pushes the amount committed from a modest initial order to a large number within a handful of trades, and a market that keeps falling in one direction keeps pulling more in. OKX states the risk plainly in its own documentation: with continued doubling, the probability of loss is effectively open-ended, and a trader who runs out of funds and exits mid-strategy can face a substantial loss.

There's a subtler outcome too. After several doubled losses, the eventual winning cycle may do nothing more than bring you back to break even, which means you carried escalating risk for no net gain.

Leverage adds a second layer. High leverage amplifies losses when the market moves against you, and OKX reserves the right, at its discretion, to close one, several, or all of your positions. If your margin balance drops below the maintenance requirement, the position can be liquidated, potentially wiping out the funds you allocated to the bot.

Auto-transfer margin deserves its own warning. It pulls money from your trading account automatically to defend the position. Set the maximum transfer amount deliberately, because the default behavior protects the bot with funds you may have earmarked for something else.

The scenario these bots are built for is a market that moves sharply but briefly, or chops sideways with short rebounds and pullbacks. A sustained trend against your direction is the scenario that drains the account.

👉 [Create a Futures DCA bot with your own parameters](https://okx.com/join/CASH20)

## After you hit create: stop, end cycle, and the two options people confuse

The most misunderstood behavior first. Stopping a bot cancels all pending orders and sells your position at market price, with the proceeds returned to your trading account. It does not leave your position open for later. If you stop during a drawdown, you realize that drawdown at whatever the market gives you.

End cycle is a different action. It closes the current cycle without stopping the bot, so the next cycle can still start. Use it when a cycle has gone stale but you want the strategy to continue.

The rest of the dashboard:

- **Add manual orders**: once every safety order has filled, you can still place additional manual buys to push the average cost lower.
- **Edit cycle take profit**: change the TP target of the cycle currently running, from the bot detail page.
- **Adjust margin**: move extra funds in ahead of a volatile event rather than waiting for a margin call.
- **Replicate parameters**: recreate the same setup in one click, useful when you want a second bot on a different pair with identical logic.
- **Details**: the per-bot page with orders and cycle history.

You manage all of this from the Trading Bots homepage under My bots, on both web and app.

## Which bot for which trader

If you want to accumulate spot on dips and never face a liquidation, use Spot DCA with a small price step and a cap on max safety orders. You keep the coins, and the worst case is holding an asset that fell.

If you already trade perpetuals, read margin levels without prompting, and accept that a bad sequence can liquidate you, Futures DCA gives you the automated version. Set the stop condition to End of cycle or to a price trigger before you start, and set the auto-transfer cap at a number you're willing to lose.

If you're not sure yet, run the AI strategy on the conservative profile with the minimum capital required, and let one full cycle complete before adding anything.

Two things to confirm before you press create. First, total capital at risk isn't the initial order; add up the initial order plus every safety order with your multiplier applied, and check that number against your balance. Second, remember that the actual number of safety orders placed depends on your margin, so a plan that assumes all of them fill needs enough headroom to fund them.

👉 [Start your first OKX DCA bot with a small test size](https://okx.com/join/CASH20)
