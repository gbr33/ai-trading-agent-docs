# AI TRADING AGENT

## Master Project Blueprint & Engineering Specification

**Project Type:** AI-assisted quantitative intraday trading system
**Primary Objective:** Build, test, validate, and eventually operate an 
autonomous intraday trading system in which AI may propose trading 
decisions, but deterministic software has absolute authority over whether 
any trade can actually occur.

**Development Principle:**

> **Simulation first → Historical validation → Paper broker → Sustained 
paper qualification → Restricted live trading**

No broker execution is considered production-ready until the simulation 
and historical engines have passed their qualification gates.

**Revision note: Part I (Sections 1–73) is the core architecture. Part II 
(Sections 74–82) is a mandatory pre-live addendum. Both are authoritative.

---

# 1. PROJECT MISSION

The system shall automate the complete lifecycle of an intraday trade:

```text
MARKET DATA
     ↓
DATA QUALITY
     ↓
TECHNICAL FEATURES
     ↓
MARKET REGIME
     ↓
OPPORTUNITY SCANNER
     ↓
NEWS / SENTIMENT
     ↓
AI ANALYSIS
     ↓
DETERMINISTIC VALIDATION
     ↓
RISK ENGINE
     ↓
PORTFOLIO ENGINE
     ↓
EXECUTION AUTHORIZATION
     ↓
SIMULATED / PAPER / LIVE BROKER
     ↓
ORDER LIFECYCLE
     ↓
POSITION MONITORING
     ↓
EXIT MANAGEMENT
     ↓
FLATTENING
     ↓
RECONCILIATION
     ↓
JOURNAL / AUDIT
     ↓
PERFORMANCE ANALYSIS
     ↓
RESEARCH / LEARNING
```

The system must be capable of operating the same decision process in three 
environments:

```text
SIMULATION
    ↓
BROKER PAPER
    ↓
LIVE
```

The decision logic should not be rewritten when moving between these 
environments.

Only the execution adapter should change.

---

# 2. CORE DESIGN PRINCIPLE

The most important rule in the entire project is:

```text
AI PROPOSES
     ↓
SYSTEM VALIDATES
     ↓
RISK AUTHORIZES
     ↓
PORTFOLIO AUTHORIZES
     ↓
EXECUTION OCCURS
```

Never:

```text
AI → BROKER
```

The AI must never have direct authority to:

* bypass risk limits;
* increase position size;
* disable a stop;
* override the daily loss limit;
* bypass portfolio exposure limits;
* trade when the market is closed;
* trade stale data;
* trade after the entry cutoff;
* ignore a kill switch;
* modify deterministic safety rules;
* directly submit broker orders.

If the AI says:

```text
BUY
```

but the deterministic system says:

```text
REJECT
```

the final result is:

```text
NO TRADE
```

This is the fundamental safety architecture.

---

# 3. SYSTEM MODES

The project has three operational modes.

## 3.1 SIMULATION

No broker connection.

The system receives:

* historical market data;
* historical news;
* historical corporate/event information;
* simulated account state.

It produces:

* opportunities;
* AI decisions;
* risk decisions;
* simulated orders;
* simulated fills;
* positions;
* P&L;
* complete audit records.

This is the primary development environment.

---

# 3.2 BROKER PAPER

The system connects to the broker's paper environment.

The strategy and safety pipeline remain unchanged.

Only the execution layer changes:

```text
SimulationBroker
       ↓
PaperBroker
```

The purpose is to discover problems that historical simulation cannot 
reveal:

* connection failures;
* broker order rejection;
* real order state transitions;
* partial fills;
* latency;
* market-data discrepancies;
* reconciliation problems;
* disconnect/reconnect behavior;
* real session behavior.

---

# 3.3 LIVE

Live execution is the final stage.

It is disabled until all qualification gates have passed.

Live mode must require explicit configuration.

It must never accidentally activate because a developer forgot to change a 
setting.

---

# 4. AUTHORITATIVE SYSTEM ARCHITECTURE

```text
                         
┌─────────────────────────────┐
                         │       MARKET DATA           │
                         │                             │
                         │ Historical / Paper / Live  │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │      DATA QUALITY           │
                         │                             │
                         │ Timestamp                   │
                         │ Missing data                │
                         │ Duplicates                  │
                         │ Price validity              │
                         │ Volume validity             │
                         │ Session validity            │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │     FEATURE ENGINE          │
                         │                             │
                         │ OHLCV                       │
                         │ RSI                         │
                         │ RVOL                        │
                         │ ATR                         │
                         │ Momentum                    │
                         │ Volatility                  │
                         │ Trend                       │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │      REGIME ENGINE          │
                         │                             │
                         │ Trend                       │
                         │ Volatility                  │
                         │ Market condition            │
                         │ Risk environment            │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │     OPPORTUNITY SCANNER      │
                         │                             │
                         │ Momentum                    │
                         │ RVOL                        │
                         │ RSI                         │
                         │ Breakout                    │
                         │ Liquidity                   │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │      NEWS ENGINE             │
                         │                             │
                         │ News                        │
                         │ Sentiment                   │
                         │ Event context               │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                         
┌─────────────────────────────┐
                         │          AI ENGINE           │
                         │                             │
                         │ Candidate analysis           │
                         │ Context interpretation       │
                         │ Confidence                   │
                         │ Reasoning                    │
                         │ Risk flags                   │
                         
└──────────────┬──────────────┘
                                        │
                                        ▼
                 
┌────────────────────────────────────────────┐
                 │        DETERMINISTIC DECISION FIREWALL     │
                 │                                            │
                 │ AI output validation                        │
                 │ Confidence                                  │
                 │ Opportunity quality                          │
                 │ AI risk flags                               │
                 │ Data freshness                               │
                 │ Session state                                │
                 
└────────────────────┬───────────────────────┘
                                      │
                                      ▼
                 
┌────────────────────────────────────────────┐
                 │              RISK ENGINE                    │
                 │                                            │
                 │ Daily loss                                  │
                 │ Position limits                             │
                 │ Trade limits                                │
                 │ Risk/trade                                  │
                 │ Stop distance                               │
                 │ Position sizing                             │
                 │ Exposure                                    │
                 
└────────────────────┬───────────────────────┘
                                      │
                                      ▼
                 
┌────────────────────────────────────────────┐
                 │             PORTFOLIO ENGINE                 │
                 │                                            │
                 │ Existing positions                           │
                 │ Total exposure                               │
                 │ Sector exposure                              │
                 │ Correlation                                 │
                 │ Available capital                            │
                 │ Pending orders                               │
                 
└────────────────────┬───────────────────────┘
                                      │
                                      ▼
                 
┌────────────────────────────────────────────┐
                 │          EXECUTION AUTHORIZATION             │
                 
└────────────────────┬───────────────────────┘
                                      │
                    
┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
              SIMULATED          BROKER PAPER         LIVE
                BROKER              BROKER            BROKER
                    │                 │                 │
                    
└─────────────────┼─────────────────┘
                                      ▼
                            ORDER LIFECYCLE
                                      │
                                      ▼
                              POSITION MANAGER
                                      │
                                      ▼
                           EXIT / STOP / TARGET
                                      │
                                      ▼
                             FLATTEN ENGINE
                                      │
                                      ▼
                           RECONCILIATION ENGINE
                                      │
                                      ▼
                              AUDIT JOURNAL
                                      │
                                      ▼
                           PERFORMANCE ENGINE
```

