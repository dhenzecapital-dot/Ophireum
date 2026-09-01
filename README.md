

SYMMETRIX GOLD SMC

A high-selectivity, multi-timeframe XAUUSD trading system for MetaTrader 5, engineered around Smart Money Concepts, liquidity events, market structure, and strict risk controls.

Status: Production candidate specification — validation and broker-specific integration are still required before live deployment.

Document Purpose

This document is the authoritative high-level reference for the strategy owner, MQL5 developers, Python engineers, quantitative reviewers, testers, deployment operators, and compliance reviewers working on SYMMETRIX GOLD SMC.

It defines:

the economic and structural logic of the trading strategy;

the required software architecture and module boundaries;

mandatory entry, sizing, execution, and risk-control rules;

state transitions, invalidation rules, and failure behavior;

audit, testing, validation, and production-acceptance requirements; and

the limitations and disclosures that must accompany any performance presentation.

This README does not replace source-level API documentation, unit-test specifications, release notes, broker configuration records, or validation reports. Those artifacts should be maintained separately and versioned with the production code.

Table of Contents

Overview

System Profile

Core Design Principles

Strategy Workflow

Entry Model

Market Direction and Structure

Liquidity Model

Mandatory Risk Controls

Stop-Loss and Position Management

Session, Volatility, and News Protection

Software Architecture

Programming Requirements

Reference MQL5 Implementation

State Expiration Rules

Installation

Configuration

Logging and Auditability

Testing and Validation

Production Deployment

Performance and Risk Disclosures

Overview

SYMMETRIX GOLD SMC is a fully automated Expert Advisor (EA) specification for trading XAUUSD (gold) on MetaTrader 5. The strategy translates Smart Money Concepts (SMC/ICT) into a deterministic state machine rather than relying on a single indicator or discretionary signal.

The system evaluates higher-timeframe direction, maps external liquidity, waits for a qualifying sweep and reclaim, confirms a change of character (CHoCH), requires displacement and a break of structure (BOS), and then evaluates a retracement into a confluence-based entry zone.

Capital preservation takes priority over trade frequency. The EA may take up to three qualified trades per day, but zero trades is considered a valid outcome when no setup satisfies every required condition.

System Profile

Parameter

Value

Product

SYMMETRIX GOLD SMC

Organization

Ophireum Multimedia Productions

Version

3.0 Production Candidate

Target platform

MetaTrader 5

Production language

MQL5

Reference implementation

Python with the official MetaTrader5 package

Primary instrument

XAUUSD

Execution timeframe

M5

Confirmation timeframe

M15

Structural timeframe

H1

Trend timeframe

H4

Macro timeframe

D1

Strategy model

Multi-Timeframe SMC / Liquidity / Structure

Execution

Fully automatic

Position sizing

Fully automatic; manual fixed lots prohibited

Protective stop-loss

Mandatory

Maximum trades

3 per day

Default risk

0.25% of current equity per trade

Martingale / grid

Prohibited

Production lock

Enabled

Core Design Principles

The system enforces the following hierarchy:

Protect capital.

Validate the market context.

Wait for a complete structural setup.

Require high-quality confluence.

Define a mandatory protective stop-loss.

Calculate position risk from live account data.

Control aggregate exposure and margin use.

Verify spread, liquidity, and execution conditions.

Execute only when every mandatory gate passes.

No rule marked mandatory, hard gate, protected, or production locked should be bypassable in a production build.

Strategy Workflow

flowchart TD
    A["Higher-timeframe bias"] --> B["Liquidity identified"]
    B --> C["Sweep and reclaim"]
    C --> D["CHoCH confirmed"]
    D --> E["Displacement"]
    E --> F["BOS confirmed"]
    F --> G["FVG + OB + Fibonacci zone"]
    G --> H["M5 confirmation"]
    H --> I["Score, RR, volatility and risk gates"]
    I --> J["Order with structural SL"]
    J --> K["Managed exit"]

Any failed or expired stage invalidates the setup. Signals from unrelated market regimes must never be chained together.

Entry Model

A trade is eligible only after completing the required sequence:

D1, H4, and H1 market context is evaluated.

H4 direction persists for at least three closed candles.

EMA separation and ADX thresholds confirm a tradeable trend regime.

The active H1 dealing range is recent, structurally meaningful, and outside its equilibrium exclusion zone.

An eligible liquidity pool is selected using the defined priority hierarchy.

Price performs a time-bounded liquidity sweep and reclaim.

A confirmed structural swing produces a strict CHoCH.

Directional displacement demonstrates follow-through.

Price closes through a pre-existing confirmed structural level to establish BOS.

An entry zone contains qualifying FVG, Order Block, and Fibonacci confluence.

An explicit M5 confirmation trigger occurs within the permitted window.

Reward-to-risk, volatility, session, news, spread, exposure, and margin gates all pass.

Supported M5 Confirmation Triggers

Engulfing candle: the current body fully engulfs the previous candle body in the intended direction.

Rejection candle: the rejection wick represents at least 60% of the candle range and price closes in the intended direction.

Internal micro-BOS: an M5 close breaks the most recent internal swing point in the intended direction.

Market Direction and Structure

The direction engine uses EMA alignment, ATR-normalized EMA separation, persistence, and ADX strength. A raw moving-average crossover is insufficient.

H4 EMA separation: at least 0.15 × H4 ATR

H1 EMA separation: at least 0.10 × H1 ATR

H4 persistence: at least 3 consecutive closed candles

ADX(14): at least 20

H1 range age: no more than 40 candles

H1 range size: at least 3.0 × H1 ATR

Equilibrium exclusion: ±5% around the range midpoint

When these requirements are not satisfied, the market direction is treated as neutral and new entries are blocked.

Liquidity Model

Eligible liquidity is ranked in the following order:

Previous Week High / Low

Previous Day High / Low

Equal Highs / Equal Lows

Asian Session High / Low

Confirmed H1 swings

Confirmed M15 swings

