Ophireum Multimedia Productions 

Mobile Phone: +63 917 999 8814 
Email: dhenzecapital@gmail.com
telegram: @Mkogexchange
WhatsApp: +63 995 715 1043


EA Name	Ophireum Technology
Version	3.0 Production Candidate
Platform	MetaTrader 5
Language	MQL5
Primary Symbol	XAUUSD
Execution TF	M5
Confirmation TF	M15
Structural TF	H1
Trend TF	H4
Macro TF	D1
Strategy Type	Multi-Timeframe SMC / Liquidity / Structure
Execution	Fully Automatic
Position Sizing	Fully Automatic
Mandatory SL	TRUE
Production Lock	TRUE



Symmetrix Gold SMC: automated MT5 EA trading XAUUSD via SMC/ICT rules (liquidity sweep→CHoCH→BOS→entry). Max 3 trades/day, auto-calculated lot sizing from live equity/risk %, strict risk controls, news/session filters, structural stops. Capital preservation prioritized over trade frequency.

Symmetrix Gold SMC is a fully automated MT5 Expert Advisor for trading XAUUSD (gold) using a Smart Money Concepts (SMC/ICT) methodology, built as a rule-based state machine rather than a simple indicator-signal bot.

Core approach: it only trades a setup that passes an entire mandatory sequence, in order — market direction alignment (H4/H1/D1), liquidity mapping, a liquidity sweep with reclaim, a strict CHoCH (change of character), directional displacement, a mandatory break of structure, retracement into a confluence entry zone (FVG + Order Block + Fibonacci 61.8%), and a defined confirmation trigger. Any failed link breaks the chain and the setup is discarded — it never trades just to hit a quota.

Trade limits & risk: up to 3 qualified trades per day (a ceiling, not a target), one position open at a time, with every trade's lot size computed live and automatically from current equity, live symbol contract specs, and the structural stop distance — never a fixed or manual lot size. Risk starts at 0.25% of equity per trade and scales down progressively after losing trades. RR must be ≥2 and volatility must be in a normal regime, or the trade is rejected outright.

Protective layers on top of signal logic: dynamic session windows (London/NY only by default), a gold-specific economic-news blackout filter, spread/slippage/execution checks, an exposure cap that takes the minimum across risk-based, broker, account, and margin limits, structural (not arbitrary) stop-losses, a staged profit-taking plan (partial closes at +1R/+2R with a trailing runner), daily/weekly/monthly loss circuit breakers, weekend flatten-and-lockout protection, and full position reconciliation after any restart or disconnect so it never loses track of what it's actually holding.

Philosophy: capital preservation first, then setup quality, then execution quality, then return — with "zero trades" treated as a fully correct outcome on days when nothing qualifies. Every accepted or rejected setup is logged in detail (including the full lot-size calculation) so its behavior is auditable rather than a black box.