---

# 5. PROJECT DIRECTORY

The project shall ultimately be organized around these responsibilities:

```text
ai-trading-agent/
│
├── app/
│   │
│   ├── ai/
│   │   └── engine.py
│   │
│   ├── backtest/
│   │   ├── engine.py
│   │   ├── data.py
│   │   ├── fills.py
│   │   ├── accounting.py
│   │   ├── metrics.py
│   │   └── experiment.py
│   │
│   ├── data/
│   │   ├── historical.py
│   │   ├── live.py
│   │   ├── quality.py
│   │   └── models.py
│   │
│   ├── execution/
│   │   ├── broker.py
│   │   ├── simulated.py
│   │   ├── ibkr.py
│   │   ├── order_manager.py
│   │   ├── reconciliation.py
│   │   ├── flatten.py
│   │   ├── session.py
│   │   └── state_machine.py
│   │
│   ├── features/
│   │   └── technical.py
│   │
│   ├── journal/
│   │   └── database.py
│   │
│   ├── monitoring/
│   │   ├── service.py
│   │   └── kill_switch.py
│   │
│   ├── news/
│   │   └── engine.py
│   │
│   ├── portfolio/
│   │   └── controller.py
│   │
│   ├── regime/
│   │   └── engine.py
│   │
│   ├── risk/
│   │   └── engine.py
│   │
│   ├── runtime/
│   │   └── orchestrator.py
│   │
│   ├── strategy/
│   │   └── scanner.py
│   │
│   └── validation/
│       └── decision.py
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── manifests/
│
├── experiments/
│   ├── manifests/
│   ├── results/
│   └── logs/
│
├── tests/
│
├── docs/
│
├── scripts/
│
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

---

# 6. CONFIGURATION AUTHORITY

There must be one authoritative configuration model.

It controls:

* run mode;
* initial capital;
* maximum risk per trade;
* maximum daily loss;
* maximum trades per day;
* maximum open positions;
* maximum total exposure;
* maximum sector exposure;
* AI confidence threshold;
* opportunity threshold;
* stop policy;
* target policy;
* entry cutoff;
* flatten time;
* market timezone;
* allowed universe;
* data source;
* AI model;
* AI timeout;
* retry policy;
* execution friction;
* commission assumptions.

No strategy module should contain hidden risk constants.

No execution module should secretly change risk.

No AI prompt should redefine the risk policy.

---

# 7. MARKET DATA LAYER

The market-data system must support three sources.

```text
HistoricalDataProvider
PaperMarketDataProvider
LiveMarketDataProvider
```

All three must produce the same canonical market-event structure.

The strategy must not care where the data came from.

Each market event must contain, at minimum:

* symbol;
* timestamp;
* open;
* high;
* low;
* close;
* volume;
* trading session;
* source;
* data-quality status.

---

# 8. HISTORICAL DATA REQUIREMENTS

Historical data must be treated as an information source with a strict 
timestamp boundary.

At time:

```text
10:15:00
```

the system may use only information that would actually have been 
available by:

```text
10:15:00
```

It must not use:

* future candles;
* future volume;
* future news;
* future corporate events;
* future adjusted prices;
* future fundamentals;
* future market state.

This is one of the most important requirements in the project.

---

# 9. POINT-IN-TIME DATA RULE

Every historical input must answer:

> “Could the trading system have known this at this exact timestamp?”

If the answer is unknown:

```text
DO NOT USE IT
```

The historical engine must prefer missing information over accidentally 
using future information.

---

# 10. SIMULATION CLOCK

The historical engine needs its own clock.

It must not use the computer's current time.

The simulation clock controls:

* current timestamp;
* trading day;
* market session;
* entry cutoff;
* flatten time;
* order timing;
* fill timing;
* news availability;
* position monitoring.

Every component involved in historical replay must receive simulation 
time.

No component should secretly call:

```text
datetime.now()
```

to determine trading behavior during historical replay.

---

# 11. MARKET CALENDAR

The system must use an authoritative exchange calendar.

It must understand:

* regular trading days;
* weekends;
* holidays;
* early closes;
* market-open time;
* market-close time;
* pre-market;
* regular session;
* post-market.

The system must never assume that every weekday is automatically:

```text
09:30 → 16:00
```

---

# 12. DATA QUALITY ENGINE

Before market data reaches the strategy, validate:

### Timestamp

* present;
* valid;
* ordered;
* timezone-aware;
* no impossible timestamps.

### OHLC

* open > 0;
* high > 0;
* low > 0;
* close > 0;
* high >= low;
* high >= open;
* high >= close;
* low <= open;
* low <= close.

### Volume

* non-negative;
* no impossible values;
* missing volume identified.

### Continuity

Identify:

* missing bars;
* duplicate bars;
* gaps;
* suspicious jumps.

Bad data must prevent a new trade.

---

# 13. TECHNICAL FEATURE ENGINE

The deterministic feature engine calculates features before AI evaluation.

Core features include:

* RSI;
* RVOL;
* ATR;
* momentum;
* price change;
* volume change;
* moving averages;
* volatility;
* range;
* breakout state;
* trend state.

The original strategy uses RVOL and RSI as important scanner filters. 

The RVOL calculation must exclude the current bar from the historical 
baseline.

The RSI implementation must correctly handle:

* all gains;
* all losses;
* flat prices.

---

# 14. OPPORTUNITY SCANNER

The scanner is responsible for reducing the universe to candidate trades.

It should evaluate:

* momentum;
* relative volume;
* RSI;
* price movement;
* breakout characteristics;
* liquidity;
* volatility;
* spread;
* market regime;
* trading session.

The original design uses a momentum scanner with RVOL > 3x and RSI 
filtering. 

These values should remain configuration parameters rather than hard-coded 
throughout the code.

---

# 15. MARKET REGIME ENGINE

The system must determine the broader environment.

Possible states include:

```text
TRENDING
RANGING
HIGH_VOLATILITY
LOW_VOLATILITY
RISK_ON
RISK_OFF
UNKNOWN
```

The regime engine must be deterministic.

AI may interpret the regime but cannot redefine the actual regime 
classification.

If regime information is unavailable:

```text
UNKNOWN
```

The system should become more conservative rather than assuming a 
favorable environment.

---

# 16. NEWS AND SENTIMENT ENGINE

The news system supplies context to the decision engine.

It must distinguish:

```text
NEWS_TIMESTAMP
```

from:

```text
PROCESSING_TIMESTAMP
```

The historical system must only provide news whose 
publication/availability time is less than or equal to the simulated 
decision time.

The engine should record:

* source;
* timestamp;
* symbol;
* headline;
* sentiment;
* relevance;
* event category;
* confidence;
* data source.

The AI should receive structured information rather than unrestricted 
uncontrolled web content.

---

# 17. AI ENGINE

The AI engine is a reasoning component.

It does not execute trades.

It receives:

* symbol;
* current market state;
* technical features;
* regime;
* news;
* opportunity score;
* risk context;
* portfolio context.

It returns a structured proposal.

The proposal should contain:

* action;
* confidence;
* thesis;
* risk flags;
* invalidation conditions;
* expected setup quality.

The AI engine must fail safely.

If:

* API fails;
* timeout occurs;
* response is malformed;
* required fields are missing;
* confidence is invalid;

then:

```text
HOLD / REJECT
```

The original blueprint explicitly requires malformed AI output, 
unavailable API, or false-positive AI output to be overridden by local 
deterministic controls. 

---

# 18. AI HISTORICAL REPRODUCIBILITY

AI creates a special problem in backtesting.

If the same historical run produces a different AI answer tomorrow, the 
experiment cannot be reproduced perfectly.

Therefore every AI decision must record:

* model;
* model version if available;
* system prompt version;
* user prompt version;
* input payload;
* output;
* timestamp;
* experiment ID;
* decision ID.

For deterministic historical replay:

```text
Recorded AI decision
        ↓
