# Alpha-Bet Decisions

## Purpose
Tracks all significant technical and strategic decisions, their rationale, and open questions requiring resolution. Serves as the single source of truth for "why did we decide that?"

## When to Use
- **Making a new decision** - Log it here with context and rationale
- **Questioning a past decision** - Look up why it was made
- **Finding open questions** - See what needs to be decided (organized by priority)
- **Onboarding** - Understand decision history and context
- **Planning** - Check blockers and pending decisions

## Called By
- Team members making technical/strategic choices
- [README.md](../README.md) - Links for decision transparency
- [docs/ROADMAP.md](ROADMAP.md) - References blocking questions
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - Documents tech stack decisions
- Sprint planning meetings

## Calls
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - Links to technical implications
- [docs/API_STRATEGY.md](API_STRATEGY.md) - Provider selection decisions
- [docs/ROADMAP.md](ROADMAP.md) - Feature priority decisions

---

Decision log and open questions for the Alpha-Bet project.

---

## Decision Log

Track all significant decisions with context and rationale.

| Date | Decision | Rationale | Status | Owner |
|------|----------|-----------|--------|-------|
| 2024-11-20 | Use Python/FastAPI for backend | Fast development, rich ML ecosystem, good enough performance for MVP | ✅ Final | Team |
| 2024-11-20 | Use PostgreSQL + Redis | ACID guarantees, time-series support, fast cache for current odds | ✅ Final | Team |
| 2024-11-20 | Start with manual execution | Lower risk, validate strategies before automating | ✅ Final | Team |
| 2024-11-20 | Focus on arbitrage first | Simpler than value betting, lower model risk | 🟡 Proposed | Team |
| 2024-11-20 | Containerize with Docker | Consistent environments, easy deployment | ✅ Final | Team |

### Status Legend
- ✅ **Final** - Decision is locked in, implementation underway
- 🟡 **Proposed** - Decision suggested, awaiting approval/validation
- 🔴 **Blocked** - Decision needed but blocked by external factor
- ⚪ **Deferred** - Decision postponed to later phase

---

## Open Questions

Questions organized by priority and timeline.

### 🔴 Blocking MVP (Need answers this week)

#### Which sportsbook APIs to prioritize?
**Context:** Need to integrate 2+ providers for MVP arbitrage detection.

**Options:**
1. DraftKings + FanDuel (largest market share)
2. DraftKings + BetMGM (good API docs)
3. FanDuel + Caesars (good coverage)

**Considerations:**
- API documentation quality
- Rate limits and costs
- Historical reliability
- Liquidity/market depth

**Decision needed by:** End of Week 1
**Owner:** TBD

---

#### Starting sport (NBA, NFL, MLB)?
**Context:** Need to pick one sport for MVP to focus development.

**Options:**
1. **NBA** - High volume of games, good API coverage, active betting season
2. **NFL** - Most popular for betting, but fewer games per week
3. **MLB** - Lots of games, but more complex modeling

**Considerations:**
- Current season timing
- API data availability
- Betting volume/liquidity
- Model complexity

**Decision needed by:** End of Week 1
**Owner:** TBD

---

#### What's our minimum acceptable edge?
**Context:** Need to set threshold for arbitrage opportunities (e.g., 1%, 2%, 3%).

**Tradeoffs:**
- Lower threshold = more opportunities, but higher execution risk
- Higher threshold = fewer opportunities, but better risk/reward

**Proposed:** Start with 2% minimum, adjust based on data

**Decision needed by:** Week 2
**Owner:** TBD

---

### 🟡 Important (Need answers this month)

#### WebSocket vs. polling for odds updates?
**Context:** How to get real-time odds data from sportsbooks.

**Options:**
1. **WebSocket** - Real-time updates, lower latency
2. **HTTP Polling** - Simpler implementation, more reliable
3. **Hybrid** - WebSocket with polling fallback

**Considerations:**
- Latency requirements (arbitrage needs speed)
- API support (not all providers offer WebSockets)
- Implementation complexity
- Cost (polling = more API calls)

**Proposed:** Start with polling (simpler), add WebSocket in Phase 2

**Decision needed by:** Week 3
**Owner:** TBD

---

#### How much capital to start with?
**Context:** Need bankroll for initial testing and validation.

**Considerations:**
- Start small ($1,000-$5,000) for validation
- Scale up as strategies prove profitable
- Need enough to place meaningful bets but not risk too much

**Decision needed by:** Before MVP launch
**Owner:** TBD

---

#### Arbitrage vs. value betting first?
**Context:** Which strategy to implement for MVP?

**Comparison:**
| Factor | Arbitrage | Value Betting |
|--------|-----------|---------------|
| Complexity | Lower (math-based) | Higher (requires model) |
| Edge | Lower (1-3%) | Higher (5%+) |
| Risk | Lower (guaranteed profit) | Higher (model risk) |
| Opportunities | Fewer | More |

**Proposed:** Arbitrage first (lower risk, faster to implement)

