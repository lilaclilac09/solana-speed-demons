# Module 3: Transaction Journey

### Teaching Arc
- **Metaphor:** A sorting facility at a major airport — packages (transactions) arrive chaotically from many directions, get checked, labeled, deduplicated, sorted by priority, and loaded onto exactly the right flight. Every step has a different specialist. Packages never travel backwards.
- **Opening hook:** "You click Swap on Jupiter. In the next 400 milliseconds, your transaction will pass through six distinct checkpoints inside a Firedancer validator before it's committed to the chain."
- **Key insight:** The pipeline is lock-free — tiles communicate through ring buffers with no mutual exclusion. Every checkpoint either stamps a transaction "valid and continue" or silently drops it. Nothing goes backward.
- **"Why should I care?":** When a transaction fails or stalls, knowing which checkpoint dropped it — QUIC ingest, sig verify, dedup, pack, or execution — is the difference between debugging in 5 minutes and debugging for days.

### Code Snippets

File: /tmp/delorean-client/delorean-client/src/main.rs (build_mock_bank function, lines ~450-480)
```rust
fn build_mock_bank(f: &TransactionFixture) -> MockBank {
    let bank = MockBank::default();
    {
        let mut accounts = bank.accounts.write().unwrap();

        for fa in &f.pre_accounts {
            accounts.insert(fa.pubkey, fixture_account_to_shared(fa));
        }

        for sv in f.sysvars.iter() {
            let mut acct = AccountSharedData::default();
            acct.set_data_from_slice(&sv.data);
            acct.set_owner(solana_sdk_ids::sysvar::id());
            acct.set_lamports(1);
            accounts.insert(sv.sysvar_id, acct);
        }

        for prog in &f.programs {
            let programdata = build_loader_v3_programdata_account(prog);
            accounts.insert(get_program_data_address(&prog.program_id), programdata);
        }
    }
    bank
}
```

File: /tmp/delorean-client/delorean-client/src/main.rs (run() execution section)
```rust
println!("executing...\n");
let env = TransactionProcessingEnvironment {
    blockhash: fixture.recent_blockhash,
    blockhash_lamports_per_signature: fixture.lamports_per_signature,
    epoch_total_stake: 0,
    feature_set,
    program_runtime_environments_for_execution: batch_processor.get_environments_for_epoch(0),
    program_runtime_environments_for_deployment: batch_processor.get_environments_for_epoch(0),
    rent: extract_rent(&fixture.sysvars),
};
```

### Interactive Elements

- [x] **Data flow animation (main hero)** — id="flow-module3". This is the centerpiece. 7 steps:
  Step 1: highlight NIC tile — "Your transaction arrives as raw UDP packets via QUIC"
  Step 2: packet from actor-1 to actor-2 — "NIC tile extracts the transaction payload"
  Step 3: highlight SigVerify tile — "ed25519 signature is verified against your public key"
  Step 4: packet from actor-2 to actor-3 — "Verified transaction tagged with a 64-bit fingerprint"
  Step 5: highlight Dedup tile — "Fingerprint checked against recent cache — first time seen? Continue."
  Step 6: highlight Pack tile — "Ranked by fee priority, competing for one of 2,400 block slots"
  Step 7: highlight Execute tile — "SVM runs the program. Accounts updated. Result committed."
  Actors: flow-actor-1=NIC, flow-actor-2=SigVerify, flow-actor-3=Dedup, flow-actor-4=Pack, flow-actor-5=Execute
  
- [x] **Code↔English translation** — use build_mock_bank snippet above. Left: Rust code. Right: "Create an empty in-memory database... For each account the transaction will touch, load its pre-transaction balance and data... Load all Solana system variables (like current rent prices)... Load all program bytecode. Now the SVM has a complete snapshot of blockchain state."

- [x] **Callout box (info)** — "The Zero-Copy Insight: Firedancer's tiles never copy transaction data between steps. The NIC tile writes the raw bytes once into a shared memory region (dcache), and every downstream tile reads from the same region via a pointer. This is why Firedancer is so fast — memory bandwidth is never wasted on duplicating data."

- [x] **Step cards** — 6 numbered step cards summarizing the pipeline checkpoints: (1) QUIC Ingest, (2) Signature Verify, (3) Dedup Tag, (4) Block Pack, (5) SVM Execute, (6) Consensus Commit. Short, punchy descriptions.

- [x] **Quiz** — 3 scenario questions:
  1. A user says "my transaction was never confirmed, but it didn't fail either — it just disappeared." Based on the pipeline, which stage is the most likely culprit? (Answer: Dedup — the transaction may have been deduplicated as a resend of one the validator already saw)
  2. Compute units (CUs) are a measure of how much work the SVM did executing a transaction. If a transaction uses more CUs than its budget, what happens? (Answer: it fails at the Execute stage — the SVM halts and rolls back all account changes)
  3. The Pack tile selects which transactions go in a block. If two transactions touch the same account, can they both be in the same block? (Answer: yes, but they must be ordered sequentially for that account — Pack enforces this)

### Reference Files to Read
- `references/interactive-elements.md` → "Message Flow / Data Flow Animation", "Code ↔ English Translation Blocks", "Numbered Step Cards", "Multiple-Choice Quizzes", "Glossary Tooltips", "Callout Boxes"
- `references/content-philosophy.md` → full doc
- `references/gotchas.md` → full doc

### Connections
- **Previous module:** Firedancer: The C Engine — explained what tiles are and how they communicate
- **Next module:** Samba: MEV at Wire Speed — will show how Harmonic forks this pipeline to add MEV capabilities
- **Tone/style notes:** Accent teal. Module uses --color-bg-warm (odd). Actor colors for flow animation: NIC=actor-1, SigVerify=actor-2, Dedup=actor-3, Pack=actor-4, Execute=actor-5. No single-quotes inside data-steps JSON — use &apos; or rewrite with double-quote delimiters.