Replay
        ↓
Same recorded decision
```

For new AI research:

```text
New experiment
        ↓
New AI calls
        ↓
New experiment ID
```

Do not silently mix the two.

---

# 19. DECISION VALIDATION FIREWALL

The validator sits between AI and risk.

It checks:

* valid action;
* valid symbol;
* valid confidence;
* minimum confidence;
* minimum opportunity score;
* AI risk flags;
* valid entry;
* valid stop;
* valid target;
* valid market session;
* valid data freshness.

A failed validation means:

```text
NO ORDER
```

The validator must not modify the risk policy to accommodate AI.

---

# 20. RISK ENGINE

The risk engine is the highest authority over trade admission.

The original blueprint correctly defines it as the final deterministic 
safety layer before execution. 

It must evaluate:

### Account-level risk

* current equity;
* daily starting equity;
* current daily P&L;
* maximum daily loss.

### Trade-level risk

* entry;
* stop;
* target;
* risk per share;
* maximum dollar risk.

### Position limits

* maximum open positions;
* maximum total exposure;
* maximum symbol exposure;
* maximum sector exposure.

### Activity limits

* maximum daily trades;
* cooldown;
* entry cutoff.

---

# 21. POSITION SIZING

Position sizing must be based on actual stop distance.

The calculation is:

```text
risk_per_share =
    abs(entry_price - stop_price)

maximum_dollar_risk =
    account_equity × allowed_risk_percentage

shares =
    floor(
        maximum_dollar_risk /
        risk_per_share
    )
```

Then additional exposure limits are applied.

This is fundamentally different from simply allocating a percentage of 
capital to the stock.

The final quantity must satisfy every constraint.

---

# 22. PORTFOLIO ENGINE

The portfolio engine evaluates the proposed trade in the context of 
existing positions.

It must know:

* cash;
* equity;
* open positions;
* pending orders;
* gross exposure;
* net exposure;
* symbol exposure;
* sector exposure;
* realized P&L;
* unrealized P&L;
* daily P&L.

A trade that is individually safe may still be unsafe at the portfolio 
level.

Therefore:

```text
Trade Risk
      +
Portfolio Risk
      =
Final Risk Decision
```

---

# 23. EXECUTION AUTHORIZATION

The system must produce an explicit authorization object.

Conceptually:

```text
ExecutionAuthorization

authorized = true / false

reason = ...

symbol = ...

side = ...

quantity = ...

entry = ...

stop = ...

target = ...

risk_dollars = ...

exposure = ...

decision_id = ...

risk_decision_id = ...

portfolio_decision_id = ...
```

The broker must never construct its own independent trading decision.

It should receive an already-authorized execution instruction.

---

# 24. BROKER ABSTRACTION

The system must have a generic broker interface.

Required operations:

```text
connect
disconnect
get_account
get_positions
get_open_orders
submit_order
cancel_order
modify_order
get_order
close_position
cancel_all
```

There are three implementations:

```text
SimulatedBroker
PaperBroker
LiveBroker
```

The trading strategy should not know which one is active.

---

# 25. SIMULATED BROKER

The simulated broker is not a fake shortcut.

It must reproduce the mechanics of a real broker sufficiently well for 
research.

It needs:

* account;
* cash;
* equity;
* positions;
* orders;
* fills;
* commissions;
* spread;
* slippage;
* partial fills;
* order status;
* order timestamps;
* cancellation;
* rejection.

---

# 26. ORDER STATE MACHINE

Every order must have a defined lifecycle.

Example states:

```text
CREATED
   ↓
