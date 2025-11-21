# Alpha-Bet API Strategy

## Purpose
Defines the strategy for integrating with sportsbook APIs and data providers, including provider priorities, integration patterns, error handling, and testing approaches.

## When to Use
- **Selecting providers to integrate** - Check priority rankings and pros/cons
- **Implementing provider adapters** - Reference design patterns and code examples
- **Handling API errors** - Consult error handling strategies
- **Rate limit issues** - See optimization and caching approaches
- **Testing integrations** - Use testing patterns and mock implementations

## Called By
- Developers implementing sportsbook integrations
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - References adapter patterns from here
- [docs/DECISIONS.md](DECISIONS.md) - Links to provider selection rationale
- [docs/ROADMAP.md](ROADMAP.md) - References provider integration timeline
- AI coding assistants implementing APIs

## Calls
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - Implements architectural patterns
- [docs/DECISIONS.md](DECISIONS.md) - Documents provider choices
- External sportsbook API documentation

---

Sportsbook and data provider integration strategy.

---

## Table of Contents

- [Sportsbook API Priorities](#sportsbook-api-priorities)
- [Integration Approach](#integration-approach)
- [Data Provider APIs](#data-provider-apis)
- [API Design Patterns](#api-design-patterns)
- [Rate Limiting & Optimization](#rate-limiting--optimization)
- [Error Handling](#error-handling)
- [Testing Strategy](#testing-strategy)

---

## Sportsbook API Priorities

### Phase 1: MVP (2-3 Providers)

**Primary Targets:**

#### 1. DraftKings
**Priority:** 🔴 Critical (Must-Have for MVP)

**Pros:**
- Well-documented public API
- High liquidity and market coverage
- Real-time odds updates
- Good rate limits

**Cons:**
- May require partnership for full access
- Potential geographic restrictions

**Integration Status:** 🟡 Research Phase

**API Endpoints Needed:**
- `GET /odds/{sport}` - Current odds for all games
- `GET /events/{eventId}` - Specific event details
- `GET /markets/{eventId}` - Available markets for event

---

#### 2. FanDuel
**Priority:** 🟡 High (Important for MVP)

**Pros:**
- Largest US market share
- Comprehensive coverage
- Good for arbitrage opportunities

**Cons:**
- Limited public API documentation
- May need unofficial/scraping approach
- Stricter terms of service

**Integration Status:** 🔴 Blocked (Need API access)

**Next Steps:**
- Research API partnership options
- Evaluate alternative data sources
- Consider web scraping as fallback (check ToS)

---

#### 3. BetMGM
**Priority:** 🟢 Medium (Nice-to-Have for MVP)

**Pros:**
- Good odds comparison target
- Decent market coverage
- API exists

**Cons:**
- Sparse documentation
- Unclear rate limits
- May require partnership

**Integration Status:** 🟡 Research Phase

---

### Phase 2: Expansion (4-6 Providers)

Additional providers to add after MVP validation:

4. **Caesars Sportsbook** - Additional coverage, good for arbitrage
5. **PointsBet** - Unique products, differentiated odds
6. **Barstool** - Growing market share

### Phase 3: Scale (7+ Providers)

- **Regional books** - State-specific operators
- **Sharp books** - Pinnacle, Bookmaker (if accessible)
- **Exchanges** - Betfair (if legal in US markets)

---

## Integration Approach

### Architecture Pattern: Provider Adapters

Each sportsbook gets its own adapter implementing a common interface.

```python
class OddsProvider(ABC):
    """Base interface for all sportsbook integrations."""

    @abstractmethod
    async def fetch_odds(self, sport: SportType, **filters) -> List[OddsQuote]:
        """Fetch current odds from provider."""
        pass

    @abstractmethod
    async def fetch_event(self, event_id: str) -> SportEvent:
        """Fetch specific event details."""
        pass

    @abstractmethod
    async def fetch_markets(self, event_id: str) -> List[Market]:
        """Fetch available markets for an event."""
        pass

    @abstractmethod
    async def place_bet(self, bet_order: BetOrder) -> BetConfirmation:
        """Place bet with provider (Phase 2+)."""
        pass
```

### Implementation Phases

**Phase 1: Read-Only (MVP)**
- Fetch odds data only
- No bet placement
- Focus on data normalization
- Build adapter pattern

**Phase 2: Write (Automation)**
- Add bet placement
- Implement authentication
- Handle order management
- Track execution

**Phase 3: Advanced (Scale)**
- WebSocket real-time feeds
- Bulk operations
- Advanced order types
- Position management

---

## Data Provider APIs

Non-betting APIs for model building and context.

### Historical Data

**Purpose:** Train models, backtest strategies

**Options:**
1. **The Odds API** - Historical odds data, affordable
2. **SportsDataIO** - Comprehensive sports data
3. **Custom scraping** - Build own historical database

**MVP Decision:** Start with The Odds API (cost-effective)

### Live Game Data

**Purpose:** In-game betting, context for odds changes

**Sources:**
- **ESPN API** - Free, good coverage, rate-limited
- **SportsRadar** - Professional-grade, expensive
- **Official league APIs** - NBA API, NFL API, etc.

**MVP Decision:** Use free APIs (ESPN), upgrade if needed

### Injury Reports

**Purpose:** Factor injuries into models

**Sources:**
- **Rotoworld** - Free injury news
- **FantasyData** - API for injury/status updates
- **Twitter feeds** - Real-time injury news

**MVP Decision:** Manual monitoring initially, automate in Phase 2

### Weather Data

**Purpose:** Outdoor sports modeling (NFL, MLB)

**Sources:**
- **OpenWeather API** - Free tier available
- **Weather.gov** - Free US weather data
- **DarkSky** - Paid, very accurate (acquired by Apple)

**MVP Decision:** Not needed for NBA start, defer

---

## API Design Patterns

### Adapter Pattern (Provider Abstraction)

```python
class DraftKingsAdapter(OddsProvider):
    """Adapter for DraftKings API."""

    def __init__(self, api_key: str, base_url: str = "https://api.draftkings.com"):
        self.api_key = api_key
        self.base_url = base_url
        self.session = aiohttp.ClientSession()

    async def fetch_odds(self, sport: SportType, **filters) -> List[OddsQuote]:
        """Fetch odds and normalize to our format."""
        # Call DraftKings API
        raw_data = await self._make_request(f"/odds/{sport.value}", filters)

        # Normalize to OddsQuote format
        return [self._normalize_quote(quote) for quote in raw_data["quotes"]]

    def _normalize_quote(self, raw: dict) -> OddsQuote:
        """Convert DraftKings format to our format."""
        return OddsQuote(
            quote_id=raw["id"],
            event_id=self._normalize_event_id(raw["eventId"]),
            provider="draftkings",
            market_type=self._map_market_type(raw["marketType"]),
            selection=raw["selection"],
            odds=self._convert_odds_format(raw["odds"]),
            timestamp=datetime.fromisoformat(raw["timestamp"]),
            metadata=raw.get("extra", {})
        )

    def _map_market_type(self, dk_market: str) -> MarketType:
        """Map DraftKings market names to our enum."""
        mapping = {
            "moneyline": MarketType.MONEYLINE,
            "spread": MarketType.SPREAD,
            "total": MarketType.TOTAL,
            # ... more mappings
        }
        return mapping.get(dk_market.lower(), MarketType.OTHER)
```

### Circuit Breaker Pattern (Failure Handling)

```python
class CircuitBreaker:
    """Prevent cascading failures from flaky APIs."""

    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN

    async def call(self, func, *args, **kwargs):
        """Execute function with circuit breaker protection."""
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise CircuitBreakerOpen("Circuit breaker is OPEN")

        try:
            result = await func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        """Reset failure count on success."""
        self.failure_count = 0
        self.state = "CLOSED"

    def on_failure(self):
        """Increment failure count, open circuit if threshold met."""
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"
```

### Retry Pattern (Transient Failures)

```python
class RetryPolicy:
    """Handle transient API failures with exponential backoff."""

    def __init__(self, max_attempts: int = 3, base_delay: float = 1.0):
        self.max_attempts = max_attempts
        self.base_delay = base_delay

    async def execute(self, func, *args, **kwargs):
        """Execute function with retry logic."""
        for attempt in range(self.max_attempts):
            try:
                return await func(*args, **kwargs)
            except (aiohttp.ClientError, asyncio.TimeoutError) as e:
                if attempt == self.max_attempts - 1:
                    raise
                delay = self.base_delay * (2 ** attempt)  # Exponential backoff
                await asyncio.sleep(delay)
```

---

## Rate Limiting & Optimization

### Rate Limit Tracking

```python
class RateLimiter:
    """Track and enforce rate limits per provider."""

    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = deque()

    async def acquire(self):
        """Wait if necessary to stay within rate limit."""
        now = time.time()

        # Remove old requests outside window
        while self.requests and self.requests[0] < now - self.window_seconds:
            self.requests.popleft()

        # Wait if at limit
        if len(self.requests) >= self.max_requests:
            sleep_time = self.window_seconds - (now - self.requests[0])
            await asyncio.sleep(sleep_time)

        self.requests.append(now)
```

### Caching Strategy

**What to Cache:**
- Current odds (Redis, 1-5 second TTL)
- Event metadata (Redis, 1 minute TTL)
- Historical odds (PostgreSQL, permanent)

**Cache Invalidation:**
- Time-based (TTL)
- Event-based (game start, score change)
- Manual (admin override)

### Batch Operations

```python
async def fetch_odds_batch(providers: List[OddsProvider], sport: SportType):
    """Fetch odds from multiple providers in parallel."""
    tasks = [provider.fetch_odds(sport) for provider in providers]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Handle failures gracefully
    odds = []
    for provider, result in zip(providers, results):
        if isinstance(result, Exception):
            logger.error(f"Failed to fetch from {provider.name}: {result}")
        else:
            odds.extend(result)

    return odds
```

---

## Error Handling

### Error Categories

**1. Transient Errors (Retry)**
- Network timeouts
- Rate limit exceeded (429)
- Server errors (5xx)

**2. Client Errors (Don't Retry)**
- Authentication failed (401)
- Invalid request (400)
- Not found (404)

**3. Circuit Breaker Errors (Stop Trying)**
- Repeated failures
- Provider down
- API deprecated

### Error Response Format

```python
class APIError(Exception):
    """Base class for API errors."""
    def __init__(self, provider: str, status_code: int, message: str):
        self.provider = provider
        self.status_code = status_code
        self.message = message
        super().__init__(f"{provider} API error ({status_code}): {message}")

class RateLimitError(APIError):
    """Rate limit exceeded."""
    pass

class AuthenticationError(APIError):
    """Authentication failed."""
    pass

class ProviderUnavailableError(APIError):
    """Provider is down or unreachable."""
    pass
```

---

## Testing Strategy

### Mock Providers

```python
class MockOddsProvider(OddsProvider):
    """Mock provider for testing without real API calls."""

    def __init__(self, mock_data: dict):
        self.mock_data = mock_data

    async def fetch_odds(self, sport: SportType, **filters) -> List[OddsQuote]:
        """Return mock odds data."""
        return self.mock_data.get(sport, [])

    async def fetch_event(self, event_id: str) -> SportEvent:
        """Return mock event data."""
        return self.mock_data.get("events", {}).get(event_id)
```

### Integration Tests

```python
@pytest.mark.asyncio
async def test_draftkings_adapter():
    """Test DraftKings adapter with real API."""
    adapter = DraftKingsAdapter(api_key=TEST_API_KEY)

    # Fetch odds
    odds = await adapter.fetch_odds(SportType.NBA)

    # Validate response
    assert len(odds) > 0
    assert all(isinstance(q, OddsQuote) for q in odds)
    assert all(q.provider == "draftkings" for q in odds)

    # Validate normalization
    quote = odds[0]
    assert quote.event_id is not None
    assert quote.market_type in MarketType
    assert quote.odds > 0
```

### Contract Tests

Ensure our adapters correctly implement the interface:

```python
def test_provider_contract(provider: OddsProvider):
    """Test that provider implements required interface."""
    # Check methods exist
    assert hasattr(provider, "fetch_odds")
    assert hasattr(provider, "fetch_event")
    assert hasattr(provider, "fetch_markets")

    # Check method signatures
    sig = inspect.signature(provider.fetch_odds)
    assert "sport" in sig.parameters
```

---

## API Documentation

### Provider-Specific Docs

Maintain separate docs for each provider:

- `docs/api/draftkings.md` - DraftKings integration guide
- `docs/api/fanduel.md` - FanDuel integration guide
- `docs/api/betmgm.md` - BetMGM integration guide

### API Playground

Build internal tool to test API calls:

```bash
# Fetch odds from DraftKings
python api_playground.py --provider draftkings --sport nba

# Compare odds across providers
python api_playground.py --compare --sport nba --market moneyline
```

---

## Security Considerations

### API Key Management

- Store keys in environment variables, never in code
- Use separate keys for dev/staging/prod
- Rotate keys regularly
- Implement key expiration alerts

### Request Signing

Some providers require signed requests:

```python
def sign_request(secret: str, payload: dict) -> str:
    """Sign request with HMAC-SHA256."""
    message = json.dumps(payload, sort_keys=True)
    signature = hmac.new(
        secret.encode(),
        message.encode(),
        hashlib.sha256
    ).hexdigest()
    return signature
```

### SSL/TLS

- Always use HTTPS for API calls
- Verify SSL certificates
- Pin certificates for critical providers

---

## Version History

- **v1.0 (2024-11-20):** Initial API strategy document created
