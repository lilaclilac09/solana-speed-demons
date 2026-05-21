# Module 6: The Bigger Picture

### Teaching Arc
- **Metaphor:** Three different laboratories studying the same physics problem — different teams, different tools, shared scientific record. When their measurements agree, we have confidence. When they disagree, we know exactly where to look.
- **Opening hook:** "You have seen Firedancer build the engine, Samba modify it for MEV, and Delorean replay history. Here is how these three tools connect — and what they tell you about where Solana is headed."
- **Key insight:** Client diversity is not fragmentation — it is resilience. A blockchain where every validator runs identical code is a single point of failure. Multiple independent implementations catch bugs that no single team can find alone.
- **"Why should I care?":** Knowing who builds what, and why, helps you make better decisions: which RPC endpoint to use, which tooling to trust, what to watch as Solana matures, and how to position yourself as an engineer or protocol developer.

### Code Snippets

No new code snippets needed for this module. Use a conceptual diagram and relationship map instead.

Relationship summary (for the architecture diagram):
- Firedancer (Jump Crypto, C) → Samba is a fork of Firedancer with MEV extensions
- Firedancer (validator) → Delorean debugs transactions that validators produce
- Agave (Solana Labs, Rust) → Frankendancer uses Agave for execution (discoh tiles)
- Solana Network → both Frankendancer and Agave validators participate in consensus

Who runs what:
- Jump Crypto → Firedancer / Frankendancer
- Harmonic Finance → Samba (Firedancer fork + MEV)
- Temporal XYZ → Penrose (historical data service) + delorean (client)
- Solana Labs / Anza → Agave validator (the incumbent)

Why it matters:
- If every validator ran Agave, a single Agave bug could halt the chain
- Firedancer and Agave independently implement the Solana protocol
- When they disagree on a transaction result, it surfaces protocol ambiguity
- This is called "consensus diversity" — the same reason blockchains are more resilient than single servers

### Interactive Elements

- [x] **Architecture diagram (interactive)** — show the full ecosystem. Zones: "The Network" (Solana validators), "Validator Clients" (Agave, Frankendancer, Firedancer, Samba), "Tooling" (Penrose, delorean). Clickable components show descriptions. Use arch-diagram with arch-zone dividers. Components: Agave (actor-5 forest), Firedancer (actor-1 vermillion), Frankendancer (actor-1 with note "hybrid"), Samba (actor-2 teal), Penrose (actor-3 plum), delorean (actor-3 plum), Solana Network (actor-4 golden).

- [x] **Icon rows** — 4 rows for the 4 organizations: (1) Jump Crypto 🔥 "Built Firedancer from scratch in C. Runs Frankendancer on mainnet today. Full Firedancer coming.", (2) Harmonic Finance 🎶 "Forked Firedancer into Samba. Added MEV bundle infrastructure. Runs MEV validators on mainnet.", (3) Temporal XYZ ⏱️ "Built Penrose, a historical transaction data service. Built delorean as the reference client.", (4) Anza (Solana Labs) 🌿 "Maintains Agave, the original Solana validator. Still runs the majority of mainnet validators."

- [x] **Callout box (info)** — "What Client Diversity Means in Practice: In March 2023, a bug in Agave caused a 4.5-hour mainnet outage. With only one validator client, the fix had to come from one team. With Firedancer running 30%+ of stake, a future similar bug might not halt the chain — Firedancer validators could keep producing blocks while Agave gets patched."

- [x] **Drag-and-drop quiz** — match each tool to its primary purpose. Chips: [delorean, Samba, Firedancer, Penrose]. Zones: "Replay historical transactions for debugging", "Add MEV bundle ordering to a validator", "Run a high-performance Solana validator from scratch", "Capture and serve transaction fixtures from the chain".

- [x] **Multiple-choice quiz** — 3 scenario questions:
  1. You are building a new Solana program and want to test it against a specific historical transaction that broke a competitor's protocol. Which tool gives you this capability? (Answer: delorean with --replace-program)
  2. A large MEV firm wants to guarantee that their arbitrage bundles land before competitors on Solana mainnet. At which layer does this guarantee need to be implemented? (Answer: at the validator/pack tile layer — application-layer solutions cannot guarantee ordering, only the validator itself can)
  3. Firedancer reports a transaction as valid; Agave reports it as failed. This is a consensus split. What should happen? (Answer: this triggers protocol-level investigation — one implementation has a bug. Client diversity made the bug visible instead of silently corrupting state on all validators.)

### Reference Files to Read
- `references/interactive-elements.md` → "Interactive Architecture Diagram", "Icon-Label Rows", "Drag-and-Drop Matching", "Multiple-Choice Quizzes", "Callout Boxes", "Glossary Tooltips"
- `references/content-philosophy.md` → full doc
- `references/gotchas.md` → full doc

### Connections
- **Previous module:** Delorean: The Time Machine — completed the deep-dives on all 3 repos
- **Next module:** None — this is the finale
- **Tone/style notes:** Accent teal. Module uses --color-bg (even module). End on an aspirational note: Solana is building the infrastructure layer for global finance, and these tools are the foundation. No need to be salesy — the engineering speaks for itself. Include a small "what to explore next" section pointing to each repo's GitHub.