SUBMITTED
   ↓
ACCEPTED
   ↓
PARTIALLY_FILLED
   ↓
FILLED
```

or:

```text
CREATED
   ↓
SUBMITTED
   ↓
CANCELLED
```

or:

```text
CREATED
   ↓
REJECTED
```

No code should assume an order filled simply because submission succeeded.

---

# 27. SIMULATED FILL ENGINE

Historical simulation must model execution friction.

It must account for:

* bid/ask spread;
* slippage;
* commissions;
* fill timing;
* order type;
* liquidity;
* partial fills;
* price gaps.

The system must never assume:

```text
signal price = fill price
```

because this creates unrealistically favorable results.

---

# 28. STOP/TARGET EXECUTION

Every open position must have explicit exit logic.

At minimum:

```text
STOP LOSS
TAKE PROFIT
TIME EXIT
END-OF-DAY FLATTEN
EMERGENCY EXIT
```

If historical bar data shows that both the stop and target could have been 
touched inside the same bar, the simulator must not automatically assume 
the favorable outcome.

It must use a documented conservative rule or higher-resolution data.

---

# 29. ACCOUNTING ENGINE

The simulator must maintain:

```text
cash
equity
buying power
positions
average entry price
market value
realized P&L
unrealized P&L
commissions
slippage
gross exposure
net exposure
```

Every fill changes account state.

Every account-state change must be deterministic.

---

# 30. TRADE LIFECYCLE

A trade must be represented as a complete chain:

```text
Market Event
     ↓
Features
     ↓
Opportunity
     ↓
AI Decision
     ↓
Validation
     ↓
Risk Decision
     ↓
Portfolio Decision
     ↓
Execution Authorization
     ↓
Order
     ↓
Fill
     ↓
Position
     ↓
Exit
     ↓
Closing Fill
     ↓
Trade
     ↓
P&L
```

This chain is the core audit structure.

---

# 31. JOURNAL / AUDIT SYSTEM

The database is the system's permanent memory.

The original blueprint already defines the ledger as recording scanner 
signals, AI decisions, orders and fills. 

The authoritative implementation expands this into:

```text
market_events
feature_snapshots
opportunities
ai_decisions
validation_decisions
risk_decisions
portfolio_decisions
market_regimes
news_events
orders
fills
positions
trades
health_events
system_events
experiments
```

Every important action must have an ID.

---

# 32. TRACEABILITY

The system must allow you to answer:

> Why did the system buy this stock?

Starting from the trade, you should be able to trace backwards:

```text
Trade
 ↓
Exit Fill
 ↓
Entry Fill
 ↓
Order
 ↓
Authorization
 ↓
Portfolio Decision
 ↓
Risk Decision
 ↓
Validation
 ↓
AI Decision
 ↓
Opportunity
 ↓
Features
 ↓
Market Data
 ↓
News / Regime
```

Nothing important should be a black box.

---

# 33. SIMULATION ENGINE

The simulation engine controls the complete historical event loop.

Its responsibility is:

```text
LOAD DATA
     ↓
ADVANCE CLOCK
     ↓
PROCESS MARKET EVENT
     ↓
UPDATE ACCOUNT
     ↓
UPDATE POSITIONS
     ↓
CHECK EXITS
     ↓
CALCULATE FEATURES
     ↓
SCAN
     ↓
NEWS
     ↓
AI
     ↓
VALIDATION
     ↓
RISK
     ↓
PORTFOLIO
     ↓
SUBMIT SIMULATED ORDER
     ↓
SIMULATE FILL
     ↓
UPDATE ACCOUNT
     ↓
JOURNAL
     ↓
NEXT EVENT
```

The loop continues until the historical dataset ends.

---

# 34. HISTORICAL ENGINE

The historical engine must support:

* one trading day;
* one week;
* one month;
* three months;
* one year;
* multiple years;
* multiple symbols.

It must preserve the exact chronological ordering of events.

The historical engine must not process future events early.

---

# 35. EXPERIMENT MANIFEST

Every historical experiment must have a manifest.

The manifest records:

```text
experiment_id
strategy_version
code_version
dataset_id
dataset_version
start_date
end_date
universe
timeframe
initial_capital
risk_policy
commission_model
spread_model
slippage_model
fill_model
AI model
AI prompt version
regime configuration
scanner configuration
random seed
created_at
```

This makes results reproducible.

---

# 36. DETERMINISTIC REPLAY

If you execute:

```text
Experiment A
```

twice with exactly the same:

* code;
* data;
* configuration;
* AI decisions;
* seed;
* execution model;

the results should be identical.

The following should match:

* trades;
* fills;
* position sizes;
* P&L;
* drawdown;
* equity curve;
* rejection reasons.

If they don't, investigate before trusting the backtester.

---

# 37. ANTI-LOOKAHEAD TESTING

The system must deliberately attempt to catch itself cheating.

Required tests:

### Future-bar test

Inject impossible future information.

Result must not change.

### Future-volume test

Change future volume.

Earlier decisions must not change.

### Future-news test

Add news after the decision.

The decision must not see it.

### Future-event test

Add future market events.

Earlier decisions must remain unchanged.

### Dataset truncation test

Run:

```text
January → December
```

and:

```text
January → September
```

Decisions occurring before September must remain identical.

This is a powerful leakage test.

---

# 38. DATASET INTEGRITY

Each historical dataset must have:

* source;
* download date;
* symbol list;
* timeframe;
* timezone;
* corporate-action treatment;
* adjustment methodology;
* missing-data report;
* checksum/version identifier.

Do not casually replace datasets and continue comparing results.

A changed dataset means a new experiment.

---

# 39. BENCHMARKS

The strategy must be compared against simple baselines.

At minimum:

```text
SPY buy-and-hold
QQQ buy-and-hold
cash
simple deterministic strategy
```

The purpose is not merely to produce a positive return.

The question is:

> Does the complexity of this AI trading system produce sufficiently 
better risk-adjusted behavior than simpler alternatives?

---

# 40. PERFORMANCE METRICS

Every historical experiment should calculate:

### Return

* total return;
* annualized return;
* daily return.

### Risk

* maximum drawdown;
* volatility;
* downside volatility.

### Trade statistics

* number of trades;
* winning trades;
* losing trades;
* win rate;
* average win;
* average loss;
* largest win;
* largest loss;
* profit factor;
* expectancy.

### Risk-adjusted metrics

* Sharpe;
* Sortino;
* Calmar where appropriate.

### Execution metrics

* average slippage;
* median slippage;
* commissions;
* spread cost;
* fill rate;
* rejected orders.

---

# 41. STRATEGY QUALITY

Do not judge the system using return alone.

A strategy that produces:

```text
+100%
```

with:

```text
-80% drawdown
```

is fundamentally different from one producing:

```text
+40%
```

with:

```text
-10% drawdown
```

The system must evaluate:

```text
RETURN
RISK
DRAWDOWN
CONSISTENCY
EXECUTION COST
TRADE QUALITY
ROBUSTNESS
```

---

# 42. COST STRESS TESTING

Run the strategy with progressively worse execution assumptions.

At minimum test:

```text
normal friction
moderate friction
high friction
extreme friction
```

Evaluate whether the strategy's edge survives.

If profitability disappears under modest realistic transaction costs, the 
strategy should not advance.

---

# 43. PARAMETER SENSITIVITY

Test changes to:

* RVOL threshold;
* RSI thresholds;
* confidence threshold;
* stop distance;
* target distance;
* risk percentage;
* maximum positions;
* opportunity threshold;
* slippage;
* commission.

The objective is to determine whether performance exists only at one 
extremely specific parameter combination.

If a tiny parameter change destroys the strategy, that is a warning sign 
for overfitting.

---

# 44. TRAIN / VALIDATION / OUT-OF-SAMPLE

Historical data must eventually be separated.

Conceptually:

```text
TRAINING
    ↓