Within the same priority tier, the nearest qualifying pool in the anticipated sweep direction is selected. Levels untouched for more than ten trading days are downgraded by one tier.

Mandatory Risk Controls

Dynamic Position Sizing

Fixed or manually entered lot sizes are prohibited. Volume must be recalculated immediately before every order using:

current account equity;

the active risk tier;

structural stop distance;

live tick size and tick value;

broker volume minimum, maximum, and step;

current XAUUSD exposure;

available margin; and

configured account and execution ceilings.

The conceptual calculation is:

Risk capital = Current equity × Active risk percentage
Raw volume   = Risk capital ÷ Monetary loss per lot at the structural stop
Final volume = Minimum of all risk, broker, exposure, liquidity, and margin ceilings

Volume is always rounded down to the broker's valid step. If the broker's minimum volume would exceed the configured risk tolerance, the trade is rejected.

Progressive Risk Reduction

Daily result state

Maximum risk per new trade

No losing trade

0.25% of equity

After 1 loss

0.20% of equity

After 2 losses

0.15% of equity

Six hours before Friday close

50% of the active tier

Loss and Exposure Protection

One open strategy position at a time

Maximum three trades per day

Minimum six M5 candles between a close and the next order

Daily, weekly, and monthly equity circuit breakers

Margin safety buffer of 20%

Live exposure reconciliation before every order

Emergency stop after repeated broker rejections

Defensive handling of orphaned positions after restart or state corruption

Weekend entry restrictions, de-risking, and configurable flattening controls

Stop-Loss and Position Management

Every entry requires a structural stop-loss. For long positions, the stop is placed below the relevant structural low with an ATR and live-spread buffer; short positions use the mirrored rule.

The stop must satisfy the broker's minimum-distance requirement. If it does not, the system must recalculate the compliant stop distance, position size, and realized reward-to-risk ratio. The order is rejected if the revised setup fails any mandatory gate.

Default profit management:

Close 30% at +1R.

Close 40% at +2R.

Manage the remaining 30% as a structural runner.

Apply broker-volume rounding fallbacks when partial closes are not technically possible.

Flag trades that remain between entry and +1R for 20 M5 candles as STALLED for review.

Session, Volatility, and News Protection

New entries are permitted during configured London and New York windows.

The Asian session is used for accumulation and liquidity context by default, not for entry execution.

New entries are blocked around defined session dead zones.

M5 ATR must remain between 60% and 250% of its 50-period average.

Spread must be below both the absolute safety ceiling and 15% of current M5 ATR.

High-impact USD and gold-relevant events trigger configurable blackout windows.

FOMC events use an extended default blackout of 90 minutes before and after.

If reliable calendar data is unavailable, the production system must fail safely and block new entries.

Software Architecture

The production EA should use a modular architecture with strict separation between market analysis, decision logic, risk calculation, order execution, and position management. A signal module must never submit an order directly.

flowchart TD
    A["MT5 market and account data"] --> B["Data validation and normalization"]
    B --> C["Indicators and market structure"]
    C --> D["Strategy state machine"]
    D --> E["Risk and exposure engine"]
    E --> F["Execution gateway"]
    F --> G["Broker / MT5 trade server"]
    G --> H["Position reconciliation"]
    H --> I["Management, journal and alerts"]
    I --> D

Required Modules

Module

Responsibility

Must not do

Configuration

Load, validate, and freeze versioned parameters

Submit orders or modify live positions

Market Data

Retrieve candles, ticks, spreads, account data, and symbol properties

Infer strategy direction

Indicator Engine

Compute EMA, ATR, ADX, RSI, and supporting values from closed bars

Use future or incomplete-bar data unless explicitly permitted

Structure Engine

Detect confirmed swings, dealing ranges, CHoCH, displacement, and BOS

Size or execute orders

Liquidity Engine

Map, rank, age, and invalidate liquidity pools

Override the state machine

Confluence Engine

Detect FVG, Order Block, Fibonacci overlap, and confirmation candles

Relax mandatory gates to increase frequency

State Machine

Control sequencing, timeouts, invalidation, and duplicate prevention

Bypass risk or execution checks

Risk Engine

Calculate risk capital, structural stop, volume, exposure, margin, and RR

Send an order using stale values

Execution Gateway

Validate the final request, submit it, interpret broker responses, and control retries

Recalculate strategy signals after submission begins

Position Manager

Manage partial exits, runner logic, emergency policies, and close reconciliation

Open unrelated positions

Persistence Layer

Store state, setup IDs, baselines, journal data, and recovery information

Treat local state as superior to the broker's live position record

Monitoring and Alerts

Report circuit breakers, orphan positions, data failure, and execution faults

Automatically re-enable an emergency stop

Source-of-Truth Rules

The production program must explicitly define the authoritative source for each type of information:

The broker is authoritative for live positions, fills, deals, margin, symbol specifications, and account equity.

Closed candles are authoritative for structural confirmation and scoring.

The persisted journal is authoritative for historical strategy decisions, but it must be reconciled against broker records.

The active signed configuration is authoritative for strategy parameters.

Cached values are never authoritative for position sizing, exposure, margin, or order-state decisions.

Deterministic Processing Cycle

On every new M5 closed candle, the orchestrator should process the following sequence:

1. Confirm terminal and data-feed health.
2. Reconcile broker positions, orders, and recent deals.
3. Refresh account equity, margin, symbol specifications, and spread.
4. Update daily, weekly, and monthly circuit-breaker status.
5. Update higher-timeframe indicators only from closed candles.
6. Refresh structural swings, dealing range, and liquidity map.
7. Advance or invalidate the current setup state.
8. Evaluate session, news, volatility, spread, cooldown, and duplicate gates.
9. Build the entry, stop, and target model.
10. Calculate volume from fresh broker and account values.
11. Revalidate exposure, margin, stop distance, and RR.
12. Submit once through the execution gateway or log the rejection.
13. Persist the state transition and complete audit record.

Position-management events may also be evaluated on ticks, but signal formation and structural confirmation must use the documented closed-bar rules.