**Decision needed by:** Week 2
**Owner:** TBD

---

### 🟢 Nice to Know (Can decide later)

#### AWS vs. GCP vs. Azure?
**Context:** Cloud provider for hosting.

**Options:**
- **AWS** - Largest ecosystem, most mature
- **GCP** - Better pricing, good ML tools
- **Azure** - Good if using Microsoft stack

**Proposed:** AWS (most familiar, best docs)

**Decision needed by:** Week 4
**Owner:** TBD

---

#### TimescaleDB vs. standard PostgreSQL?
**Context:** Whether to use TimescaleDB extension for time-series odds data.

**Options:**
1. **Standard PostgreSQL** - Simpler, no extensions needed
2. **TimescaleDB** - Optimized for time-series, better performance at scale

**Proposed:** Start with standard PostgreSQL, migrate to TimescaleDB if performance issues

**Decision needed by:** Month 2
**Owner:** TBD

---

#### Self-host vs. managed database?
**Context:** Database hosting approach.

**Options:**
1. **Managed** (RDS, Cloud SQL) - Less ops, automatic backups, higher cost
2. **Self-hosted** (EC2, Docker) - More control, lower cost, more maintenance

**Proposed:** Managed for MVP (less ops burden), re-evaluate at scale

**Decision needed by:** Week 4
**Owner:** TBD

---

#### Monitoring/alerting stack?
**Context:** How to monitor system health and performance.

**Options:**
1. **Datadog** - Comprehensive, expensive
2. **Prometheus + Grafana** - Open source, more setup
3. **CloudWatch** - Native AWS, basic features

**Proposed:** CloudWatch for MVP (simple, cheap), upgrade if needed

**Decision needed by:** Month 2
**Owner:** TBD

---

#### CI/CD pipeline tools?
**Context:** Automated testing and deployment.

**Options:**
1. **GitHub Actions** - Native to GitHub, free tier
2. **Jenkins** - More powerful, more setup
3. **GitLab CI** - Good if using GitLab

**Proposed:** GitHub Actions (simple, integrated)

**Decision needed by:** Week 3
**Owner:** TBD

---

#### Cost budget for MVP?
**Context:** Monthly infrastructure and API costs.

**Estimated Costs:**
- Cloud hosting: $50-$200/month
- Database (managed): $50-$100/month
- Sportsbook API access: $0-$500/month (varies by provider)
- Twilio (SMS alerts): $10-$50/month
- Total: $110-$850/month

**Proposed:** $300/month budget for MVP

**Decision needed by:** Before MVP launch
**Owner:** TBD

---

## Decision-Making Process

### How to Make a Decision

1. **Research** - Gather information, evaluate options
2. **Propose** - Document options and recommendation
3. **Review** - Team discussion, consider tradeoffs
4. **Decide** - Make final call, document rationale
5. **Execute** - Implement decision, track results
6. **Review** - Revisit decision if circumstances change

### When to Revisit a Decision

- New information becomes available
- Requirements change
- Initial assumptions proven wrong
- Better alternatives emerge
- Cost/benefit analysis shifts

### Documentation Guidelines

When adding a decision to the log:
- Include date and clear decision statement
- Explain rationale (why this choice?)
- Document alternatives considered
- Note any dissenting opinions
- Assign owner/responsibility

---

## Assumptions Log

Track key assumptions that decisions are based on.

| Assumption | Impact if Wrong | Validation Plan |
|------------|----------------|-----------------|
| Sportsbook APIs will remain accessible | Critical - entire platform depends on it | Monitor API terms, build fallbacks |
| Arbitrage opportunities exist frequently enough | High - need volume for profitability | Validate with test data before full build |
| 2% arbitrage edge is achievable after fees | High - affects profitability | Calculate with real fee structures |
| Manual execution is fast enough for MVP | Medium - could miss opportunities | Test with real scenarios |
| Python performance is sufficient | Medium - could need rewrite | Benchmark critical paths |
| US market will remain legal | Critical - regulatory risk | Monitor legislation, have exit plan |

---

## Research Notes

Track research findings that inform decisions.

### Sportsbook API Research
**Date:** 2024-11-20
**Status:** In Progress

**Findings:**
- DraftKings: Good API docs, reasonable rate limits
- FanDuel: Limited public API, may need unofficial access
- BetMGM: API available but docs are sparse
- Caesars: API access requires partnership

**Next Steps:**
- [ ] Test DraftKings API with trial account
- [ ] Research FanDuel API alternatives
- [ ] Contact BetMGM for API documentation

---

### Arbitrage Frequency Analysis
**Date:** TBD
**Status:** Not Started

**Questions to Answer:**
- How many arbitrage opportunities occur per day?
- What's the average profit margin?
- How long do opportunities last?
- Which sports have the most arbs?

**Data Needed:**
- Historical odds from 2+ providers
- Time-series of odds changes

---

## Version History

- **v1.0 (2024-11-20):** Initial decisions document created