VALIDATION
    ↓
OUT-OF-SAMPLE
```

The final performance claim must rely heavily on data that was not used to 
tune the strategy.

---

# 45. WALK-FORWARD TESTING

The final research process should use rolling periods.

Conceptually:

```text
Train
───────
       Validate
       ────────

        Train
        ─────────
               Validate
               ────────

                Train
                ─────────
                       Validate
                       ────────
```

This tests whether the strategy continues working as market conditions 
change.

---

# 46. PORTFOLIO CORRELATION

Once multiple symbols are traded, the system must consider correlation.

Five individual positions can each appear safe but collectively create one 
enormous market bet.

Therefore portfolio risk must consider:

* symbol exposure;
* sector exposure;
* market exposure;
* correlation;
* gross exposure;
* net exposure.

---

# 47. SESSION CONTROL

The session engine defines:

```text
PRE_MARKET
OPENING
TRADING
ENTRY_CUTOFF
FLATTENING
CLOSED
```

New entries must be disabled after the configured cutoff.

Existing positions continue to be managed.

---

# 48. END-OF-DAY FLATTEN

The original design uses a 15:45 Eastern flattening process. 

The authoritative implementation must:

```text
ENTRY CUTOFF
      ↓
NO NEW POSITIONS
      ↓
CANCEL WORKING ENTRY ORDERS
      ↓
CLOSE OPEN POSITIONS
      ↓
WAIT FOR CONFIRMED FILLS
      ↓
QUERY POSITIONS
      ↓
CONFIRM ZERO EXPOSURE
      ↓
SESSION COMPLETE
```

Simply sending a close order does not mean the portfolio is flat.

The system must verify it.

---

# 49. KILL SWITCH

There are two levels.

## SOFT HALT

Stops:

```text
NEW ENTRIES
```

but allows:

```text
EXITS
RECONCILIATION
MONITORING
```

## HARD HALT

Used for emergency conditions.

Depending on policy:

```text
CANCEL ORDERS
+
FLATTEN
+
STOP NEW TRADING
+
ALERT OPERATOR
```

---

# 50. AUTOMATIC HALT CONDITIONS

The system should be capable of entering a safe state after:

* database failure;
* corrupted market data;
* stale data;
* broker disconnect;
* reconciliation mismatch;
* unexpected account state;
* impossible position;
* excessive daily loss;
* system health failure;
* execution anomaly;
* repeated order rejection;
* emergency operator command.

---

# 51. RECONCILIATION

Reconciliation compares:

```text
INTERNAL STATE
       VS
BROKER STATE
```

It must compare:

* positions;
* quantities;
* average prices;
* open orders;
* order statuses;
* fills;
* account equity.

If they disagree:

```text
NO NEW ENTRIES
```

until the discrepancy is resolved.

---

# 52. MONITORING

The monitoring service tracks:

* process health;
* data health;
* AI health;
* database health;
* broker health;
* order health;
* reconciliation health;
* session state;
* kill-switch state.

A heartbeat should be emitted periodically.

The system must make health observable.

---

# 53. ALERTING

Alerts should be generated for:

### Trading

* opportunity;
* approved trade;
* rejected trade;
* fill;
* stop;
* target;
* flatten.

### System

* startup;
* shutdown;
* broker disconnect;
* data failure;
* AI failure;
* database failure;
* reconciliation mismatch;
* kill switch;
* unexpected position.

The original design includes Discord notifications for transaction and 
system telemetry. 

---

# 54. LEARNING SYSTEM

The word “learning” must be defined carefully.

The production system must **not automatically rewrite its own strategy or 
risk parameters based on recent losses**.

Instead:

```text
TRADING
   ↓
DATA COLLECTION
   ↓
PERFORMANCE ANALYSIS
   ↓
RESEARCH
   ↓
NEW CANDIDATE STRATEGY
   ↓
BACKTEST
   ↓
OUT-OF-SAMPLE
   ↓
PAPER
   ↓
QUALIFICATION
   ↓
HUMAN APPROVAL
   ↓
