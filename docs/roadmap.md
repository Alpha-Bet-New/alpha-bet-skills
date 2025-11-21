# Alpha-Bet Roadmap

## Purpose
Outlines the development timeline, feature priorities, and milestones for the Alpha-Bet platform across MVP, automation, and scale phases.

## When to Use
- **Planning sprints** - Determine what to build next
- **Checking priorities** - See Must-Have vs Should-Have vs Nice-to-Have features
- **Understanding timeline** - Know which phase introduces which capabilities
- **Setting expectations** - Communicate what's coming and when
- **Reviewing progress** - Track actual vs planned milestones

## Called By
- Product/project managers planning work
- [README.md](../README.md) - Links to current phase
- Developers checking feature priorities
- Stakeholders reviewing timeline
- AI agents determining implementation scope

## Calls
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - References technical constraints
- [docs/DECISIONS.md](DECISIONS.md) - Links to open questions blocking features
- [docs/API_STRATEGY.md](API_STRATEGY.md) - Provider integration timeline

---

Development phases, timeline, and priorities.

---

## Timeline Overview

```
Month 1-3:  MVP           ████████░░░░░░░░░░░░ (Data pipeline, arbitrage, alerts)
Month 4-6:  Automation    ░░░░░░░░████████░░░░ (Auto-execution, multi-sport)
Month 7-12: Scale         ░░░░░░░░░░░░████████ (ML models, optimization)
```

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

## Related Documents

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Technical design principles and patterns
- **[DECISIONS.md](DECISIONS.md)** - Decision log and open questions
- **[API_STRATEGY.md](API_STRATEGY.md)** - Sportsbook integration strategy

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

- **v2.0 (2024-11-20):** Restructured into focused phases document
- **v1.0 (2024-11-20):** Initial roadmap created
