# Module 5: Delorean — The Time Machine

### Teaching Arc
- **Metaphor:** An aircraft's flight data recorder (black box) — it captures a complete snapshot of every instrument reading the moment before impact. Investigators can re-run the last moments with perfect fidelity. Temporal's Penrose service is the black box; delorean is the investigation software.
- **Opening hook:** "A trader's bot lost $40,000 on a transaction that should have succeeded. The transaction is buried in a block from 3 days ago. How do you debug it? With delorean, you can re-run it in under a second on your laptop."
- **Key insight:** A "fixture" is a complete snapshot of every account, program, and Solana system variable that a transaction touched — enough information to replay it perfectly in isolation, long after the fact.
- **"Why should I care?":** Debugging Solana programs against historical state is normally impossible — you can't go back in time. Penrose+delorean changes this. Understanding fixtures also helps you understand what the SVM needs to execute any transaction.

### Code Snippets

File: /tmp/delorean-client/penrose-types/src/fixture.rs (TransactionFixture struct)
```rust
pub struct TransactionFixture {
    pub schema_version: u32,
    pub slot: u64,
    pub recent_blockhash: Hash,
    pub signature: Signature,
    pub client_version: Vec<u8>,
    pub enabled_features: Vec<Pubkey>,
    pub lamports_per_signature: u64,
    pub transaction: Vec<u8>,
    pub alt_writable: Vec<Pubkey>,
    pub alt_readonly: Vec<Pubkey>,
    pub pre_accounts: Vec<FixtureAccount>,
    pub programs: Vec<FixtureProgramData>,
    pub sysvars: Arc<Vec<FixtureSysvar>>,
    pub result: ExecutionResult,
    pub post_accounts: Vec<FixtureAccount>,
}
```

File: /tmp/delorean-client/delorean-client/src/main.rs (get_transaction_fixture)
```rust
fn get_transaction_fixture(
    rpc: &RpcClient,
    signature: &Signature,
) -> Result<Option<TransactionFixture>, ClientError> {
    let encoded: Option<String> = rpc.send(
        RpcRequest::Custom {
            method: "getTransactionFixture",
        },
        json!([signature.to_string()]),
    )?;
    let Some(b64) = encoded else {
        return Ok(None);
    };
    let bytes = BASE64_STANDARD.decode(&b64)...
    let fixture = TransactionFixture::deserialize(&bytes)...
    Ok(Some(fixture))
}
```

File: /tmp/delorean-client/delorean-client/src/main.rs (apply_program_replacements snippet)
```rust
fn apply_program_replacements(
    bank: &MockBank,
    rent: &Rent,
    replacements: &[ProgramReplacement],
) -> Result<(), Box<dyn std::error::Error>> {
    if replacements.is_empty() {
        return Ok(());
    }
    let mut accounts = bank.accounts.write().unwrap();
    for rep in replacements {
        let program_account = accounts.get(&rep.program_id)...
        let elf = read_replacement_elf(&rep.elf_path)?;
        // patch the programdata account with the new ELF bytes
        accounts.insert(programdata_address, updated);
    }
    Ok(())
}
```

### Interactive Elements

- [x] **Code↔English translation (hero)** — use TransactionFixture struct. Left: the Rust struct fields. Right: "schema_version — which version of this format we are reading... slot — the specific Solana block number where this happened... signature — the unique ID of this exact transaction... enabled_features — Solana has hundreds of feature flags; this captures which were active at that moment... pre_accounts — every wallet and contract balance BEFORE the transaction ran... programs — the bytecode of every smart contract involved... sysvars — system-wide values like current rent prices... post_accounts — what every account looked like AFTER. With all this, you can replay the transaction perfectly."

- [x] **Group chat animation** — id="chat-module5". Actors: Developer (actor-3, plum), Delorean CLI (actor-1, vermillion), Penrose Service (actor-2, teal), Local SVM (actor-5, forest).
  1. Developer: "delorean WRM4dvt... https://penrose.temporal.xyz"
  2. Delorean: "Connecting to Penrose... fetching fixture for signature WRM4dvt..."
  3. Penrose: "Found it. Slot 420195494, captured 3 days ago. Sending fixture (2.1 MB)..."
  4. Delorean: "Building mock bank... loading 25 pre-accounts, 9 sysvars, 3 programs..."
  5. Delorean: "Sanitizing transaction... executing..."
  6. Local SVM: "Execution complete. Status: Ok(()). CUs: 104291."
  7. Delorean: "Status: MATCH. CUs: MATCH. Post-state: MATCH. All 25 accounts match."

- [x] **Step cards** — 5 steps of how delorean works: (1) Fetch fixture from Penrose via getTransactionFixture RPC, (2) Deserialize the fixture blob into pre_accounts/programs/sysvars, (3) Build a MockBank (in-memory Solana state), (4) Execute through the real SVM (agave TransactionBatchProcessor), (5) Compare actual vs expected status/CUs/post-accounts.

- [x] **Code↔English translation (secondary)** — use get_transaction_fixture snippet. Left: Rust code. Right: "Call the Penrose server with a custom RPC method called getTransactionFixture... The server returns the fixture as a base64-encoded string (a way to send binary data as text)... Decode from base64 back to bytes... Deserialize into a TransactionFixture struct. Done — the entire historical state is now in memory."

- [x] **Callout box (accent)** — "The --replace-program Flag: Delorean has a killer feature — you can swap out any program's bytecode before replay. Testing whether a bug fix actually works? Replace the deployed program with your new version, replay the failing transaction, and see if it passes. This is historical simulation: same accounts, same state, different code."

- [x] **Quiz** — 3 scenario questions:
  1. You want to debug why a specific Raydium swap failed 2 weeks ago. Raydium has upgraded their program since then. Which approach lets you debug the ORIGINAL failure accurately? (Answer: use delorean with the original fixture — it captures the program bytecode as it was at that exact slot, so you replay with the old code)
  2. The delorean output shows "CUs: MISMATCH — actual 95,000, expected 104,291." What does this suggest? (Answer: your local SVM environment differs from the validator's — possibly different feature flags or a program that was changed. The fixture captured what the real validator did; your replay is doing something different)
  3. You want to test whether your new contract fix handles the edge case that caused a $40k loss. Using delorean, what is the correct workflow? (Answer: use --replace-program to swap in your new ELF, then replay the historical failing transaction — if it passes now, your fix works against real state)

### Reference Files to Read
- `references/interactive-elements.md` → "Group Chat Animation", "Code ↔ English Translation Blocks", "Numbered Step Cards", "Multiple-Choice Quizzes", "Glossary Tooltips", "Callout Boxes"
- `references/content-philosophy.md` → full doc
- `references/gotchas.md` → full doc

### Connections
- **Previous module:** Samba: MEV at Wire Speed — showed how the pipeline is modified for MEV
- **Next module:** The Bigger Picture — shows how all 3 repos relate to each other and the Solana ecosystem
- **Tone/style notes:** Accent teal. Delorean=actor-3 (plum). Module uses --color-bg-warm (odd module). Penrose is currently in alpha — mention it lightly. ELF is a binary executable format — tooltip it. SVM = Solana Virtual Machine — tooltip it.
