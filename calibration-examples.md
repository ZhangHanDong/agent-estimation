# Calibration Examples

Real-world project estimates using the round-based system. Use these to calibrate your estimates for similar tasks.

## Small Projects (< 20 rounds)

### CLI Tool — File Format Converter
Convert JSON to YAML with schema validation.

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| Arg parsing + I/O | 1 | 1.0 | 1 | clap/structopt, one-shot |
| JSON→YAML core | 1 | 1.0 | 1 | serde, trivial |
| Schema validation | 3 | 1.3 | 4 | jsonschema crate, edge cases |
| Error handling + UX | 2 | 1.0 | 2 | polish |
| **Total** | **7** | | **8** | ~24 min |

### Static HTML Page with WebSocket Client
Phone-side client for a remote control app.

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| HTML layout + buttons | 2 | 1.0 | 2 | standard web dev |
| WebSocket connection | 1 | 1.0 | 1 | known pattern |
| Button → command mapping | 2 | 1.0 | 2 | straightforward |
| Visual feedback + reconnect | 2 | 1.3 | 3 | edge cases |
| **Total** | **7** | | **8** | ~24 min |

---

## Medium Projects (20-50 rounds)

### Desktop App — Keyboard Broadcaster (Makepad + Rust)
One Mac keyboard controlling 27 devices over LAN. (The KeyboardCast example.)

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| HTTP/WS server (axum) | 3 | 1.0 | 3 | mature crate, standard pattern |
| Phone web client | 3 | 1.0 | 3 | static HTML + WS |
| Makepad main UI | 8 | 1.3 | 10 | layout iteration needed |
| CGEvent keyboard capture | 5 | 1.5 | 8 | macOS permissions, platform quirks |
| QR code generation | 1 | 1.0 | 1 | qrcode crate |
| Client management state | 3 | 1.3 | 4 | connect/disconnect/list |
| Category filtering UI | 2 | 1.0 | 2 | data-driven, simple |
| Integration | +4 | 1.3 | 5 | wiring async events to UI |
| **Total** | **29** | | **36** | ~1.5-2 hours |

**Wave Analysis** (3 agents):

| Wave | Modules | Max Effective Rounds | Notes |
|------|---------|---------------------|-------|
| Wave 1 | HTTP/WS server, QR code gen, Phone web client | 3 | all independent, run in parallel |
| Wave 2 | Makepad main UI, Client mgmt state, Category filtering UI | 10 | depend on server/client contracts |
| Wave 3 | CGEvent keyboard capture | 8 | depends on UI + server |
| Integration | wiring async events to UI | 5 | after all waves |

- Coordination overhead: +3 rounds (contract definition for server↔client protocol)
- **Sequential wallclock**: ~108 min (36 rounds × 3 min)
- **Parallel wallclock**: ~57 min (3 + 10 + 8 + 5 = 26 rounds × 3 min, minus overlap; effective ~19 wave rounds + 3 coordination + 5 integration = 27 rounds)
- **Speedup vs sequential**: ~47%

### REST API with Auth + DB
Standard CRUD API with JWT auth and Postgres.

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| Project scaffold | 1 | 1.0 | 1 | template |
| DB schema + migrations | 3 | 1.0 | 3 | sqlx/diesel |
| CRUD endpoints | 4 | 1.0 | 4 | boilerplate |
| JWT auth middleware | 3 | 1.3 | 4 | token edge cases |
| Input validation | 2 | 1.0 | 2 | standard |
| Error handling | 2 | 1.0 | 2 | standard |
| Tests | 5 | 1.3 | 7 | integration tests fiddly |
| Integration | +3 | 1.0 | 3 | well-defined boundaries |
| **Total** | **23** | | **26** | ~1.3 hours |

---

## Large Projects (50-100+ rounds)

### Full-Stack Dashboard with Real-Time Charts
React frontend + Rust backend + WebSocket streaming.

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| Backend API | 5 | 1.0 | 5 | standard REST |
| WebSocket streaming | 4 | 1.3 | 5 | backpressure, reconnection |
| React scaffold + routing | 3 | 1.0 | 3 | standard |
| Dashboard layout | 6 | 1.3 | 8 | responsive, component hierarchy |
| Chart components | 8 | 1.5 | 12 | recharts config, data transforms |
| Auth flow (frontend) | 4 | 1.3 | 5 | token refresh, protected routes |
| State management | 5 | 1.3 | 7 | real-time + REST sync |
| Tests | 6 | 1.5 | 9 | E2E flaky, mocking WS |
| Integration | +6 | 1.5 | 9 | cross-stack debugging |
| **Total** | **47** | | **63** | ~3-3.5 hours |

