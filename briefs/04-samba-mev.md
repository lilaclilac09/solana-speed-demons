# Module 4: Samba — MEV at Wire Speed

### Teaching Arc
- **Metaphor:** A post office that also offers a "guaranteed express window" — most packages go through the regular queue by arrival time, but couriers with special envelopes (bundles) can guarantee their package arrives before or after a specific other one. The post office has been modified to support this guarantee.
- **Opening hook:** "Harmonic Finance took Firedancer — the fastest validator engine in the world — and added one capability: the ability to accept transaction bundles with guaranteed ordering. The result is Samba."
- **Key insight:** MEV (Maximum Extractable Value) is money that exists purely in the ordering of transactions. If you can guarantee that your arbitrage trade lands *after* a price-moving trade and *before* anyone else reacts, you extract that value. Harmonic built this guarantee directly into the validator.
- **"Why should I care?":** MEV is one of the largest sources of revenue for sophisticated Solana traders. Understanding how bundle submission works — and that it lives at the validator layer, not the application layer — changes how you think about transaction timing.

### Code Snippets

From samba git log (key commits):
```
56ada6c3e Update submodule URL to use HTTPS
6f2068da5 v3.1.14 rebase
534cf9efb fix harmonic buffer + cleanup arrival_ns
2dce39b7d reduce buffer time
943729e15 pack: fix harmonic rejection condition
3b8614b6b bundle: fix reset submit_leader_window_info_wait
15156b7d1 initial streaming impl
5dd65de1a only append if not upcoming leader
b8af7803d update .gitmodules
f6db98e2a fix: account for block txns in bundle/txn insert
21cf12dec harmonic fd client id
7c0ae4c32 pack: skip zero size failed block signal link
```

From Samba source tree differences vs Firedancer:
```
samba/src/discof/sender/   ← Harmonic-added: bundle sender tile
samba/src/discof/send/     ← Harmonic-added: streaming tx submission
samba/src/discof/resolv/   ← Modified: resolv tile with MEV routing
```

### Interactive Elements

- [x] **Pattern cards** — 4 cards explaining MEV types: (1) Arbitrage — "buy low on DEX A, sell high on DEX B in the same block", (2) Sandwich — "buy before a large trade, sell after it moves the price", (3) Liquidation — "be first to liquidate an undercollateralized loan", (4) Bundle — "group of transactions guaranteed to land atomically in order". Use actor-2 (teal) for the accent color cards.

- [x] **Group chat animation** — id="chat-module4". Shows a bundle being processed. Actors: Trader (actor-4, golden), Harmonic Bundle API (actor-2, teal), Samba Pack Tile (actor-1, vermillion), Validator (actor-5, forest).
  1. Trader: "I want to send a bundle: tx A then tx B, guaranteed in that order, or both fail."
  2. Harmonic API: "Bundle received. Checking if we are the next leader for an upcoming slot..."
  3. Harmonic API: "We are leader in 2 slots. Forwarding bundle to the pack tile."
  4. Samba Pack: "Bundle queued. This pair is locked — tx A gets slot N, tx B gets slot N+1."
  5. Samba Pack: "Regular transactions filling remaining block space by fee priority..."
  6. Validator: "Block produced. Bundle landed. Tx A and B confirmed in order."

- [x] **Code↔English translation** — use the git log snippet above. Left: git commit messages. Right: "v3.1.14 rebase = Harmonic keeps pulling in the latest Firedancer updates... fix harmonic buffer = the internal queue that holds bundles before they enter the pack tile... pack: fix harmonic rejection condition = when to refuse a bundle (e.g., can't fit in the block)... initial streaming impl = bundles can be streamed in real-time, not just submitted as one batch... harmonic fd client id = Samba identifies itself differently from Firedancer on the network."

- [x] **Callout box (accent)** — "Why Fork Firedancer, Not Build on Agave? Firedancer's tile architecture makes it much easier to add a new capability: just add a new tile (the bundle sender) and modify the pack tile's selection logic. In Agave (the Rust validator), the equivalent change would require threading MEV logic through a much more complex codebase. Harmonic chose the C engine because it's surgically modifiable."

- [x] **Quiz** — 3 scenario questions:
  1. A bundle contains 3 transactions: buy, arbitrage, sell. The block is almost full with only 2 slots left. What does Samba do with the bundle? (Answer: it rejects the entire bundle — bundles are atomic, partial inclusion would break the ordering guarantee)
  2. You notice Samba's `pack: fix harmonic rejection condition` commit. What does this suggest about MEV bundle development? (Answer: the rejection logic — when to refuse a bundle vs accept it — is a core challenge. Getting this wrong means either losing revenue or breaking ordering guarantees)
  3. Harmonic runs Samba validators on mainnet. When a user sends a regular transaction (not a bundle) to a Samba validator, what happens? (Answer: it goes through the regular pack queue by fee priority — Samba is backward-compatible, bundles just get a fast-lane)

### Reference Files to Read
- `references/interactive-elements.md` → "Group Chat Animation", "Pattern/Feature Cards", "Code ↔ English Translation Blocks", "Multiple-Choice Quizzes", "Glossary Tooltips", "Callout Boxes"
- `references/content-philosophy.md` → full doc
- `references/gotchas.md` → full doc

### Connections
- **Previous module:** Transaction Journey — traced a normal transaction through the Firedancer pipeline
- **Next module:** Delorean: The Time Machine — shifts to Temporal's debugging tool for replaying historical transactions
- **Tone/style notes:** Accent teal. Samba=actor-2 (teal). Module uses --color-bg (even module). MEV is a sensitive topic — frame it neutrally as a market-efficiency mechanism, not as "cheating." Mention that Jito (another MEV system on Solana) works at the app level while Samba works at the validator level.
