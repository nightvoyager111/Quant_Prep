Two things you might mean — here's both, quickly.

## If you mean "my project is the RL agent, why are you talking about TWAP/VWAP/AC?"

You did choose RL only. The baselines aren't a second project — they're the ruler you measure the RL agent with.

Here's the problem without them. You train PPO, it finishes, and it reports: **4.2 basis points of implementation shortfall.** Is that good? You have no idea. There's no natural zero in execution cost. The only way the number means anything is next to another number:

| Strategy | IS (bps) |
|---|---|
| TWAP | 6.1 |
| VWAP | 5.4 |
| Almgren-Chriss | 5.0 |
| **PPO agent** | **4.2** |

Now "4.2" is a result. Without the three rows above it, it's a number with no story.

Three more reasons they earn their place:

- **They're nearly free.** TWAP is ~10 lines. VWAP is a weighted split. AC is one closed-form formula. All three are already written and tested in your repo — maybe 100 lines total against ~1,100. Rounding error on effort, huge payoff in interpretability.
- **They're your debugging floor during training.** If PPO can't beat *equal slices on a timer*, the bug is in your environment or reward, not your hyperparameters. That's an enormously useful signal to have before you burn a million timesteps.
- **They're what the desk actually sells.** Scotiabank EES offers clients TWAP and VWAP algos. Walking in able to discuss why AC's schedule is front-loaded, or when VWAP's volume curve breaks down, is on-topic for the job in a way the RL part isn't yet.

So the framing stays: **the RL agent is the deliverable, the baselines are the control group.** Any paper on an RL execution agent has the same table.

## If you mean the button you clicked

In [the original session](#local_539a8679-757c-458c-9435-2c7eb980fb55) you picked **"Wire baselines E2E"** — that builds the pipeline from schedule → engine → IS distribution so you get actual numbers out. It isn't RL training.

That ordering is deliberate, though: that pipeline is the same machinery the RL agent runs through. The env calls the same engine and the same `implementation_shortfall_bps`. So wiring baselines end-to-end isn't a detour before RL — it's the plumbing RL needs, plus the comparison table, in one step. And it's what gets you real numbers to quote this week, which training a PPO agent will not.

If you'd rather that session skip ahead to PPO training instead, say the word and I'll pass it along.



No — PPO is the more impressive result. It's just **downstream** of the baselines, and riskier. Let me separate those.

## 1. It isn't a preference, it's a dependency

PPO can't train until the plumbing exists. Look at what `ExecutionEnv` needs to run even one episode:

- an `event_stream_factory` that yields a real day of events → **doesn't exist yet**
- an arrival price snapshot → needs that stream
- `implementation_shortfall_bps` called on real fills → needs the engine driven over real data
- episode slicing — how many events or seconds per slice → currently a hardcoded `for _ in range(200)`, flagged `TODO`

Every one of those is the same machinery a baseline run needs. "Wire baselines end-to-end" isn't an alternative to RL — it's roughly 80% of the work required before `model.learn()` can be called at all. The baseline run is just the cheapest thing to point that plumbing at first, because a TWAP schedule is a fixed array and can't itself be buggy.

So the order isn't *baselines instead of RL*. It's *build the pipeline, prove it with a strategy that can't fail, then swap the strategy for a learner.*

## 2. Risk per hour spent

| | Baseline pipeline | PPO training |
|---|---|---|
| Time | hours | days to weeks |
| Can it fail? | No — TWAP always produces a schedule | Yes, routinely |
| Output if it works | A cost distribution + comparison table | One more row in that table |
| Output if it doesn't | n/a | Nothing quotable |

RL on sparse terminal rewards with a 5-dim state is genuinely finicky. PPO may converge to "always passive" and never fill, or "always cross" and eat the spread. Diagnosing that *without* a baseline number to compare against is miserable — you can't tell a broken reward from a weak policy.

## 3. The interview angle specifically

Your interview is soon. Consider what each outcome lets you say:

**With baselines wired:** "I ran 200 episodes across five symbols. TWAP costs 6.1 bps, Almgren-Chriss 5.0. When I turn queue modeling off, both drop by 2 bps — so a naive fill model would have overstated my passive strategy by 40%." That ablation is a real finding, and it's yours.

**With a half-trained PPO:** "I trained an agent, it got 5.8 bps." *"Versus what?"* "...I haven't run the baselines yet."

The second one is worse than saying nothing. And remember the resume problem from earlier — "training an RL agent" and "validated on live paper trading" are both currently unbacked. A rushed PPO run doesn't fix that; it just gives an interviewer a trained model to ask hard questions about.

## 4. One thing you need to know either way

**There's no market data on this machine.** No `data/` directory, no LOBSTER CSVs, nothing in Downloads — I just checked. `.gitignore` excludes `data/raw/` and `data/processed/`, so nothing was committed either.

That blocks *both* paths equally. Two ways through:

- **LOBSTER free samples** — lobsterdata.com publishes a handful of sample symbol-days (AAPL, MSFT, INTC among them) at no cost. Real NASDAQ messages, real queue dynamics. This is what you want for quotable numbers.
- **A synthetic event generator** — a Poisson order-flow process I can write in ~80 lines. It gets the pipeline running and tested today, and it's honest as long as you label the numbers synthetic. Not quotable as "I measured this on NASDAQ data."

Ideally both: synthetic to build and test against, real samples to produce the numbers you actually cite.

## So, to answer directly

PPO is the more desirable *result*. The baseline pipeline is the more desirable *next action* — because it's a prerequisite, it can't fail, it produces quotable output in hours, and it's what makes the PPO number mean something when you do train it.

If you want to flip the priority anyway, that's a legitimate call — just know you'd be building the same pipeline first regardless, only without a comparison row at the end of it.