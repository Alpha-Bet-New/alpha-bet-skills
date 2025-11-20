# Alpha-Bet Development Roadmap

**Company:** Alpha-Bet  
**Mission:** Build algorithmic sports betting platform with systematic edge discovery and automated execution  
**Last Updated:** 2024-11-20

---

## Strategic Direction

### Core Philosophy
- **Data-Driven:** Every betting decision backed by quantitative analysis
- **Systematic:** Repeatable processes, not gut feelings
- **Scalable:** Build for multiple sports, markets, and strategies
- **Flexible:** Architecture supports rapid strategy iteration

### What We're Building
Algorithmic trading platform for sports betting that:
1. Ingests real-time odds from multiple providers
2. Applies quantitative models to identify edge
3. Executes trades automatically when criteria met
4. Tracks performance and continuously improves models

---

## Current Phase: MVP (Months 1-3)

### Goals
- Single sport (start with NBA or NFL)
- Single strategy (arbitrage or value betting)
- Manual execution with automated alerts
- Basic performance tracking

### Technical Stack (Finalized)
- **Backend:** Python (FastAPI)
- **Database:** PostgreSQL + Redis cache
- **Data Sources:** TBD (researching APIs)
- **Deployment:** Docker + AWS/GCP

### MVP Features
1. **Data Pipeline**
   - Ingest odds from 2-3 providers
   - Store historical odds data
   - Real-time odds updates (WebSocket or polling)

2. **Strategy Engine**
   - Arbitrage detector (cross-book opportunities)
   - Value betting detector (model vs market odds)
   - Configurable thresholds and filters

3. **Alerting System**
   - SMS/email alerts for opportunities
   - Dashboard showing current opportunities
   - Manual execution tracking

4. **Performance Tracking**
   - Track all bets placed
   - ROI calculation
   - Bankroll management

---

## Phase 2: Automation (Months 4-6)

### Goals
- Automated execution for approved strategies
- Expand to 2-3 sports
- Add 2-3 more strategies
- Improve models with historical data

### New Capabilities
1. **Auto-Execution**
   - Direct API integration with sportsbooks
   - Automated bet placement within risk limits
   - Position sizing algorithms

2. **Multi-Sport Support**
   - Unified data model for all sports
   - Sport-specific strategy parameters
   - Cross-sport arbitrage detection

3. **Advanced Strategies**
   - Line movement tracking
   - Steam chasing
   - Closing line value (CLV) optimization
   - Live betting models

4. **Risk Management**
   - Per-strategy bankroll allocation
   - Exposure limits
   - Correlation tracking (avoid correlated bets)

---

## Phase 3: Scale & Optimization (Months 7-12)

### Goals
- 5+ sports covered
- 10+ strategies running
- Sub-second execution latency
- Machine learning model improvements

### Advanced Features
1. **ML Model Pipeline**
   - Automated feature engineering
   - Model training and backtesting
   - A/B testing framework for strategies

2. **Low-Latency Execution**
   - Optimize for speed (sub-100ms decision to execution)
   - Edge cases: arb opportunities disappear fast
   - WebSocket connections to all books

3. **Portfolio Optimization**
   - Kelly criterion position sizing
   - Correlation-adjusted exposure
   - Dynamic bankroll allocation

4. **Market Making**
   - Provide liquidity on betting exchanges
   - Earn spread while managing inventory

---

## Architectural Principles

### ✅ DO: Build for Flexibility

**Multi-Strategy Architecture:**
```python
# GOOD: Strategy interface allows easy addition
class Strategy(ABC):
    @abstractmethod
    def evaluate(self, odds_data: OddsSnapshot) -> Optional[BettingOpportunity]:
        pass

# Add new strategies without changing core system
class ArbitrageStrategy(Strategy): ...
class ValueBettingStrategy(Strategy): ...
class SteamChaseStrategy(Strategy): ...
```