Programming Requirements

Implementation Standards

Use explicit data types for price, points, volume, money, percentages, timestamps, and trade identifiers.

Normalize price and volume using the broker's symbol digits, tick size, and volume step.

Use UTC internally wherever possible; retain broker server time and local presentation time as separate values.

Evaluate structural rules on completed candles to prevent repainting and provisional-bar inconsistencies.

Assign every setup a stable, reproducible ID derived from the liquidity event, CHoCH timestamp, direction, and symbol.

Make all order operations idempotent so a restart or repeated event cannot create a duplicate trade.

Persist every state transition before or atomically with any externally visible action where practical.

Reject invalid numerical states, including zero tick size, zero tick value, non-finite ATR, negative margin, missing candles, and inverted stop/target geometry.

Never silently substitute a default when a mandatory live value is unavailable.

Keep strategy parameters immutable during an active setup and position unless a documented migration rule applies.

Closed-Bar and Look-Ahead Protection

All backtest and live implementations must produce the same decision from the same completed market data. To enforce parity:

do not confirm CHoCH, BOS, displacement, or entry candles before bar close;

do not use a swing before the confirming candles required by the fractal definition exist;

do not select a structural level formed after the event that claims to break it;

timestamp every derived object with both its source candle and confirmation candle; and

include automated tests that fail when future bars influence an earlier decision.

Automatic Lot-Size Algorithm

The risk engine must execute the following calculation immediately before order submission:

equity        = live account equity
risk_percent  = current progressive risk tier × weekend modifier
risk_capital  = equity × (risk_percent / 100)

tick_size     = live symbol trade tick size
tick_value    = live symbol trade tick value in account currency
stop_ticks    = abs(expected_fill - stop_price) / tick_size
loss_per_lot  = stop_ticks × tick_value
raw_lots      = risk_capital / loss_per_lot

grid_lots     = floor(raw_lots / volume_step) × volume_step
final_lots    = minimum of:
                - grid_lots
                - broker order maximum
                - configured order maximum
                - remaining XAUUSD exposure
                - account exposure ceiling
                - margin-derived ceiling
                - liquidity/execution ceiling

The program must then call the platform's margin-calculation function using the intended order type, expected fill price, and final volume. If required margin exceeds 80% of current free margin by default, reject the order.

The following conditions are mandatory rejection events:

stop distance is zero or structurally invalid;

tick size, tick value, or volume step is unavailable or invalid;

normalized minimum volume exceeds permitted risk tolerance;

final volume is below the broker minimum;

available exposure is zero or negative;

margin safety requirements fail;

final reward-to-risk is below 2.0; or

account or symbol data changed materially before order submission.

Order Submission Contract

Immediately before sending an order, the execution gateway must verify:

Gate

Required result

Emergency stop

Not active

Connection and trading permission

Healthy and authorized

Setup identity

Current and not previously executed

State

RISK_VALID and not expired

Market session

Permitted

News calendar

Available and clear

Spread and volatility

Within limits

Entry, SL, and TP geometry

Valid for direction

Broker stop level

Satisfied

Position volume

Broker-valid and risk-valid

Existing exposure

Within all ceilings

Margin

Safety buffer satisfied

Reward-to-risk

At least 2.0

An order request must include the strategy magic number, setup ID or traceable comment, normalized volume, protective SL, and the approved execution policy. A market order without a valid protective stop is prohibited.

Failure and Recovery Behavior

The safest valid behavior is the default behavior:

Missing or stale market data: freeze new setup evaluation.

Unavailable economic calendar: block new entries.

Broker disconnection: stop new orders and continue recovery attempts without losing persisted state.

Ambiguous order result: reconcile orders, positions, and deals before retrying.

Five qualifying rejections within 30 minutes: activate EmergencyStop and require manual review.

Orphaned strategy position: notify, apply the documented defensive-management policy, and block conflicting new exposure.

Corrupted or incompatible persisted state: quarantine the state, reconcile live positions, and require operator review.

Restart during an open trade: rebuild management state from broker positions and deal history before taking any new action.

Security Requirements

No credentials, account identifiers, tokens, or private endpoints may be hardcoded or committed.

Runtime secrets must use operating-system or approved secret-storage facilities.

Configuration changes must be attributable, versioned, and reviewable.

Trading permissions should use the least privilege supported by the operating environment.

Logs must avoid exposing credentials while retaining sufficient broker response data for audit.

Release binaries and configuration packages should be checksummed and archived.

Emergency controls must remain available independently of the normal signal-processing loop.

Reference MQL5 Implementation

The following code provides a production-oriented foundation for the final EA. It implements the core types, closed-bar timing, state handling, live risk calculation, margin checks, mandatory stop protection, duplicate-position checks, and a guarded order-submission path.

It is intentionally a framework, not a claim of a completed profitable EA. The structural functions for liquidity, CHoCH, displacement, BOS, FVG, Order Block, Fibonacci confluence, scoring, and calendar integration must be implemented and tested against the exact rules in this document.

Main Expert Advisor: SymmetrixGoldSMC.mq5

#property strict
#property version   "3.00"
#property description "SYMMETRIX GOLD SMC — production-candidate framework"

#include <Trade/Trade.mqh>

CTrade Trade;

// -----------------------------------------------------------------------------
// Production-locked inputs
// -----------------------------------------------------------------------------
input string InpSymbol                    = "XAUUSD";
input ulong  InpMagicNumber               = 990011;
input double InpBaseRiskPercent           = 0.25;
input double InpRiskAfterOneLoss          = 0.20;
input double InpRiskAfterTwoLosses        = 0.15;
input double InpMinimumRR                 = 2.00;
input double InpMarginSafetyFactor        = 0.80;
input double InpStopATRBuffer             = 0.10;
input double InpMaxLotsPerOrder           = 5.00;
input double InpMaxTotalXAUExposure       = 5.00;
input double InpMinLotRiskTolerance       = 1.50;
input int    InpMaximumTradesPerDay       = 3;
input int    InpCooldownM5Bars            = 6;
input int    InpMaxSpreadPoints           = 500;
input bool   InpFailSafeNewsFilter        = true;

