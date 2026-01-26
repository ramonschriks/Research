# Football Updates Skill Learnings

**Source:** Today's session (football-updates skill)  
**Extracted:** 2026-01-26

---

## Key Learnings

### 1. Stale News Problem
- Cron every 30 min caused rate limiting
- Fetched cached/stale tweets repeatedly
- Same news appearing multiple times

### 2. Solution: Smart Filtering
- Filter by 24-hour window
- Skip known completed transfers (Taylor, Moro, Vicente)
- Only show breaking news (JUST IN, DEVELOPING, etc.)
- Deduplicate results

### 3. Cron Frequency Adjustment
- Changed from every 30 min → every hour
- Balances freshness with rate limit avoidance

### 4. Data Sources
- Twitter/X via bird CLI
- 20 trusted accounts (Tier 1-3 + Must Include)
- Accounts: FabrizioRomano, MatteMoretto, Inside_020, etc.

---

## Implementation Details

```python
# Filter patterns for stale news
COMPLETED_TRANSFERS = {
    'kenneth taylor': 'Lazio',
    'raul moro': 'Osasuna',
    'carlos vicente': 'Birmingham City',
}

# Breaking news patterns
BREAKING_PATTERNS = ['JUST IN', 'BREAKING', 'DEVELOPING', 'IMMINENT']
```

---

## Tags
#football #twitter #automation #filtering