**Wave Analysis** (3 agents):

| Wave | Modules | Max Effective Rounds | Notes |
|------|---------|---------------------|-------|
| Wave 1 | Backend API, React scaffold + routing, Auth flow (frontend) | 5 | all independent foundations |
| Wave 2 | WebSocket streaming, Dashboard layout, Chart components | 12 | depend on backend/frontend scaffolds |
| Wave 3 | State management, Tests | 9 | depend on WS + UI components |
| Integration | cross-stack debugging | 9 | after all waves |

- Coordination overhead: +3 rounds (API contract, data schema alignment)
- **Sequential wallclock**: ~189 min (63 rounds × 3 min)
- **Parallel wallclock**: ~108 min (5 + 12 + 9 + 9 = 35 wave rounds + 3 coordination = 38 rounds × 3 min; with 3 agents effective ~36 rounds)
- **Speedup vs sequential**: ~43%

---

## Estimation Accuracy Notes

These examples assume:
- The agent (Claude Code or similar) has access to the full codebase
- The user reviews but doesn't heavily rewrite agent output
- Standard development environment (no exotic toolchains)
- `minutes_per_round` = 3

Common sources of estimate blowup:
- **Unfamiliar framework** (e.g., first time with Makepad): +30-50% on UI modules
- **Platform permissions** (macOS accessibility, Android intents): +50-100% on that module
- **Undocumented APIs**: can 2x a module easily
- **"One more thing" scope creep**: user adds features mid-build, not captured in initial estimate

Common sources of estimate shrinkage:
- **User provides existing code to extend**: modules may drop to 1-2 rounds
- **Agent has done this exact pattern before in the conversation**: 1 round
- **Copy-paste from a working sibling module**: 1 round

## Real-World Calibration Log

### 2026-04-17 — ChronicleCore-Ark: Expert Routing Table Injection

Inject a markdown table of 38 experts (codename, title, "use when") into all Claude sessions' systemPrompt so the model can proactively suggest summoning appropriate experts.

**Initial estimate** (before reconnaissance):

| Module | Base | Risk | Effective | Notes |
|--------|------|------|-----------|-------|
| Parse `Use when` from EXPERT.md | 2 | 1.0 | 2 | Boilerplate |
| Add `Use when` to all 38 experts | 3 | 1.3 | 4 | Content writing × 38 |
| `buildRoutingTable()` formatter | 2 | 1.0 | 2 | Pure string |
| Inject in `getExpertSystemPrompt()` | 2 | 1.3 | 3 | Integration |
| Verify | 2 | 1.0 | 2 | Log inspection |

**Total: 13 rounds × 3 min = 39 min wallclock**

**Actual outcome: ~5 rounds × 1 min = 5.5 min wallclock**

**What blew the estimate (7× overestimate):**

1. **Phantom module: "Add Use when to all 38 experts" (-4 rounds).** All 38 experts' SKILL.md `description` fields ALREADY contained `Use when: ...`. A single `grep -c "Use when" .../*/SKILL.md` would have revealed this BEFORE listing the module. This is exactly the trap Step 0 (Reconnaissance) now prevents.
2. **Phantom verification module (-2 rounds).** Once `tsc --noEmit` passed and `pnpm build:electron` succeeded, no separate verification pass was needed. For low-risk changes, the compile/build itself is verification.
3. **Wrong min/round tier (-2 min × 5 rounds = 10 min).** User was in rapid batch mode ("好", "繼續", "好") — no per-step review. Should have used 1 min/round, not 3.

**Lessons that updated the skill:**

- Added **Step 0: Reconnaissance** to force `grep`/`ls`/`read` before listing modules
- Added **1 min/round tier (rapid batch execution)** for no-gate scenarios
- Added signal-detection for rapid batch mode (user says "go", "continue" without reviewing)
