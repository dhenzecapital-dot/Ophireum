Ophireum Multimedia Productions 

Mobile Phone: +63 917 999 8814 
Email: dhenzecapital@gmail.com
telegram: @Mkogexchange
WhatsApp: +63 995 715 1043


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