enum StrategyState
{
   STATE_IDLE = 0,
   STATE_HTF_DIRECTION_VALID,
   STATE_LIQUIDITY_IDENTIFIED,
   STATE_SWEEP_CONFIRMED,
   STATE_CHOCH_CONFIRMED,
   STATE_DISPLACEMENT_CONFIRMED,
   STATE_BOS_CONFIRMED,
   STATE_ENTRY_ZONE_CREATED,
   STATE_WAITING_FOR_RETRACE,
   STATE_ENTRY_CONFIRMATION,
   STATE_SCORE_VALID,
   STATE_RISK_VALID,
   STATE_ORDER_SENT,
   STATE_ORDER_CONFIRMED,
   STATE_POSITION_MANAGEMENT,
   STATE_POSITION_CLOSED
};

enum TradeDirection
{
   DIRECTION_NONE = 0,
   DIRECTION_BUY,
   DIRECTION_SELL
};

struct SetupContext
{
   string         setup_id;
   StrategyState  state;
   TradeDirection direction;
   datetime       state_started;
   datetime       liquidity_time;
   datetime       choch_time;
   double         liquidity_price;
   double         entry_price;
   double         stop_price;
   double         target_price;
   double         score;
};

struct RiskResult
{
   bool   approved;
   string reason;
   double equity;
   double risk_percent;
   double risk_capital;
   double tick_size;
   double tick_value;
   double stop_ticks;
   double raw_lots;
   double final_lots;
   double margin_required;
   double free_margin;
   double realized_rr;
};

SetupContext CurrentSetup;
datetime LastM5BarTime = 0;
datetime LastPositionCloseTime = 0;
bool EmergencyStop = false;
int DailyLossCount = 0;
int TradesToday = 0;

// -----------------------------------------------------------------------------
// Initialization
// -----------------------------------------------------------------------------
int OnInit()
{
   if(!SymbolSelect(InpSymbol, true))
   {
      Print("INIT_FAILED | Cannot select symbol: ", InpSymbol);
      return INIT_FAILED;
   }

   Trade.SetExpertMagicNumber(InpMagicNumber);
   Trade.SetTypeFillingBySymbol(InpSymbol);
   Trade.SetAsyncMode(false);

   ResetSetup("INITIALIZATION");
   Print("SYMMETRIX_INITIALIZED | symbol=", InpSymbol,
         " | magic=", InpMagicNumber);
   return INIT_SUCCEEDED;
}

void OnDeinit(const int reason)
{
   Print("SYMMETRIX_STOPPED | reason=", reason);
}

void OnTick()
{
   ReconcileLivePosition();
   ManageOpenPosition();

   if(!IsNewClosedM5Bar())
      return;

   ProcessNewM5Bar();
}

// -----------------------------------------------------------------------------
// Closed-bar clock
// -----------------------------------------------------------------------------
bool IsNewClosedM5Bar()
{
   datetime current_open = iTime(InpSymbol, PERIOD_M5, 0);
   if(current_open <= 0 || current_open == LastM5BarTime)
      return false;

   LastM5BarTime = current_open;
   return true; // Bar index 1 is now the newly completed candle.
}

void ProcessNewM5Bar()
{
   if(!TerminalAndAccountHealthy())
   {
      LogDecision("BLOCKED", "TERMINAL_OR_ACCOUNT_UNHEALTHY");
      return;
   }

   UpdateLossCircuitBreakers();
   if(EmergencyStop)
   {
      LogDecision("BLOCKED", "EMERGENCY_STOP_ACTIVE");
      return;
   }

   if(HasStrategyPosition())
      return;

   if(TradesToday >= InpMaximumTradesPerDay)
   {
      LogDecision("BLOCKED", "DAILY_TRADE_LIMIT");
      return;
   }

   if(!CooldownComplete())
   {
      LogDecision("BLOCKED", "POST_TRADE_COOLDOWN");
      return;
   }

   if(StateExpired(CurrentSetup.state, CurrentSetup.state_started))
      ResetSetup("STATE_TIMEOUT");

   AdvanceStrategyState();
}

State-Machine Controller

Each function below must evaluate only its named event. It must return true only when all requirements for that state have been satisfied using closed-bar data.

void AdvanceStrategyState()
{
   switch(CurrentSetup.state)
   {
      case STATE_IDLE:
         if(ValidateHigherTimeframeDirection(CurrentSetup.direction))
            SetState(STATE_HTF_DIRECTION_VALID, "HTF_DIRECTION_VALID");
         break;

      case STATE_HTF_DIRECTION_VALID:
         if(IdentifyQualifiedLiquidity(CurrentSetup))
            SetState(STATE_LIQUIDITY_IDENTIFIED, "LIQUIDITY_IDENTIFIED");
         break;

      case STATE_LIQUIDITY_IDENTIFIED:
         if(ConfirmLiquiditySweep(CurrentSetup))
            SetState(STATE_SWEEP_CONFIRMED, "SWEEP_CONFIRMED");
         break;

      case STATE_SWEEP_CONFIRMED:
         if(ConfirmStrictCHoCH(CurrentSetup))
            SetState(STATE_CHOCH_CONFIRMED, "CHOCH_CONFIRMED");
         break;

      case STATE_CHOCH_CONFIRMED:
         if(ConfirmDisplacement(CurrentSetup))
            SetState(STATE_DISPLACEMENT_CONFIRMED, "DISPLACEMENT_CONFIRMED");
         break;

      case STATE_DISPLACEMENT_CONFIRMED:
         if(InvalidatedBeforeBOS(CurrentSetup))
         {
            ResetSetup("CHOCH_PROTECTION_BROKEN");
            break;
         }
         if(ConfirmBreakOfStructure(CurrentSetup))
            SetState(STATE_BOS_CONFIRMED, "BOS_CONFIRMED");
         break;

      case STATE_BOS_CONFIRMED:
         if(CreateConfluenceEntryZone(CurrentSetup))
            SetState(STATE_ENTRY_ZONE_CREATED, "ENTRY_ZONE_CREATED");
         break;

      case STATE_ENTRY_ZONE_CREATED:
         SetState(STATE_WAITING_FOR_RETRACE, "WAITING_FOR_RETRACE");
         break;

      case STATE_WAITING_FOR_RETRACE:
         if(SetupStructurallyInvalid(CurrentSetup))
         {
            ResetSetup("STRUCTURAL_INVALIDATION");
            break;
         }
         if(ConfirmM5EntryTrigger(CurrentSetup))
            SetState(STATE_ENTRY_CONFIRMATION, "ENTRY_CONFIRMATION");
         break;

      case STATE_ENTRY_CONFIRMATION:
         if(!AllPreTradeGatesPass(CurrentSetup))
         {
            ResetSetup("PRE_TRADE_GATE_FAILED");
            break;
         }
         SetState(STATE_SCORE_VALID, "SCORE_VALID");
         ExecuteValidatedSetup();
         break;

      default:
         break;
   }
}