DEPLOYMENT
```

Learning is therefore a controlled research/deployment process.

---

# 55. NO SELF-MODIFYING LIVE TRADER

The production trading process must never automatically modify:

* risk limits;
* stop rules;
* position limits;
* strategy logic;
* broker permissions;
* AI authority.

AI can recommend changes.

Research can produce changes.

Deployment requires explicit qualification.

---

# 56. VERSIONING

Every production-relevant component must have a version.

Examples:

```text
strategy_version
risk_policy_version
prompt_version
AI_model_version
feature_version
execution_model_version
dataset_version
```

Every trade must identify the versions that produced it.

---

# 57. ERROR HANDLING

Errors must be classified.

### Recoverable

Example:

```text
temporary API timeout
```

Action:

```text
retry
```

### Trading-blocking

Example:

```text
stale market data
```

Action:

```text
no new entry
```

### Critical

Example:

```text
unknown broker position
```

Action:

```text
halt + reconcile
```

### Emergency

Example:

```text
uncontrolled exposure
```

Action:

```text
hard halt + emergency handling
```

---

# 58. FAILURE POLICY

The default philosophy is:

> **When uncertain, do not open additional risk.**

Examples:

```text
AI uncertain
→ NO TRADE

Data uncertain
→ NO TRADE

Risk state uncertain
→ NO TRADE

Portfolio state uncertain
→ NO TRADE

Broker state uncertain
→ NO TRADE

Reconciliation uncertain
→ NO TRADE
```

Existing positions are handled separately.

---

# 59. TESTING PYRAMID

Testing must occur at several levels.

## Unit tests

Test:

* RSI;
* RVOL;
* ATR;
* risk sizing;
* validation;
* portfolio limits;
* session state;
* flatten;
* reconciliation.

## Integration tests

Test:

```text
market event
→ features
→ scanner
→ AI
→ validator
→ risk
→ portfolio
→ simulated broker
```

## End-to-end tests

Run complete simulated trading sessions.

## Safety tests

Deliberately trigger failures.

## Historical tests

Run the complete system over real historical data.

---

# 60. CRITICAL INVARIANTS

The following must always be true.

### Invariant 1

No position can exist without a fill.

### Invariant 2

No order can exist without authorization.

### Invariant 3

No new entry can occur after the entry cutoff.

### Invariant 4

No new entry can occur while the system is halted.

### Invariant 5

No new entry can occur with invalid/stale data.

### Invariant 6

Position size can never exceed deterministic risk limits.

### Invariant 7

Portfolio exposure can never exceed portfolio limits.

### Invariant 8

Flattening is not considered successful until positions are verified flat.

### Invariant 9

Historical decisions cannot access future information.

### Invariant 10

Simulation and broker execution must use the same decision authorization 
logic.

---

# 61. FIRST AUTHORITATIVE END-TO-END SCENARIO

Before running years of historical data, create one tiny deterministic 
scenario.

The scenario must contain:

```text
historical bars
     ↓
valid setup
     ↓
scanner detects opportunity
     ↓
AI produces BUY
     ↓
validator approves
     ↓
risk approves
     ↓
portfolio approves
     ↓
simulated order
     ↓
fill
     ↓
position
     ↓
stop/target
     ↓
exit
     ↓
closed trade
     ↓
P&L
     ↓
journal
```

Then create failure scenarios:

```text
AI BUY
→ confidence too low
→ REJECT
```

```text
AI BUY
→ daily loss exceeded
→ REJECT
```

```text
AI BUY
→ maximum positions reached
→ REJECT
```

```text
AI BUY
→ stale data
→ REJECT
```

```text
AI BUY
→ kill switch
→ REJECT
```

```text
AI BUY
→ after entry cutoff
→ REJECT
```

---

# 62. HISTORICAL TEST PROGRESSION

Never immediately run five years of data.

Use this progression:

```text
1 trading day
      ↓
1 week
      ↓
1 month
      ↓
3 months
      ↓
6 months
      ↓
1 year
      ↓
multiple years
      ↓
multiple symbols
```

At every stage:

```text
TEST
→ INSPECT
→ FIX
→ REPEAT
```

Do not proceed merely because the program completed.

---

# 63. HISTORICAL QUALIFICATION GATE

The historical engine is not qualified merely because it produces a 
performance report.

It must demonstrate:

### Data integrity

* point-in-time data;
* no future leakage;
* correct sessions;
* correct timestamps.

### Execution realism

* spread;
* slippage;
* commission;
* fill timing;
* partial fills;
* stop/target handling.

### Strategy integrity

* actual production strategy;
* actual feature pipeline;
* actual AI interface;
* actual deterministic validation;
* actual risk engine.

### Safety

* risk limits work;
* kill switch works;
* flatten works;
* reconciliation logic works;
* invalid states fail closed.

### Reproducibility

* identical experiment produces identical results.

### Robustness

* results survive cost stress;
* results survive parameter sensitivity;
* results survive out-of-sample testing;
* results survive walk-forward testing.

---

# 64. BROKER-PAPER GATE

Only after historical qualification:

```text
SIMULATION
     ↓
QUALIFIED
     ↓
BROKER PAPER
```

The broker-paper system must then be tested for:

* connection;
* authentication;
* market data;
* order submission;
* order rejection;
* partial fill;
* full fill;
* cancellation;
* modification;
* stop;
* target;
* flatten;
* disconnect;
* reconnect;
* reconciliation.

The existing IBKR adapter is deliberately not considered paper-ready until 
actual contract/order mapping and lifecycle handling are implemented and 
qualified.

---

# 65. PAPER-TRADING QUALIFICATION

Paper trading should run for a sustained period.

A suitable gate is at least:

```text
20 consecutive trading days
```

with:

* no unexplained crashes;
* no unintended overnight positions;
* successful reconciliation;
* correct flattening;
* correct risk enforcement;
* no unauthorized trades;
* documented execution behavior;
* acceptable slippage;
* complete audit trail.

The exact thresholds should be defined before the test begins rather than 
changed afterward.

---

# 66. LIVE DEPLOYMENT GATE

Live trading requires all of the following:

```text
Historical Qualification
        +
Paper Qualification
        +
Safety Qualification
        +
Operational Qualification
        +
Explicit Deployment Approval
```

Then live deployment should begin with:

```text
restricted capital
restricted universe
restricted position count
restricted daily loss
```

The system should earn permission to increase exposure through evidence 
rather than starting at maximum authority.

---

# 67. LIVE ARCHITECTURE

Once qualified:

```text
LIVE MARKET DATA
      ↓