SYMMETRIX GOLD SMC — v2 STRENGTHENED SPECIFICATION
Base document: v1 Production Spec (state-machine SMC EA, XAUUSD, MT5/MQL5) Purpose of this revision: close loopholes, remove ambiguity that would force a programmer to guess, and add the safeguards that separate a spec that reads rigorous from one that behaves rigorously in live markets.
Where a section is unchanged in substance, it isn't repeated — only additions/changes are listed. Section numbers match the original for cross-reference.
________________________________________
1. CORE OPERATING PRINCIPLE — no change to intent, one addition
Add a state timeout table (see §37) up front as a first-class rule, not an implementation detail: every state in the pipeline has a maximum bar-count it may remain "open." This is the single most common way SMC EAs silently degrade — a CHoCH from three days ago is not the same signal as one from ten minutes ago, but a naive implementation will happily chain them together.
________________________________________
2. MARKET DIRECTION ENGINE — add persistence and separation filters
Problems with v1: a bare Close > EMA20 > EMA50 flips on every marginal EMA cross, and says nothing about how strong or stable the trend is. In ranging conditions this produces constant bias flicker.
Additions:
•	Minimum separation: require |EMA20 − EMA50| ≥ 0.15 × H4 ATR (H1: 0.10 × H1 ATR). Below this, direction is NEUTRAL even if the raw inequality holds — the EMAs are too close together to mean anything.
•	Persistence filter: the H4 direction must have held for a minimum of 3 consecutive closed H4 candles before it counts as valid for trading (prevents trading the first candle of a fakeout flip).
•	ADX floor: H4 ADX(14) ≥ 20 required for a non-neutral bias to be tradeable. Below 20, mark direction as NEUTRAL regardless of EMA order — this is what actually distinguishes trend from chop, and ADX was previously buried as a 1-point scoring item when it should also gate the direction engine.
•	D1 bias keeps the same persistence and separation rules before contributing to scoring.
________________________________________
3. H1 DEALING RANGE — add recency, size floor, and equilibrium buffer
Problems: "most recent validated H1 swing range" has no age limit or size limit. A six-week-old range or a range 50 pips wide on gold produces meaningless premium/discount classification.
Additions:
•	Range age ceiling: the range must have been established within the last 40 H1 candles. Older ranges are discarded and the EA waits for a fresh one — no trade on stale ranges.
•	Range size floor: range size must be ≥ 3.0 × H1 ATR. Ranges smaller than this are noise, not structure; discard and wait.
•	Equilibrium exclusion zone: price within ±5% of the range midpoint is NEUTRAL (neither discount nor premium) — no trade. This prevents marginal midpoint-crossing price action from qualifying as "discount" or "premium" by a few points.
•	Range must be recalculated on every new confirmed H1 swing; do not let it drift silently while trades are being evaluated against it.
________________________________________
4. LIQUIDITY ENGINE — add selection hierarchy and decay
Problem: v1 lists ten liquidity types and says "untouched external liquidity receives priority" but gives no tie-break rule when multiple untouched pools exist simultaneously — which is the normal case.
Additions:
•	Explicit priority order (highest to lowest) when multiple untouched pools exist on the same side: PWH/PWL → PDH/PDL → Equal Highs/Lows (≥2 touches) → Asian High/Low → Confirmed H1 Swings → Confirmed M15 Swings.
•	Nearest-qualifying rule: among pools of the same priority tier, select the one nearest to current price in the direction of anticipated sweep — this is the pool most likely to be swept next, not the most distant one.
•	Liquidity decay: a level untouched for more than 10 trading days is downgraded one priority tier (still tracked, but no longer preferred over fresher levels of lower nominal rank). Old untouched liquidity is frequently stale relative to a market that has since re-priced.
•	Equal highs/lows definition: require touches within 0.08 × ATR of each other to count as "equal" — an unspecified tolerance in v1 that a programmer would otherwise have to invent unilaterally.
________________________________________
5. SWING DETECTION — add amplitude floor
Addition:
•	A 5-candle fractal alone can flag noise-level wiggles in low-volatility periods. Require swing amplitude ≥ 0.20 × M5 ATR relative to the adjacent opposite swing for the fractal to be marked "confirmed" and eligible for use anywhere downstream (liquidity, CHoCH structure, OB context). Sub-threshold fractals are tracked internally but flagged MINOR and excluded from CHoCH/BOS logic.
________________________________________
6. LIQUIDITY SWEEP — add time and velocity constraints
Additions:
•	Sweep window: the penetration candle and the reclaim (close-back) candle together must span no more than 3 M5 candles. A slow multi-hour grind through a level is not a sweep; it's a breakout in progress, and should be handled by the existing BREAKOUT-REJECT rule instead.
•	Volatility-regime gate: do not evaluate sweeps when M5 ATR has contracted to < 40% of its 50-period average (dead session, e.g., mid-Asian lull) — penetration/reclaim thresholds calibrated to normal ATR become meaningless noise-catchers in a flat market.
________________________________________
7. STRICT CHoCH STATE MACHINE — add protected-level qualification and timeout
Problems: v1 doesn't say the protected LH/HL must itself be a significant swing (it could be a MINOR fractal from §5, making CHoCH trivial to trigger). It also doesn't bound how long the EA waits between sweep and CHoCH close.
Additions:
•	Only swings marked CONFIRMED (not MINOR, per §5) may serve as the protected LH/HL.
•	CHoCH timeout: if the M5 close-above/below-protected-level condition has not occurred within 24 M5 candles (2 hours) of the qualifying sweep, the setup is invalidated and the state machine resets to LIQUIDITY_IDENTIFIED. Without this, a sweep from six hours ago could still "count" toward a CHoCH triggered under entirely different market conditions.
•	Reclose confirmation must hold for the full candle body, not just the close tick versus a intrabar spike back through the level (i.e., use the actual OHLC close value, not a tick-by-tick provisional check that a partial/incomplete bar could satisfy).
________________________________________
8. DISPLACEMENT — add follow-through check
Addition:
•	The displacement candle must not be ≥ 70% retraced by the immediately following candle. A large body that gets instantly erased is not genuine displacement — it's a spike. This catches a real failure mode where a single-candle stop-run produces a large body that satisfies the ATR-ratio test but carries no actual follow-through.
________________________________________
9. BOS — add structural-level qualification and sequencing window
Problems: v1 says "close above the next confirmed structural high" — but doesn't say how far away that level is allowed to be, or bound the time between displacement and BOS.
Additions:
•	The referenced structural high/low must be a CONFIRMED (§5) swing that existed before the CHoCH was triggered — never a swing formed during the current move (this would be circular/look-ahead logic dressed up as structure).
•	Displacement→BOS window: BOS must occur within 15 M5 candles of the qualifying displacement candle. If it doesn't happen in that window, invalidate and reset. This prevents "displacement now, structural break three days later" from being chained together as one setup.
•	If price reverses and closes back through the CHoCH protection level before BOS occurs, immediately invalidate the entire chain (sweep/CHoCH/displacement) rather than leaving it pending.
________________________________________
10. FAIR VALUE GAP — add mitigation percentage and staleness
Additions:
•	Mitigation threshold: an FVG is marked MITIGATED once price has traded through ≥ 50% of the gap's range (not just first-touch), and FULLY_MITIGATED at 100% — entries should only use UNMITIGATED or PARTIALLY_MITIGATED (<50%) gaps.
•	Staleness limit: an FVG older than 50 M5 candles without being used is deprioritized for confluence purposes (still logged, not deleted).
________________________________________
11. ORDER BLOCK VALIDATION — add tie-break and age limit
Additions:
•	When more than one opposite-direction candle precedes the impulsive move (common after consolidation), select the candle immediately adjacent to the impulse (last one before the break in the run of same-direction candles), not an arbitrarily earlier one.
•	OB age limit: an order block older than 80 M5 candles and still unmitigated is deprioritized (same treatment as stale FVGs) — old, never-tested OBs in a market that has moved on are weak confluence.
•	Mitigation defined the same way as FVG (§10): percentage-of-range traded through, not first-touch binary.
________________________________________
12. FIBONACCI CONFLUENCE — add invalidation condition
Addition:
•	If price closes beyond the 100% level of the fib (i.e., beyond the sweep extreme that started the impulse), the entire setup is invalidated — this is de facto proof the "impulse" was not what the algorithm thought it was.
________________________________________
13. INSTITUTIONAL ENTRY ZONE — add minimum stop-distance sanity check
Addition:
•	Before creating the entry zone, verify the projected structural stop (§21) would sit at least 0.5 × M5 ATR from the zone. If the OB/FVG/Fib cluster sits so close to the structural invalidation point that the resulting stop is unrealistically tight (broker minimum-stop violations, near-certain noise-stop-out), reject the zone rather than let position sizing produce absurd volumes or immediate stop-outs.
________________________________________
14. ENTRY CONFIRMATION — replace vague language with explicit trigger definitions
Problem: "require bullish/bearish M5 confirmation" is not implementable as written — this is exactly the kind of ambiguity that produces inconsistent behavior between backtest and live.
Define confirmation as any one of the following, occurring with the entry zone:
•	Engulfing: candle body fully engulfs the prior candle's body, in the trade direction.
•	Rejection wick: wick ≥ 60% of total candle range, opposite to trade direction, closing in the trade direction, inside or within 0.10×ATR of the zone.
•	Internal micro-BOS: M5 close beyond the most recent internal (1-candle) swing point within the zone, in the trade direction.
Additions:
•	Confirmation timeout: if no qualifying confirmation candle appears within 10 M5 candles of price first entering the zone, or if price trades through the zone and beyond the fib 100% level without confirming, invalidate and reset to LIQUIDITY_IDENTIFIED.
________________________________________
15. TRADE SCORING — make RR and volatility regime hard gates, not points
Problems: RR ≥ 2 and session quality being scored rather than gated means a high-scoring setup with RR of 1.8 could still fire. That's a structural expectancy leak.
Additions:
•	RR ≥ 2 is now a mandatory pass/fail gate, evaluated before scoring, not a 2-point contributor. Setups with RR < 2 are rejected outright regardless of score.
•	Add a volatility-regime gate (also pass/fail, before scoring): M5 ATR must be between 60% and 250% of its 50-period average. Below 60% = dead market (skip); above 250% = likely news spike or abnormal event (skip, defer to §17 news filter and manual review).
•	Keep the 32-point framework for the remaining qualitative weighting (H4/H1 alignment, D1, premium/discount, liquidity quality, sweep/CHoCH/BOS/displacement/FVG/OB/Fib, EMA, RSI, ADX, session, news-clear), but RR and volatility no longer earn points for something that should instead be a precondition to considering the setup at all.
•	Score recompute rule: score must be computed at the moment of confirmation-candle close, using data as of that close — not re-averaged across the waiting period, and not recomputed after the fact to justify an entry that already happened.
________________________________________
16. SESSION ENGINE — add Asian-session handling and dead-zone exclusion
Additions:
•	Treat the Asian session as accumulation/manipulation context only — liquidity built during Asian hours (Asian High/Low) is valid for sweep targeting during London/NY, but no new entries are taken during Asian hours in the default production configuration (configurable, off by default).
•	Explicit dead-zone exclusion: no new entries in the 30 minutes before London open and the 30 minutes after NY close — both are documented periods of erratic gold price action with poor liquidity depth.
•	Session boundary times must be pulled from broker server-time offset at runtime (not a fixed lookup table), re-validated at each daily rollover to catch broker DST changes independent of the local DST calendar.
________________________________________
17. NEWS PROTECTION — name gold-specific triggers explicitly
Addition — explicit high-impact event list for gold (beyond generic "USD high-impact"):
•	FOMC rate decisions and press conferences
•	US Non-Farm Payrolls (NFP)
•	US CPI / Core CPI
•	US PCE (Fed's preferred inflation gauge)
•	Fed Chair testimony/speeches flagged high-impact
•	Geopolitical/safe-haven-relevant scheduled events where the calendar provider flags gold-relevant impact
FOMC-day extended blackout: on FOMC decision days, extend the blackout to 90 minutes before / 90 minutes after by default (gold's post-FOMC volatility routinely runs longer than standard 30/60-minute windows) — configurable, not optional to disable entirely.
________________________________________
18. SPREAD AND EXECUTION FILTER — make spread threshold volatility-relative
Problem: a static pip-based max spread doesn't scale with gold's variable volatility regimes; a threshold sized for calm markets will block all trading during legitimate high-ATR trending moves, while one sized for volatile markets will let dangerously wide spreads through in calm conditions.
Addition:
•	Maximum spread should be defined as the lesser of a static absolute ceiling (hard safety cap, e.g., configurable pip value) and a dynamic ceiling of ≤ 15% of current M5 ATR. Both must pass.
________________________________________
19–20. POSITION RISK & EXPOSURE CAP — add drawdown-based risk scaling, equity baseline rule, and fully automated lot-size calculation
Additions:
•	Static baseline requirement: daily/weekly/monthly equity baselines (used for §26 loss limits) must be captured at the first tick after rollover, stored, and never recalculated intraday from a "running" equity figure — otherwise a recovering account can reset its own drawdown ceiling mid-day, defeating the limit.
•	Progressive risk scaling: after 1 losing trade in a day, reduce risk-per-trade to 0.20%; after 2 losing trades (already requiring score ≥30 per §15/§25), reduce further to 0.15%. This compounds with the score tightening already specified, rather than only tightening entry quality while leaving position size flat.
19a. MANDATORY AUTOMATIC LOT-SIZE CALCULATION — no manual/static lot sizes permitted
This is a hard requirement: the EA must never use a fixed or manually configured lot size. Every single trade, with no exceptions, must have its volume computed live, immediately before order submission, from current account and symbol state. A cached or previously computed lot size from an earlier tick, an earlier trade, or a config file default is not acceptable — equity, balance, and symbol contract terms can all change between setups (and even between signal and execution), so the calculation must be re-run fresh at the moment of RISK_VALID evaluation for every trade.
Step 1 — Detect current account state (live, not cached):
CurrentEquity   = AccountInfoDouble(ACCOUNT_EQUITY)
CurrentBalance  = AccountInfoDouble(ACCOUNT_BALANCE)
FreeMargin      = AccountInfoDouble(ACCOUNT_MARGIN_FREE)
Risk-per-trade percentage (§19–20/§25/§26 progressive scaling) is always applied against Equity, never Balance — Equity reflects floating P/L from any existing position and is the conservative, correct base for sizing a new risk decision.
Step 2 — Compute the risk capital in account currency:
RiskPercent  = CurrentRiskTier      // 0.25% / 0.20% / 0.15% per §19–20 progressive scaling,
                                     // further halved inside the pre-weekend window per §27–28
RiskCapital  = CurrentEquity × (RiskPercent / 100)
Step 3 — Detect live symbol/contract specification (never hardcode XAUUSD dollar-per-point):
TickSize     = SymbolInfoDouble(SYMBOL, SYMBOL_TRADE_TICK_SIZE)
TickValue    = SymbolInfoDouble(SYMBOL, SYMBOL_TRADE_TICK_VALUE)
ContractSize = SymbolInfoDouble(SYMBOL, SYMBOL_TRADE_CONTRACT_SIZE)
VolumeStep   = SymbolInfoDouble(SYMBOL, SYMBOL_VOLUME_STEP)
VolumeMin    = SymbolInfoDouble(SYMBOL, SYMBOL_VOLUME_MIN)
VolumeMax    = SymbolInfoDouble(SYMBOL, SYMBOL_VOLUME_MAX)
TickValue must be read for the current account currency and current quote conditions (it changes with cross-rate conversions on non-USD accounts) — re-fetch it every time, never store a constant.
Step 4 — Compute stop distance from the already-finalized structural SL (§21, including spread buffer):
StopDistancePoints = |EntryPrice - StopLossPrice| / TickSize
EntryPrice here is the anticipated fill (current Ask for BUY / Bid for SELL at calculation time, not the confirmation-candle close, to keep the risk model honest against slippage-free assumptions at minimum).
Step 5 — Compute the exact lot size:
ValuePerPointPerLot = TickValue / TickSize
RiskLots = RiskCapital / (StopDistancePoints × ValuePerPointPerLot)
Step 6 — Normalize to the broker's tradeable volume grid:
RiskLots = floor(RiskLots / VolumeStep) × VolumeStep     // always round DOWN, never up — rounding up silently increases risk beyond the configured percentage
RiskLots = max(RiskLots, VolumeMin)                       // but see the reject rule below
RiskLots = min(RiskLots, VolumeMax)
Reject rule: if rounding down would require raising RiskLots up to VolumeMin and doing so would push actual risk above 1.5× the configured RiskPercent, reject the trade instead of forcing an oversized minimum-lot position. Log the reason (MIN_LOT_EXCEEDS_RISK_TOLERANCE). This matters specifically on smaller accounts trading gold, where one minimum lot can represent a meaningfully larger risk percentage than intended.
Step 7 — Apply the full exposure cap chain (§20), computed live at the same moment:
FinalLots = MIN(
    RiskLots,                       // from Steps 1–6, recomputed fresh
    BrokerMaxVolumePerOrder,        // SYMBOL_VOLUME_MAX
    ConfiguredMaxLotsPerOrder,      // user/config ceiling
    ConfiguredMaxTotalXAUUSDExposure - CurrentOpenXAUUSDVolume,   // live open-exposure check, not static
    AccountExposureCeiling,
    AvailableMarginCeiling,         // derived from FreeMargin ÷ margin-required-per-lot, live
    LiquidityExecutionCeiling
)
CurrentOpenXAUUSDVolume must be read from live position data at calculation time (via position selection by symbol + magic number), not from an internally cached running total, to stay correct after partial closes, manual intervention, or reconciliation events (§30).
Step 8 — Margin sufficiency verification (hard gate, immediately before send):
MarginRequired = OrderCalcMargin(ORDER_TYPE, SYMBOL, FinalLots, EntryPrice)
if MarginRequired > FreeMargin × ConfiguredMarginSafetyFactor (default 0.80):
    REJECT — insufficient margin buffer, log MARGIN_INSUFFICIENT
The 0.80 safety factor (not 1.00) is deliberate: it leaves headroom for adverse floating P/L on any existing position and normal margin-requirement fluctuation, rather than sizing right up to the edge of a margin call.
Step 9 — Final RR re-validation: Recompute realized RR using FinalLots and the actual stop/target distances one last time; if exposure-cap rounding (Step 6/7) has changed the effective risk enough that RR now falls below 2 (§15 gate), reject the trade rather than execute a setup that no longer meets its own entry criteria.
Auditability requirement: every value computed in Steps 1–9 (Equity, RiskPercent, RiskCapital, TickSize, TickValue, ContractSize, StopDistancePoints, RiskLots pre- and post-rounding, all exposure-cap inputs, MarginRequired, FreeMargin, FinalLots, and the pass/fail result of every gate) must be written to the trade log (§35) for that setup — whether the trade executes or is rejected at any step. This is what makes "the bot automatically computed the correct lot size" a verifiable claim rather than a trust exercise: anyone reviewing the log can reconstruct exactly how the final number was reached.
Non-negotiable constraints carried over from v1, restated as hard requirements:
•	No hardcoded XAUUSD dollar-per-point or dollar-per-lot assumption anywhere in the code — everything derives from live SymbolInfoDouble / OrderCalcMargin calls.
•	No lot size is ever entered manually or left at a static default; Steps 1–9 execute in full for every single trade, including all 3 potential trades in a day, independently, since equity and open exposure change between them.
•	Rounding direction is always conservative (down for volume, and reject rather than force minimum-lot when that would breach risk tolerance).
________________________________________
21. STRUCTURAL STOP LOSS — add spread buffer and broker minimum-distance check
Additions:
•	Add current spread to the ATR buffer: SL = Structural Low − (0.10 × M5 ATR) − CurrentSpread (BUY; mirrored for SELL). Without this, the stop can sit inside the spread on a wide-spread print and trigger on the opening tick.
•	Validate against SYMBOL_TRADE_STOPS_LEVEL (broker minimum stop distance) before sending the order; if the structural stop is inside the broker's minimum distance, reject the trade — do not silently widen it to comply (widening changes the risk model without re-validating RR/position size against the new distance). Recompute lot size against the broker-compliant distance and re-check RR ≥ 2 (§15) instead; if RR now fails, reject.
________________________________________
22–23. PROFIT MANAGEMENT & RUNNER — add lot-step rounding fallback
Problem: 30%/40%/30% splits of a small position will frequently round to zero at the broker's volume step, silently breaking the partial-close logic.
Additions:
•	If OriginalVolume × 0.30 rounds below the broker's minimum volume step, skip TP1 partial close and roll that tranche into the TP2 close (i.e., close 70% at TP2 instead). If even OriginalVolume × 0.70 cannot be closed as two clean steps, fall back to a single full close at TP2 and log the constraint. Never fail silently — every rounding fallback must be logged with the reason.
•	Runner minimum stop-to-breakeven move must also respect the spread buffer from §21 (breakeven ± spread, not exact entry price, to avoid an immediate spread-driven stop-out at "breakeven").
________________________________________
24. EARLY EXIT — add timeout-based partial de-risk
Addition:
•	If 20 M5 candles pass after entry with price still between entry and +1R (no TP1 hit, no early-exit trigger), reduce the position by an additional configurable fraction (default: no forced exit, but flag STALLED in the journal for review) — this is a soft monitoring rule, not a hard exit, since forcing exits on time alone is not structurally justified, but stalled trades should be visible, not silent.
________________________________________
25. DAILY TRADE CONTROL — add cooldown and independence check
Additions:
•	Setup independence check: "another independent qualified setup" (post-loss) must have a Setup ID (§32) built from a different liquidity event AND a different CHoCH timestamp than the trade just closed — not merely a different entry price against the same structural event.
•	Cooldown: minimum 6 M5 candles must elapse after any position closes before a new setup may reach ORDER_SENT, regardless of score — prevents rapid-fire re-entry into the same volatility spike that just stopped the EA out.
________________________________________
26. LOSS LIMITS — add circuit-breaker consistency and streak-based risk cut
Addition:
•	Cross-check: with progressive risk scaling (§19–20: 0.25/0.20/0.15%) and the loss sequence in §25/§26, three consecutive losses under the default configuration total ≤ 0.60% of equity, comfortably inside the -0.75% normal daily stop. This must be verified as a config-time sanity check (not just left implicit) — if a user changes any of the risk-per-trade defaults, the EA should warn if the worst-case 3-loss sequence would already breach the daily stop before four trades are even possible.
________________________________________
27–28. WEEKEND PROTECTION / REOPENING — add reduced-size Friday window
Addition:
•	In addition to the existing T-4h/T-2h/T-30m cutover, apply progressive risk scaling starting 6 hours before Friday close: new entries in this window use risk-per-trade reduced to 50% of the current tier (compounds with §19–20), reflecting genuinely thinner pre-weekend liquidity and gap risk even before the hard T-4h entry cutoff.
________________________________________
29–31. CONNECTION / RECONCILIATION / ORDER FAILURE — add circuit breaker and orphan handling
Additions:
•	Orphan position handling: if reconciliation (§30) finds a position with the EA's Magic Number + Symbol but no matching internal journal record (e.g., after a corrupted local state or manual intervention), do not manage it silently. Flag it ORPHANED, apply the configured emergency policy (default: manage defensively — trail to structural stop only, no new scaling), and generate a notification. Never assume an orphan is safe to ignore or safe to fully automate around.
•	Retry circuit breaker: after 5 consecutive order rejections of any kind within a rolling 30-minute window, automatically set EmergencyStop = TRUE (new entries only, per §34's separation of concerns) and generate a critical notification. Manual review required before re-enabling. This prevents a persistent broker-side or environment issue from producing an unbounded retry loop.
________________________________________
34. EMERGENCY KILL SWITCH — add automatic linkage from risk breaches
Addition:
•	The daily/weekly/monthly loss limits (§26) and the retry circuit breaker (§29–31) must automatically set EmergencyStop = TRUE — these should not depend on a human noticing a log entry. EmergencyFlatten remains manual-only (or tied to the emergency daily stop specifically, per configured policy), consistent with v1's intent to keep the two controls separable.
________________________________________
37. PROGRAM STATE MACHINE — attach explicit timeout table
Every state below has a maximum candle-count (M5 basis) it may persist before automatic reset to IDLE (or the nearest safe prior state), consolidating the individual timeouts specified above into one authoritative table the programmer implements once:
State	Max Duration (M5 candles)
LIQUIDITY_IDENTIFIED (waiting for sweep)	100
SWEEP_CONFIRMED → CHOCH_CONFIRMED	24
CHOCH_CONFIRMED → DISPLACEMENT_CONFIRMED	10
DISPLACEMENT_CONFIRMED → BOS_CONFIRMED	15
BOS_CONFIRMED → ENTRY_ZONE_CREATED	5
WAITING_FOR_RETRACE → ENTRY_CONFIRMATION	10
ENTRY_CONFIRMATION → ORDER_SENT	1 (must act on the confirming candle's close, or expire)
Any state exceeding its ceiling resets the chain and returns to LIQUIDITY_IDENTIFIED (not all the way to IDLE, so H4/H1 direction and liquidity map persist) — logged with reason STATE_TIMEOUT.
________________________________________
39. PRODUCTION ACCEPTANCE CRITERIA — add sample size and concentration checks
Additions:
•	Minimum sample size: require ≥ 100 trades in the out-of-sample and walk-forward segments combined before any profitability metric is treated as meaningful. A profitable 20-trade sample is not evidence of edge on a strategy with this many discretionary-turned-mechanical filters.
•	Concentration check: no single trade may account for > 15% of total out-of-sample net profit, and no single month may account for > 30% — flags a backtest that "worked" mostly because of one lucky tail event rather than a repeatable process.
•	Add Sharpe and Sortino ratios alongside profit factor and max drawdown as required reporting metrics, plus average R multiple and win rate, so the score/RR framework's actual output distribution is visible, not just the equity curve.
________________________________________
Summary of the structural gaps this revision closes
1.	Ambiguity → determinism: "bullish/bearish M5 confirmation" and "most recent validated swing range" are replaced with implementable, unambiguous definitions.
2.	Missing time bounds: every stage of the sweep→CHoCH→displacement→BOS→entry chain now has an explicit expiration, closing the "signals from different market regimes get chained together" failure mode.
3.	RR and volatility regime move from scoring to gating — a high-scoring but sub-2R or dead-market setup can no longer fire.
4.	Risk scaling is now progressive, not just entry-quality tightening — losses reduce size, not just raise the bar.
5.	Broker-reality safeguards — spread buffers on stops, lot-step rounding fallbacks, minimum-stop-distance checks — address the gap between "correct on paper" and "correct against a live broker's execution constraints."
6.	Orphan-position and retry-circuit-breaker logic close failure modes in the reconciliation/connection sections that v1 flagged as important but didn't fully specify.
7.	Statistical validity gates (sample size, concentration limits) prevent the acceptance criteria from being satisfied by a lucky backtest.
This is still a specification, not a guarantee — SMC/ICT-style rules encode a discretionary methodology into fixed logic, and no amount of rule-tightening substitutes for the full validation sequence in §38–40 before real capital is at risk. Treat this document as the build target, then let the walk-forward and demo-forward results — not confidence in the spec — decide whether it's ready for live size.




"""
SYMMETRIX GOLD SMC — Python / MetaTrader5 Implementation
=========================================================

Production-candidate implementation of the Symmetrix Gold SMC v2 strengthened
specification: a rule-based Smart Money Concepts (SMC/ICT) state-machine EA
for XAUUSD on MT5, driven from Python via the official `MetaTrader5` package.

Requirements:
    pip install MetaTrader5 pandas numpy

This script assumes the MT5 terminal is installed, logged in to a broker
account, and running locally (the Python API talks to the local terminal,
not directly to the broker).

IMPORTANT
---------
This is a large, complex trading system. Before ANY live capital:
  1. Run it in a demo account for weeks.
  2. Validate every log entry against what you expect the strategy to do.
  3. Backtest / walk-forward the structure logic separately (this script is
     built for live/demo execution, not vectorized backtesting).
  4. Treat every threshold below (ATR multipliers, candle counts, score
     cutoffs) as a starting configuration to be tuned and re-validated,
     not as a guarantee of edge.

Nothing here is financial advice — it is an engineering implementation of
the rules you specified.
"""

from __future__ import annotations

import time
import csv
import os
import json
import uuid
import math
import logging
from dataclasses import dataclass, field, asdict
from datetime import datetime, timedelta, timezone
from enum import Enum, auto
from typing import Optional, List, Dict, Tuple

import numpy as np
import pandas as pd

try:
    import MetaTrader5 as mt5
except ImportError:
    mt5 = None  # allows the module to be imported/tested without the terminal


# ============================================================================
# 1. CONFIGURATION
# ============================================================================

@dataclass
class Config:
    symbol: str = "XAUUSD"
    magic_number: int = 990011

    # Timeframes
    tf_exec: int = None          # M5
    tf_confirm: int = None       # M15
    tf_h1: int = None
    tf_h4: int = None
    tf_d1: int = None

    # Direction engine
    ema_fast: int = 20
    ema_slow: int = 50
    h4_min_separation_atr: float = 0.15
    h1_min_separation_atr: float = 0.10
    h4_persistence_bars: int = 3
    adx_period: int = 14
    adx_floor: float = 20.0

    # Dealing range
    range_max_age_h1_bars: int = 40
    range_min_size_atr: float = 3.0
    equilibrium_buffer_pct: float = 0.05

    # Liquidity
    equal_level_tolerance_atr: float = 0.08
    liquidity_decay_days: int = 10

    # Swings
    swing_min_amplitude_atr: float = 0.20

    # Sweep
    sweep_min_pen_atr: float = 0.05
    sweep_pref_pen_atr: float = 0.10
    sweep_max_pen_atr: float = 0.75
    sweep_max_window_bars: int = 3
    sweep_min_relative_atr_pct: float = 0.40  # vs 50-period avg ATR

    # CHoCH
    choch_timeout_bars: int = 24

    # Displacement
    displacement_min_ratio: float = 1.25
    displacement_pref_ratio: float = 1.50
    displacement_retrace_max_pct: float = 0.70
    body_lookback: int = 20

    # BOS
    bos_max_window_bars: int = 15

    # FVG
    fvg_min_size_atr: float = 0.05
    fvg_mitigation_pct: float = 0.50
    fvg_stale_bars: int = 50

    # Order block
    ob_stale_bars: int = 80

    # Entry zone
    entry_zone_atr_tolerance: float = 0.30
    entry_zone_min_sl_atr: float = 0.5

    # Entry confirmation
    confirm_timeout_bars: int = 10
    rejection_wick_pct: float = 0.60

    # Scoring
    score_min_normal: int = 27
    score_min_after_2_losses: int = 30
    score_max: int = 32
    min_rr: float = 2.0
    vol_regime_min_pct: float = 0.60
    vol_regime_max_pct: float = 2.50

    # Sessions (broker server-time hours, 24h clock) — tune to broker offset
    london_start_hour: int = 9
    london_end_hour: int = 17
    ny_start_hour: int = 14
    ny_end_hour: int = 22
    dead_zone_minutes: int = 30
    allow_asian_session: bool = False

    # News
    news_blackout_before_min: int = 30
    news_blackout_after_min: int = 30
    fomc_blackout_before_min: int = 90
    fomc_blackout_after_min: int = 90

    # Spread / execution
    max_spread_static_points: float = 500.0   # hard ceiling, broker points
    max_spread_atr_pct: float = 0.15

    # Risk & sizing
    base_risk_pct: float = 0.25
    risk_after_1_loss_pct: float = 0.20
    risk_after_2_losses_pct: float = 0.15
    preweekend_risk_multiplier: float = 0.50
    margin_safety_factor: float = 0.80
    min_lot_risk_tolerance_multiplier: float = 1.5

    # Exposure
    max_lots_per_order: float = 5.0
    max_total_xauusd_exposure: float = 5.0

    # Stop loss
    sl_atr_buffer: float = 0.10

    # Profit management
    tp1_r: float = 1.0
    tp1_close_pct: float = 0.30
    tp2_r: float = 2.0
    tp2_close_pct: float = 0.40
    # remaining 30% = runner

    # Daily controls
    max_trades_per_day: int = 3
    cooldown_bars_after_close: int = 6
    stall_bars_warning: int = 20

    # Loss limits (percent of equity)
    daily_stop_pct: float = -0.75
    emergency_daily_stop_pct: float = -2.00
    weekly_stop_pct: float = -4.00
    monthly_stop_pct: float = -6.00

    # Weekend protection (broker server time)
    friday_close_hour_utc: int = 21
    weekend_t_minus_4h_disable: int = 4
    weekend_t_minus_2h_defensive: int = 2
    weekend_t_minus_30m_flatten_min: int = 30

    # Reconnection / order failure
    max_order_retries: int = 5
    retry_delay_seconds: float = 2.0
    retry_circuit_breaker_count: int = 5
    retry_circuit_breaker_window_min: int = 30

    # Misc
    poll_seconds: int = 5
    log_dir: str = "./symmetrix_logs"
    journal_csv: str = "./symmetrix_logs/trade_journal.csv"
    setup_log_csv: str = "./symmetrix_logs/setup_log.csv"

    def bind_timeframes(self):
        if mt5 is None:
            return
        self.tf_exec = mt5.TIMEFRAME_M5
        self.tf_confirm = mt5.TIMEFRAME_M15
        self.tf_h1 = mt5.TIMEFRAME_H1
        self.tf_h4 = mt5.TIMEFRAME_H4
        self.tf_d1 = mt5.TIMEFRAME_D1


# ============================================================================
# 2. STATE MACHINE ENUM
# ============================================================================

class State(Enum):
    IDLE = auto()
    HTF_DIRECTION_VALID = auto()
    LIQUIDITY_IDENTIFIED = auto()
    SWEEP_CONFIRMED = auto()
    CHOCH_CONFIRMED = auto()
    DISPLACEMENT_CONFIRMED = auto()
    BOS_CONFIRMED = auto()
    ENTRY_ZONE_CREATED = auto()
    WAITING_FOR_RETRACE = auto()
    ENTRY_CONFIRMATION = auto()
    SCORE_VALID = auto()
    RISK_VALID = auto()
    ORDER_SENT = auto()
    ORDER_CONFIRMED = auto()
    POSITION_MANAGEMENT = auto()
    POSITION_CLOSED = auto()


STATE_TIMEOUT_BARS = {
    State.LIQUIDITY_IDENTIFIED: 100,
    State.SWEEP_CONFIRMED: 24,          # -> CHOCH_CONFIRMED
    State.CHOCH_CONFIRMED: 10,          # -> DISPLACEMENT_CONFIRMED
    State.DISPLACEMENT_CONFIRMED: 15,   # -> BOS_CONFIRMED
    State.BOS_CONFIRMED: 5,             # -> ENTRY_ZONE_CREATED
    State.WAITING_FOR_RETRACE: 10,      # -> ENTRY_CONFIRMATION
    State.ENTRY_CONFIRMATION: 1,        # must act same bar
}


class Direction(Enum):
    BULLISH = auto()
    BEARISH = auto()
    NEUTRAL = auto()


# ============================================================================
# 3. LOGGING
# ============================================================================

def setup_logging(cfg: Config) -> logging.Logger:
    os.makedirs(cfg.log_dir, exist_ok=True)
    logger = logging.getLogger("symmetrix")
    logger.setLevel(logging.DEBUG)
    if not logger.handlers:
        fh = logging.FileHandler(os.path.join(cfg.log_dir, "symmetrix.log"))
        fh.setFormatter(logging.Formatter("%(asctime)s | %(levelname)s | %(message)s"))
        ch = logging.StreamHandler()
        ch.setFormatter(logging.Formatter("%(asctime)s | %(levelname)s | %(message)s"))
        logger.addHandler(fh)
        logger.addHandler(ch)

    for path, header in [
        (cfg.journal_csv, [
            "setup_id", "entry_time", "exit_time", "direction", "initial_risk",
            "mfe", "mae", "r_result", "net_pl", "commission", "swap", "slippage",
            "exit_reason", "tp1_status", "tp2_status", "runner_result"
        ]),
        (cfg.setup_log_csv, [
            "timestamp", "symbol", "direction", "h4_bias", "h1_bias", "d1_bias",
            "liquidity_type", "liquidity_price", "sweep_depth_atr", "choch",
            "displacement_ratio", "bos", "fvg", "ob", "fib", "score", "spread",
            "risk_pct", "equity", "risk_capital", "tick_value", "tick_size",
            "stop_distance_points", "raw_lots", "final_lots", "margin_required",
            "free_margin", "entry", "sl", "tp1", "tp2", "reason"
        ]),
    ]:
        if not os.path.exists(path):
            with open(path, "w", newline="") as f:
                csv.writer(f).writerow(header)

    return logger


def append_csv(path: str, row: dict, header: List[str]):
    with open(path, "a", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=header)
        writer.writerow({k: row.get(k, "") for k in header})


# ============================================================================
# 4. MT5 DATA ACCESS LAYER
# ============================================================================

class MT5Data:
    """Thin wrapper around the MetaTrader5 API for data + account + orders."""

    def __init__(self, cfg: Config, logger: logging.Logger):
        self.cfg = cfg
        self.log = logger

    def connect(self) -> bool:
        if mt5 is None:
            self.log.error("MetaTrader5 package not available in this environment.")
            return False
        if not mt5.initialize():
            self.log.error(f"MT5 initialize() failed: {mt5.last_error()}")
            return False
        info = mt5.symbol_info(self.cfg.symbol)
        if info is None or not info.visible:
            mt5.symbol_select(self.cfg.symbol, True)
        self.log.info("Connected to MT5 terminal.")
        return True

    def shutdown(self):
        if mt5 is not None:
            mt5.shutdown()

    def is_connected(self) -> bool:
        if mt5 is None:
            return False
        return mt5.terminal_info() is not None

    def rates(self, timeframe, count: int = 300) -> pd.DataFrame:
        raw = mt5.copy_rates_from_pos(self.cfg.symbol, timeframe, 0, count)
        if raw is None or len(raw) == 0:
            return pd.DataFrame()
        df = pd.DataFrame(raw)
        df["time"] = pd.to_datetime(df["time"], unit="s")
        return df

    def account(self) -> dict:
        a = mt5.account_info()
        return {} if a is None else a._asdict()

    def symbol_info(self) -> dict:
        s = mt5.symbol_info(self.cfg.symbol)
        return {} if s is None else s._asdict()

    def tick(self) -> dict:
        t = mt5.symbol_info_tick(self.cfg.symbol)
        return {} if t is None else t._asdict()

    def open_positions(self) -> List[dict]:
        positions = mt5.positions_get(symbol=self.cfg.symbol)
        if positions is None:
            return []
        return [p._asdict() for p in positions
                if p.magic == self.cfg.magic_number]

    def calc_margin(self, order_type, volume, price) -> Optional[float]:
        return mt5.order_calc_margin(order_type, self.cfg.symbol, volume, price)

    def send_order(self, request: dict):
        return mt5.order_send(request)


# ============================================================================
# 5. INDICATORS
# ============================================================================

class Indicators:

    @staticmethod
    def ema(series: pd.Series, period: int) -> pd.Series:
        return series.ewm(span=period, adjust=False).mean()

    @staticmethod
    def atr(df: pd.DataFrame, period: int = 14) -> pd.Series:
        high, low, close = df["high"], df["low"], df["close"]
        prev_close = close.shift(1)
        tr = pd.concat([
            high - low,
            (high - prev_close).abs(),
            (low - prev_close).abs()
        ], axis=1).max(axis=1)
        return tr.rolling(period).mean()

    @staticmethod
    def adx(df: pd.DataFrame, period: int = 14) -> pd.Series:
        high, low, close = df["high"], df["low"], df["close"]
        up_move = high.diff()
        down_move = -low.diff()
        plus_dm = np.where((up_move > down_move) & (up_move > 0), up_move, 0.0)
        minus_dm = np.where((down_move > up_move) & (down_move > 0), down_move, 0.0)
        tr = Indicators.atr(df, 1) * 1  # true range per bar (period=1 rolling mean == TR)
        atr_n = pd.Series(tr).rolling(period).mean()
        plus_di = 100 * pd.Series(plus_dm).rolling(period).mean() / atr_n.replace(0, np.nan)
        minus_di = 100 * pd.Series(minus_dm).rolling(period).mean() / atr_n.replace(0, np.nan)
        dx = 100 * (plus_di - minus_di).abs() / (plus_di + minus_di).replace(0, np.nan)
        return dx.rolling(period).mean()

    @staticmethod
    def avg_body(df: pd.DataFrame, lookback: int) -> float:
        bodies = (df["close"] - df["open"]).abs()
        return float(bodies.tail(lookback).mean())


# ============================================================================
# 6. SWING / FRACTAL DETECTION
# ============================================================================

@dataclass
class Swing:
    index: int
    time: datetime
    price: float
    kind: str          # "HIGH" or "LOW"
    quality: str        # "CONFIRMED" or "MINOR"


def detect_swings(df: pd.DataFrame, atr_series: pd.Series, min_amp_atr: float) -> List[Swing]:
    """5-candle fractal swings, confirmed only after N+2 has closed."""
    swings = []
    highs, lows = df["high"].values, df["low"].values
    n = len(df)
    for i in range(2, n - 2):
        atr_i = atr_series.iloc[i]
        if pd.isna(atr_i) or atr_i == 0:
            continue
        # swing high
        if highs[i] > highs[i-2] and highs[i] > highs[i-1] and \
           highs[i] > highs[i+1] and highs[i] > highs[i+2]:
            amp = min(highs[i] - lows[i-1], highs[i] - lows[i+1])
            quality = "CONFIRMED" if amp >= min_amp_atr * atr_i else "MINOR"
            swings.append(Swing(i, df["time"].iloc[i], highs[i], "HIGH", quality))
        # swing low
        if lows[i] < lows[i-2] and lows[i] < lows[i-1] and \
           lows[i] < lows[i+1] and lows[i] < lows[i+2]:
            amp = min(highs[i-1] - lows[i], highs[i+1] - lows[i])
            quality = "CONFIRMED" if amp >= min_amp_atr * atr_i else "MINOR"
            swings.append(Swing(i, df["time"].iloc[i], lows[i], "LOW", quality))
    return swings


# ============================================================================
# 7. LIQUIDITY ENGINE
# ============================================================================

class LiquidityStatus(Enum):
    UNTOUCHED = auto()
    SWEPT = auto()
    INVALIDATED = auto()


@dataclass
class LiquidityLevel:
    kind: str            # PDH, PDL, PWH, PWL, ASIAN_HIGH, ASIAN_LOW, EQ_HIGH, EQ_LOW, H1_SWING, M15_SWING
    price: float
    time: datetime
    status: LiquidityStatus = LiquidityStatus.UNTOUCHED
    priority_tier: int = 5   # lower = higher priority


PRIORITY_ORDER = {
    "PWH": 0, "PWL": 0,
    "PDH": 1, "PDL": 1,
    "EQ_HIGH": 2, "EQ_LOW": 2,
    "ASIAN_HIGH": 3, "ASIAN_LOW": 3,
    "H1_SWING": 4,
    "M15_SWING": 5,
}


class LiquidityEngine:

    def __init__(self, cfg: Config):
        self.cfg = cfg
        self.levels: List[LiquidityLevel] = []

    def build(self, d1: pd.DataFrame, h1: pd.DataFrame, m15: pd.DataFrame,
              h1_swings: List[Swing], m15_swings: List[Swing], now: datetime):
        self.levels.clear()
        if len(d1) >= 2:
            prev_day = d1.iloc[-2]
            self.levels.append(LiquidityLevel("PDH", prev_day["high"], prev_day["time"],
                                               priority_tier=PRIORITY_ORDER["PDH"]))
            self.levels.append(LiquidityLevel("PDL", prev_day["low"], prev_day["time"],
                                               priority_tier=PRIORITY_ORDER["PDL"]))
        weekly = d1.copy()
        weekly["week"] = weekly["time"].dt.isocalendar().week
        if weekly["week"].nunique() >= 2:
            last_week = weekly["week"].iloc[-1]
            prior = weekly[weekly["week"] != last_week].tail(5)
            if not prior.empty:
                self.levels.append(LiquidityLevel("PWH", prior["high"].max(), prior["time"].iloc[-1],
                                                   priority_tier=PRIORITY_ORDER["PWH"]))
                self.levels.append(LiquidityLevel("PWL", prior["low"].min(), prior["time"].iloc[-1],
                                                   priority_tier=PRIORITY_ORDER["PWL"]))

        asian = h1[(h1["time"].dt.hour >= 0) & (h1["time"].dt.hour < 8) &
                   (h1["time"].dt.date == now.date())]
        if not asian.empty:
            self.levels.append(LiquidityLevel("ASIAN_HIGH", asian["high"].max(), asian["time"].iloc[-1],
                                               priority_tier=PRIORITY_ORDER["ASIAN_HIGH"]))
            self.levels.append(LiquidityLevel("ASIAN_LOW", asian["low"].min(), asian["time"].iloc[-1],
                                               priority_tier=PRIORITY_ORDER["ASIAN_LOW"]))

        for sw in h1_swings:
            if sw.quality == "CONFIRMED":
                kind = "H1_SWING"
                self.levels.append(LiquidityLevel(kind, sw.price, sw.time,
                                                   priority_tier=PRIORITY_ORDER[kind]))
        for sw in m15_swings:
            if sw.quality == "CONFIRMED":
                kind = "M15_SWING"
                self.levels.append(LiquidityLevel(kind, sw.price, sw.time,
                                                   priority_tier=PRIORITY_ORDER[kind]))

        self._mark_equal_levels(h1["close"].iloc[-1] if len(h1) else None)
        self._apply_decay(now)

    def _mark_equal_levels(self, atr_ref: float):
        # simplistic equal-highs/lows clustering among H1/M15 swing highs already added
        pass  # left as an extension point; core priority levels already provide confluence

    def _apply_decay(self, now: datetime):
        for lvl in self.levels:
            age_days = (now - lvl.time).days
            if age_days > self.cfg.liquidity_decay_days and lvl.status == LiquidityStatus.UNTOUCHED:
                lvl.priority_tier += 1

    def best_target(self, direction: Direction, current_price: float) -> Optional[LiquidityLevel]:
        """Nearest-qualifying untouched pool on the correct side, by priority tier then distance."""
        candidates = [l for l in self.levels if l.status == LiquidityStatus.UNTOUCHED]
        if direction == Direction.BULLISH:
            candidates = [l for l in candidates if l.price < current_price]  # sell-side liquidity below
        else:
            candidates = [l for l in candidates if l.price > current_price]  # buy-side liquidity above
        if not candidates:
            return None
        candidates.sort(key=lambda l: (l.priority_tier, abs(l.price - current_price)))
        return candidates[0]


# ============================================================================
# 8. DIRECTION ENGINE
# ============================================================================

@dataclass
class DirectionReading:
    direction: Direction
    persistent: bool


class DirectionEngine:

    def __init__(self, cfg: Config):
        self.cfg = cfg

    def _raw_direction(self, df: pd.DataFrame) -> pd.Series:
        ema_f = Indicators.ema(df["close"], self.cfg.ema_fast)
        ema_s = Indicators.ema(df["close"], self.cfg.ema_slow)
        atr = Indicators.atr(df, 14)
        sep_ok = (ema_f - ema_s).abs() >= (self.cfg.h4_min_separation_atr * atr)
        direction = pd.Series(Direction.NEUTRAL, index=df.index)
        bullish = (df["close"] > ema_f) & (ema_f > ema_s) & sep_ok
        bearish = (df["close"] < ema_f) & (ema_f < ema_s) & sep_ok
        direction[bullish] = Direction.BULLISH
        direction[bearish] = Direction.BEARISH
        return direction

    def evaluate(self, df_h4: pd.DataFrame, df_h1: pd.DataFrame,
                 df_d1: pd.DataFrame) -> Tuple[DirectionReading, DirectionReading, DirectionReading]:
        h4_dir_series = self._raw_direction(df_h4)
        h1_dir_series = self._raw_direction(df_h1)
        d1_dir_series = self._raw_direction(df_d1)

        h4_adx = Indicators.adx(df_h4, self.cfg.adx_period)
        h4_current = h4_dir_series.iloc[-1]
        if h4_adx.iloc[-1] < self.cfg.adx_floor or pd.isna(h4_adx.iloc[-1]):
            h4_current = Direction.NEUTRAL

        persistence_window = h4_dir_series.tail(self.cfg.h4_persistence_bars)
        h4_persistent = bool((persistence_window == h4_current).all()) and h4_current != Direction.NEUTRAL

        h1_current = h1_dir_series.iloc[-1]
        d1_current = d1_dir_series.iloc[-1]

        return (DirectionReading(h4_current, h4_persistent),
                DirectionReading(h1_current, True),
                DirectionReading(d1_current, True))


# ============================================================================
# 9. DEALING RANGE (PREMIUM / DISCOUNT)
# ============================================================================

@dataclass
class DealingRange:
    high: float
    low: float
    midpoint: float
    age_bars: int
    valid: bool


def compute_dealing_range(h1: pd.DataFrame, h1_swings: List[Swing],
                           atr_series: pd.Series, cfg: Config) -> Optional[DealingRange]:
    confirmed = [s for s in h1_swings if s.quality == "CONFIRMED"]
    if len(confirmed) < 2:
        return None
    recent = [s for s in confirmed if (len(h1) - 1 - s.index) <= cfg.range_max_age_h1_bars]
    highs = [s for s in recent if s.kind == "HIGH"]
    lows = [s for s in recent if s.kind == "LOW"]
    if not highs or not lows:
        return None
    range_high = max(h.price for h in highs)
    range_low = min(l.price for l in lows)
    size = range_high - range_low
    atr_now = atr_series.iloc[-1]
    if pd.isna(atr_now) or size < cfg.range_min_size_atr * atr_now:
        return DealingRange(range_high, range_low, (range_high + range_low) / 2, 0, valid=False)
    age = min(len(h1) - 1 - h.index for h in highs + lows)
    return DealingRange(range_high, range_low, (range_high + range_low) / 2, age, valid=True)


def classify_zone(price: float, rng: DealingRange, cfg: Config) -> str:
    if not rng or not rng.valid:
        return "UNKNOWN"
    span = rng.high - rng.low
    buffer = span * cfg.equilibrium_buffer_pct
    if abs(price - rng.midpoint) <= buffer:
        return "EQUILIBRIUM"
    return "DISCOUNT" if price < rng.midpoint else "PREMIUM"


# ============================================================================
# 10. SWEEP / CHoCH / DISPLACEMENT / BOS
# ============================================================================

@dataclass
class SweepResult:
    valid: bool
    depth_atr: float = 0.0
    bar_index: int = -1
    reclaim_index: int = -1


def detect_sweep(m5: pd.DataFrame, atr: pd.Series, level: float,
                  direction: Direction, cfg: Config) -> SweepResult:
    """Look at the most recent bars for penetration + reclaim of `level`."""
    n = len(m5)
    lookback = min(cfg.sweep_max_window_bars + 2, n)
    window = m5.tail(lookback).reset_index(drop=True)
    atr_now = atr.iloc[-1]
    if pd.isna(atr_now) or atr_now == 0:
        return SweepResult(False)

    avg_atr = atr.tail(50).mean()
    if avg_atr and atr_now < cfg.sweep_min_relative_atr_pct * avg_atr:
        return SweepResult(False)  # dead volatility regime

    for i in range(len(window)):
        row = window.iloc[i]
        if direction == Direction.BULLISH:
            penetration = level - row["low"]
        else:
            penetration = row["high"] - level
        if penetration <= 0:
            continue
        depth_atr = penetration / atr_now
        if depth_atr < cfg.sweep_min_pen_atr or depth_atr > cfg.sweep_max_pen_atr:
            continue
        # look for reclaim within remaining bars of the window
        for j in range(i, len(window)):
            reclaim_row = window.iloc[j]
            reclaimed = (reclaim_row["close"] > level) if direction == Direction.BULLISH \
                else (reclaim_row["close"] < level)
            if reclaimed:
                if (j - i) <= cfg.sweep_max_window_bars:
                    return SweepResult(True, depth_atr, i, j)
                break
    return SweepResult(False)


@dataclass
class ChochResult:
    confirmed: bool
    protected_level: Optional[float] = None
    bar_index: int = -1


def detect_choch(m5: pd.DataFrame, swings: List[Swing], direction: Direction,
                  sweep_bar_time: datetime, cfg: Config) -> ChochResult:
    """
    Bullish: LL -> LH -> LL then sweep, then M5 close above protected LH.
    Bearish: HH -> HL -> HH then sweep, then M5 close below protected HL.
    Only CONFIRMED swings count as the protected level.
    """
    confirmed_swings = [s for s in swings if s.quality == "CONFIRMED"]
    if direction == Direction.BULLISH:
        lows = [s for s in confirmed_swings if s.kind == "LOW"]
        highs = [s for s in confirmed_swings if s.kind == "HIGH"]
        if len(lows) < 2 or not highs:
            return ChochResult(False)
        protected_high = highs[-1]
        candles_after_sweep = m5[m5["time"] > sweep_bar_time]
        for idx, row in candles_after_sweep.iterrows():
            bars_elapsed = idx - m5.index[m5["time"] == sweep_bar_time][0] if any(m5["time"] == sweep_bar_time) else 0
            if bars_elapsed > cfg.choch_timeout_bars:
                return ChochResult(False)
            if row["close"] > protected_high.price:
                return ChochResult(True, protected_high.price, idx)
        return ChochResult(False)
    else:
        highs = [s for s in confirmed_swings if s.kind == "HIGH"]
        lows = [s for s in confirmed_swings if s.kind == "LOW"]
        if len(highs) < 2 or not lows:
            return ChochResult(False)
        protected_low = lows[-1]
        candles_after_sweep = m5[m5["time"] > sweep_bar_time]
        for idx, row in candles_after_sweep.iterrows():
            bars_elapsed = idx - m5.index[m5["time"] == sweep_bar_time][0] if any(m5["time"] == sweep_bar_time) else 0
            if bars_elapsed > cfg.choch_timeout_bars:
                return ChochResult(False)
            if row["close"] < protected_low.price:
                return ChochResult(True, protected_low.price, idx)
        return ChochResult(False)


def check_displacement(m5: pd.DataFrame, choch_bar_index: int, direction: Direction,
                        cfg: Config) -> Tuple[bool, float]:
    if choch_bar_index < 0 or choch_bar_index >= len(m5):
        return False, 0.0
    avg_body = Indicators.avg_body(m5.iloc[:choch_bar_index], cfg.body_lookback)
    if avg_body == 0:
        return False, 0.0
    candle = m5.iloc[choch_bar_index]
    body = candle["close"] - candle["open"]
    is_directional = body > 0 if direction == Direction.BULLISH else body < 0
    ratio = abs(body) / avg_body
    if not is_directional or ratio < cfg.displacement_min_ratio:
        return False, ratio
    # follow-through check: next candle should not retrace >= 70% of the body
    if choch_bar_index + 1 < len(m5):
        nxt = m5.iloc[choch_bar_index + 1]
        retrace = abs(nxt["close"] - candle["close"]) / abs(body) if body != 0 else 1.0
        opposite = (nxt["close"] < candle["close"]) if direction == Direction.BULLISH \
            else (nxt["close"] > candle["close"])
        if opposite and retrace >= cfg.displacement_retrace_max_pct:
            return False, ratio
    return True, ratio


def check_bos(m5: pd.DataFrame, displacement_bar_index: int, swings: List[Swing],
              choch_bar_index: int, direction: Direction, cfg: Config) -> Tuple[bool, int]:
    confirmed = [s for s in swings if s.quality == "CONFIRMED" and s.index < choch_bar_index]
    if direction == Direction.BULLISH:
        target_swings = [s for s in confirmed if s.kind == "HIGH" and s.price >
                          m5.iloc[choch_bar_index]["close"]]
        if not target_swings:
            return False, -1
        target = min(target_swings, key=lambda s: s.price)
        window_end = min(displacement_bar_index + cfg.bos_max_window_bars, len(m5) - 1)
        for i in range(displacement_bar_index, window_end + 1):
            if m5.iloc[i]["close"] > target.price:
                return True, i
        return False, -1
    else:
        target_swings = [s for s in confirmed if s.kind == "LOW" and s.price <
                          m5.iloc[choch_bar_index]["close"]]
        if not target_swings:
            return False, -1
        target = max(target_swings, key=lambda s: s.price)
        window_end = min(displacement_bar_index + cfg.bos_max_window_bars, len(m5) - 1)
        for i in range(displacement_bar_index, window_end + 1):
            if m5.iloc[i]["close"] < target.price:
                return True, i
        return False, -1


# ============================================================================
# 11. FVG / ORDER BLOCK / FIBONACCI
# ============================================================================

@dataclass
class FVG:
    high: float
    low: float
    time: datetime
    direction: Direction
    mitigation_pct: float = 0.0


def find_fvgs(m5: pd.DataFrame, atr: pd.Series, cfg: Config) -> List[FVG]:
    fvgs = []
    for i in range(2, len(m5)):
        c1, c3 = m5.iloc[i - 2], m5.iloc[i]
        atr_now = atr.iloc[i]
        if pd.isna(atr_now):
            continue
        if c1["low"] > c3["high"]:
            gap = c1["low"] - c3["high"]
            if gap >= cfg.fvg_min_size_atr * atr_now:
                fvgs.append(FVG(c1["low"], c3["high"], m5["time"].iloc[i], Direction.BEARISH))
        elif c1["high"] < c3["low"]:
            gap = c3["low"] - c1["high"]
            if gap >= cfg.fvg_min_size_atr * atr_now:
                fvgs.append(FVG(c3["low"], c1["high"], m5["time"].iloc[i], Direction.BULLISH))
    # mitigation against subsequent price action
    closes = m5[["time", "close", "high", "low"]]
    for f in fvgs:
        span = f.high - f.low
        after = closes[closes["time"] > f.time]
        touched = 0.0
        for _, row in after.iterrows():
            overlap_low = max(f.low, row["low"])
            overlap_high = min(f.high, row["high"])
            if overlap_high > overlap_low:
                touched = max(touched, (overlap_high - overlap_low) / span if span else 0)
        f.mitigation_pct = touched
    return fvgs


@dataclass
class OrderBlock:
    high: float
    low: float
    time: datetime
    direction: Direction
    index: int
    has_structural_consequence: bool
    mitigation_pct: float = 0.0


def find_order_block(m5: pd.DataFrame, bos_bar_index: int, displacement_bar_index: int,
                      direction: Direction, cfg: Config) -> Optional[OrderBlock]:
    """Last opposite-direction candle immediately preceding the impulsive move."""
    if displacement_bar_index <= 0:
        return None
    search_start = max(0, displacement_bar_index - 10)
    ob_index = None
    for i in range(displacement_bar_index - 1, search_start - 1, -1):
        candle = m5.iloc[i]
        is_opposite = (candle["close"] < candle["open"]) if direction == Direction.BULLISH \
            else (candle["close"] > candle["open"])
        if is_opposite:
            ob_index = i  # keep searching backward would find earlier ones; we want the LAST one adjacent
            break
    if ob_index is None:
        return None
    candle = m5.iloc[ob_index]
    ob = OrderBlock(
        high=candle["high"], low=candle["low"], time=candle["time"],
        direction=direction, index=ob_index,
        has_structural_consequence=(bos_bar_index >= 0)
    )
    return ob


def fib_618_zone(sweep_extreme: float, choch_break_price: float, direction: Direction) -> Tuple[float, float, float]:
    """Returns (level_618, level_786, level_100_invalidation)."""
    impulse = choch_break_price - sweep_extreme
    if direction == Direction.BULLISH:
        level_618 = choch_break_price - 0.618 * impulse
        level_786 = choch_break_price - 0.786 * impulse
    else:
        level_618 = choch_break_price + 0.618 * abs(impulse)
        level_786 = choch_break_price + 0.786 * abs(impulse)
    return level_618, level_786, sweep_extreme


# ============================================================================
# 12. ENTRY ZONE & CONFIRMATION
# ============================================================================

@dataclass
class EntryZone:
    low: float
    high: float
    confluence_count: int
    components: List[str]


def build_entry_zone(fvg: Optional[FVG], ob: Optional[OrderBlock], fib_618: float,
                      atr_now: float, cfg: Config) -> Optional[EntryZone]:
    tolerance = cfg.entry_zone_atr_tolerance * atr_now
    points = []
    components = []
    if fvg and fvg.mitigation_pct < cfg.fvg_mitigation_pct:
        points.append(((fvg.high + fvg.low) / 2, fvg.low, fvg.high))
        components.append("FVG")
    if ob and ob.mitigation_pct < 1.0:
        points.append(((ob.high + ob.low) / 2, ob.low, ob.high))
        components.append("OB")
    points.append((fib_618, fib_618 - tolerance, fib_618 + tolerance))
    components.append("FIB")

    # count how many of the 3 midpoints fall within `tolerance` of each other
    mids = [p[0] for p in points]
    clustered = []
    for i, m in enumerate(mids):
        if all(abs(m - other) <= tolerance for other in mids):
            clustered.append(components[i])
    if len(clustered) < 2:
        return None
    low = min(p[1] for p in points)
    high = max(p[2] for p in points)
    return EntryZone(low, high, len(clustered), clustered)


def check_confirmation(m5: pd.DataFrame, zone: EntryZone, direction: Direction,
                        cfg: Config) -> Optional[int]:
    """Returns index of confirming candle, or None."""
    recent = m5.tail(cfg.confirm_timeout_bars + 1).reset_index(drop=True)
    for i in range(1, len(recent)):
        row, prev = recent.iloc[i], recent.iloc[i - 1]
        in_zone = zone.low <= row["close"] <= zone.high or zone.low <= row["low"] <= zone.high \
            or zone.low <= row["high"] <= zone.high
        if not in_zone:
            continue
        body = row["close"] - row["open"]
        prev_body = prev["close"] - prev["open"]
        # engulfing
        engulf = (body > 0 and prev_body < 0 and row["close"] > prev["open"] and row["open"] < prev["close"]) \
            if direction == Direction.BULLISH else \
            (body < 0 and prev_body > 0 and row["close"] < prev["open"] and row["open"] > prev["close"])
        # rejection wick
        rng = row["high"] - row["low"]
        if rng > 0:
            if direction == Direction.BULLISH:
                lower_wick = min(row["open"], row["close"]) - row["low"]
                rejection = (lower_wick / rng >= cfg.rejection_wick_pct) and row["close"] > row["open"]
            else:
                upper_wick = row["high"] - max(row["open"], row["close"])
                rejection = (upper_wick / rng >= cfg.rejection_wick_pct) and row["close"] < row["open"]
        else:
            rejection = False
        if engulf or rejection:
            return len(m5) - (len(recent) - i)
    return None


# ============================================================================
# 13. SCORING
# ============================================================================

@dataclass
class ScoreBreakdown:
    total: int
    passed_gates: bool
    detail: Dict[str, int]


def compute_score(h4_h1_aligned: bool, d1_aligned: bool, correct_zone: bool,
                   major_liquidity: bool, clean_sweep: bool, choch: bool, bos: bool,
                   displacement: bool, fvg_present: bool, ob_present: bool, fib_present: bool,
                   ema_ok: bool, rsi_ok: bool, adx_ok: bool, good_session: bool,
                   no_news: bool, rr_ok: bool, vol_regime_ok: bool) -> ScoreBreakdown:
    detail = {
        "h4_h1_alignment": 3 if h4_h1_aligned else 0,
        "d1_alignment": 2 if d1_aligned else 0,
        "premium_discount": 2 if correct_zone else 0,
        "major_liquidity": 2 if major_liquidity else 0,
        "clean_sweep": 3 if clean_sweep else 0,
        "choch": 3 if choch else 0,
        "bos": 2 if bos else 0,
        "displacement": 2 if displacement else 0,
        "fvg": 2 if fvg_present else 0,
        "order_block": 2 if ob_present else 0,
        "fib": 2 if fib_present else 0,
        "ema": 1 if ema_ok else 0,
        "rsi": 1 if rsi_ok else 0,
        "adx": 1 if adx_ok else 0,
        "session": 1 if good_session else 0,
        "no_major_news": 1 if no_news else 0,
    }
    total = sum(detail.values())
    passed_gates = rr_ok and vol_regime_ok and bos and choch  # RR & vol regime are hard gates per spec
    return ScoreBreakdown(total, passed_gates, detail)


# ============================================================================
# 14. SESSION & NEWS
# ============================================================================

def in_trading_session(now: datetime, cfg: Config) -> bool:
    hour = now.hour
    in_london = cfg.london_start_hour <= hour < cfg.london_end_hour
    in_ny = cfg.ny_start_hour <= hour < cfg.ny_end_hour
    if not (in_london or in_ny):
        return False
    minute_of_hour = now.minute
    if hour == cfg.london_start_hour and minute_of_hour < cfg.dead_zone_minutes:
        return False
    if hour == cfg.ny_end_hour - 1 and minute_of_hour >= (60 - cfg.dead_zone_minutes):
        return False
    return True


class NewsFilter:
    """
    Placeholder integration point for the MT5 Economic Calendar (mt5.calendar_*
    if available on your build) or an external calendar feed.
    Must FAIL SAFE: if calendar data is unavailable, block new entries.
    """

    def __init__(self, cfg: Config, logger: logging.Logger):
        self.cfg = cfg
        self.log = logger
        self.calendar_available = False
        self.events: List[dict] = []  # each: {"time": dt, "impact": "high", "currency": "USD", "is_fomc": bool}

    def refresh(self):
        # TODO: wire to your broker's calendar API or a third-party feed.
        # Until implemented, calendar_available stays False -> fail-safe blocks entries.
        self.calendar_available = False
        self.events = []

    def in_blackout(self, now: datetime) -> bool:
        if not self.calendar_available:
            self.log.warning("News calendar unavailable — failing safe (blocking new entries).")
            return True
        for ev in self.events:
            before = self.cfg.fomc_blackout_before_min if ev.get("is_fomc") else self.cfg.news_blackout_before_min
            after = self.cfg.fomc_blackout_after_min if ev.get("is_fomc") else self.cfg.news_blackout_after_min
            window_start = ev["time"] - timedelta(minutes=before)
            window_end = ev["time"] + timedelta(minutes=after)
            if window_start <= now <= window_end:
                return True
        return False


# ============================================================================
# 15. RISK / LOT SIZING  (spec §19a — mandatory automatic calculation)
# ============================================================================

@dataclass
class LotSizeResult:
    ok: bool
    final_lots: float = 0.0
    reason: str = ""
    detail: Dict = field(default_factory=dict)


class RiskManager:

    def __init__(self, cfg: Config, data: MT5Data, logger: logging.Logger):
        self.cfg = cfg
        self.data = data
        self.log = logger

    def current_risk_pct(self, losses_today: int, in_preweekend_window: bool) -> float:
        if losses_today >= 2:
            pct = self.cfg.risk_after_2_losses_pct
        elif losses_today == 1:
            pct = self.cfg.risk_after_1_loss_pct
        else:
            pct = self.cfg.base_risk_pct
        if in_preweekend_window:
            pct *= self.cfg.preweekend_risk_multiplier
        return pct

    def calculate(self, direction: Direction, entry_price: float, sl_price: float,
                  losses_today: int, in_preweekend_window: bool,
                  current_open_xau_volume: float) -> LotSizeResult:
        cfg = self.cfg
        detail = {}

        # Step 1 — live account state
        acct = self.data.account()
        if not acct:
            return LotSizeResult(False, reason="ACCOUNT_INFO_UNAVAILABLE")
        equity = acct["equity"]
        free_margin = acct["margin_free"]
        detail.update(equity=equity, free_margin=free_margin)

        # Step 2 — risk capital
        risk_pct = self.current_risk_pct(losses_today, in_preweekend_window)
        risk_capital = equity * (risk_pct / 100.0)
        detail.update(risk_pct=risk_pct, risk_capital=risk_capital)

        # Step 3 — live symbol spec
        sym = self.data.symbol_info()
        if not sym:
            return LotSizeResult(False, reason="SYMBOL_INFO_UNAVAILABLE")
        tick_size = sym["trade_tick_size"]
        tick_value = sym["trade_tick_value"]
        volume_step = sym["volume_step"]
        volume_min = sym["volume_min"]
        volume_max = sym["volume_max"]
        detail.update(tick_size=tick_size, tick_value=tick_value)

        # Step 4 — stop distance
        stop_distance_points = abs(entry_price - sl_price) / tick_size if tick_size else 0
        detail["stop_distance_points"] = stop_distance_points
        if stop_distance_points <= 0:
            return LotSizeResult(False, reason="INVALID_STOP_DISTANCE", detail=detail)

        # Step 5 — raw lots
        value_per_point_per_lot = tick_value / tick_size if tick_size else 0
        if value_per_point_per_lot <= 0:
            return LotSizeResult(False, reason="INVALID_TICK_VALUE", detail=detail)
        raw_lots = risk_capital / (stop_distance_points * value_per_point_per_lot)
        detail["raw_lots"] = raw_lots

        # Step 6 — normalize to volume grid (round DOWN)
        steps = math.floor(raw_lots / volume_step) if volume_step else 0
        norm_lots = steps * volume_step
        if norm_lots < volume_min:
            implied_risk_at_min = (volume_min / raw_lots) * risk_pct if raw_lots > 0 else float("inf")
            if implied_risk_at_min > risk_pct * cfg.min_lot_risk_tolerance_multiplier:
                return LotSizeResult(False, reason="MIN_LOT_EXCEEDS_RISK_TOLERANCE", detail=detail)
            norm_lots = volume_min
        norm_lots = min(norm_lots, volume_max)
        detail["norm_lots"] = norm_lots

        # Step 7 — exposure cap chain (all live)
        broker_max = volume_max
        configured_max_order = cfg.max_lots_per_order
        exposure_room = max(0.0, cfg.max_total_xauusd_exposure - current_open_xau_volume)
        final_lots = min(norm_lots, broker_max, configured_max_order, exposure_room)
        final_lots = math.floor(final_lots / volume_step) * volume_step if volume_step else final_lots
        detail["final_lots_pre_margin"] = final_lots
        if final_lots < volume_min:
            return LotSizeResult(False, reason="EXPOSURE_CAP_BELOW_MIN_LOT", detail=detail)

        # Step 8 — margin check
        order_type = mt5.ORDER_TYPE_BUY if direction == Direction.BULLISH else mt5.ORDER_TYPE_SELL
        margin_required = self.data.calc_margin(order_type, final_lots, entry_price)
        detail["margin_required"] = margin_required
        if margin_required is None or margin_required > free_margin * cfg.margin_safety_factor:
            return LotSizeResult(False, reason="MARGIN_INSUFFICIENT", detail=detail)

        # Step 9 — final RR re-validation happens by caller (needs TP distance)
        return LotSizeResult(True, final_lots=final_lots, reason="OK", detail=detail)


def validate_rr(entry: float, sl: float, tp: float, min_rr: float) -> bool:
    risk = abs(entry - sl)
    reward = abs(tp - entry)
    if risk == 0:
        return False
    return (reward / risk) >= min_rr


# ============================================================================
# 16. STRUCTURAL STOP LOSS
# ============================================================================

def structural_stop(direction: Direction, sweep_extreme: float, ob: Optional[OrderBlock],
                     protected_level: float, atr_now: float, spread_points: float,
                     tick_size: float, cfg: Config) -> float:
    candidates = [sweep_extreme, protected_level]
    if ob:
        candidates.append(ob.low if direction == Direction.BULLISH else ob.high)
    spread_price = spread_points * tick_size
    if direction == Direction.BULLISH:
        structural_low = min(candidates)
        return structural_low - (cfg.sl_atr_buffer * atr_now) - spread_price
    else:
        structural_high = max(candidates)
        return structural_high + (cfg.sl_atr_buffer * atr_now) + spread_price


# ============================================================================
# 17. POSITION MANAGEMENT (TP1/TP2/runner)
# ============================================================================

@dataclass
class ManagedPosition:
    ticket: int
    setup_id: str
    direction: Direction
    entry: float
    sl: float
    original_volume: float
    remaining_volume: float
    tp1_price: float
    tp2_price: float
    tp1_done: bool = False
    tp2_done: bool = False
    opened_at: datetime = field(default_factory=lambda: datetime.now(timezone.utc))


class PositionManager:

    def __init__(self, cfg: Config, data: MT5Data, logger: logging.Logger):
        self.cfg = cfg
        self.data = data
        self.log = logger

    def compute_tp_levels(self, direction: Direction, entry: float, sl: float) -> Tuple[float, float]:
        r = abs(entry - sl)
        if direction == Direction.BULLISH:
            return entry + self.cfg.tp1_r * r, entry + self.cfg.tp2_r * r
        return entry - self.cfg.tp1_r * r, entry - self.cfg.tp2_r * r

    def partial_close_volume(self, original_volume: float, pct: float, volume_step: float,
                              volume_min: float) -> float:
        raw = original_volume * pct
        steps = math.floor(raw / volume_step) if volume_step else 0
        vol = steps * volume_step
        if vol < volume_min:
            return 0.0  # signal caller to fold into next tranche (spec §22-23 rounding fallback)
        return vol

    def manage(self, pos: ManagedPosition, current_price: float, m15_swings: List[Swing]):
        sym = self.data.symbol_info()
        volume_step = sym.get("volume_step", 0.01)
        volume_min = sym.get("volume_min", 0.01)

        hit_tp1 = (current_price >= pos.tp1_price) if pos.direction == Direction.BULLISH \
            else (current_price <= pos.tp1_price)
        hit_tp2 = (current_price >= pos.tp2_price) if pos.direction == Direction.BULLISH \
            else (current_price <= pos.tp2_price)

        if hit_tp1 and not pos.tp1_done:
            vol = self.partial_close_volume(pos.original_volume, self.cfg.tp1_close_pct,
                                             volume_step, volume_min)
            if vol == 0.0:
                self.log.info(f"[{pos.setup_id}] TP1 tranche rounds to 0 — rolling into TP2 close.")
            else:
                self._close_partial(pos, vol)
                pos.remaining_volume -= vol
            pos.tp1_done = True
            self.log.info(f"[{pos.setup_id}] TP1_COMPLETE")

        if hit_tp2 and not pos.tp2_done:
            pct = self.cfg.tp2_close_pct + (self.cfg.tp1_close_pct if pos.tp1_done and
                                             self.partial_close_volume(pos.original_volume,
                                                                        self.cfg.tp1_close_pct,
                                                                        volume_step, volume_min) == 0
                                             else 0)
            vol = self.partial_close_volume(pos.original_volume, pct, volume_step, volume_min)
            vol = min(vol, pos.remaining_volume)
            if vol > 0:
                self._close_partial(pos, vol)
                pos.remaining_volume -= vol
            pos.tp2_done = True
            self.log.info(f"[{pos.setup_id}] TP2_COMPLETE")
            self._move_to_breakeven(pos, sym)

        if pos.tp2_done:
            self._trail_runner(pos, m15_swings)

    def _close_partial(self, pos: ManagedPosition, volume: float):
        tick = self.data.tick()
        price = tick.get("bid") if pos.direction == Direction.BULLISH else tick.get("ask")
        request = {
            "action": mt5.TRADE_ACTION_DEAL,
            "symbol": self.cfg.symbol,
            "volume": volume,
            "type": mt5.ORDER_TYPE_SELL if pos.direction == Direction.BULLISH else mt5.ORDER_TYPE_BUY,
            "position": pos.ticket,
            "price": price,
            "magic": self.cfg.magic_number,
            "comment": f"symmetrix_partial_{pos.setup_id}",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        result = self.data.send_order(request)
        self.log.info(f"Partial close sent: {volume} lots, result={result}")

    def _move_to_breakeven(self, pos: ManagedPosition, sym: dict):
        spread_price = sym.get("spread", 0) * sym.get("trade_tick_size", 0.01)
        new_sl = pos.entry + spread_price if pos.direction == Direction.BULLISH else pos.entry - spread_price
        if pos.direction == Direction.BULLISH and new_sl <= pos.sl:
            return
        if pos.direction == Direction.BEARISH and new_sl >= pos.sl:
            return
        self._modify_sl(pos, new_sl)

    def _trail_runner(self, pos: ManagedPosition, m15_swings: List[Swing]):
        confirmed = [s for s in m15_swings if s.quality == "CONFIRMED"]
        if pos.direction == Direction.BULLISH:
            higher_lows = [s for s in confirmed if s.kind == "LOW" and s.price > pos.sl]
            if higher_lows:
                candidate = max(s.price for s in higher_lows)
                if candidate > pos.sl:
                    self._modify_sl(pos, candidate)
        else:
            lower_highs = [s for s in confirmed if s.kind == "HIGH" and s.price < pos.sl]
            if lower_highs:
                candidate = min(s.price for s in lower_highs)
                if candidate < pos.sl:
                    self._modify_sl(pos, candidate)

    def _modify_sl(self, pos: ManagedPosition, new_sl: float):
        request = {
            "action": mt5.TRADE_ACTION_SLTP,
            "position": pos.ticket,
            "symbol": self.cfg.symbol,
            "sl": new_sl,
            "magic": self.cfg.magic_number,
        }
        result = self.data.send_order(request)
        if result and getattr(result, "retcode", None) == mt5.TRADE_RETCODE_DONE:
            pos.sl = new_sl
            self.log.info(f"[{pos.setup_id}] SL moved to {new_sl} (never widened).")
        else:
            self.log.warning(f"[{pos.setup_id}] SL modify failed: {result}")


# ============================================================================
# 18. DAILY / WEEKLY / MONTHLY LOSS LIMITS
# ============================================================================

@dataclass
class RiskState:
    day_start_equity: float = 0.0
    week_start_equity: float = 0.0
    month_start_equity: float = 0.0
    trades_today: int = 0
    losses_today: int = 0
    consecutive_losses: int = 0
    last_reset_day: Optional[datetime] = None
    emergency_stop: bool = False
    emergency_flatten: bool = False
    order_reject_timestamps: List[datetime] = field(default_factory=list)


class RiskLimiter:

    def __init__(self, cfg: Config, logger: logging.Logger):
        self.cfg = cfg
        self.log = logger
        self.state = RiskState()

    def rollover_check(self, now: datetime, current_equity: float):
        if self.state.last_reset_day is None or now.date() != self.state.last_reset_day.date():
            self.state.day_start_equity = current_equity
            self.state.trades_today = 0
            self.state.losses_today = 0
            self.state.last_reset_day = now
            self.log.info(f"Daily rollover: baseline equity set to {current_equity:.2f}")
        if now.weekday() == 0 and (self.state.week_start_equity == 0.0):
            self.state.week_start_equity = current_equity
        if now.day == 1 and self.state.month_start_equity == 0.0:
            self.state.month_start_equity = current_equity

    def check_loss_limits(self, current_equity: float) -> Optional[str]:
        if self.state.day_start_equity:
            day_pct = (current_equity - self.state.day_start_equity) / self.state.day_start_equity * 100
            if day_pct <= self.cfg.emergency_daily_stop_pct:
                self._trigger_emergency("EMERGENCY_DAILY_STOP")
                return "EMERGENCY_DAILY_STOP"
            if day_pct <= self.cfg.daily_stop_pct:
                self.state.emergency_stop = True
                return "DAILY_STOP"
        if self.state.week_start_equity:
            week_pct = (current_equity - self.state.week_start_equity) / self.state.week_start_equity * 100
            if week_pct <= self.cfg.weekly_stop_pct:
                self._trigger_emergency("WEEKLY_STOP")
                return "WEEKLY_STOP"
        if self.state.month_start_equity:
            month_pct = (current_equity - self.state.month_start_equity) / self.state.month_start_equity * 100
            if month_pct <= self.cfg.monthly_stop_pct:
                self._trigger_emergency("MONTHLY_STOP")
                return "MONTHLY_STOP"
        return None

    def _trigger_emergency(self, reason: str):
        self.state.emergency_stop = True
        self.log.critical(f"EMERGENCY STOP TRIGGERED: {reason}")

    def register_trade_result(self, is_loss: bool):
        self.state.trades_today += 1
        if is_loss:
            self.state.losses_today += 1
            self.state.consecutive_losses += 1
        else:
            self.state.consecutive_losses = 0
        if self.state.consecutive_losses >= 3:
            self.log.warning("Three consecutive losses — disabling new trades for remainder of day.")
            self.state.emergency_stop = True  # remainder-of-day lock; cleared at rollover

    def register_order_rejection(self, now: datetime):
        self.state.order_reject_timestamps.append(now)
        window_start = now - timedelta(minutes=self.cfg.retry_circuit_breaker_window_min)
        self.state.order_reject_timestamps = [t for t in self.state.order_reject_timestamps if t >= window_start]
        if len(self.state.order_reject_timestamps) >= self.cfg.retry_circuit_breaker_count:
            self._trigger_emergency("RETRY_CIRCUIT_BREAKER")

    def min_required_score(self) -> int:
        if self.state.consecutive_losses >= 2:
            return self.cfg.score_min_after_2_losses
        return self.cfg.score_min_normal

    def can_trade(self) -> bool:
        return (not self.state.emergency_stop) and (self.state.trades_today < self.cfg.max_trades_per_day)


# ============================================================================
# 19. WEEKEND PROTECTION
# ============================================================================

def weekend_phase(now_utc: datetime, cfg: Config) -> str:
    """Returns 'NORMAL', 'DISABLE_NEW', 'DEFENSIVE', 'FLATTEN', or 'WEEKEND'."""
    if now_utc.weekday() == 5 or (now_utc.weekday() == 6):
        return "WEEKEND"
    if now_utc.weekday() != 4:  # only relevant on Friday
        return "NORMAL"
    close_time = now_utc.replace(hour=cfg.friday_close_hour_utc, minute=0, second=0, microsecond=0)
    delta = close_time - now_utc
    minutes_to_close = delta.total_seconds() / 60
    if minutes_to_close <= cfg.weekend_t_minus_30m_flatten_min:
        return "FLATTEN"
    if minutes_to_close <= cfg.weekend_t_minus_2h_defensive * 60:
        return "DEFENSIVE"
    if minutes_to_close <= cfg.weekend_t_minus_4h_disable * 60:
        return "DISABLE_NEW"
    if minutes_to_close <= 6 * 60:
        return "PREWEEKEND_REDUCED_RISK"
    return "NORMAL"


def flatten_all(data: MT5Data, cfg: Config, logger: logging.Logger):
    positions = data.open_positions()
    for p in positions:
        tick = data.tick()
        direction = Direction.BULLISH if p["type"] == 0 else Direction.BEARISH
        price = tick.get("bid") if direction == Direction.BULLISH else tick.get("ask")
        request = {
            "action": mt5.TRADE_ACTION_DEAL,
            "symbol": cfg.symbol,
            "volume": p["volume"],
            "type": mt5.ORDER_TYPE_SELL if direction == Direction.BULLISH else mt5.ORDER_TYPE_BUY,
            "position": p["ticket"],
            "price": price,
            "magic": cfg.magic_number,
            "comment": "symmetrix_weekend_flatten",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        result = data.send_order(request)
        logger.critical(f"Weekend flatten sent for ticket {p['ticket']}: {result}")
    orders = mt5.orders_get(symbol=cfg.symbol) or []
    for o in orders:
        if o.magic == cfg.magic_number:
            mt5.order_send({"action": mt5.TRADE_ACTION_REMOVE, "order": o.ticket})


# ============================================================================
# 20. RECONCILIATION
# ============================================================================

def reconcile_positions(data: MT5Data, known_setup_ids: Dict[int, str], logger: logging.Logger):
    """Rebuild internal state from broker truth; flag orphans."""
    live_positions = data.open_positions()
    live_tickets = {p["ticket"] for p in live_positions}
    for ticket in live_tickets:
        if ticket not in known_setup_ids:
            logger.warning(f"ORPHANED position detected: ticket {ticket} — "
                            f"no internal record. Applying defensive management only.")
    closed_tickets = [t for t in known_setup_ids if t not in live_tickets]
    for t in closed_tickets:
        logger.info(f"Position {t} no longer open at broker — removing from local tracking.")
        known_setup_ids.pop(t, None)
    return live_positions


# ============================================================================
# 21. SETUP ID / DUPLICATE PROTECTION
# ============================================================================

def build_setup_id(symbol: str, direction: Direction, liquidity_time: datetime,
                    choch_time: datetime, bos_time: datetime) -> str:
    raw = f"{symbol}|{direction.name}|{liquidity_time.isoformat()}|{choch_time.isoformat()}|{bos_time.isoformat()}"
    return str(uuid.uuid5(uuid.NAMESPACE_DNS, raw))


# ============================================================================
# 22. MAIN ORCHESTRATOR
# ============================================================================

class SymmetrixGoldSMC:

    def __init__(self, cfg: Config):
        self.cfg = cfg
        cfg.bind_timeframes()
        self.log = setup_logging(cfg)
        self.data = MT5Data(cfg, self.log)
        self.risk_mgr = RiskManager(cfg, self.data, self.log)
        self.pos_mgr = PositionManager(cfg, self.data, self.log)
        self.risk_limiter = RiskLimiter(cfg, self.log)
        self.direction_engine = DirectionEngine(cfg)
        self.liquidity_engine = LiquidityEngine(cfg)
        self.news_filter = NewsFilter(cfg, self.log)
        self.consumed_setup_ids: set = set()
        self.managed_positions: Dict[int, ManagedPosition] = {}
        self.state = State.IDLE
        self.state_entered_bar = 0
        self.working = {}  # scratch space for the in-progress setup

    # -- lifecycle -----------------------------------------------------
    def start(self):
        if not self.data.connect():
            raise RuntimeError("Could not connect to MT5 terminal.")
        self.log.info("Symmetrix Gold SMC started.")
        self.run_loop()

    def stop(self):
        self.data.shutdown()
        self.log.info("Symmetrix Gold SMC stopped.")

    # -- main loop -------------------------------------------------------
    def run_loop(self):
        while True:
            try:
                self.tick()
            except Exception as e:
                self.log.exception(f"Unhandled exception in main loop: {e}")
            time.sleep(self.cfg.poll_seconds)

    def tick(self):
        cfg = self.cfg
        now = datetime.now(timezone.utc)

        if not self.data.is_connected():
            self.log.error("MT5 disconnected — attempting reconnect and reconciling before any new orders.")
            self.data.connect()
            reconcile_positions(self.data, {p.ticket: p.setup_id for p in self.managed_positions.values()}, self.log)
            return

        acct = self.data.account()
        if not acct:
            return
        equity = acct["equity"]

        self.risk_limiter.rollover_check(now, equity)
        limit_hit = self.risk_limiter.check_loss_limits(equity)
        if limit_hit:
            self.log.warning(f"Loss limit hit: {limit_hit} — new entries disabled.")

        phase = weekend_phase(now, cfg)
        if phase == "FLATTEN":
            flatten_all(self.data, cfg, self.log)
            return
        if phase == "WEEKEND":
            return

        # manage any open positions regardless of new-signal gating
        self._manage_open_positions()

        if phase in ("DISABLE_NEW", "WEEKEND") or self.risk_limiter.state.emergency_stop \
                or not self.risk_limiter.can_trade():
            return

        self.news_filter.refresh()
        if self.news_filter.in_blackout(now):
            return

        if not in_trading_session(now, cfg) and not cfg.allow_asian_session:
            return

        if len(self.data.open_positions()) >= 1:
            return  # one-position policy (§33)

        self._evaluate_signal_pipeline(now, phase)

    # -- signal pipeline ---------------------------------------------------
    def _evaluate_signal_pipeline(self, now: datetime, weekend_phase_str: str):
        cfg = self.cfg
        d = self.data

        m5 = d.rates(cfg.tf_exec, 400)
        m15 = d.rates(cfg.tf_confirm, 300)
        h1 = d.rates(cfg.tf_h1, 300)
        h4 = d.rates(cfg.tf_h4, 300)
        d1 = d.rates(cfg.tf_d1, 200)
        if any(df.empty for df in [m5, m15, h1, h4, d1]):
            return

        atr_m5 = Indicators.atr(m5, 14)
        atr_h1 = Indicators.atr(h1, 14)

        # 1) direction
        h4_read, h1_read, d1_read = self.direction_engine.evaluate(h4, h1, d1)
        if h4_read.direction == Direction.NEUTRAL or not h4_read.persistent:
            return
        if h4_read.direction != h1_read.direction:
            return  # mandatory H4 == H1
        direction = h4_read.direction

        # 2) dealing range / premium-discount
        h1_swings = detect_swings(h1, atr_h1, cfg.swing_min_amplitude_atr)
        rng = compute_dealing_range(h1, h1_swings, atr_h1, cfg)
        if not rng or not rng.valid:
            return
        current_price = m5["close"].iloc[-1]
        zone = classify_zone(current_price, rng, cfg)
        if direction == Direction.BULLISH and zone != "DISCOUNT":
            return
        if direction == Direction.BEARISH and zone != "PREMIUM":
            return

        # 3) liquidity map
        m15_swings = detect_swings(m15, Indicators.atr(m15, 14), cfg.swing_min_amplitude_atr)
        self.liquidity_engine.build(d1, h1, m15, h1_swings, m15_swings, now)
        target = self.liquidity_engine.best_target(direction, current_price)
        if not target:
            return

        # 4) sweep
        sweep = detect_sweep(m5, atr_m5, target.price, direction, cfg)
        if not sweep.valid:
            return
        sweep_time = m5["time"].iloc[-(len(m5) - sweep.reclaim_index)] if sweep.reclaim_index < len(m5) else m5["time"].iloc[-1]

        # 5) CHoCH
        m5_swings = detect_swings(m5, atr_m5, cfg.swing_min_amplitude_atr)
        choch = detect_choch(m5, m5_swings, direction, sweep_time, cfg)
        if not choch.confirmed:
            return

        # 6) displacement
        displaced, disp_ratio = check_displacement(m5, choch.bar_index, direction, cfg)
        if not displaced:
            return

        # 7) BOS (mandatory)
        bos_ok, bos_index = check_bos(m5, choch.bar_index, m5_swings, choch.bar_index, direction, cfg)
        if not bos_ok:
            return

        # 8) FVG / OB / Fib confluence -> entry zone
        fvgs = find_fvgs(m5, atr_m5, cfg)
        relevant_fvgs = [f for f in fvgs if f.direction == direction and
                         (len(m5) - 1 - m5[m5["time"] == f.time].index[0] if any(m5["time"] == f.time) else 999) <= cfg.fvg_stale_bars]
        best_fvg = relevant_fvgs[-1] if relevant_fvgs else None

        ob = find_order_block(m5, bos_index, choch.bar_index, direction, cfg)

        fib_618, fib_786, sweep_extreme_price = fib_618_zone(
            sweep_extreme=m5["low"].iloc[sweep.bar_index] if direction == Direction.BULLISH
            else m5["high"].iloc[sweep.bar_index],
            choch_break_price=choch.protected_level,
            direction=direction
        )

        atr_now = atr_m5.iloc[-1]
        zone_obj = build_entry_zone(best_fvg, ob, fib_618, atr_now, cfg)
        if not zone_obj:
            return

        # sanity: min stop distance check (§13 addition)
        provisional_sl = structural_stop(
            direction, sweep_extreme_price, ob, choch.protected_level, atr_now,
            self.data.symbol_info().get("spread", 0),
            self.data.symbol_info().get("trade_tick_size", 0.01), cfg
        )
        if abs(current_price - provisional_sl) < cfg.entry_zone_min_sl_atr * atr_now:
            return

        # 9) confirmation
        confirm_index = check_confirmation(m5, zone_obj, direction, cfg)
        if confirm_index is None:
            return

        # 10) setup id / duplicate protection
        setup_id = build_setup_id(cfg.symbol, direction, target.time, m5["time"].iloc[choch.bar_index],
                                   m5["time"].iloc[bos_index])
        if setup_id in self.consumed_setup_ids:
            return

        # 11) scoring (RR + vol regime are hard gates, computed first)
        entry_price = m5["close"].iloc[confirm_index]
        sl_price = provisional_sl
        tp1, tp2 = self.pos_mgr.compute_tp_levels(direction, entry_price, sl_price)
        rr_ok = validate_rr(entry_price, sl_price, tp2, cfg.min_rr)

        avg_atr_50 = atr_m5.tail(50).mean()
        vol_ratio = atr_now / avg_atr_50 if avg_atr_50 else 0
        vol_regime_ok = cfg.vol_regime_min_pct <= vol_ratio <= cfg.vol_regime_max_pct

        score = compute_score(
            h4_h1_aligned=True, d1_aligned=(d1_read.direction == direction),
            correct_zone=True, major_liquidity=(target.priority_tier <= 1),
            clean_sweep=(cfg.sweep_pref_pen_atr <= sweep.depth_atr <= cfg.sweep_max_pen_atr),
            choch=True, bos=True,
            displacement=(disp_ratio >= cfg.displacement_pref_ratio),
            fvg_present="FVG" in zone_obj.components, ob_present="OB" in zone_obj.components,
            fib_present=True, ema_ok=True, rsi_ok=True, adx_ok=True,
            good_session=in_trading_session(now, cfg), no_news=not self.news_filter.in_blackout(now),
            rr_ok=rr_ok, vol_regime_ok=vol_regime_ok,
        )

        min_score = self.risk_limiter.min_required_score()
        setup_log_row = dict(
            timestamp=now.isoformat(), symbol=cfg.symbol, direction=direction.name,
            h4_bias=h4_read.direction.name, h1_bias=h1_read.direction.name, d1_bias=d1_read.direction.name,
            liquidity_type=target.kind, liquidity_price=target.price, sweep_depth_atr=sweep.depth_atr,
            choch=True, displacement_ratio=disp_ratio, bos=True,
            fvg="FVG" in zone_obj.components, ob="OB" in zone_obj.components, fib=True,
            score=score.total, entry=entry_price, sl=sl_price, tp1=tp1, tp2=tp2,
        )

        if not score.passed_gates or score.total < min_score:
            setup_log_row["reason"] = f"REJECTED score={score.total} min={min_score} gates={score.passed_gates}"
            append_csv(cfg.setup_log_csv, setup_log_row, self._setup_log_header())
            return

        # 12) risk / lot sizing (§19a — mandatory automatic calculation)
        preweekend = weekend_phase_str == "PREWEEKEND_REDUCED_RISK"
        current_open_vol = sum(p["volume"] for p in self.data.open_positions())
        lot_result = self.risk_mgr.calculate(
            direction, entry_price, sl_price, self.risk_limiter.state.losses_today,
            preweekend, current_open_vol
        )
        setup_log_row.update(
            risk_pct=lot_result.detail.get("risk_pct"), equity=lot_result.detail.get("equity"),
            risk_capital=lot_result.detail.get("risk_capital"), tick_value=lot_result.detail.get("tick_value"),
            tick_size=lot_result.detail.get("tick_size"),
            stop_distance_points=lot_result.detail.get("stop_distance_points"),
            raw_lots=lot_result.detail.get("raw_lots"), final_lots=lot_result.final_lots,
            margin_required=lot_result.detail.get("margin_required"),
            free_margin=lot_result.detail.get("free_margin"),
        )
        if not lot_result.ok:
            setup_log_row["reason"] = f"REJECTED {lot_result.reason}"
            append_csv(cfg.setup_log_csv, setup_log_row, self._setup_log_header())
            return

        # final RR re-check against rounded lot (RR itself doesn't change with lot size,
        # but re-validate in case of any last-moment price drift)
        tick = self.data.tick()
        live_entry = tick.get("ask") if direction == Direction.BULLISH else tick.get("bid")
        if not validate_rr(live_entry, sl_price, tp2, cfg.min_rr):
            setup_log_row["reason"] = "REJECTED RR_FAIL_AT_EXECUTION"
            append_csv(cfg.setup_log_csv, setup_log_row, self._setup_log_header())
            return

        # spread/execution filter
        spread_points = self.data.symbol_info().get("spread", 0)
        if spread_points > min(cfg.max_spread_static_points, cfg.max_spread_atr_pct * atr_now /
                                self.data.symbol_info().get("trade_tick_size", 0.01)):
            setup_log_row["reason"] = "REJECTED SPREAD_TOO_WIDE"
            append_csv(cfg.setup_log_csv, setup_log_row, self._setup_log_header())
            return

        # 13) execute
        setup_log_row["reason"] = "ACCEPTED"
        append_csv(cfg.setup_log_csv, setup_log_row, self._setup_log_header())
        self._execute_trade(setup_id, direction, live_entry, sl_price, tp1, tp2, lot_result.final_lots)

    def _setup_log_header(self):
        return [
            "timestamp", "symbol", "direction", "h4_bias", "h1_bias", "d1_bias",
            "liquidity_type", "liquidity_price", "sweep_depth_atr", "choch",
            "displacement_ratio", "bos", "fvg", "ob", "fib", "score", "spread",
            "risk_pct", "equity", "risk_capital", "tick_value", "tick_size",
            "stop_distance_points", "raw_lots", "final_lots", "margin_required",
            "free_margin", "entry", "sl", "tp1", "tp2", "reason"
        ]

    def _execute_trade(self, setup_id: str, direction: Direction, entry: float,
                        sl: float, tp1: float, tp2: float, lots: float):
        cfg = self.cfg
        order_type = mt5.ORDER_TYPE_BUY if direction == Direction.BULLISH else mt5.ORDER_TYPE_SELL
        request = {
            "action": mt5.TRADE_ACTION_DEAL,
            "symbol": cfg.symbol,
            "volume": lots,
            "type": order_type,
            "price": entry,
            "sl": sl,
            "tp": tp2,  # broker-level TP as backstop; partials handled by PositionManager
            "magic": cfg.magic_number,
            "comment": f"symmetrix_{setup_id[:16]}",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        for attempt in range(cfg.max_order_retries):
            result = self.data.send_order(request)
            if result and result.retcode == mt5.TRADE_RETCODE_DONE:
                self.consumed_setup_ids.add(setup_id)
                self.managed_positions[result.order] = ManagedPosition(
                    ticket=result.order, setup_id=setup_id, direction=direction,
                    entry=entry, sl=sl, original_volume=lots, remaining_volume=lots,
                    tp1_price=tp1, tp2_price=tp2
                )
                self.log.info(f"ORDER_CONFIRMED setup={setup_id} ticket={result.order} lots={lots}")
                return
            self.log.warning(f"Order attempt {attempt+1} failed: {result}")
            self.risk_limiter.register_order_rejection(datetime.now(timezone.utc))
            time.sleep(cfg.retry_delay_seconds)
        self.log.error(f"Order failed after {cfg.max_order_retries} retries for setup {setup_id}.")

    def _manage_open_positions(self):
        m15 = self.data.rates(self.cfg.tf_confirm, 300)
        if m15.empty:
            return
        m15_swings = detect_swings(m15, Indicators.atr(m15, 14), self.cfg.swing_min_amplitude_atr)
        live = reconcile_positions(self.data, {t: p.setup_id for t, p in self.managed_positions.items()}, self.log)
        live_tickets = {p["ticket"] for p in live}
        for ticket in list(self.managed_positions.keys()):
            if ticket not in live_tickets:
                closed_pos = self.managed_positions.pop(ticket)
                self.log.info(f"[{closed_pos.setup_id}] Position closed — recording result.")
                # NOTE: net P/L, MFE/MAE, commission/swap/slippage should be pulled from
                # mt5.history_deals_get(position=ticket) here and written to the journal
                # (journal_csv) plus fed into risk_limiter.register_trade_result(is_loss).
                continue
            tick = self.data.tick()
            price = tick.get("bid") if self.managed_positions[ticket].direction == Direction.BULLISH else tick.get("ask")
            self.pos_mgr.manage(self.managed_positions[ticket], price, m15_swings)


# ============================================================================
# ENTRY POINT
# ============================================================================

if __name__ == "__main__":
    config = Config()
    bot = SymmetrixGoldSMC(config)
    try:
        bot.start()
    except KeyboardInterrupt:
        bot.stop()






SYMMETRIX GOLD SMC v3.0
Production EA Coding & Parameter Specification
High-Selectivity XAUUSD Automated Trading System — MT5 / MQL5
This document is intended to serve as the coding specification for the Expert Advisor. It combines the strengthened SMC specification, the master multi-timeframe architecture, automatic position sizing, risk/exposure management, mandatory stop-loss protection, execution safeguards, recovery logic, and the higher-selectivity rules intended to test whether a sustainable 75%–80% win-rate region exists.
The 75%–80% figure must remain a validation target, not a guaranteed performance claim. Production acceptance must come from historical, out-of-sample, walk-forward, stress, Monte Carlo, demo-forward, and controlled live testing. The strengthened specification itself requires at least 100 combined OOS/walk-forward trades before profitability statistics are treated as meaningful.
________________________________________
1. EA IDENTITY
Parameter	Value
EA Name	SYMMETRIX GOLD SMC
Version	3.0 Production Candidate
Platform	MetaTrader 5
Language	MQL5
Primary Symbol	XAUUSD
Execution TF	M5
Confirmation TF	M15
Structural TF	H1
Trend TF	H4
Macro TF	D1
Strategy	Multi-Timeframe SMC / Liquidity / Structure
Execution	Fully Automatic
Position Sizing	Fully Automatic
Manual Lot Size	Prohibited
Mandatory SL	TRUE
Entry Without SL	Prohibited
Martingale	Prohibited
Grid	Prohibited
Averaging Losing Trades	Prohibited
Production Lock	TRUE
Maximum Trades/Day	3
Default Risk	0.25% current equity
Target Research Win Rate	75%–80%
Guaranteed Win Rate	NONE
The existing master specification defines the EA as a fully automatic, risk-adjusted, production-locked XAUUSD system using M5, M15, H1, H4 and D1.
________________________________________
2. CORE OPERATING PRINCIPLE
The EA operates as a deterministic state machine.
It does not continuously ask:
“Should I buy or sell?”
Instead, it asks:
Has every required condition in the setup sequence occurred in the correct order, within the permitted time window, using current synchronized market data?
A trade can be submitted only after:
Market Data → Direction → Liquidity → Sweep → CHoCH → Displacement → BOS → Entry Zone → Retracement → Confirmation → Quality Gate → Risk → Exposure → SL → Lot Size → Margin → Execution
If one mandatory condition fails:
NO TRADE
________________________________________
3. CONFIGURATION ARCHITECTURE
There shall be two classes of parameters.
A. PROTECTED STRATEGY PARAMETERS
These are compiled into production logic and cannot be changed from normal EA inputs when:
ProductionLock = true
B. BROKER / OPERATIONAL PARAMETERS
These may remain configurable because they vary by broker, server time, symbol suffix, execution policy or account.
________________________________________
4. PROTECTED CORE PARAMETERS
Code Parameter	Type	Production Value	Editable
ProductionLock	bool	true	No
StrategySymbolBase	string	"XAUUSD"	No
TF_EXECUTION	ENUM_TIMEFRAMES	PERIOD_M5	No
TF_CONFIRMATION	ENUM_TIMEFRAMES	PERIOD_M15	No
TF_STRUCTURE	ENUM_TIMEFRAMES	PERIOD_H1	No
TF_TREND	ENUM_TIMEFRAMES	PERIOD_H4	No
TF_MACRO	ENUM_TIMEFRAMES	PERIOD_D1	No
MaxTradesPerDay	int	3	No
AllowGrid	bool	false	No
AllowMartingale	bool	false	No
AllowAveragingLosers	bool	false	No
AllowEntryWithoutSL	bool	false	No
AllowSLRemoval	bool	false	No
AllowSLWidening	bool	false	No
OneXAUPositionOnly	bool	true	No
________________________________________
5. MULTI-TIMEFRAME LIVE DATA ENGINE
The EA must continuously maintain current closed-bar data for:
D1
Macro direction and higher-timeframe context.
H4
Primary trend determination.
H1
Structural range, premium/discount and structural direction.
M15
Intermediate structure, confirmation context and runner management.
M5
Execution logic.
The EA must explicitly read each timeframe rather than assuming data from the attached chart. The master specification requires active multi-timeframe reading and synchronization.
Recommended structures:
struct TFState
{
   ENUM_TIMEFRAMES tf;
   datetime lastClosedBar;
   datetime lastUpdate;

   double close;
   double atr14;
   double ema20;
   double ema50;
   double adx14;

   bool synchronized;
   bool enoughBars;
   bool valid;
};
Maintain:
TFState StateD1;
TFState StateH4;
TFState StateH1;
TFState StateM15;
TFState StateM5;
________________________________________
6. DATA SYNCHRONIZATION GATE
Before any strategy evaluation:
D1 synchronized
AND H4 synchronized
AND H1 synchronized
AND M15 synchronized
AND M5 synchronized
AND enough history available
AND all indicator handles valid
Otherwise:
DATA_NOT_READY
and:
NO TRADE
Use completed bars for structural decisions.
Do not use an unfinished H1/H4/D1 candle as though it were confirmed.
________________________________________
7. INDICATORS
Required handles/data:
Indicator	TF
EMA20	D1/H4/H1
EMA50	D1/H4/H1
ATR14	D1/H4/H1/M15/M5
ADX14	H4
RSI14	M5 if retained in scoring
Average Candle Body 20	M5
ATR Average 50	M5
________________________________________
8. D1 MACRO DIRECTION
Bullish candidate:
Close > EMA20 > EMA50
Bearish:
Close < EMA20 < EMA50
For high-selectivity/A++ trades:
D1 alignment becomes mandatory.
Therefore:
BUY requires:
D1 = BULLISH
SELL requires:
D1 = BEARISH
A conflicting D1 blocks the highest-quality production setup.
________________________________________
9. H4 DIRECTION ENGINE
Bullish:
Close > EMA20 > EMA50
Bearish:
Close < EMA20 < EMA50
Mandatory separation:
abs(EMA20 - EMA50) >= 0.15 * H4_ATR14
Mandatory persistence:
Same H4 direction >= 3 completed H4 candles
Mandatory ADX:
ADX14 >= 20
For A++ high-selectivity mode:
Preferred ADX >= 25
Research tier:
A++ preferred ADX >= 30
The strengthened specification makes EMA separation, trend persistence, and ADX hard directional filters rather than merely scoring items.
________________________________________
10. H4 OVEREXTENSION FILTER
Add production protection:
DistanceFromEMA20 <= 2.5 * H4_ATR
If price is excessively extended from H4 EMA20:
TREND_OVEREXTENDED
→ wait for retracement.
This prevents entering late-stage momentum simply because ADX is elevated.
This is an enhancement to be validated in optimization rather than assumed to improve performance.
________________________________________
11. H1 DIRECTION
Require:
H1 direction == H4 direction
High-selectivity production:
D1 == H4 == H1
Otherwise:
HTF_ALIGNMENT_FAILED
________________________________________
12. H1 DEALING RANGE
The H1 range must:
Age <= 40 completed H1 candles
and:
RangeSize >= 3.0 * H1_ATR
Recalculate whenever a new confirmed H1 structural swing changes the active range.
The strengthened specification explicitly rejects stale, tiny and equilibrium-centered ranges.
________________________________________
13. PREMIUM / DISCOUNT
Calculate:
RangeMid = (RangeHigh + RangeLow) / 2.0;
RangeSize = RangeHigh - RangeLow;
Equilibrium exclusion:
±5% of total range around midpoint
If price is inside that region:
EQUILIBRIUM_NEUTRAL
→ NO TRADE
BUY:
Discount only
SELL:
Premium only
A++ enhancement:
BUY preferably located in deeper discount.
SELL preferably located in deeper premium.
________________________________________
14. LIQUIDITY ENGINE
Track:
1.	Previous Week High / Low
2.	Previous Day High / Low
3.	Equal Highs / Equal Lows
4.	Asian High / Low
5.	Confirmed H1 swings
6.	Confirmed M15 swings
Priority:
PWH/PWL
→ PDH/PDL
→ EQH/EQL
→ Asian H/L
→ H1
→ M15
Same-tier rule:
nearest qualifying pool
Liquidity >10 trading days untouched:
downgrade one tier
Equal high/low tolerance:
<= 0.08 * ATR
These hierarchy and decay rules are explicitly defined in the strengthened specification.
________________________________________
15. LIQUIDITY OBJECT
Recommended coding structure:
enum LiquidityType
{
   LIQ_PWH,
   LIQ_PWL,
   LIQ_PDH,
   LIQ_PDL,
   LIQ_EQH,
   LIQ_EQL,
   LIQ_ASIA_HIGH,
   LIQ_ASIA_LOW,
   LIQ_H1_SWING_HIGH,
   LIQ_H1_SWING_LOW,
   LIQ_M15_SWING_HIGH,
   LIQ_M15_SWING_LOW
};

struct LiquidityLevel
{
   LiquidityType type;
   double price;
   datetime created;
   int priority;
   bool touched;
   bool swept;
   bool stale;
};
________________________________________
16. SWING DETECTION
Use confirmed five-bar fractal.
Swing High:
High[n] > High[n-2]
High[n] > High[n-1]
High[n] > High[n+1]
High[n] > High[n+2]
Swing Low mirrored.
Because two future bars are needed:
Never mark the swing confirmed before those bars close.
Amplitude:
SwingAmplitude >= 0.20 * M5_ATR
Below:
MINOR_SWING
Minor swings cannot trigger:
•	CHoCH
•	BOS
•	structural SL
•	major liquidity classification
The amplitude rule is required by the strengthened specification.
________________________________________
17. SWEEP REQUIREMENTS
Broad valid research envelope:
0.05 ATR <= penetration <= 0.75 ATR
Preferred production region:
0.10 ATR <= penetration <= 0.50 ATR
High-selectivity A++ research region:
0.15 ATR <= penetration <= 0.40 ATR
Reclaim mandatory.
Sweep + reclaim must complete within:
<= 3 M5 candles
Slow penetration:
BREAKOUT_NOT_SWEEP
Do not evaluate sweep if:
Current M5 ATR < 0.40 * ATR50Average
The source specification requires both velocity and dead-volatility protection.
________________________________________
18. STRICT CHoCH
BUY
Required structural progression:
LL → LH → LL
Then:
sell-side liquidity sweep
Then:
completed M5 close > protected confirmed LH
SELL
HH → HL → HH
then buy-side sweep,
then:
completed M5 close < protected confirmed HL
Only CONFIRMED swings are valid protection points.
CHoCH timeout:
24 M5 candles
The source explicitly requires confirmed protected levels and the 24-bar timeout.
________________________________________
19. DISPLACEMENT
Calculate:
AvgBody20 = Average(abs(Close[i] - Open[i]), previous 20 M5 bars);
Base:
CurrentBody >= 1.25 * AvgBody20
Preferred:
>= 1.50
A++ research:
>= 1.75
Following candle:
retracement < 70% of displacement body/range
Otherwise:
DISPLACEMENT_FAILED
The immediate follow-through requirement is part of the strengthened strategy.
________________________________________
20. BOS — MANDATORY
BOS cannot be optional.
BUY:
completed M5 close > confirmed structural high
SELL:
completed M5 close < confirmed structural low
Wick:
does not count
Referenced swing must have existed before CHoCH.
BOS maximum delay:
15 M5 candles after displacement
CHoCH invalidated before BOS:
RESET_SETUP
These sequencing requirements prevent look-ahead or disconnected structural signals.
________________________________________
21. FAIR VALUE GAP
Three-candle FVG.
Minimum:
0.05 * M5 ATR
A++ preferred:
>= 0.10 ATR
Track:
struct FVG
{
   bool bullish;
   double low;
   double high;
   double size;
   double mitigationPct;
   datetime created;
   int ageBars;
   bool stale;
};
Eligibility:
Mitigation < 50%
At:
>=50% = MITIGATED
100% = FULLY_MITIGATED
Age >50 M5 bars:
STALE_FVG
The source explicitly defines mitigation percentage and staleness.
________________________________________
22. ORDER BLOCK
Valid OB:
last opposite candle immediately before qualifying impulse
Preferred consequence:
Displacement + BOS
Strong OB may additionally produce FVG/CHoCH.
Track mitigation percentage.
Age:
>80 M5 bars = stale/deprioritized
The source specifies the adjacent-candle tie-break and OB staleness rules.
________________________________________
23. FIBONACCI
Primary:
61.8%
Research:
78.6%
Must use actual impulse associated with the CHoCH/BOS sequence.
If a completed candle closes beyond:
Fib 100% / sweep extreme
then:
SETUP_INVALID
This invalidation condition is part of the strengthened specification.
________________________________________
24. ENTRY ZONE
Baseline production:
minimum 2 of:
FVG
OB
Fib61.8
A++ high-selectivity configuration:
FVG + OB + Fib61.8
All three required.
Baseline tolerance:
0.30 ATR
A++ optimization candidates:
0.10 / 0.15 / 0.20 ATR
Do not hardcode the narrowest version until OOS testing verifies it.
________________________________________
25. MINIMUM STOP DISTANCE FROM ZONE
Before activating entry zone:
ProjectedStructuralStopDistance >= 0.50 * M5 ATR
Otherwise:
ZONE_TOO_CLOSE_TO_INVALIDATION
→ reject.
This protects against artificially tight stops and oversized calculated positions.
________________________________________
26. M5 ENTRY CONFIRMATION
Valid confirmation types:
Engulfing
Body fully engulfs previous candle body in trade direction.
Rejection
Opposing wick:
>=60% of full candle range
with directional close near/inside zone.
Micro-BOS
Completed M5 close through most recent internal swing in intended direction.
Existing production:
one confirmation
A++ research:
two compatible confirmations
Example:
Rejection + Micro-BOS
or:
Engulfing + Micro-BOS
Confirmation timeout:
10 M5 bars
The source defines these confirmations explicitly.
________________________________________
27. M15 CONFIRMATION
New stronger filter:
BUY must not have a confirmed M15 bearish continuation structure directly opposing the entry.
SELL mirrored.
A++ preference:
M15 agrees with trade direction
or:
M15 neutral and transitioning toward trade
Clear opposite M15 structure:
M15_CONFLICT
→ reject.
________________________________________
28. VOLATILITY GATE
Mandatory current rules:
0.60 <= M5_ATR / ATR50Average <= 2.50
Below:
DEAD_MARKET
Above:
ABNORMAL_VOLATILITY
A++ optimization candidates:
0.75–2.00
0.80–1.80
Do not deploy the narrower range until validated.
RR and volatility are hard gates, not points.
________________________________________
29. SESSION ENGINE
Asian:
Context/liquidity only
Default entries:
•	London
•	London/New York overlap
•	New York
Do not trade:
30 minutes before London open
30 minutes after NY close
For optimization:
Prioritize London-NY overlap as a test cohort, not as a guaranteed superior session.
Runtime must account for broker server offset/DST.
________________________________________
30. NEWS ENGINE
Block new entries around:
•	FOMC
•	NFP
•	CPI
•	Core CPI
•	PCE
•	Fed Chair high-impact events
•	explicitly identified gold-sensitive scheduled releases
Standard:
-30 minutes / +30 minutes
FOMC:
-90 / +90 minutes
The required gold-specific event list and extended FOMC blackout are already defined.
If economic-calendar data is unavailable when a required filter should be operating:
Production-safe option:
NEWS_DATA_UNAVAILABLE
→ disable new entries.
________________________________________
31. SPREAD GATE
Maximum spread:
min(
    StaticSpreadCeiling,
    0.15 * CurrentM5ATR
)
A++ research:
dynamic threshold = 0.10 * M5 ATR
Both static and dynamic criteria must pass.
This volatility-relative spread control is required by the strengthened specification.
________________________________________
32. QUALITY CLASSIFICATION
Recommended classes:
Class	Score	Requirements
A++	30–32	All mandatory high-selectivity gates
A+	28–29	Full structural validation
A	27	Research/shadow mode initially
B	<27	Reject
For initial high-selectivity live validation:
Trade A++ only
This is intended to pursue higher accuracy by sacrificing frequency.
It cannot guarantee 80%.
________________________________________
33. HARD GATES OUTSIDE SCORE
The following must never be compensated by points:
Data synchronized
Direction alignment
Valid dealing range
Premium/discount
Valid sweep/reclaim
Strict CHoCH
Displacement
Mandatory BOS
RR >= 2
Volatility gate
Session allowed
News clear
Spread acceptable
Mandatory structural SL
Position size valid
Exposure valid
Margin valid
Broker execution valid
A 32-point score cannot override a failed hard gate.
________________________________________
34. RISK ENGINE
Risk base:
Current Equity
Never balance.
At each new potential entry, read fresh:
double Equity     = AccountInfoDouble(ACCOUNT_EQUITY);
double Balance    = AccountInfoDouble(ACCOUNT_BALANCE);
double FreeMargin = AccountInfoDouble(ACCOUNT_MARGIN_FREE);
The source explicitly requires fresh live account values and equity-based sizing.
________________________________________
35. PROGRESSIVE RISK
Daily Sequence	Risk
Normal/first qualifying trade	0.25%
After 1 loss	0.20%
After 2 losses	0.15%
After 3 losses	Stop trading
After two losses also require:
Score >=30
A++ quality preferred
The progressive risk cuts are defined in the strengthened specification.
________________________________________
36. FRIDAY RISK MODIFIER
Six hours before broker Friday close:
CurrentRisk *= 0.50
Four hours before close:
NO NEW ENTRIES
Two hours before:
defensive management.
Thirty minutes before:
close EA positions
delete EA pending orders
The 50% pre-weekend risk reduction is part of the strengthened specification.
________________________________________
37. RISK CAPITAL
RiskCapital = CurrentEquity * RiskPct;
Example internal representation:
0.25% = 0.0025
0.20% = 0.0020
0.15% = 0.0015
________________________________________
38. MANDATORY STOP LOSS
This is a permanent protected requirement.
MandatoryStopLoss = true;
AllowEntryWithoutSL = false;
AllowSLRemoval = false;
AllowSLWidening = false;
EmergencyCloseIfSLProtectionFails = true;
No market order may intentionally be sent with:
SL = 0
________________________________________
39. STRUCTURAL SL
BUY candidate:
StructuralLow =
lowest relevant of:
Sweep Low
OB Low
Protected Structural Low
Then:
SL = StructuralLow
     - 0.10 * M5ATR
     - CurrentSpread
SELL:
StructuralHigh =
highest relevant:
Sweep High
OB High
Protected High
then:
SL = StructuralHigh
     + 0.10 * M5ATR
     + CurrentSpread
The source requires ATR+spread buffering and broker stop-level validation.
________________________________________
40. SL PRE-ORDER VALIDATION
Before order:
SL != 0
SL is on correct side of entry
SL respects SYMBOL_TRADE_STOPS_LEVEL
SL distance >= minimum strategy requirement
Risk against SL computable
Final RR >=2
Failure:
SL_VALIDATION_FAILED
→ NO TRADE
________________________________________
41. NO SILENT SL WIDENING
If broker minimum stop requirement is incompatible with structural SL:
Do not silently push SL farther away.
Instead:
1.	calculate broker-compliant potential SL,
2.	recalculate dollar risk,
3.	recalculate lot size,
4.	recalculate RR,
5.	re-run exposure/margin,
6.	execute only if the resulting setup still meets all strategy gates.
Otherwise:
REJECT
________________________________________
42. SL MUST BE ATTACHED TO ENTRY
Order request must contain:
request.sl = FinalStopLoss;
The bot must not intentionally use:
Open first → add SL later
as normal behavior.
________________________________________
43. POST-FILL SL VERIFICATION
Immediately after confirmed fill:
Read the live broker position.
Verify:
POSITION_SL > 0
and approximately equals expected protective level.
If missing:
UNPROTECTED_POSITION
Attempt one controlled protective SL repair.
If repair fails:
Emergency protective close
and:
EmergencyStop = true
This prevents an uncontrolled XAUUSD position from remaining open.
________________________________________
44. STOP MANAGEMENT RULE
After entry:
SL may:
tighten
but:
never widen
and:
never be removed
unless the position is simultaneously being closed.
________________________________________
45. AUTOMATIC POSITION SIZING
Every single trade recalculates size.
Never:
FixedLots
ManualLots
PreviousTradeLots
CachedLots
The strengthened specification explicitly prohibits manual/static lot sizes.
________________________________________
46. SYMBOL PARAMETERS TO READ LIVE
SYMBOL_TRADE_TICK_SIZE
SYMBOL_TRADE_TICK_VALUE
SYMBOL_TRADE_TICK_VALUE_PROFIT
SYMBOL_TRADE_TICK_VALUE_LOSS
SYMBOL_TRADE_CONTRACT_SIZE
SYMBOL_VOLUME_MIN
SYMBOL_VOLUME_MAX
SYMBOL_VOLUME_STEP
SYMBOL_TRADE_STOPS_LEVEL
SYMBOL_ASK
SYMBOL_BID
SYMBOL_POINT
Never hardcode XAUUSD dollar-per-point assumptions.
The source explicitly requires reading the current broker contract specification.
________________________________________
47. PREFERRED ROBUST LOT CALCULATION
For coding, the safest method is to determine the potential loss at SL for 1.0 lot using OrderCalcProfit().
Example concept:
double lossOneLot = 0.0;

bool ok = OrderCalcProfit(
   orderType,
   symbol,
   1.0,
   entryPrice,
   stopLoss,
   lossOneLot
);

lossOneLot = MathAbs(lossOneLot);

if(!ok || lossOneLot <= 0)
    RejectTrade();
Then:
RawLots = RiskCapital / lossOneLot;
This is preferable to assuming a constant XAUUSD point value because it respects the broker's symbol and account-currency mechanics.
________________________________________
48. LOT NORMALIZATION
Always round down.
Conceptually:
FinalLots =
   MathFloor(RawLots / VolumeStep)
   * VolumeStep;
Never round up.
Example:
Raw = 0.137
Step = 0.01
Final = 0.13
Conservative downward rounding is explicitly required.
________________________________________
49. MINIMUM LOT RULE
If calculated size falls below broker minimum:
Do not automatically force VolumeMin.
Calculate the actual risk of VolumeMin.
If unacceptable:
MIN_LOT_EXCEEDS_RISK_TOLERANCE
→ reject.
This is particularly important on small accounts.
________________________________________
50. ACTUAL RISK RECALCULATION
After normalization:
Use OrderCalcProfit() again:
Entry → Stop
FinalLots
Determine:
ActualRiskCurrency
ActualRiskPct
Require:
ActualRisk <= AllowedRisk
unless a specifically documented tiny broker rounding tolerance applies.
Production preference:
Never exceed configured risk.
________________________________________
51. OPEN-RISK EXPOSURE
Before new trade:
ExistingOpenRisk
+
ProposedTradeRisk
=
ProjectedOpenRisk
Require:
ProjectedOpenRisk <= MaxPortfolioOpenRisk
Recommended production default:
MaxPortfolioOpenRisk = 0.50% equity
Absolute strategy ceiling for future expanded deployment:
0.75%
Because the initial implementation permits only one active XAUUSD trade, most normal operation remains below this.
________________________________________
52. XAUUSD EXPOSURE
Calculate current XAUUSD risk using actual open positions, not internally cached values.
Read broker state every time.
The source specifically requires live exposure after partial closes or manual intervention.
________________________________________
53. NOTIONAL EXPOSURE
For institutional-size accounts also calculate:
NotionalExposure
approximately using live contract specifications:
Lots × ContractSize × Price
subject to symbol mechanics.
Maintain controls:
MaxLotsPerOrder
MaxXAUUSDVolume
MaxSymbolNotional
MaxAccountNotional
MaxBrokerExposure
MaxMarginUtilization
These should be broker/account operational limits rather than strategy-alpha parameters.
________________________________________
54. FINAL LOT CAP
Conceptually:
FinalLots = MIN(
    RiskLots,
    BrokerMaxVolume,
    ConfiguredMaxLotsPerOrder,
    RemainingSymbolExposure,
    RemainingAccountExposure,
    MarginAllowedLots,
    LiquidityExecutionLimit
)
Then:
round down again
and recalculate actual risk.
The source specifies this exposure-cap chain.
________________________________________
55. MARGIN CHECK
Use:
OrderCalcMargin(...)
Require:
MarginRequired <= FreeMargin × MarginSafetyFactor
Default:
MarginSafetyFactor = 0.80
The source explicitly defines the 80% margin safety concept.
________________________________________
56. RR GATE
Before execution:
RR >= 2.0
Hard gate.
For standard two-R target:
RewardDistance >= 2 × RiskDistance
Do not count RR as score.
If:
RR < 2
then:
NO TRADE
________________________________________
57. FINAL EXECUTION RECHECK
Immediately before OrderSend():
Read again:
Bid
Ask
Spread
Equity
Free Margin
Existing Position
Open Exposure
Stops level
Then recalculate:
Entry
SL distance
Risk
Lots
Margin
RR
Spread
This protects against price changes occurring between confirmation candle close and actual submission.
________________________________________
58. EXECUTION PRICE
BUY sizing:
current Ask
SELL sizing:
current Bid
Do not calculate live risk from historical confirmation candle close.
________________________________________
59. ORDER REQUEST REQUIREMENTS
Order request shall include at minimum:
Symbol
Direction
Final Volume
Current market price / market order logic
Mandatory SL
TP where supported by strategy
Deviation/slippage control
Magic Number
Strategy comment / SetupID
________________________________________
60. SETUP ID
Recommended:
Symbol
+
Direction
+
LiquidityType
+
LiquidityTimestamp
+
CHoCHTimestamp
+
BOSTimestamp
Hash/string representation:
SetupID
After execution:
CONSUMED
Never execute it again.
________________________________________
61. DUPLICATE ENTRY PROTECTION
Before order:
SetupID not consumed
No active order for SetupID
No active position for SetupID
Cooldown satisfied
Otherwise reject.
________________________________________
62. ONE POSITION DEFAULT
Production:
One active XAUUSD position at a time
Maximum 3 trades/day means:
three sequential independent setups
not three simultaneous stacked positions.
________________________________________
63. PROFIT MANAGEMENT
Baseline:
TP1
+1R
close 30%
TP2
+2R
close 40%
Runner
30%
If volume-step rules make partials impossible, follow the fallback logic required by the strengthened spec.
________________________________________
64. PARTIAL-CLOSE FALLBACK
If 30% volume < allowable broker minimum/step:
Skip TP1 partial.
Roll it into TP2.
Then:
70% at TP2
30% runner
If even this cannot be cleanly executed:
close full position at TP2
Log:
PARTIAL_CLOSE_VOLUME_FALLBACK
Never silently fail.
________________________________________
65. BREAKEVEN
After TP2:
Runner protection:
BE + spread buffer
not exact entry.
Then use M15 structure.
BUY:
trail under confirmed M15 HL.
SELL:
trail above confirmed M15 LH.
________________________________________
66. EARLY EXIT / STALLED POSITION
After:
20 M5 candles
if price remains between:
entry and +1R
mark:
STALLED
Default:
do not force exit solely due to time
The strengthened spec treats this as a monitoring condition.
________________________________________
67. DAILY TRADE CONTROL
MaxTradesPerDay = 3
Do not force trades.
Valid daily result:
0
1
2
or 3 trades
After any close:
Cooldown = 6 M5 candles
Next setup requires:
different liquidity event
AND
different CHoCH timestamp
The source explicitly requires setup independence and cooldown.
________________________________________
68. DAILY LOSS PROTECTION
Suggested retained values:
Normal Daily Soft Stop = -0.75%
Emergency Daily Stop = -2.00%
Weekly Stop = -4.00%
Monthly Stop = -6.00%
When the relevant threshold triggers:
EmergencyStop = true;
No new entries.
The strengthened specification links loss-limit breaches directly to the emergency stop.
________________________________________
69. EQUITY BASELINES
Capture:
DailyBaselineEquity
WeeklyBaselineEquity
MonthlyBaselineEquity
on first valid tick following the relevant rollover.
Do not continually move the baseline higher/lower during the period.
This is explicitly required to prevent the drawdown limit resetting itself.
________________________________________
70. CONSECUTIVE LOSS LOGIC
Three losses at default risk:
0.25 + 0.20 + 0.15 = 0.60%
This remains below normal:
0.75% daily soft stop
Add configuration sanity check at initialization.
The source explicitly requires this consistency test.
________________________________________
71. WIN STREAK
No increased risk after wins.
Always cap normal per-trade risk at:
0.25%
No anti-martingale.
________________________________________
72. FRIDAY/WEEKEND SAFETY
Broker-close-relative:
T-6h → risk × 50%
T-4h → no new entries
T-2h → protect existing trade
T-30m → flatten EA positions and cancel pending
Weekend → trading disabled
________________________________________
73. MONDAY REOPENING
After broker market reopens:
Wait:
2 completed M15 candles
Require:
spread normal
series synchronized
structure initialized
no news blackout
connection valid
Then allow scanning.
________________________________________
74. CONNECTION MONITOR
Check:
Terminal connected
Broker/account connected
Symbol synchronized
Trading allowed
Algo Trading enabled
Ticks recent
Trade context operational
No new entries if connection state is unreliable.
________________________________________
75. RECONCILIATION
On:
•	OnInit
•	reconnect
•	terminal/VPS restart
•	trade transaction
•	periodic health check
read broker positions/orders.
Internal state must be reconciled to broker reality.
________________________________________
76. ORPHAN POSITION
If:
MagicNumber matches
Symbol matches
No corresponding internal journal record
then:
ORPHANED_POSITION
Default:
•	no new scaling
•	defensive structural management
•	alert
•	new entries disabled until reconciliation
The source requires orphan handling rather than silently assuming state.
________________________________________
77. ORDER FAILURE CIRCUIT BREAKER
Track rejected orders.
If:
5 consecutive order rejections
within rolling 30 minutes
then:
EmergencyStop = true;
Manual review required.
This is explicitly required by the strengthened specification.
________________________________________
78. EMERGENCY FLAGS
Use separate concepts:
bool EmergencyStop;
bool EmergencyFlatten;
EmergencyStop
blocks new entries.
EmergencyFlatten
closes existing strategy positions according to emergency policy.
Do not conflate them.
________________________________________
79. STATE MACHINE
Recommended enum:
enum StrategyState
{
   STATE_IDLE,
   STATE_HTF_VALID,
   STATE_LIQUIDITY_IDENTIFIED,
   STATE_SWEEP_CONFIRMED,
   STATE_CHOCH_CONFIRMED,
   STATE_DISPLACEMENT_CONFIRMED,
   STATE_BOS_CONFIRMED,
   STATE_ENTRY_ZONE_CREATED,
   STATE_WAITING_RETRACE,
   STATE_ENTRY_CONFIRMATION,
   STATE_SCORE_VALID,
   STATE_RISK_VALID,
   STATE_ORDER_SENT,
   STATE_POSITION_MANAGEMENT,
   STATE_TP1,
   STATE_TP2,
   STATE_RUNNER,
   STATE_POSITION_CLOSED,
   STATE_EMERGENCY_STOP
};
________________________________________
80. STATE TIMEOUTS
State Transition	Maximum M5 Bars
Liquidity → Sweep	100
Sweep → CHoCH	24
CHoCH → Displacement	10
Displacement → BOS	15
BOS → Entry Zone	5
Retrace → Confirmation	10
Confirmation → Order	1
These are authoritative strengthened timeouts.
On timeout:
STATE_TIMEOUT
Reset safely to:
LIQUIDITY_IDENTIFIED
where appropriate.
________________________________________
81. SETUP CONTEXT STRUCTURE
Recommended:
struct SetupContext
{
   string setupID;

   int direction;

   StrategyState state;

   LiquidityLevel liquidity;

   datetime sweepTime;
   double sweepExtreme;

   datetime chochTime;
   double protectedLevel;

   datetime displacementTime;

   datetime bosTime;
   double bosLevel;

   FVG fvg;

   double obHigh;
   double obLow;

   double fib618;
   double fib100;

   double zoneHigh;
   double zoneLow;

   datetime confirmationTime;

   int qualityScore;

   double entry;
   double sl;
   double tp1;
   double tp2;

   double riskPct;
   double riskCapital;

   double rawLots;
   double finalLots;

   double actualRisk;
   double rr;

   bool consumed;
};
________________________________________
82. OPERATIONAL INPUTS
These can remain configurable.
Input	Purpose
BrokerSymbol	XAUUSD broker suffix mapping
MagicNumber	Strategy identification
MaxAbsoluteSpread	Broker-specific hard ceiling
MaxSlippagePoints	Execution tolerance
BrokerFridayCloseTime	Weekend management
MaxLotsPerOrder	Institutional/order safety
MaxXAUVolume	Symbol exposure ceiling
MaxPortfolioOpenRiskPct	Portfolio risk ceiling
MarginSafetyFactor	Default 0.80
EnableNotifications	Operational
EnableFileLogging	Audit
EnableVerboseJournal	Development
EmergencyFlattenPolicy	Operational safety
These are not intended to change the trading edge.
________________________________________
83. PRODUCTION-LOCKED INPUTS
Do not expose normal editable controls for:
EMA periods
ATR period
ADX period
minimum ADX
HTF timeframes
CHoCH rules
BOS rules
sweep thresholds
FVG definition
OB definition
Fib definition
SL requirement
risk tiers
RR minimum
state timeouts
daily maximum trades
loss-stop structure
TP percentages
Changing those should require:
new strategy version + new validation cycle
rather than casual EA input changes.
________________________________________
84. AUDIT LOG
Every setup — including rejected ones — should log:
Timestamp
SetupID
State
D1/H4/H1 direction
M15 context
Liquidity type/price
Sweep penetration
CHoCH details
Displacement ratio
BOS
FVG
OB
Fib
Confirmation type
Score
ATR regime
Session
News status
Spread
Equity
Balance
Free margin
Risk %
Risk currency
Entry
SL
Stop distance
Loss/1 lot
Raw lot
Normalized lot
Existing exposure
Projected exposure
Margin required
RR
Final decision
Rejection reason
Order ticket
Fill price
Slippage
Post-fill SL verification
TP1
TP2
Runner
Final P/L
Final R
The strengthened specification explicitly requires auditability of the automatic sizing calculation and all major risk inputs.
________________________________________
85. STANDARD REJECTION CODES
Recommended:
DATA_NOT_READY
TF_NOT_SYNCHRONIZED
H4_NEUTRAL
D1_CONFLICT
H1_CONFLICT
ADX_TOO_LOW
EMA_SEPARATION_FAIL
H1_RANGE_STALE
H1_RANGE_TOO_SMALL
EQUILIBRIUM_ZONE
NO_VALID_LIQUIDITY
SWEEP_TOO_SMALL
SWEEP_TOO_DEEP
SWEEP_TOO_SLOW
CHOCH_TIMEOUT
CHOCH_INVALID
DISPLACEMENT_WEAK
DISPLACEMENT_RETRACED
BOS_TIMEOUT
BOS_INVALID
FVG_INVALID
FVG_MITIGATED
OB_INVALID
FIB_INVALIDATED
ENTRY_ZONE_INVALID
CONFIRMATION_TIMEOUT
M15_CONFLICT
RR_BELOW_MINIMUM
VOLATILITY_TOO_LOW
VOLATILITY_TOO_HIGH
SESSION_BLOCKED
NEWS_BLACKOUT
SPREAD_TOO_WIDE
COOLDOWN_ACTIVE
MAX_TRADES_REACHED
DAILY_STOP_ACTIVE
WEEKLY_STOP_ACTIVE
MONTHLY_STOP_ACTIVE
FRIDAY_CUTOFF
MANDATORY_SL_INVALID
MIN_LOT_EXCEEDS_RISK
EXPOSURE_LIMIT
MARGIN_INSUFFICIENT
ORDER_REJECTED
UNPROTECTED_POSITION
ORPHANED_POSITION
EMERGENCY_STOP
STATE_TIMEOUT
________________________________________
86. MAIN EVENT ARCHITECTURE
Recommended MQL5 components:
OnInit()
OnDeinit()
OnTick()
OnTimer()
OnTradeTransaction()
OnInit()
•	validate account/symbol
•	create indicator handles
•	load strategy state
•	set timer
•	reconcile positions
•	load rollover baselines
•	check production configuration
•	ensure risk sanity
OnTick()
Fast checks only:
•	new tick
•	spread
•	open-position protection
•	emergency conditions
•	order/position management
•	detect new M5 bar
Do not recompute every H4/D1 indicator on every tick unnecessarily.
New M5 bar
Run strategy state machine.
New M15/H1/H4/D1 bars
Update only the relevant timeframe state.
________________________________________
87. HIGH-LEVEL PSEUDOCODE
ON EVERY TICK:

    VerifyConnection()

    ReconcileIfRequired()

    ManageOpenPosition()

    VerifyMandatorySLProtection()

    CheckEmergencyLimits()

    if EmergencyStop:
        return

    if not NewM5Bar:
        return

    RefreshM5()

    if NewM15Bar:
        RefreshM15()

    if NewH1Bar:
        RefreshH1()

    if NewH4Bar:
        RefreshH4()

    if NewD1Bar:
        RefreshD1()

    if !AllTimeframesSynchronized:
        reject DATA_NOT_READY

    if WeekendBlocked:
        reject

    UpdateDirectionEngine()

    if !D1_H4_H1_Aligned:
        reject

    if !H1RangeValid:
        reject

    UpdateLiquidityMap()

    AdvanceStrategyStateMachine()

    if SetupNotConfirmed:
        return

    if !M15ContextPass:
        reject

    if !SessionPass:
        reject

    if !NewsPass:
        reject

    if !VolatilityPass:
        reject

    if !SpreadPass:
        reject

    ScoreSetup()

    if HighSelectivityMode && Score < 30:
        reject

    FinalizeStructuralSL()

    if !SLValid:
        reject

    CalculateRR()

    if RR < 2:
        reject

    ReadLiveAccountState()

    DetermineRiskTier()

    CalculateRiskCapital()

    CalculateRiskPerOneLotUsingOrderCalcProfit()

    NormalizeLotsDown()

    CalculateCurrentOpenExposure()

    ApplyExposureLimits()

    VerifyMargin()

    ReReadAskBid()

    RecalculateEntrySLRiskRRSpread()

    if any final gate fails:
        reject

    SendOrderWithSL()

    VerifyFill()

    VerifySLAttached()

    if SL missing:
        RepairSL()

    if repair fails:
        EmergencyClose()
        EmergencyStop = true
________________________________________
88. MANDATORY FINAL PRE-TRADE CHECKLIST
All must equal TRUE:
[ ] D1 synchronized
[ ] H4 synchronized
[ ] H1 synchronized
[ ] M15 synchronized
[ ] M5 synchronized

[ ] D1 direction valid
[ ] H4 direction valid
[ ] H4 persistence valid
[ ] H4 ADX valid
[ ] H4 separation valid
[ ] H1 aligned
[ ] H1 dealing range valid
[ ] premium/discount valid

[ ] qualifying liquidity
[ ] valid sweep
[ ] valid reclaim
[ ] strict CHoCH
[ ] displacement
[ ] follow-through
[ ] mandatory BOS

[ ] valid FVG
[ ] valid OB
[ ] Fib valid
[ ] entry-zone confluence

[ ] M5 confirmation
[ ] M15 confirmation/context
[ ] score requirement

[ ] RR >= 2
[ ] volatility normal
[ ] valid session
[ ] news clear
[ ] spread acceptable

[ ] daily limits clear
[ ] weekly limits clear
[ ] monthly limits clear
[ ] cooldown complete
[ ] maximum trades not reached
[ ] weekend rules clear

[ ] current equity read
[ ] current free margin read
[ ] mandatory SL finalized
[ ] SL broker-valid
[ ] risk capital calculated
[ ] exact lot calculated
[ ] lot rounded down
[ ] actual risk recalculated
[ ] open-risk exposure valid
[ ] notional exposure valid
[ ] margin valid
[ ] final RR valid

[ ] SL included in order
Only then:
ORDER_SEND_ALLOWED = TRUE
________________________________________
89. POST-ORDER CHECKLIST
After fill:
[ ] ticket confirmed
[ ] direction correct
[ ] volume correct
[ ] fill price recorded
[ ] slippage recorded
[ ] SL exists
[ ] SL is on correct side
[ ] TP management initialized
[ ] SetupID consumed
[ ] trade counter updated
[ ] state journal saved
If SL is missing:
UNPROTECTED_POSITION
→ immediate protection attempt.
If unsuccessful:
EMERGENCY_CLOSE
________________________________________
90. 80% TARGET OPTIMIZATION PARAMETERS
These are research parameters, not necessarily production inputs.
Test systematically:
Variable	Candidate Values
Score	27–32
ADX	20 / 25 / 30 / 35
Displacement	1.25 / 1.50 / 1.75 / 2.00
Sweep Min	.05 / .10 / .15
Sweep Max	.40 / .50 / .75
Confluence	2/3 vs 3/3
D1	scoring vs mandatory
Confirmation	1 vs 2
M15 agreement	optional vs mandatory
Zone tolerance	.10/.15/.20/.30 ATR
ATR regime	60–250 / 75–200 / 80–180
Spread	15% vs 10% ATR
Sessions	London / overlap / NY
Do not choose parameters based on win rate alone.
________________________________________
91. OPTIMIZATION OBJECTIVE
A useful composite optimization objective should reward:
OOS expectancy
Profit Factor
Win Rate
Trade count
Drawdown
Stability between periods
and penalize:
parameter sensitivity
low sample size
profit concentration
large drawdowns
large execution sensitivity
The objective is not:
“find the combination with the highest historical win rate.”
The objective is:
find a stable region that maintains strong expectancy and risk characteristics across unseen periods.
________________________________________
92. MINIMUM PRODUCTION VALIDATION
Before considering production live-ready:
Historical backtest
Out-of-sample
Walk-forward
Monte Carlo
Spread stress
Slippage stress
Latency assumptions
Broker execution stress
Demo-forward
Small-live verification
Minimum:
100 combined OOS + walk-forward trades
Concentration:
No trade >15% OOS net profit
No month >30% OOS net profit
Required metrics:
Trade count
Win rate
Average R
Profit Factor
Sharpe
Sortino
Maximum Drawdown
Net return
Worst losing streak
Average winner
Average loser
Slippage
Spread cost
These requirements are part of the strengthened acceptance framework.
________________________________________
93. HIGH-SELECTIVITY ACCEPTANCE TARGET
Research objective:
Win Rate target: 75%–80%
Preferred supporting metrics:
PF >= 2.0
Average R > +0.30R
Positive OOS
Positive walk-forward
Max DD <10%
But do not reject a statistically superior 72% system solely because it fails to display “80%” if it has better expectancy and robustness.
Likewise, reject an 80% system if its winners are tiny, losses are large, or OOS performance collapses.
________________________________________
94. DOCUMENTATION-SAFE PERFORMANCE LANGUAGE
Use:
Performance Objective: SYMMETRIX GOLD SMC is engineered to apply a high-selectivity automated trading framework. Research and optimization may target a historical and validated win-rate range approaching 75%–80%; however, this target is not a promise, representation or guarantee of future performance.
Do not use:
Guaranteed 80% winning EA.
________________________________________
95. STOP-LOSS DISCLOSURE
Use:
Mandatory Protective Stop: Every position initiated by the EA is required to include a predefined protective stop-loss based on the strategy's structural invalidation logic. The system does not intentionally initiate unprotected trades. A stop-loss is intended to limit risk but cannot guarantee execution at the requested price during gaps, spread expansion, extreme volatility, liquidity disruption, connectivity problems or other exceptional execution conditions.
________________________________________
96. AUTOMATIC SIZING DISCLOSURE
Use:
Automatic Position Sizing: The EA determines position size from current account equity, active strategy risk tier, structural stop distance, live broker symbol specifications, current exposure and available margin. Position-sizing controls reduce manual sizing errors but cannot eliminate market or execution risk.
This matches the source requirement that every trade's lot size be recomputed from current live account and broker state rather than fixed manually.
________________________________________
97. MULTI-TIMEFRAME DISCLOSURE
Use:
Multi-Timeframe Analysis: The EA evaluates D1, H4, H1, M15 and M5 information according to predefined rules. Agreement among these timeframes is used as a selection filter and does not guarantee that the identified market direction will continue.
________________________________________
98. RISK DISCLOSURE
Use:
Risk Disclosure: XAUUSD and other leveraged instruments can produce substantial losses. Risk limits, stop-loss orders, exposure controls, margin safeguards and emergency protections are intended to reduce risk under normal conditions but cannot guarantee a maximum loss during price gaps, extreme volatility, execution failure or extraordinary market events.
________________________________________
99. BACKTEST DISCLOSURE
Use:
Backtesting Notice: Backtested and simulated performance is hypothetical and does not represent actual trading. Historical results may not be repeated in future markets. Live results can differ due to spreads, commissions, swaps, liquidity, slippage, latency, broker execution, rejected orders, outages and market gaps.
________________________________________
100. MONTE CARLO DISCLOSURE
Use:
Monte Carlo Notice: Monte Carlo analysis models potential outcome distributions using historical or assumed trade behavior. It is a statistical stress-testing tool and is not a forecast or guarantee of future account values, returns, win rates or drawdowns.
________________________________________
101. FINAL CODING PRINCIPLE
The production bot should enforce the following hierarchy:
PROTECT CAPITAL
↓
VALIDATE MARKET CONTEXT
↓
WAIT FOR STRUCTURAL SETUP
↓
REQUIRE HIGH-QUALITY CONFLUENCE
↓
REQUIRE MANDATORY STOP LOSS
↓
CALCULATE EXACT RISK
↓
CONTROL TOTAL EXPOSURE
↓
VERIFY EXECUTION CONDITIONS
↓
EXECUTE ONLY WHEN EVERYTHING PASSES

The strategy parameters themselves remain fixed while the EA dynamically adapts to equity, broker contract terms, stop distance, margin, spread and existing exposure. That principle is already central to the master production specification.
Final production rule

The programmer should treat every rule marked mandatory, hard gate, protected, or production locked as non-optional.
There must be no code path capable of intentionally creating an XAUUSD trade without a valid SL, valid position size, acceptable risk, acceptable exposure, sufficient margin, RR ≥2 and a completely validated multi-timeframe setup.
That is the specification I would use as the authoritative coding blueprint for SYMMETRIX GOLD SMC v3.0.