void SetState(const StrategyState next_state, const string reason)
{
   StrategyState previous = CurrentSetup.state;
   CurrentSetup.state = next_state;
   CurrentSetup.state_started = TimeCurrent();
   Print("STATE_CHANGE | setup=", CurrentSetup.setup_id,
         " | from=", EnumToString(previous),
         " | to=", EnumToString(next_state),
         " | reason=", reason);
}

void ResetSetup(const string reason)
{
   Print("SETUP_RESET | setup=", CurrentSetup.setup_id,
         " | reason=", reason);

   ZeroMemory(CurrentSetup);
   CurrentSetup.setup_id = BuildSetupId();
   CurrentSetup.state = STATE_IDLE;
   CurrentSetup.direction = DIRECTION_NONE;
   CurrentSetup.state_started = TimeCurrent();
}

bool StateExpired(const StrategyState state, const datetime started)
{
   int maximum_bars = 0;

   switch(state)
   {
      case STATE_LIQUIDITY_IDENTIFIED:      maximum_bars = 100; break;
      case STATE_SWEEP_CONFIRMED:           maximum_bars = 24;  break;
      case STATE_CHOCH_CONFIRMED:           maximum_bars = 10;  break;
      case STATE_DISPLACEMENT_CONFIRMED:    maximum_bars = 15;  break;
      case STATE_BOS_CONFIRMED:             maximum_bars = 5;   break;
      case STATE_WAITING_FOR_RETRACE:       maximum_bars = 10;  break;
      case STATE_ENTRY_CONFIRMATION:        maximum_bars = 1;   break;
      default: return false;
   }

   return BarsElapsed(PERIOD_M5, started) > maximum_bars;
}

Live Automatic Position Sizing

This calculation intentionally reads every broker and account value again immediately before order submission. It rounds volume down and rejects unsafe minimum-lot conditions.

RiskResult CalculatePositionRisk(const TradeDirection direction,
                                 const double entry,
                                 const double stop,
                                 const double target)
{
   RiskResult result;
   ZeroMemory(result);
   result.approved = false;

   result.equity       = AccountInfoDouble(ACCOUNT_EQUITY);
   result.free_margin  = AccountInfoDouble(ACCOUNT_MARGIN_FREE);
   result.risk_percent = ActiveRiskPercent();
   result.risk_capital = result.equity * result.risk_percent / 100.0;
   result.tick_size    = SymbolInfoDouble(InpSymbol, SYMBOL_TRADE_TICK_SIZE);
   result.tick_value   = SymbolInfoDouble(InpSymbol, SYMBOL_TRADE_TICK_VALUE);

   double volume_min  = SymbolInfoDouble(InpSymbol, SYMBOL_VOLUME_MIN);
   double volume_max  = SymbolInfoDouble(InpSymbol, SYMBOL_VOLUME_MAX);
   double volume_step = SymbolInfoDouble(InpSymbol, SYMBOL_VOLUME_STEP);

   if(result.equity <= 0.0 || result.free_margin < 0.0 ||
      result.tick_size <= 0.0 || result.tick_value <= 0.0 ||
      volume_min <= 0.0 || volume_max <= 0.0 || volume_step <= 0.0)
   {
      result.reason = "INVALID_ACCOUNT_OR_SYMBOL_SPECIFICATION";
      return result;
   }

   double stop_distance = MathAbs(entry - stop);
   double reward_distance = MathAbs(target - entry);
   if(stop_distance <= 0.0 || reward_distance <= 0.0)
   {
      result.reason = "INVALID_ORDER_GEOMETRY";
      return result;
   }

   result.realized_rr = reward_distance / stop_distance;
   if(result.realized_rr < InpMinimumRR)
   {
      result.reason = "RR_BELOW_MINIMUM";
      return result;
   }

   result.stop_ticks = stop_distance / result.tick_size;
   double loss_per_lot = result.stop_ticks * result.tick_value;
   if(loss_per_lot <= 0.0)
   {
      result.reason = "INVALID_LOSS_PER_LOT";
      return result;
   }

   result.raw_lots = result.risk_capital / loss_per_lot;
   double rounded_lots = MathFloor(result.raw_lots / volume_step) * volume_step;

   if(rounded_lots < volume_min)
   {
      double min_lot_loss = loss_per_lot * volume_min;
      if(min_lot_loss > result.risk_capital * InpMinLotRiskTolerance)
      {
         result.reason = "MIN_LOT_EXCEEDS_RISK_TOLERANCE";
         return result;
      }
      rounded_lots = volume_min;
   }

   double open_volume = CurrentStrategyXAUVolume();
   double remaining_exposure = MathMax(0.0, InpMaxTotalXAUExposure - open_volume);

   result.final_lots = MathMin(rounded_lots, volume_max);
   result.final_lots = MathMin(result.final_lots, InpMaxLotsPerOrder);
   result.final_lots = MathMin(result.final_lots, remaining_exposure);
   result.final_lots = MathFloor(result.final_lots / volume_step) * volume_step;

   if(result.final_lots < volume_min)
   {
      result.reason = "EXPOSURE_CAP_REJECTED_VOLUME";
      return result;
   }

   ENUM_ORDER_TYPE order_type = direction == DIRECTION_BUY
                              ? ORDER_TYPE_BUY : ORDER_TYPE_SELL;

   if(!OrderCalcMargin(order_type, InpSymbol, result.final_lots,
                       entry, result.margin_required))
   {
      result.reason = "MARGIN_CALCULATION_FAILED";
      return result;
   }

   if(result.margin_required > result.free_margin * InpMarginSafetyFactor)
   {
      result.reason = "MARGIN_INSUFFICIENT";
      return result;
   }

   result.approved = true;
   result.reason = "APPROVED";
   return result;
}