DATA QUALITY
      ↓
FEATURES
      ↓
REGIME
      ↓
SCANNER
      ↓
NEWS
      ↓
AI
      ↓
VALIDATION
      ↓
RISK
      ↓
PORTFOLIO
      ↓
EXECUTION AUTHORIZATION
      ↓
IBKR
      ↓
ORDER EVENTS
      ↓
RECONCILIATION
      ↓
MONITORING
      ↓
JOURNAL
```

The architecture should be almost identical to simulation.

That is intentional.

---

# 68. SINGLE SOURCE OF TRUTH

The following must have one authoritative implementation:

```text
risk policy
position sizing
validation
portfolio limits
session state
flatten logic
trade state
journal schema
```

Do not maintain separate versions such as:

```text
backtest risk logic
paper risk logic
live risk logic
```

That creates dangerous divergence.

Instead:

```text
COMMON DECISION ENGINE
        ↓
┌───────┼────────┐
SIM     PAPER     LIVE
```

---

# 69. CURRENT IMPLEMENTATION STATUS

The rebuilt project already contains the foundation for this architecture.

Current major components include:

```text
app/ai/engine.py
app/backtest/engine.py
app/config.py
app/data/quality.py
app/execution/broker.py
app/execution/flatten.py
app/execution/ibkr.py
app/execution/order_manager.py
app/execution/reconciliation.py
app/execution/session.py
app/execution/state_machine.py
app/features/technical.py
app/journal/database.py
app/monitoring/kill_switch.py
app/monitoring/service.py
app/news/engine.py
app/portfolio/controller.py
app/regime/engine.py
app/risk/engine.py
app/runtime/orchestrator.py
app/strategy/scanner.py
app/validation/decision.py
```

The project currently has a passing baseline test suite, but that does 
**not** mean the historical engine or IBKR execution system is finished.

---

# 70. WHAT MUST BE BUILT NEXT

The immediate engineering sequence is:

## Stage A — Simulation foundation

1. Canonical market-event model.
2. Simulation clock.
3. Historical data provider.
4. Historical dataset validation.
5. Point-in-time information boundary.

## Stage B — Simulated broker

6. Simulated account.
7. Simulated positions.
8. Simulated orders.
9. Order state machine.
10. Fill engine.
11. Spread model.
12. Slippage model.
13. Commission model.
14. Partial fills.
15. Accounting.

## Stage C — Common decision pipeline

16. Central decision pipeline.
17. Shared validator.
18. Shared risk engine.
19. Shared portfolio engine.
20. Execution authorization object.

## Stage D — Complete replay

21. Historical event loop.
22. Scanner integration.
23. Regime integration.
24. News integration.
25. AI integration.
26. Simulated execution.
27. Position management.
28. Exit management.
29. Flattening.
30. Reconciliation.

## Stage E — Audit

31. Complete decision chain.
32. Experiment IDs.
33. Experiment manifests.
34. AI replay recording.
35. Deterministic replay.

## Stage F — Qualification

36. One-day test.
37. One-week test.
38. One-month test.
39. Multi-month test.
40. Multi-year test.
41. Cost stress.
42. Parameter sensitivity.
43. Out-of-sample.
44. Walk-forward.
45. Benchmark comparison.

## Stage G — Paper

46. Complete IBKR order implementation.
47. Broker lifecycle events.
48. Broker reconciliation.
49. Paper execution.
50. Sustained paper qualification.

## Stage H — Live

51. Restricted live deployment.
52. Continuous monitoring.
53. Controlled performance evaluation.
54. Controlled strategy updates.

---

# 71. DEFINITION OF “DONE”

The project is **not done** when:

```text
the program runs
```

It is **not done** when:

```text
the backtest makes money
```

It is **not done** when:

```text
the AI produces good-looking decisions
```

It is **not done** when:

```text
IBKR accepts an order
```

The system is ready for the next stage only when the corresponding 
qualification criteria have been demonstrated.

---

# 72. FINAL AUTHORITY HIERARCHY

The final authority hierarchy is:

```text
1. SAFETY / KILL SWITCH
          ↓
2. MARKET / DATA VALIDITY
          ↓
3. SESSION RULES
          ↓
4. RISK POLICY
          ↓
5. PORTFOLIO LIMITS
          ↓
6. EXECUTION RULES
          ↓
7. STRATEGY
          ↓
8. AI PROPOSAL
```

AI is intentionally near the bottom.

It can improve the quality of a candidate decision.

It cannot override the layers above it.

---

# 73. FINAL SYSTEM OBJECTIVE

The completed system should ultimately behave as follows:

```text
                         ┌──────────────┐
                         │ MARKET DATA  │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ DATA QUALITY │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ QUANT MODEL  │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ REGIME MODEL │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ NEWS ENGINE  │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ AI ANALYSIS  │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ VALIDATION   │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ RISK ENGINE  │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ PORTFOLIO    │
                         └──────┬───────┘
                                ↓
                         ┌──────────────┐
                         │ AUTHORIZATION│
                         └──────┬───────┘
                                ↓
                    ┌───────────┴───────────┐
                    ↓                       ↓
              SIMULATED BROKER        REAL BROKER
                    ↓                       ↓
                 FILLS                    FILLS
                    ↓                       ↓
                POSITIONS                POSITIONS
                    ↓                       ↓
                 EXITS                   EXITS
                    ↓                       ↓
                P&L / AUDIT             P&L / AUDIT
                    └───────────┬───────────┘
                                ↓
                         PERFORMANCE
                                ↓
                          RESEARCH
                                ↓
                       CONTROLLED UPDATES
