# Betting Strategy Learnings

**Source:** Today's session (accumulator-betting + value-edge-betting skills)  
**Extracted:** 2026-01-26

---

## Key Learnings

### 1. Two Complementary Betting Strategies

#### Accumulator Betting (High-Fold)
- 10-12 fold accumulators
- Minimum stake, high-upside betting
- Focus on slip completion, not profitability
- Max 2 selections per league
- Tier 1 markets: Double Chance, DNB, Over 1.5

#### Value Edge Betting (Single Bets)
- Positive EV only (edge > 3-5%)
- Flat stake 0.5-1% per bet
- Max 6 bets per matchday
- Each bet stands independently
- Long-term profit focus

### 2. Skill Dependencies
| Skill | Purpose |
|-------|---------|
| odds_harvester | Source of real odds data |
| market-mechanics-betting | Edge calculation, Kelly Criterion |
| accumulator-betting-strategy | Framework for accumulators |
| value-edge-betting-strategy | Framework for value betting |

### 3. Market Priority Hierarchy
| Tier | Markets | Use |
|------|---------|-----|
| T1 | DC, DNB, O1.5, U4.5 | Preferred |
| T2 | O2.5, Match Winner | Sparingly |
| T3 | Exact score, HT/FT, Props | Never |

### 4. Risk Management
- Never chase losses
- Never increase stakes
- Maximum 6 value bets per day
- 10-12 folds = high risk by design

---

## Today's Implementation

1. Created `accumulator-betting-strategy/` (pure documentation)
2. Created `value-edge-betting-strategy/` (pure documentation)
3. Created `accumulator-betting.py` (executable skill)
4. Scheduled crons at 15:00 daily

---

## Tags
#betting #strategy #accumulator #value #risk-management