double ActiveRiskPercent()
{
   double risk = InpBaseRiskPercent;
   if(DailyLossCount == 1)
      risk = InpRiskAfterOneLoss;
   else if(DailyLossCount >= 2)
      risk = InpRiskAfterTwoLosses;

   if(IsWithinSixHoursOfFridayClose())
      risk *= 0.50;

   return risk;
}

Protected Order Submission

The final gate repeats critical checks because market and account values may change between signal confirmation and execution.

void ExecuteValidatedSetup()
{
   if(EmergencyStop || HasStrategyPosition() || !TerminalAndAccountHealthy())
   {
      ResetSetup("FINAL_SAFETY_GATE_FAILED");
      return;
   }

   MqlTick tick;
   if(!SymbolInfoTick(InpSymbol, tick))
   {
      ResetSetup("TICK_UNAVAILABLE");
      return;
   }

   double entry = CurrentSetup.direction == DIRECTION_BUY ? tick.ask : tick.bid;
   double stop  = BuildStructuralStop(CurrentSetup, tick);
   double target = BuildMinimumTarget(CurrentSetup, entry, stop, InpMinimumRR);

   if(!DirectionGeometryValid(CurrentSetup.direction, entry, stop, target))
   {
      ResetSetup("INVALID_DIRECTION_GEOMETRY");
      return;
   }

   if(!BrokerStopDistanceValid(entry, stop))
   {
      ResetSetup("BROKER_STOP_DISTANCE_FAILED");
      return;
   }

   RiskResult risk = CalculatePositionRisk(CurrentSetup.direction,
                                           entry, stop, target);
   LogRiskCalculation(CurrentSetup, risk, entry, stop, target);

   if(!risk.approved)
   {
      ResetSetup(risk.reason);
      return;
   }

   CurrentSetup.entry_price = entry;
   CurrentSetup.stop_price = stop;
   CurrentSetup.target_price = target;
   SetState(STATE_RISK_VALID, "RISK_VALID");

   string comment = "SYM|" + CurrentSetup.setup_id;
   bool sent = false;

   if(CurrentSetup.direction == DIRECTION_BUY)
      sent = Trade.Buy(risk.final_lots, InpSymbol, 0.0, stop, target, comment);
   else if(CurrentSetup.direction == DIRECTION_SELL)
      sent = Trade.Sell(risk.final_lots, InpSymbol, 0.0, stop, target, comment);

   if(!sent)
   {
      RegisterOrderRejection(Trade.ResultRetcode(),
                             Trade.ResultRetcodeDescription());
      ResetSetup("ORDER_REJECTED");
      return;
   }

   SetState(STATE_ORDER_SENT, "ORDER_SENT");
   TradesToday++;
}

bool DirectionGeometryValid(const TradeDirection direction,
                            const double entry,
                            const double stop,
                            const double target)
{
   if(direction == DIRECTION_BUY)
      return stop < entry && target > entry;
   if(direction == DIRECTION_SELL)
      return stop > entry && target < entry;
   return false;
}

bool BrokerStopDistanceValid(const double entry, const double stop)
{
   long stops_level = SymbolInfoInteger(InpSymbol, SYMBOL_TRADE_STOPS_LEVEL);
   double point = SymbolInfoDouble(InpSymbol, SYMBOL_POINT);
   if(point <= 0.0)
      return false;

   return MathAbs(entry - stop) >= (double)stops_level * point;
}

Position and Exposure Reconciliation

bool HasStrategyPosition()
{
   for(int index = PositionsTotal() - 1; index >= 0; index--)
   {
      ulong ticket = PositionGetTicket(index);
      if(ticket == 0 || !PositionSelectByTicket(ticket))
         continue;

      string symbol = PositionGetString(POSITION_SYMBOL);
      ulong magic = (ulong)PositionGetInteger(POSITION_MAGIC);
      if(symbol == InpSymbol && magic == InpMagicNumber)
         return true;
   }
   return false;
}

double CurrentStrategyXAUVolume()
{
   double total = 0.0;
   for(int index = PositionsTotal() - 1; index >= 0; index--)
   {
      ulong ticket = PositionGetTicket(index);
      if(ticket == 0 || !PositionSelectByTicket(ticket))
         continue;

      if(PositionGetString(POSITION_SYMBOL) == InpSymbol &&
         (ulong)PositionGetInteger(POSITION_MAGIC) == InpMagicNumber)
      {
         total += PositionGetDouble(POSITION_VOLUME);
      }
   }
   return total;
}

void ReconcileLivePosition()
{
   bool live_position = HasStrategyPosition();

   if(live_position &&
      CurrentSetup.state != STATE_POSITION_MANAGEMENT &&
      CurrentSetup.state != STATE_ORDER_SENT &&
      CurrentSetup.state != STATE_ORDER_CONFIRMED)
   {
      EmergencyStop = true;
      Print("CRITICAL | ORPHAN_POSITION_DETECTED | new entries disabled");
   }
}

Functions That Must Be Completed

The following functions are deliberate integration points. They must not be replaced with unconditional true values in any test, demo, or production build that can submit orders.