```

The fundamental goal is **not to build an AI that is allowed to trade 
whatever it wants**.

The goal is to build a **deterministic trading machine whose intelligence 
layer can improve decision quality without ever being able to bypass the 
machine's safety rules**.

That is the architecture we should build toward.

### The next build target

The blueprint above should now become the **master specification** for 
your existing project.

The next step should **not** be IBKR.

It should be:

**Stage A — turn the current simulation/historical foundation into a 
genuinely authoritative event-driven engine**, starting with the canonical 
market event, simulation clock, historical data provider, and strict 
no-lookahead boundary.

Then we build the simulated broker and accounting system on top of that.

That gives you one clean path:

**Blueprint → Simulation → Historical Qualification → Paper → Live**, 
without rebuilding the system at every stage.

# PART II — PRE-LIVE COMPLIANCE ADDENDUM

# 74. LEGAL, COMPLIANCE, AND DATA LICENSING
The system must not be operated live until the following are resolved and 
documented:
- Broker agreement and account type (cash vs margin).
- Market data entitlements: which exchanges, real-time vs delayed, 
per-user vs per-device.
- News/sentiment data licensing and redistribution limits.
- Pattern Day Trader (PDT) applicability if US equities and account < 
$25,000.
- Short selling: locate requirement, borrow availability, short sale 
restriction (SSR).
- Tax reporting obligations and jurisdiction.
- Regulatory regime (SEC/FINRA, FCA, ESMA, etc.) for the operator.
- Written statement that the system is for personal/internal use and is 
not financial advice.

# 75. SECURITY AND SECRETS MANAGEMENT
- Secrets never committed to Git. `.env` is gitignored. Only 
`.env.example` is committed.
- Secrets loaded from environment or a secret manager (Vault, AWS Secrets 
Manager, 1Password CLI).
- Broker API keys scoped to least privilege. Separate keys for paper and 
live.
- IP allowlisting on broker and exchange portals where supported.
- 2FA on broker account, GitHub, email, and cloud.
- Journal database encrypted at rest. Backups encrypted.
- Audit log of every configuration change, with who and when.
- No credentials, tokens, or account numbers pasted into any chat or LLM 
prompt.
- Key rotation schedule and revocation procedure documented.

# 76. DEPLOYMENT AND OPERATIONS
- Dedicated host (VPS or colocation). Not a laptop.
- NTP time synchronization verified before each session.
- Process supervisor (systemd, supervisor, or similar) with automatic 
restart and backoff.
- Backups: journal DB, config, dataset manifests, experiment manifests.
- Documented disaster recovery: restore host, restore DB, verify 
reconciliation.
- Runbooks: start, stop, soft halt, hard halt, recover from disconnect, 
rollback.
- Rollback procedure for strategy, config, and prompt changes.
- Log retention policy.
- Alerting channel (Discord or equivalent) with heartbeat.

# 77. MARKET MICROSTRUCTURE REALITY
- Bid/ask data, not just OHLC, wherever fills are modeled.
- Spread model and its source.
- Auction behavior at open and close.
- Trading halts: LULD, market-wide circuit breakers, news halts, IPO 
halts.
- SSR and its effect on short entries.
- Symbol changes, ticker reuse, and corporate action adjustments.
- Splits, dividends, mergers, spinoffs.
- Delistings and survivorship bias in the dataset.
- Short borrow availability if shorts are allowed.
- The system must recognize when a symbol is halted and refuse to trade 
it.

# 78. BROKER API REALITIES
- Rate limits per endpoint and per second.
- Supported order types per venue.
- Partial fill semantics and how they update position and risk.
- Reject reason codes and mapping to internal error classes.
- Cancel/replace timing and race conditions.
- Reconnect and resync procedure after disconnect.
- Contract mapping (internal symbol ↔ broker contract ID).
- Per-exchange market data permissions required for the universe.
- Differences between paper and live API behavior.

# 79. AI GOVERNANCE AND SAFETY
- Prompt injection defense: never place untrusted text in a position of 
authority.
- Structured output enforced by schema. Malformed = HOLD/REJECT.
- Hallucination checks: numbers, symbols, timestamps must match supplied 
inputs.
- Confidence calibration tracked over time.
- Model version pinned and recorded per decision.
- Deterministic replay via recorded AI outputs.
- Cost and latency budgets per call and per day.
- Any failure (timeout, API error, schema error, low confidence) → 
HOLD/REJECT.
- Offline evaluation harness for prompt and model changes.
- Human review required before any prompt or model change reaches live.

# 80. STATISTICAL RIGOR
- Multiple-testing correction across parameter sweeps.
- Deflated Sharpe ratio and similar adjustments.
- Probability of backtest overfitting (PBO) estimate.
- Monte Carlo / bootstrap on trades and equity curve.
- Minimum track record length before any live claim.
- Regime-conditional performance reporting.
- Comparison against benchmarks and null strategies.
- Out-of-sample discipline: no tuning on OOS data.
- Walk-forward validation across multiple regimes.

# 81. CI/CD AND TESTING INFRASTRUCTURE
- Unit tests for features, risk, validation, portfolio, session, flatten, 
reconciliation.
- Integration tests across the decision pipeline.
- End-to-end simulation tests over real historical data.
- Safety tests that deliberately trigger every halt condition.
- Property-based tests (hypothesis) for risk and sizing invariants.
- Fuzz tests on data parsers and broker message handlers.
- Contract tests for every broker adapter.
- Pre-commit hooks: lint, type check, tests.
- CI pipeline that runs the full suite on every commit.
- Versioned artifacts for strategy, config, prompts, datasets, execution 
model.

# 82. PRE-LIVE COMPLIANCE GATE
Live trading is forbidden until every item below is green and signed off 
in writing:
- [ ] Data: point-in-time, licensed, survivorship-adjusted, halt-aware.
- [ ] Security: secrets managed, 2FA enabled, no leaks in history.
- [ ] Operations: host, NTP, supervisor, backups, runbooks, rollback 
tested.
- [ ] Microstructure: halts, SSR, corporate actions handled.
- [ ] Broker: rate limits, reconnects, rejects, partial fills handled.
- [ ] AI: schema, calibration, replay, fallback all verified.
- [ ] Statistics: OOS, walk-forward, cost stress all passed.
- [ ] CI: all tests green, artifacts versioned and signed.
- [ ] Journal: full traceability from trade back to market event.
- [ ] Reconciliation: demonstrated to catch and resolve mismatch.
- [ ] Flatten: demonstrated to verify flat, not just send orders.
- [ ] Kill switch: soft and hard halt tested in paper.
- [ ] Paper qualification: minimum 20 consecutive trading days documented.
- [ ] Human sign-off recorded with date and version hashes.


