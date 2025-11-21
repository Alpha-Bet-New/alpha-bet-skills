# Alpha-Bet Architecture

## Purpose
Defines the technical design principles, system architecture, data models, and implementation patterns for the Alpha-Bet platform. This is the authoritative guide for all architectural decisions.

## When to Use
- **Before implementing new features** - Check architectural patterns to follow
- **Making technical decisions** - Consult principles and anti-patterns
- **Designing system components** - Reference architecture diagrams and data models
- **Code reviews** - Validate implementations follow architectural guidelines
- **Onboarding engineers** - Understand system design philosophy

## Called By
- Developers implementing features
- [README.md](../README.md) - Links to this for technical details
- [docs/ROADMAP.md](ROADMAP.md) - References architectural constraints
- [docs/API_STRATEGY.md](API_STRATEGY.md) - Implements patterns defined here
- AI coding assistants - Before generating code

## Calls
- [docs/API_STRATEGY.md](API_STRATEGY.md) - Provider integration patterns
- [docs/DECISIONS.md](DECISIONS.md) - Technology choice rationale

---

Technical design principles, patterns, and implementation guidelines.

---

## Table of Contents

- [Core Principles](#core-principles)
- [System Architecture](#system-architecture)
- [Data Architecture](#data-architecture)
- [Design Patterns](#design-patterns)
- [Technology Stack](#technology-stack)
- [Database Design](#database-design)
- [Risk Management](#risk-management)

---

## Core Principles

### ✅ Build for Flexibility

The platform must support multiple sports, strategies, and providers without major refactoring.

**Why?**
- Sports betting markets evolve rapidly
- New strategies emerge constantly
- Provider APIs change and new ones appear
- Early architectural decisions compound over time

### ❌ Avoid Premature Optimization

Start simple, measure, then optimize.

**Why?**
- Unknown where bottlenecks will be
- Early optimization adds complexity
- Speed matters less than correctness in MVP
- Can always optimize hot paths later

### 🔧 Configuration Over Code

Business logic parameters belong in config files, not hardcoded.

**Why?**
- Non-technical users can adjust parameters
- A/B testing strategies without deployments
- Environment-specific settings (dev/prod)
- Rapid iteration without code changes

---

## System Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────┐
│                    Alerting System                       │
│                  (SMS/Email/Dashboard)                   │
└─────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────┐
│                   Strategy Engine                        │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │
│  │  Arbitrage   │ │ Value Betting│ │  Steam Chase │   │
│  │   Strategy   │ │   Strategy   │ │   Strategy   │   │
│  └──────────────┘ └──────────────┘ └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────┐
│                   Data Pipeline                          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │
│  │  DraftKings  │ │   FanDuel    │ │    BetMGM    │   │
│  │   Adapter    │ │   Adapter    │ │   Adapter    │   │
│  └──────────────┘ └──────────────┘ └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                            ▲
                            │
                ┌───────────┴───────────┐
                │                       │
        ┌───────▼──────┐        ┌──────▼──────┐
        │  PostgreSQL  │        │    Redis    │
        │  (Historical)│        │   (Cache)   │
        └──────────────┘        └─────────────┘
```

### Data Flow

**1. Odds Ingestion**
```
Sportsbook API → Provider Adapter → Normalization → Redis Cache
                                                    ↓
                                            PostgreSQL (Historical)
```

**2. Strategy Evaluation**
```
Redis Cache → Strategy Engine → Opportunity Detection → Alert/Execute
```

**3. Bet Tracking**
```
Manual Entry → Bet Record → Performance Calculation → Dashboard
```

---

## Data Architecture

### Core Entities

#### Events (Games/Matches)

Sport-agnostic structure for all sporting events.

```python
class SportEvent:
    event_id: str              # Unique identifier
    sport: SportType           # Enum: NBA, NFL, MLB, etc.
    participants: List[Participant]  # Teams or players
    start_time: datetime
    status: EventStatus        # Scheduled, Live, Completed
    metadata: Dict[str, Any]   # Sport-specific data

class Participant:
    participant_id: str
    name: str
    role: str                  # "home", "away", "player1", etc.
```

**Design Rationale:**
- Generic structure works for team sports AND individual sports
- `metadata` field allows sport-specific extensions
- No hardcoded "home_team" or "away_team" fields
- Single table for all sports

#### Odds Quotes

Provider-agnostic odds format.

```python
class OddsQuote:
    quote_id: str
    event_id: str
    provider: str              # "draftkings", "fanduel", etc.
    market_type: MarketType    # Moneyline, Spread, Total, Prop
    selection: str             # What you're betting on
    odds: Decimal              # American, Decimal, or Fractional
    timestamp: datetime        # Critical for latency tracking
    metadata: Dict[str, Any]   # Provider-specific fields
```

**Design Rationale:**
- Time-series data (millions of records)
- Provider field enables cross-book arbitrage
- Timestamp enables line movement analysis
- All provider formats normalized to common structure

#### Bets Placed

Track all betting decisions (placed AND skipped).

```python
class Bet:
    bet_id: str
    event_id: str
    strategy_name: str
    market_type: MarketType
    selection: str
    stake: Decimal
    odds: Decimal
    expected_value: Decimal    # Model prediction
    placed_at: datetime
    provider: str
    result: Optional[BetResult]  # Win, Loss, Push
    profit_loss: Optional[Decimal]
    notes: str
```

**Design Rationale:**
- Audit trail for every bet
- Links back to strategy that generated it
- Stores expected value for model validation
- Result tracking for performance analysis

#### Strategies

Configuration and performance tracking for each strategy.

```python
class Strategy:
    strategy_id: str
    name: str
    type: StrategyType         # Arbitrage, Value, Steam, etc.
    enabled: bool
    parameters: Dict[str, Any]  # Strategy-specific config
    performance_metrics: Dict[str, Any]
    created_at: datetime
    updated_at: datetime
```

**Design Rationale:**
- Strategies as data, not just code
- Enable/disable without deployments
- Track performance per strategy
- Parameters stored as JSON for flexibility

---

## Design Patterns

### 1. Strategy Pattern (Multi-Strategy Support)

**Problem:** Need to support multiple betting strategies without coupling them to core system.

**Solution:**

```python
from abc import ABC, abstractmethod
from typing import Optional

class Strategy(ABC):
    """Base class for all betting strategies."""

    @abstractmethod
    def evaluate(self, odds_data: OddsSnapshot) -> Optional[BettingOpportunity]:
        """
        Evaluate current odds and return opportunity if found.

        Args:
            odds_data: Current odds snapshot from all providers

        Returns:
            BettingOpportunity if criteria met, None otherwise
        """
        pass

    @abstractmethod
    def validate_opportunity(self, opportunity: BettingOpportunity) -> bool:
        """Validate opportunity meets risk/exposure limits."""
        pass

# Example implementations
class ArbitrageStrategy(Strategy):
    def __init__(self, min_profit_pct: float = 2.0):
        self.min_profit_pct = min_profit_pct

    def evaluate(self, odds_data: OddsSnapshot) -> Optional[BettingOpportunity]:
        # Find cross-book arbitrage opportunities
        for event in odds_data.events:
            profit_pct = self._calculate_arb_profit(event.odds)
            if profit_pct >= self.min_profit_pct:
                return BettingOpportunity(
                    strategy="arbitrage",
                    event=event,
                    expected_profit=profit_pct
                )
        return None

class ValueBettingStrategy(Strategy):
    def __init__(self, model, edge_threshold: float = 5.0):
        self.model = model
        self.edge_threshold = edge_threshold

    def evaluate(self, odds_data: OddsSnapshot) -> Optional[BettingOpportunity]:
        # Compare model odds to market odds
        for event in odds_data.events:
            model_odds = self.model.predict(event)
            market_odds = event.best_odds
            edge = self._calculate_edge(model_odds, market_odds)
            if edge >= self.edge_threshold:
                return BettingOpportunity(
                    strategy="value_betting",
                    event=event,
                    expected_edge=edge
                )
        return None
```

**Benefits:**
- Add new strategies without modifying core system
- Each strategy encapsulates its own logic
- Easy to test strategies independently
- Can enable/disable strategies at runtime

### 2. Adapter Pattern (Multi-Provider Support)

**Problem:** Each sportsbook has different API format, authentication, and data structure.

**Solution:**

```python
class OddsProvider(ABC):
    """Base class for all sportsbook API integrations."""

    @abstractmethod
    def fetch_odds(self, sport: SportType) -> List[OddsQuote]:
        """Fetch current odds from provider."""
        pass

    @abstractmethod
    def place_bet(self, bet: BetOrder) -> BetConfirmation:
        """Place bet with provider."""
        pass

class DraftKingsAdapter(OddsProvider):
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.draftkings.com"

    def fetch_odds(self, sport: SportType) -> List[OddsQuote]:
        # DraftKings-specific API call
        response = requests.get(
            f"{self.base_url}/odds/{sport.value}",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )

        # Normalize to common format
        return [self._normalize_odds(quote) for quote in response.json()]

    def _normalize_odds(self, raw_data: dict) -> OddsQuote:
        # Convert DraftKings format to our format
        return OddsQuote(
            quote_id=raw_data["id"],
            event_id=raw_data["event_id"],
            provider="draftkings",
            market_type=self._map_market_type(raw_data["market"]),
            selection=raw_data["selection"],
            odds=Decimal(raw_data["odds"]),
            timestamp=datetime.fromisoformat(raw_data["timestamp"])
        )

class FanDuelAdapter(OddsProvider):
    # Similar implementation for FanDuel's API format
    pass
```

**Benefits:**
- Add new providers without changing core logic
- Mock providers for testing
- Switch providers easily
- Consistent interface regardless of underlying API

### 3. Repository Pattern (Data Access Layer)

**Problem:** Database queries scattered throughout codebase, hard to test and change.

**Solution:**

```python
class BetRepository:
    """Handle all bet-related database operations."""

    def __init__(self, db_session):
        self.db = db_session

    def save_bet(self, bet: Bet) -> None:
        """Persist bet to database."""
        self.db.execute(
            "INSERT INTO bets (bet_id, event_id, strategy_name, ...) VALUES (...)",
            bet.dict()
        )

    def get_bets_by_strategy(self, strategy_name: str) -> List[Bet]:
        """Fetch all bets for a specific strategy."""
        rows = self.db.query("SELECT * FROM bets WHERE strategy_name = ?", strategy_name)
        return [Bet(**row) for row in rows]

    def calculate_roi(self, strategy_name: str) -> Decimal:
        """Calculate ROI for a strategy."""
        result = self.db.query(
            "SELECT SUM(profit_loss) / SUM(stake) FROM bets WHERE strategy_name = ?",
            strategy_name
        )
        return Decimal(result[0][0])

class OddsRepository:
    """Handle all odds-related database operations."""
    # Similar pattern for odds data
    pass
```

**Benefits:**
- Centralized data access logic
- Easy to mock for testing
- Can optimize queries in one place
- Clear separation of concerns

---

## Technology Stack

### Why Python?

**Pros:**
- Fast development for strategy iteration
- Rich ML/data science ecosystem (pandas, numpy, scikit-learn)
- Good web frameworks (FastAPI)
- Easy to hire for

**Cons:**
- Slower than Go/Rust for hot paths
- GIL limits true parallelism

**Decision:** Use Python for MVP, consider Rust/Go for critical paths in Phase 2+

### Why PostgreSQL?

**Pros:**
- ACID guarantees (critical for financial data)
- Excellent time-series support (TimescaleDB extension)
- Rich query capabilities for analysis
- Battle-tested reliability

**Cons:**
- Not as fast as specialized time-series DBs
- Scaling requires care

**Decision:** Start with PostgreSQL, evaluate TimescaleDB extension if performance issues

### Why Redis?

**Pros:**
- Extremely fast for current odds lookup
- Built-in pub/sub for real-time updates
- Simple key-value model

**Cons:**
- In-memory only (expensive at scale)
- No complex queries

**Decision:** Use Redis for hot data (current odds), PostgreSQL for historical

### Why FastAPI?

**Pros:**
- Modern async Python framework
- Auto-generated API docs (OpenAPI)
- Type hints = better code quality
- WebSocket support for real-time updates

**Cons:**
- Smaller ecosystem than Flask/Django

**Decision:** FastAPI for API + WebSocket needs

### Why Docker?

**Pros:**
- Consistent dev/prod environments
- Easy local development
- Portable across cloud providers

**Decision:** Docker + docker-compose for all services

---

## Database Design

### Schema Overview

```sql
-- Events table (sport-agnostic)
CREATE TABLE events (
    event_id TEXT PRIMARY KEY,
    sport TEXT NOT NULL,
    start_time TIMESTAMP NOT NULL,
    status TEXT NOT NULL,
    participants JSONB NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_events_sport_start ON events(sport, start_time);
CREATE INDEX idx_events_status ON events(status);

-- Odds quotes (time-series data)
CREATE TABLE odds_quotes (
    quote_id TEXT PRIMARY KEY,
    event_id TEXT REFERENCES events(event_id),
    provider TEXT NOT NULL,
    market_type TEXT NOT NULL,
    selection TEXT NOT NULL,
    odds NUMERIC NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    metadata JSONB
);

CREATE INDEX idx_odds_event_provider ON odds_quotes(event_id, provider);
CREATE INDEX idx_odds_timestamp ON odds_quotes(timestamp DESC);

-- Bets placed
CREATE TABLE bets (
    bet_id TEXT PRIMARY KEY,
    event_id TEXT REFERENCES events(event_id),
    strategy_name TEXT NOT NULL,
    market_type TEXT NOT NULL,
    selection TEXT NOT NULL,
    stake NUMERIC NOT NULL,
    odds NUMERIC NOT NULL,
    expected_value NUMERIC,
    placed_at TIMESTAMP NOT NULL,
    provider TEXT NOT NULL,
    result TEXT,
    profit_loss NUMERIC,
    notes TEXT
);

CREATE INDEX idx_bets_strategy ON bets(strategy_name);
CREATE INDEX idx_bets_placed_at ON bets(placed_at DESC);

-- Strategies
CREATE TABLE strategies (
    strategy_id TEXT PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    type TEXT NOT NULL,
    enabled BOOLEAN DEFAULT true,
    parameters JSONB NOT NULL,
    performance_metrics JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Design Principles

**1. Time-Series Optimization**
- Partition `odds_quotes` by date for performance
- Consider TimescaleDB for automatic time-series optimization
- Index on timestamp for recent data queries

**2. JSONB for Flexibility**
- Use JSONB for sport-specific fields
- Allows schema evolution without migrations
- Can still query/index JSONB fields if needed

**3. Audit Trail**
- Never delete data (soft deletes only)
- Track creation/update timestamps
- Store all bet decisions (placed AND skipped)

**4. Denormalization for Speed**
- Cache current odds in Redis
- Materialize views for common aggregate queries
- Trade storage for query speed

---

## Risk Management

### Bankroll Management

```python
class BankrollManager:
    """Enforce bankroll limits across all strategies."""

    def __init__(self, total_bankroll: Decimal):
        self.total_bankroll = total_bankroll
        self.per_bet_limit = total_bankroll * Decimal("0.02")  # 2% max
        self.daily_loss_limit = total_bankroll * Decimal("0.10")  # 10% max

    def validate_bet(self, bet: BetOrder) -> bool:
        """Check if bet meets risk limits."""
        # Check per-bet limit
        if bet.stake > self.per_bet_limit:
            return False

        # Check daily loss limit
        today_loss = self._calculate_daily_loss()
        if today_loss >= self.daily_loss_limit:
            return False

        # Check exposure limits
        if self._get_exposure(bet.event_id) > self.exposure_limit:
            return False

        return True
```

### Position Limits

```python
class ExposureManager:
    """Track and limit exposure across events."""

    def __init__(self, max_event_exposure: Decimal, max_sport_exposure: Decimal):
        self.max_event_exposure = max_event_exposure
        self.max_sport_exposure = max_sport_exposure

    def get_event_exposure(self, event_id: str) -> Decimal:
        """Calculate total stake on this event."""
        return self.bet_repo.sum_stakes_by_event(event_id)

    def get_sport_exposure(self, sport: SportType) -> Decimal:
        """Calculate total stake on this sport."""
        return self.bet_repo.sum_stakes_by_sport(sport)

    def check_correlation(self, bets: List[Bet]) -> bool:
        """Check if bets are correlated (same game parlay, etc.)."""
        # Avoid betting on correlated outcomes
        pass
```

### Strategy Validation

```python
class StrategyValidator:
    """Validate strategies before going live."""

    def backtest(self, strategy: Strategy, historical_data: DataFrame) -> BacktestResult:
        """Run strategy against historical data."""
        pass

    def paper_trade(self, strategy: Strategy, duration: timedelta) -> PaperTradingResult:
        """Run strategy with fake money for testing."""
        pass

    def gradual_rollout(self, strategy: Strategy, initial_stake: Decimal):
        """Start with small stakes, increase as performance validates."""
        pass
```

---

## Anti-Patterns to Avoid

### ❌ Don't Hardcode Provider-Specific Logic

```python
# BAD
def get_odds():
    return draftkings_api.fetch_odds()

# GOOD
def get_odds(provider: OddsProvider):
    return provider.fetch_odds()
```

### ❌ Don't Use Sport-Specific Field Names

```python
# BAD
class Game:
    home_team_score: int  # What about individual sports?
    away_team_score: int

# GOOD
class Event:
    participants: List[Participant]
    results: Dict[str, Any]  # Flexible for any sport
```

### ❌ Don't Put Business Logic in API Routes

```python
# BAD
@app.get("/arbitrage")
def find_arbitrage():
    odds = fetch_odds()
    # 50 lines of arbitrage logic here...
    return opportunities

# GOOD
@app.get("/arbitrage")
def find_arbitrage():
    strategy = ArbitrageStrategy(config.min_profit_pct)
    return strategy.evaluate(odds_service.get_current_odds())
```

### ❌ Don't Skip Tests for "Simple" Code

```python
# Write tests for ALL strategy logic
def test_arbitrage_detection():
    strategy = ArbitrageStrategy(min_profit_pct=2.0)
    odds = create_test_odds(dk_odds=1.9, fd_odds=2.1)
    opportunity = strategy.evaluate(odds)
    assert opportunity.profit_pct >= 2.0
```

---

## Version History

- **v1.0 (2024-11-20):** Initial architecture document created