bool ValidateHigherTimeframeDirection(TradeDirection &direction);
bool IdentifyQualifiedLiquidity(SetupContext &setup);
bool ConfirmLiquiditySweep(SetupContext &setup);
bool ConfirmStrictCHoCH(SetupContext &setup);
bool ConfirmDisplacement(SetupContext &setup);
bool ConfirmBreakOfStructure(SetupContext &setup);
bool InvalidatedBeforeBOS(const SetupContext &setup);
bool CreateConfluenceEntryZone(SetupContext &setup);
bool ConfirmM5EntryTrigger(SetupContext &setup);
bool SetupStructurallyInvalid(const SetupContext &setup);
bool AllPreTradeGatesPass(const SetupContext &setup);
bool NewsCalendarAvailableAndClear();
bool SessionAndVolatilityValid();
bool TerminalAndAccountHealthy();
bool CooldownComplete();
bool IsWithinSixHoursOfFridayClose();
int BarsElapsed(ENUM_TIMEFRAMES timeframe, datetime started);
string BuildSetupId();
double BuildStructuralStop(const SetupContext &setup, const MqlTick &tick);
double BuildMinimumTarget(const SetupContext &setup, double entry,
                          double stop, double minimum_rr);
void ManageOpenPosition();
void UpdateLossCircuitBreakers();
void RegisterOrderRejection(uint retcode, string description);
void LogDecision(string outcome, string reason);
void LogRiskCalculation(const SetupContext &setup, const RiskResult &risk,
                        double entry, double stop, double target);

Suggested Repository Layout

Symmetrix-Gold-SMC/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── src/
│   ├── SymmetrixGoldSMC.mq5
│   ├── Config.mqh
│   ├── MarketData.mqh
│   ├── Indicators.mqh
│   ├── StructureEngine.mqh
│   ├── LiquidityEngine.mqh
│   ├── ConfluenceEngine.mqh
│   ├── StrategyStateMachine.mqh
│   ├── RiskEngine.mqh
│   ├── ExecutionGateway.mqh
│   ├── PositionManager.mqh
│   ├── Reconciliation.mqh
│   └── AuditLogger.mqh
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── scenarios/
│   └── fixtures/
├── config/
│   ├── development.set
│   ├── demo.set
│   └── production.example.set
├── docs/
│   ├── STRATEGY_SPECIFICATION.md
│   ├── VALIDATION_PROTOCOL.md
│   ├── DEPLOYMENT_RUNBOOK.md
│   └── INCIDENT_RESPONSE.md
└── validation/
    └── README.md

Compilation and Testing

Compile the primary .mq5 file in MetaEditor and treat all warnings as release blockers unless formally reviewed. Run the Strategy Tester first with order submission and structural debug logging enabled. A production build must not be released until every incomplete integration function is implemented, unit-tested, and validated against known chart examples.

State Expiration Rules

State

Maximum duration

Liquidity identified, waiting for sweep

100 M5 candles

Sweep to CHoCH

24 M5 candles

CHoCH to displacement

10 M5 candles

Displacement to BOS

15 M5 candles

BOS to entry-zone creation

5 M5 candles

Retracement to entry confirmation

10 M5 candles

Confirmation to order submission

1 M5 candle

Expired states are reset and logged as STATE_TIMEOUT.

Installation

Python Reference Implementation

Requirements:

Windows with MetaTrader 5 installed

A configured MT5 terminal logged into a broker account

Python 3.10 or later recommended

Install the required packages:

python -m pip install MetaTrader5 pandas numpy

The Python API communicates with the locally installed MT5 terminal; it does not connect directly to the broker.

Production MQL5 Build

The authoritative production target is an MQL5 Expert Advisor compiled and tested in MetaEditor. Broker-specific symbol naming, contract settings, session offsets, execution policies, and calendar integration must be verified before deployment.

Configuration

Review all defaults before running the system. Important configuration groups include:

symbol and magic number;

timeframe mapping;

EMA, ATR, ADX, swing, and structure thresholds;

liquidity and state-expiration rules;

session and news windows;

spread and slippage controls;

risk tiers, exposure ceilings, and margin buffer;

daily, weekly, and monthly loss limits;

weekend controls; and

log and journal locations.

Parameter changes must be treated as a new strategy version and revalidated. Optimization against the same historical period used for acceptance testing creates overfitting risk.

Logging and Auditability

Every accepted and rejected setup should produce an auditable record containing, at minimum:

setup ID and timestamps;

higher-timeframe direction;

selected liquidity level and sweep data;

CHoCH, displacement, BOS, FVG, OB, Fibonacci, and confirmation status;

score and rejection reason;

current equity and active risk tier;

tick size, tick value, and structural stop distance;

raw and normalized volume;

every exposure-cap input;

required and available margin;

spread, intended entry, SL, and targets; and

order, fill, partial-close, and exit results.

This information makes position sizing and rule enforcement independently reviewable rather than relying on a black-box claim.

Testing and Validation

Test Layers

Layer

Required coverage

Unit tests

Indicators, swing confirmation, liquidity ranking, CHoCH, BOS, FVG, OB, confirmation patterns, sizing, normalization, and circuit breakers

Property tests

Volume never exceeds calculated risk; invalid inputs never create an order; timeouts always terminate stale setups

Integration tests

MT5 data retrieval, symbol metadata, margin calculation, order requests, deal history, restart recovery, and log persistence

Scenario tests

Gaps, spread spikes, rejected orders, partial fills, minimum lot conflicts, DST changes, disconnects, manual intervention, and orphan positions

Backtest tests

No look-ahead, deterministic reruns, cost modeling, parameter freeze, and data-quality reporting

Forward tests

Demo execution, real broker timing, slippage, swap, news blocking, reconciliation, and operational monitoring

Validation Requirements

The system is not ready for production merely because it compiles or produces a profitable backtest. At minimum, validation should include:

Unit tests for indicators, structural events, sizing, and state transitions.

Historical backtesting with realistic spreads, commissions, swaps, and latency assumptions.

Out-of-sample testing.

Walk-forward analysis.