**Multi-Sport Data Model:**
```python
# GOOD: Generic event structure
class SportEvent:
    event_id: str
    sport: SportType  # Enum: NBA, NFL, MLB, etc.
    participants: List[Participant]
    markets: List[Market]  # Moneyline, spread, totals, props
    start_time: datetime

# NOT sport-specific tables like "nba_games", "nfl_games"
```

**Multi-Provider Odds:**
```python
# GOOD: Provider-agnostic odds format
class OddsQuote:
    provider: str  # "draftkings", "fanduel", etc.
    market_type: MarketType
    selection: str
    odds: Decimal
    timestamp: datetime

# Normalize all provider formats to this structure
```

### ❌ DON'T: Hardcode Assumptions

**Avoid Single-Provider Lock-In:**
```python
# BAD: Hardcoded to one provider
def get_odds():
    return draftkings_api.fetch_odds()

# GOOD: Provider abstraction
def get_odds(provider: OddsProvider):
    return provider.fetch_odds()
```

**Avoid Single-Sport Assumptions:**
```python
# BAD: NBA-specific field names
class Game:
    home_team_score: int
    away_team_score: int

# GOOD: Generic structure
class Event:
    participants: List[Participant]
    results: Dict[str, Any]  # Flexible for different sports
```

**Avoid Single-Strategy Logic:**
```python
# BAD: Arbitrage logic in main loop
if odds_diff > threshold:
    place_bets()

# GOOD: Strategy pattern
opportunities = [s.evaluate(odds) for s in active_strategies]
for opp in opportunities:
    if opp.meets_criteria():
        execute(opp)
```

### Configuration Over Code

**Strategy Parameters:**
```yaml
# config/strategies.yaml
arbitrage:
  enabled: true
  min_profit_pct: 2.0
  max_stake: 500
  providers: ["draftkings", "fanduel", "betmgm"]

value_betting:
  enabled: false  # Not ready yet
  edge_threshold: 5.0
  model_path: "models/nba_v1.pkl"
```

**Sport-Specific Rules:**
```yaml
# config/sports.yaml
nba:
  enabled: true
  markets: ["moneyline", "spread", "total"]
  min_odds: 1.5
  max_odds: 5.0

nfl:
  enabled: false  # Coming in Phase 2
```

---

## Data Architecture

### Core Entities

**Events** (Games/Matches)
- Sport-agnostic structure
- Participant info (teams/players)
- Start time, status
- Results after completion

**Odds Quotes**
- Provider, market, selection, odds
- Timestamp (critical for latency)
- Historical storage for analysis

**Bets Placed**
- Strategy used
- Event, market, selection
- Stake, odds, expected value
- Result, profit/loss

**Strategies**
- Name, type, parameters
- Enabled/disabled status
- Performance metrics

### Database Design Principles

1. **Time-Series Optimized**
   - Odds data is time-series (millions of records)
   - Consider TimescaleDB extension for PostgreSQL
   - Partition by date for performance

2. **Denormalization for Speed**
   - Cache current odds in Redis
   - Materialize views for common queries
   - Trade storage for query speed

3. **Audit Trail**
   - Never delete data (soft deletes only)
   - Track all bet decisions (placed AND skipped)
   - Store model predictions for backtesting

---

## API Integration Strategy

### Sportsbook APIs (Critical Path)

**Priority Order:**
1. **DraftKings** - Good API, high liquidity
2. **FanDuel** - Large market share
3. **BetMGM** - Good for arbitrage
4. **Caesars** - Additional coverage

**Integration Approach:**
- Start with odds ingestion (read-only)
- Add execution (write) in Phase 2
- Build adapter pattern for each provider
- Mock APIs for development/testing

### Data Provider APIs (Nice-to-Have)

**For Model Building:**
- Historical game data (scores, stats)
- Player injury reports
- Weather data (for outdoor sports)
- Line movement history

