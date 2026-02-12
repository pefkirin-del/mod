# CULT Chess — Product & Technical Specification (MVP + Phase 2)

## 1) Product Vision
Build a competitive online chess platform with CULT-token staking, chess.com-like gameplay UX, configurable wager models, tournament pools, match timers, and transparent on-chain settlement.

---

## 2) Confirmed Business Rules

### 2.1 Token and stake constraints
- Currency: **CULT token** only.
- Minimum stake per player per match: **$100 equivalent in CULT**.
- Maximum stake per player per match: **$1,000,000 equivalent in CULT**.
- Fiat-equivalent checks are enforced at match creation/join time via an approved price oracle.

### 2.2 1v1 staking models

#### Mode A: "25% Opponent Capture"
- Both players stake a selected amount (can be equal or unequal if lobby permits).
- Winner receives:
  - 100% of their own stake back,
  - plus **25% of loser stake**.
- Loser receives back the remaining **75%** of loser stake.
- Platform fee and draw burn logic apply as defined in section 2.4 and 2.5.

#### Mode B: "All-In Winner Takes All"
- Winner receives the full distributable pot after fee/burn rules.
- Loser receives 0 from the distributable pot.

### 2.3 Tournament models
- Supported tournament sizes: **16-player** and **32-player** brackets.
- Tournament pool = sum of all participants' entry stakes.
- Default payout model: **winner takes final distributable tournament pool**.
- Optional configurable payout profile (phase 2): top-3 split.

### 2.4 Creator fee
- Creator wallet: `0xd49B8997AD2247A6b9059e3f7f0F95536c2Aa92f`.
- Fee = **2% of each player's stake**.
- Fee transfer timing: at stake lock/escrow funding time.

### 2.5 Draw burn rule
- On a draw, send **5% of total gross staked amount** to burn address:
  - `0x000000000000000000000000000000000000dead`.
- Remaining distributable amount (after fee and burn) is refunded to players pro-rata by their net contributed stake.

### 2.6 Identity privacy
- Username display modes:
  1. Alias/public handle,
  2. Wallet-obfuscated display showing only trailing 6 characters,
  3. Private mode (limited visibility in public contexts).

### 2.7 Time controls
- Timers modeled after mainstream online chess:
  - base time + increment presets (e.g. 3+2, 5+0, 10+5),
  - timeout loss,
  - disconnect grace policy,
  - reconnection handling.

---

## 3) Settlement Formulas

Let:
- `sA`, `sB` = gross stake by player A/B,
- `f = 0.02` creator fee rate,
- `d = 0.05` draw burn rate,
- `nA = sA * (1 - f)`, `nB = sB * (1 - f)` net stake after fee,
- `G = sA + sB` gross pot,
- `N = nA + nB` net pot after fees.

### 3.1 Decisive result — Mode A (25% capture)
If A wins:
- A payout = `nA + 0.25 * nB`
- B payout = `0.75 * nB`

If B wins:
- B payout = `nB + 0.25 * nA`
- A payout = `0.75 * nA`

### 3.2 Decisive result — Mode B (all-in)
If A wins:
- A payout = `N`
- B payout = `0`

If B wins:
- B payout = `N`
- A payout = `0`

### 3.3 Draw result (both modes)
- Burn amount = `d * G` (from gross basis per requirement).
- Remaining return pool = `N - d * G`.
- Pro-rata refunds:
  - A refund = `(nA / N) * (N - d * G)`
  - B refund = `(nB / N) * (N - d * G)`

> Note: Contract implementation should use integer math, token decimals, and deterministic rounding policy (round down to avoid overdrawing escrow; dust sent to treasury or burn by policy).

---

## 4) Game Rules and Chess Feature Parity Targets

### 4.1 Core chess rules
- Legal move generation and validation fully compliant with FIDE rules.
- Support:
  - castling,
  - en passant,
  - promotions,
  - check/checkmate/stalemate,
  - threefold repetition,
  - 50-move rule,
  - insufficient material.

