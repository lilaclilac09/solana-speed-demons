# Module 2: Firedancer — The C Engine

### Teaching Arc
- **Metaphor:** A chip fab assembly line — each station (tile) does exactly one job, runs on its own CPU core, and passes work to the next station through a conveyor belt (ring buffer). No station ever waits for another. No one shares tools. Zero coordination overhead.
- **Opening hook:** "Jump Crypto wrote Firedancer the same way they build high-frequency trading systems — in C, without garbage collection, with every CPU core assigned exactly one job."
- **Key insight:** The "tile" model is the central insight: one tile = one thread = one CPU core = one responsibility. Tiles never share memory directly — they communicate via Tango, a shared-memory ring buffer system borrowed from the HFT world.
- **"Why should I care?":** Understanding tiles tells you why Firedancer can scale horizontally (add more NIC tiles for more bandwidth), why bugs are easy to isolate, and how to talk about validator architecture with engineers.

### Code Snippets

File: /tmp/firedancer/doc/organization.txt (source tree section)
```
└── src/             The main Firedancer source tree
    ├── app/         Main binaries (fdctl, fddev, firedancer, firedancer-dev)
    ├── ballet/      Standalone implementations of hash functions, cryptographic algorithms
    ├── choreo/      Consensus components (fork choice, voting)
    ├── disco/       Common tiles (network stack, block production)
    ├── discof/      Full Firedancer tiles (consensus, runtime, RPC)
    ├── discoh/      Frankendancer tiles (Agave FFI)
    ├── flamenco/    Solana SVM / runtime
    ├── funk/        Fork-aware in-memory key-value store (accounts DB, program cache)
    ├── groove/      Disk-backed memory-mapped key-value cold store
    ├── tango/       IPC messaging layer
    ├── util/        C language environment, system runtime, common data structures
    ├── waltz/       Networking
    └── wiredancer/  FPGA modules
```

File: /tmp/firedancer/doc/organization.txt (pipeline diagram, abbreviated)
```
from external NIC
     |
     v
  +-- core ----+
  | NIC / QUIC |    ← tile: one CPU core, handles raw packets
  | SIG VERIFY |    ← tile: ed25519 signature verification
  |  DEDUP TAG |    ← tile: marks already-seen transactions
  +------------+
     |
     v  (metadata via MCACHE ring buffer)
  +-- core ----+
  |    DEDUP   |    ← tile: filters duplicates
  |     MUX    |    ← tile: multiplexes verified txs
  +------------+
     |
     v
  +-- core ----+
  | BLOCK PACK |    ← tile: selects which txs go in the block
  +------------+
     |
     v
  to Agave validator (Frankendancer) or flamenco (full Firedancer)
```

### Interactive Elements

- [x] **Visual file tree** — show the src/ directory with plain-English one-liners for each folder. Highlight: tango (the nervous system), ballet (the math library), disco (the main pipeline), flamenco (where Solana programs actually run).
- [x] **Group chat animation** — id="chat-module2". Actors: NIC tile (actor-1), SigVerify tile (actor-2), Pack tile (actor-4). Script:
  1. NIC: "Got 10,000 packets from the network. Filtering to valid QUIC transactions..."
  2. NIC: "Passing 8,400 verified packets through the ring buffer."
  3. SigVerify: "Checking ed25519 signatures... 8,392 valid, 8 rejected."
  4. SigVerify: "Tagging each valid tx with a fingerprint for dedup."
  5. Pack: "Got the metadata. Selecting highest-fee transactions for the next block..."
  6. Pack: "Block ready. 2,400 transactions selected. Handing off to execution."
- [x] **Pattern cards** — 4 cards for the key Tango IPC primitives: mcache (metadata ring buffer — "the postal tracking system"), dcache (data cache — "the actual package"), fseq (flow sequencer — "the ticket counter"), fctl (flow control — "the traffic cop")
- [x] **Code↔English translation** — use the file tree snippet above. Left: the directory listing. Right: plain English — "tango/ is the nervous system — all tiles communicate through here. ballet/ is the math library — sha256, ed25519, base58 are all implemented here from scratch for maximum speed."
- [x] **Quiz** — 3 scenario questions:
  1. A Firedancer validator is running with 4 NIC tiles and processing 200,000 tx/s. You want to double throughput. What is the most direct architectural approach? (Answer: add more NIC tiles / more CPU cores assigned to ingress — tiles scale horizontally)
  2. You find a bug where a signature is sometimes accepted when it should be rejected. Which folder in the Firedancer source tree would you investigate first? (Answer: ballet/ for the ed25519 implementation, or the SIG VERIFY tile in disco/)
  3. Frankendancer uses Agave for execution. What does that mean for a validator operator? (Answer: you get Firedancer networking speed but rely on Agave for the SVM/runtime — not fully independent yet)

### Reference Files to Read
- `references/interactive-elements.md` → "Group Chat Animation", "Pattern/Feature Cards", "Code ↔ English Translation Blocks", "Visual File Tree", "Multiple-Choice Quizzes", "Glossary Tooltips", "Callout Boxes"
- `references/content-philosophy.md` → full doc
- `references/gotchas.md` → full doc

### Connections
- **Previous module:** The Validator — introduced what validators do and the 3 repos
- **Next module:** Transaction Journey — will trace a real transaction through this pipeline step by step
- **Tone/style notes:** Accent teal. Firedancer=actor-1 (vermillion). Module uses --color-bg background (even module). Two Frankendancer vs full Firedancer callout box — explain the distinction clearly: Frankendancer is production-ready today, full Firedancer is still in development.