**Start Simple:**
- Free/cheap data sources first
- Avoid expensive data subscriptions early
- Focus on betting edge, not data hoarding

---

## Technology Decisions

### Why Python?
- Fast development for strategy iteration
- Rich ML/data science ecosystem
- Good enough performance for MVP
- Consider Rust/Go for hot paths later

### Why PostgreSQL?
- Reliable, battle-tested
- Good time-series support with extensions
- ACID guarantees for financial data
- Easy to query for analysis

### Why FastAPI?
- Modern, fast Python web framework
- Auto-generated API docs
- Async support for WebSockets
- Type hints = better code quality

### Why Docker?
- Consistent dev/prod environments
- Easy deployment
- Portable across cloud providers

---

## Risk Management Principles

### Bankroll Management
- Never risk more than 2% on single bet
- Per-strategy allocation limits
- Daily loss limits (circuit breakers)

### Position Limits
- Max exposure per event
- Max exposure per sport
- Correlation tracking (avoid related bets)

### Strategy Validation
- Extensive backtesting required
- Paper trading before live
- Gradual rollout with small stakes

---

## MVP Development Priorities

### Must-Have (Blocking Launch)
1. Odds ingestion from 2+ providers
2. Arbitrage detection algorithm
3. Alert system (SMS/email)
4. Manual bet tracking
5. Basic P&L dashboard

### Should-Have (Launched Soon After)
1. Historical odds storage
2. Backtest framework
3. Value betting strategy
4. WebSocket real-time updates

### Nice-to-Have (Post-MVP)
1. Mobile app
2. Advanced visualizations
3. Multiple user accounts
4. API for external strategies

---

## Success Metrics

### Technical KPIs
- **Latency:** Odds update to decision < 1 second
- **Uptime:** 99.9% availability
- **Coverage:** 90% of major games covered
- **Accuracy:** Odds data 99.99% accurate

### Business KPIs
- **ROI:** Positive returns within 3 months
- **Sharpe Ratio:** > 1.0 (risk-adjusted returns)
- **Win Rate:** > 53% (breakeven ~52.4%)
- **Closing Line Value:** Positive CLV on average

---

## Open Questions

### Technical Decisions Needed
- [ ] Which sportsbook APIs to prioritize?
- [ ] WebSocket vs. polling for odds updates?
- [ ] TimescaleDB vs. standard PostgreSQL?
- [ ] Self-host vs. managed database?

### Strategy Decisions Needed
- [ ] Starting sport (NBA, NFL, MLB)?
- [ ] Arbitrage vs. value betting first?
- [ ] What's our minimum acceptable edge?
- [ ] How much capital to start with?

### Infrastructure Decisions Needed
- [ ] AWS vs. GCP vs. Azure?
- [ ] Monitoring/alerting stack?
- [ ] CI/CD pipeline tools?
- [ ] Cost budget for MVP?

---

## How to Use This Roadmap

**For AI Coding Assistants:**
1. Check roadmap before making architectural decisions
2. Prefer flexibility over optimization early
3. Don't hardcode single-provider/sport/strategy assumptions
4. Use configuration files for parameters
5. Build abstractions that support future phases

**For Development:**
1. Focus on MVP features first
2. Build with Phase 2/3 in mind (avoid rework)
3. Document decisions that differ from roadmap
4. Update roadmap when priorities change

**For Planning:**
1. Monthly roadmap reviews
2. Adjust timelines based on learnings
3. Re-prioritize based on market opportunities
4. Keep "What We're NOT Building" section updated

---

## What We're NOT Building (Yet)

- ❌ Social features (sharing bets, leaderboards)
- ❌ User-facing betting platform (we're B2C, not B2B)
- ❌ Fantasy sports integration
- ❌ Cryptocurrency betting
- ❌ International markets (US-only for now)
- ❌ White-label solutions for other companies

---

## Version History

- **v1.0 (2024-11-20):** Initial roadmap created