Monte Carlo and execution stress testing.

Broker-specific symbol and order-behavior testing.

Several weeks of demo-forward observation.

Controlled live testing at minimal permitted exposure.

Profitability metrics should not be considered meaningful until the combined out-of-sample and walk-forward sample contains at least 100 trades.

Acceptance Metrics

Report at least:

net return;

maximum drawdown;

profit factor;

Sharpe ratio;

Sortino ratio;

average R multiple;

win rate;

worst losing streak;

average winner and loser;

spread, slippage, commission, and swap costs; and

monthly and single-trade profit concentration.

No single trade should represent more than 15% of out-of-sample net profit, and no single month should represent more than 30%.

The 75%–80% win-rate range is a research objective only. A lower win-rate model with stronger expectancy and robustness may be superior, while a high win-rate model with poor payoff asymmetry or unstable out-of-sample performance must be rejected.

Required Validation Record

Every release candidate should include a signed or otherwise controlled validation package containing:

source-code commit or release identifier;

compiled EA checksum;

complete parameter set;

broker, symbol, account currency, leverage, and execution model;

data source, timezone, quality assessment, and test period;

in-sample, out-of-sample, and walk-forward boundaries;

spread, slippage, commission, and swap assumptions;

complete metric report and monthly return table;

Monte Carlo methodology and results;

trade list and rejection-reason distribution;

known limitations and unresolved defects; and

reviewer decision: rejected, demo-only, controlled live, or production approved.

Production Deployment

Production deployment is a controlled release process, not a simple activation of the AutoTrading button.

Readiness Gates

The system may progress beyond demo only when:

all mandatory modules are complete;

the economic-calendar integration is operational and fail-safe behavior is verified;

all critical and high-severity defects are closed;

unit, integration, scenario, and recovery tests pass;

backtest and live/demo behavior are reconciled;

the required OOS and walk-forward sample is achieved;

broker-specific symbol, margin, fill, stop, and volume behavior is verified;

monitoring and emergency-stop procedures are tested; and

the strategy owner and technical reviewer formally approve the release.

Recommended Release Stages

Development: isolated logic tests with order submission disabled.

Historical validation: deterministic backtest, OOS, walk-forward, and stress analysis.

Demo forward: continuous broker-connected testing for several weeks.

Controlled live: minimum practical exposure with enhanced monitoring.

Limited production: predefined capital ceiling and formal review interval.

Production: approved release with rollback, monitoring, and periodic revalidation.

Any material parameter, broker, symbol specification, execution-policy, or code change returns the system to the appropriate validation stage.

Operating Runbook

Before each trading week, the operator should confirm terminal health, broker connection, symbol availability, account permissions, calendar-feed availability, free disk space, time synchronization, emergency controls, and valid configuration checksums.

During operation, monitor rejected orders, spread anomalies, state timeouts, data gaps, reconciliation mismatches, equity circuit breakers, orphan positions, and unexpected restart events.

After any incident, preserve logs and broker records, disable new entries, reconcile all positions and deals, identify the root cause, test the correction, and issue a new controlled release. Emergency stops must never be cleared merely to resume trading without understanding the triggering condition.

Current Implementation Status

The supplied Python code is a reference implementation and orchestration prototype, not a completed live-production release. Before live use, the following items require completion and independent verification:

integration with a reliable economic-calendar source;

complete trade-history and realized P/L reconciliation;

exhaustive broker error and retry handling;

production-grade persistence and recovery testing;

verification of session time and daylight-saving behavior;

test coverage for every hard gate and state transition;

backtest parity between the Python reference and final MQL5 build; and

security review of runtime configuration, logs, and deployment environment.

Until these requirements are completed, the system should remain restricted to development, backtesting, and demo accounts.

Security and Operational Guidance

Never commit broker credentials, terminal data paths, API secrets, or account numbers.

Store sensitive configuration outside source control.

Run the EA with a unique magic number and a dedicated broker account during validation.

Monitor terminal connectivity, order rejections, clock synchronization, disk availability, and log health.

Treat manual intervention as an auditable event and reconcile the live broker state afterward.

Activate live trading only through a documented release and rollback procedure.

Performance and Risk Disclosures

No guarantee of performance. SYMMETRIX GOLD SMC is engineered to apply a high-selectivity automated framework. Research and optimization may target a historically validated win-rate range approaching 75%–80%; this is not a promise, representation, or guarantee of future results.

Leveraged trading risk. XAUUSD and other leveraged instruments can produce substantial losses. Stop-loss orders, exposure controls, margin safeguards, and emergency protections are intended to reduce risk under normal conditions but cannot guarantee a maximum loss during price gaps, extreme volatility, execution failure, liquidity disruption, connectivity problems, or extraordinary events.

Backtesting limitations. Backtested and simulated performance is hypothetical and does not represent actual trading. Live results may differ because of spreads, commissions, swaps, liquidity, slippage, latency, rejected orders, outages, and market gaps.

Monte Carlo limitations. Monte Carlo analysis is a statistical stress-testing technique based on historical or assumed trade behavior. It is not a forecast or guarantee of future account values, returns, win rates, or drawdowns.

Not financial advice. This repository contains an engineering specification and software reference. It is not investment advice, a solicitation, or a recommendation to buy or sell any financial instrument.

Contributing

Contributions should preserve deterministic behavior and must include tests for all affected state transitions and risk gates. Proposed changes should document:

the problem being solved;

the exact rule change;

expected effects on risk and execution;

unit and integration test results; and

out-of-sample or walk-forward evidence where strategy behavior changes.

Avoid merging performance-driven parameter changes based solely on in-sample optimization.

License

No open-source license was included in the source material. Unless a license is added by the copyright owner, the code and documentation should be treated as all rights reserved. Add a LICENSE file before public distribution or third-party reuse.

Contact

Ophireum Multimedia Productions

Mobile: +63 917 999 8814

Email: dhenzecapital@gmail.com

Telegram: @Mkogexchange

WhatsApp: +63 995 715 1043