### 4.2 UX parity goals ("almost exactly like chess.com")
- Drag/drop and click-to-move.
- Move highlights and legal destination hints.
- Move list, captured pieces, board flip.
- Pre-moves (phase 2 if needed).
- Sound settings and basic analysis board (phase 2).
- Match chat and report flow (moderated).

### 4.3 Anti-cheat baseline
- Server-authoritative move validation.
- Behavior anomaly heuristics.
- Match integrity logs (timings, move confidence vectors).
- Tournament integrity checks and rematch-farming protections.

---

## 5) System Architecture

### 5.1 Smart contracts
1. **CULTStakeEscrow**
   - lock funds,
   - collect creator fee,
   - apply burn on draw,
   - settle payouts by signed result.
2. **MatchRegistry**
   - create/join/cancel/start/finish matches,
   - link off-chain game IDs to on-chain escrows,
   - enforce stake bounds and mode selection.
3. **TournamentManager**
   - create 16/32 brackets,
   - manage entries and rounds,
   - settle final pool.
4. **Config/Governance**
   - fee recipient,
   - burn address,
   - stake limits,
   - pause controls.

### 5.2 Backend services
- Real-time chess game server (authoritative state + clocks).
- Anti-cheat service.
- Result signing service (multi-sig recommended).
- Tournament orchestration service.
- Oracle service for CULT/USD conversion checks.

### 5.3 Frontend
- Lobby + match creation (stake type + amount chooser).
- Tournament browser/join flow.
- Chess board/game room with timer and identity preferences.
- Wallet connect and transaction signing.
- Settlement receipts + on-chain explorer links.

---

## 6) UX: Stake Type and Quantity Selection

### 6.1 Match creation modal
Fields:
- Mode selector:
  - 25% Opponent Capture,
  - All-In Winner Takes All.
- Stake input in CULT + fiat estimate.
- Preset chips: `$100`, `$250`, `$500`, `$1k`, `$10k`, custom.
- Validation:
  - min `$100`, max `$1,000,000` equivalent.
- Timer preset selector.
- Privacy selector for displayed identity.

### 6.2 Confirmation screen
Show exact preview:
- "You stake" amount,
- "Creator fee (2%)" amount,
- "If draw, burn (5% gross pot)" projected amount,
- Payout preview for win/loss/draw.

---

## 7) Tournament UX
- Join screen with live participant count and bracket size.
- Entry escrow confirmation with fee disclosure.
- Round timer and auto-advance for no-shows/timeouts.
- Prize pool visibility.
- Final settlement receipt.

---

## 8) Security, Risk, and Compliance

### 8.1 Security controls
- Reentrancy guards.
- Pull-based withdrawals where possible.
- Pausable contracts.
- Role-based access with timelocks.
- Formal tests + third-party smart contract audit.

### 8.2 Abuse prevention
- Botting and engine-assist detection.
- Account linkage and collusion monitoring.
- Device/network fingerprint controls (privacy-aware).

### 8.3 Legal/compliance checklist (pre-launch)
- Jurisdictional legal review for stake-based skill gaming.
- KYC/AML policy if required.
- Sanctions screening policy.
- Terms of service and dispute resolution process.

---

## 9) Delivery Plan

### Phase 0: Foundations (2–3 weeks)
- Product specs locked.
- Contract interfaces and settlement math test vectors.
- UX wireframes for lobby and game room.

### Phase 1: MVP (6–10 weeks)
- 1v1 modes (25% and all-in).
- 16-player tournament.
- Timers and draw adjudication.
- Wallet privacy display modes.
- Creator fee + burn flows live.

### Phase 2: Expansion (4–8 weeks)
- 32-player tournaments.
- Enhanced anti-cheat.
- Advanced chess UX features.
- Expanded analytics and leaderboard systems.

---

## 10) Acceptance Criteria (MVP)
- Stake bounds are strictly enforced using oracle pricing.
- Creator fee transferred exactly 2% per funded entry.
- Draws burn exactly 5% of gross stake to dead address.
- Mode A and Mode B payouts match deterministic formulas.
- Timer adjudication and disconnect policies are deterministic.
- Privacy display modes configurable and correctly rendered.
- 16-player tournament completes and settles on-chain.

