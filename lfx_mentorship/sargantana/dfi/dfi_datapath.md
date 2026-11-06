# Sargantana DFI Datapath Modifications

## 1. Overview

- Scope and reference to the DFI policy.
- Diagram of modified modules and DFIC connections.
- Summary of interfaces.

## 2. Decode
### Prefix Recognition and Payload Extraction

Recognize a DFI prefix in the custom0 opcode space using a U-format field layout.

| Instruction bits | Meaning                                                                             |
| ---------------- | ----------------------------------------------------------------------------------- |
| [31:30]          | Reserved; must be zero                                                              |
| [29:28]          | Operation class: load, store, or atomic; numeric assignments remain to be specified |
| [27:12]          | 16-bit identifier                                                                   |
| [11:7]           | rd; must be x0                                                                      |
| [6:0]            | custom0 opcode                                                                      |

The identifier selects an RDSet for a protected load and supplies a definition ID for a protected store.
#### Relevant signals:

- `instr_type`: extend with a distinct DFI-prefix instruction type.
- `imm`: carry the payload through the existing U-immediate path. Extract the identifier from \[27:12] and class from \[29:28]; the lower twelve zero bits and RV64 sign extension have no DFI meaning.
- `xcpt_illegal_instruction_int`: assert for invalid prefix encodings.
#### Condition:

`prefix_encoding_valid =`
    `custom0_opcode`
    `&& (instruction[11:7] == 0)`
    `&& (instruction[31:30] == 0)`
    `&& supported_class`

The atomic class is reserved by the policy. Whether the initial implementation rejects it remains an explicit decision.

### Register and Execution Controls

The prefix reads and writes no architectural registers and has no independent ordinary data-memory side effect.
#### Relevant signals:

- `use_rs1, use_rs2, use_fs1, use_fs2, use_fs3`: clear.
- `use_vs1, use_vs2, use_mask, use_old_vd`: clear.
- `regfile_we, fregfile_we, vregfile_we`: clear.
- `mem_type`: classify the prefix itself as having no ordinary memory access.

Payload bits overlapping conventional register fields must not create operand dependencies. The prefix’s dispatch and completion handling will be defined at register read / GL allocation.

### Pairing Boundary

Decode preserves the prefix identity, payload, and PC for downstream pairing. Validation of sequential adjacency, the following instruction’s operation class, and joint completion belongs to the pairing mechanism. The prefix and memory instruction remain separate pipeline entries while representing one architectural protected operation.

## 3. Register Read / GL Allocation
### DFI Request Generation

Generate one DFI request when a custom0 prefix successfully allocates a Graduation List entry for DFI processing. DFIC allocates a token from its free pool and captures the request at the same event. A prefix carrying an attached interrupt instead uses the ordinary allocation-exception path with token 0 and no DFI request; see [Exceptions and Interrupts](#7-exceptions-and-interrupts).

Tokens are six bits wide. Token `0` is reserved for normal operations including unprotected memory operations; values `1..63` identify protected memory operations. The prefix and its associated memory instruction share one token, distinct from the static 16-bit identifier. DFIC returns the token to the free pool after successful DFI processing and release of outstanding references. Pair association is described below; retirement and recovery handling are described in the later sections.

Use separate structures for the free-list offer from DFIC and the request sent to DFIC:

```systemverilog
typedef struct packed {
    logic       valid;
    dfi_token_t token;
} dfi_alloc_t;

typedef struct packed {
    logic       valid;
    dfi_token_t token;
    bus64_t     imm;
    gl_index_t  gl_index;
} dfi_req_t;

dfi_alloc_t dfi_alloc; // DFIC free list -> datapath
dfi_req_t   dfi_req;   // Datapath -> DFIC
```

`dfi_alloc.valid` indicates that the DFIC free list offers a usable nonzero token in `dfi_alloc.token`. `dfi_req.imm` carries the prepared U-immediate; DFIC extracts the identifier from `[27:12]` and request type from `[29:28]`.
#### Relevant signals:

- `alloc_fire_o`: proposed GL output exposing its internal write_enable; indicates accepted allocation.
- `stage_ir_rr_q.instr`: source of PC, instruction type, memory class, and prefix immediate for checks performed in parallel with register reads.
- `stage_rr_exe_d.instr`: prepared instruction metadata, including the selected token; source of the DFI request payload.
- `stage_rr_exe_d.gl_index`: prefix GL index supplied by GL's existing `assigned_gl_entry_o` connection.
- `dfi_alloc`: valid/token offer from the DFIC free list, available before the allocation edge.
- `dfi_req`: valid/token/imm/gl_index request sent to DFIC on accepted prefix allocation.
#### Condition:

`dfi_req.valid =`
    `alloc_fire_o && (stage_rr_exe_d.instr.instr_type == DFI_PREFIX) && !attach_interrupt`
#### Request payload:

- `dfi_req.token = stage_rr_exe_d.instr.dfi_token`.
- `dfi_req.imm = stage_rr_exe_d.instr.imm`.
- `dfi_req.gl_index = stage_rr_exe_d.gl_index`; DFIC retains this prefix location alongside the token for completion reporting.

Generate the request from the allocation event, not the RR–EX register’s held valid bit. Stalls therefore do not generate duplicate requests, and allocations suppressed by a flush generate no request.

Read the combinational `_d` metadata to form the request in the allocation cycle; do not delay request generation until the registered `_q` output. Prefix recognition, pair checks, token selection, and pending next-state preparation start from `stage_ir_rr_q.instr` in parallel with operand reads. Accepted GL allocation qualifies request capture and the pending-state update at the clock edge.

For normal DFI admission, prefix allocation, token reservation, and request capture must succeed together. Require `dfi_alloc.valid` for that admission; also backpressure allocation if DFIC cannot capture the request. A prefix carrying an attached interrupt bypasses these DFI capacity requirements, reserves no token, and creates no `dfi_pending_q` association; it still requires GL capacity and ordinary pipeline acceptance. The `dfi_alloc` structure contains only valid and token; request-capacity backpressure remains part of the interface handling. A later token reply is not required: DFIC offers the token before the accepting edge.

DFIC holds the assigned value on `dfi_alloc.token` until it accepts the pair outcome for that prefix. The datapath also captures the token in `dfi_pending_q.token` and uses that pending copy when tagging the suffix. Successful and malformed pair outcomes both acknowledge the held token; advancing the offered token does not release the previous token back to the free pool.

Attach a proposed `dfi_token_t dfi_token` field to `instr_entry_t`, with `dfi_token_t` defined as `logic [5:0]` in `drac_pkg.sv`. Initialize it to zero before allocation and assign the selected token to `stage_rr_exe_d.instr.dfi_token`. Add the field separately to `gl_instruction_t`; drive `instruction_decode_gl.dfi_token` from that same prepared `_d` token so GL and RR–EX capture identical metadata. Preserve the existing separate GL and execution valid / exception controls. The opcode enum `instr_type_t` remains unchanged by this metadata addition.


### Pair Association and Pending State

Keep one pending pairing structure in datapath. It records the allocated prefix while waiting for the next instruction's association outcome; it is not an additional instruction queue or a GL tracking list.

```systemverilog
typedef struct packed {
    logic       valid;
    logic [1:0] req_type;
    dfi_token_t token;
    addrPC_t    pc;
} dfi_pending_t;

dfi_pending_t dfi_pending_d, dfi_pending_q;
```

- `valid`: a prefix is waiting for its suffix to be associated.
- `req_type`: the prefix's requested operation class, taken from instruction bits `[29:28]`. Keep two bits for later class extensions; the baseline associates supported scalar loads and stores, with SIMD and atomic protection deferred.
- `token`: the nonzero token assigned to that prefix, latched rather than reread from a later free-list offer.
- `pc`: the prefix PC, used to require the suffix at `pc + 4`.

`dfi_pending_q` is the current registered state. `dfi_pending_d` is its combinational next value, prepared in parallel with register reads. Start with `dfi_pending_d = dfi_pending_q`; hold the state across stalls and instruction bubbles. The register captures `_d` at the clock edge:

```systemverilog
always_ff @(posedge clk_i or negedge rstn_i) begin
    if (!rstn_i)
        dfi_pending_q <= '0;
    else
        dfi_pending_q <= dfi_pending_d;
end
```

On accepted normal prefix allocation (`dfi_req.valid`), set `dfi_pending_d.valid` and capture its request class, prepared token, and PC. A prefix carrying an attached interrupt does not create pending state, as described above.

When `dfi_pending_q.valid` is set, first check the next allocation candidate from `stage_ir_rr_q.instr` against the pending prefix: its PC must equal `dfi_pending_q.pc + 4`, and its supported instruction type must match `dfi_pending_q.req_type`. Checking only `mem_type` is insufficient to exclude deferred SIMD or atomic operations. Perform these checks alongside RR; accepted allocation qualifies the state change and pair-outcome notification, so a held candidate does not generate repeated outcomes.

On successful association, copy `dfi_pending_q.token` into `stage_rr_exe_d.instr.dfi_token` and `instruction_decode_gl.dfi_token` before their accepting edge. The memory instruction therefore enters execution and GL with the same token as its prefix. Report successful association to DFIC and clear the pending valid bit when the outcome is accepted. This acknowledges the held token offer; it does not free the operation's token.

On a malformed association, leave the candidate's DFI metadata unchanged and let it follow its ordinary instruction path. Report `INVALID_SUFFIX` with `dfi_pending_q.token` to DFIC, which uses its saved prefix GL index to complete custom0 with the malformed-pair exception. Clear the pending valid bit when that outcome is accepted. The datapath does not retrieve or modify the prefix directly in GL.

Reset clears the structure. Recovery clears pending state when its owning prefix is cancelled; a surviving prefix retains its pending association. Flush cancellation takes priority over capturing state for a discarded allocation.

## 4. LSQ / Memory Execution
### Physical-Address Reporting

Report a protected scalar operation's translation outcome using a separate `dfi_pa` channel. A nonfaulting outcome supplies the physical address; a faulting outcome supplies the original exception instead. This updates the DFIC record already identified by the token; it does not allocate another token or repeat the prefix request.

Use a single pipeline register between translation and DFIC. The baseline assumes DFIC consumes one registered report every cycle, independently of the metadata-memory operation currently being processed. There is no ready signal, holding protocol, or additional LSQ backpressure for this interface.

`translated_instr` is the common source for both incoming and queued translations. The current LSQ selects exactly one `instr_to_translate` and has one DTLB request / response path, so it produces at most one new translation outcome per cycle (`n = 1`). DFIC consumes the previous registered report while the register captures the next one. Older translated instructions may issue to memory concurrently; this does not create a second new translation outcome.

`SARG_BYPASS_LSQ` is an optional empty-queue optimization for loads. When an incoming load can be translated immediately, `next_instr_exe_o` can take `translated_instr` directly rather than storing and rereading it through `control_table`. The bypass requires `translate_enable`; it does not skip PA generation or fault handling, and is not store-to-load data forwarding.

```systemverilog
typedef struct packed {
    logic       valid;
    dfi_token_t token;
    bus64_t     pa;
    logic [3:0] mem_size;
    exception_t ex;
} dfi_pa_t;

dfi_pa_t dfi_pa_d, dfi_pa_q;
```

`dfi_pa_q` is the registered output to DFIC. With `valid == 1` and `ex.valid == 0`, PA and size describe the access. With `valid == 1` and `ex.valid == 1`, DFIC ignores PA, does not start an RDTable lookup from it, and processes the original exception. `mem_size` retains the core encoding, from which DFIC derives the byte width and covered four-byte granules. An aligned eight-byte access requires two RDTable labels.
#### Relevant signals:

- `translated_instr.instr.dfi_token`: inherited protected-operation token; zero requires no PA report.
- `translated_instr.data_rs1`: physical address assembled from the DTLB PPN and page offset.
- `translated_instr.instr.mem_size`: original access-size encoding.
- `translated_instr.ex`: original translation / alignment exception, including cause and fault information.
- `translate_enable`: existing translation-available indication, including DTLB readiness and absence of a miss.
- `translate_incoming`, `num_to_translate`: distinguish a real incoming or queued instruction from stale queue contents.
- `translation_fire`: translation-progress event qualifying the report; no new stall or allocation signal is required.
- `flush_i`: suppresses acceptance of discarded translation work.
#### Conditions:

- `translation_fire = translate_enable && (translate_incoming || num_to_translate != 0) && !flush_i`
- `dfi_pa_d.valid = translation_fire && (translated_instr.instr.dfi_token != 0)`
- `dfi_pa_d.token = translated_instr.instr.dfi_token`
- `dfi_pa_d.pa = translated_instr.data_rs1`
- `dfi_pa_d.mem_size = translated_instr.instr.mem_size`
- `dfi_pa_d.ex = translated_instr.ex`

On each clock edge, capture `dfi_pa_d` into `dfi_pa_q`. When there is no qualifying protected translation, capture `valid = 0`; do not retain or resend the previous valid report. Reset clears the register. Applicable recovery suppresses discarded reports. The existing LSQ translation writes, counters, pointers, and bypass behavior do not need additional backpressure gating under the always-consuming DFIC assumption.

The report event must cover both queued translations and directly accepted incoming instructions, including `SARG_BYPASS_LSQ`; watching only writes to `control_table` would miss bypassed operations. DTLB misses generate no report; a later accepted translation generates one address or fault report for the supported scalar operation.

On a reported translation fault, DFIC uses the token to identify the operation, ignores the PA, and does not start an RDTable lookup from it. The report lets DFIC resolve or terminate its work without copying the ordinary fault into custom0. The memory instruction retains its original exception path: loads report through GL, while stores use the memory-commit exception channel. Retirement selects the pair's exception with custom0 taking priority and attributes the selected fault to the prefix PC. Preserve the original cause and fault information.

Faults detected after translation require a later memory-completion notification. They do not come from this translation-only tap and must not be mistaken for another newly translated PA.

Recovery invalidates registered reports belonging to discarded operations. If DFIC already consumed a report, cancel the corresponding speculative work there. A token remains owned until outstanding references have drained; a report must not be applied to a reused token.

PA reporting establishes address availability, not memory completion or retirement. Loads may begin RDTable lookup, subject to the later data/label-ordering contract. Stores retain the address for their eventual label update; address translation does not authorize an RDTable write.

## 5. Graduation List and Retirement
### GL Entry Handling

Pair association belongs to the datapath. GL stores custom0 and its associated memory instruction as separate entries carrying the shared token; no additional per-entry DFI tracking state is required. Keep the existing GL head-output logic unchanged.

On recovery, GL reports the tokens of discarded prefixes using a 64-bit cancellation mask. See Flush and Recovery for mask generation and DFIC handling.

DFIC establishes custom0's own final exception state **PRIOR to setting its `valid_bit_q` readiness bit**. Ordinary memory exceptions remain on the memory instruction's existing paths; DFIC does not copy them into custom0. Retirement selects one exception for the pair, with custom0's exception taking priority.

- DFIC reports prefix-side faults, including DFI-check failure or malformed pairing, against custom0. A faulting prefix can reach exception handling without waiting for the suffix to complete.
- A ready, nonexceptional prefix means that its own processing is settled; it does not independently authorize retirement or certify that the memory instruction succeeded.
- The memory instruction retains its existing readiness and exception mechanism: loads become ready through normal writeback, while stores are marked ready at allocation and undergo their existing memory-commit processing at retirement.
- A load's ordinary exception remains in GL; a store's commit exception reaches retirement through the existing memory-commit exception channel.
- Normal execution writeback must not independently complete custom0. Retirement holds a nonexceptional prefix until its associated memory operation's outcome permits either a pair exception or joint successful retirement.

Relevant existing signals:
- `valid_bit_q`: per-entry readiness, controlled by DFIC for custom0.
- `entries_q[index].ex_valid`: prefix exception flag established before it is exposed as ready.
- `exception_q`, `exception_index_q`: retained exception details and owning GL entry; DFIC's prefix-fault update participates in this existing mechanism.
- `instruction_o`: existing ready-qualified head entries supplied to retirement.

### Default custom0 Retirement Block

Implement the retirement decisions in `csr_interface.sv`. custom0 cannot retire from slot 1, including when it carries an exception: an older instruction may retire alone, moving custom0 to slot 0. Add the slot-1 `DFI_PREFIX` condition to the existing `commit_2_blocked` expression.

In slot 0, preserve the existing head-exception branch first. Then handle custom0 in a dedicated branch before ordinary retirement: select any associated memory exception, otherwise apply the load- or store-pair release condition below. A nonexceptional custom0 remains blocked until one of those conditions succeeds.

Relevant signals:
- `instruction_to_commit_i[0]`, `[1]`: GL head entries visible to `csr_interface`.
- `instruction_to_commit_i[0].instr_type`, `[1].instr_type`: identify custom0 using the proposed `DFI_PREFIX` instruction type.
- `commit_2_blocked`: existing restriction on dual retirement; extend it to block custom0 in slot 1.
- `commit_xcpt_i`: existing commit-exception indication; extend its generation to include the selected pair exception, and handle it before successful pair release.
- `retire_inst_o`: mask returned as `retire_inst_gl` to GL's `read_head_i`.
- `req_cpu_csr_o.csr_retire`: successful-retirement mask sent to CSR logic.

With a blocked prefix in slot 0, both masks remain `2'b00`. For a head exception, keep the existing behavior: `retire_inst_o = 2'b01` removes the exceptional head internally, while `req_cpu_csr_o.csr_retire = 2'b00`; this is exception handling, not successful retirement. Recovery discards the remaining pair entry and younger work.

### Pair Exception Selection

Resolve exceptions for both protected loads and protected stores at retirement. This replaces ordinary-memory-exception inheritance through DFIC. Prefix-side faults still use DFIC's prefix GL update; no retirement-time copy back into GL is required.

Priority and actions:

1. If slot 0 custom0 carries an exception, take that exception immediately. It wins over any exception from its suffix; a malformed prefix does not require a valid matching suffix to trap.
2. Otherwise, for a valid associated suffix with the same nonzero token, select its ordinary memory exception once custom0's own outcome is settled. Loads supply their GL exception; stores supply their existing memory-commit exception.
3. If neither source faults, apply the relevant pair-release condition. Unresolved memory processing keeps both entries blocked.

For a selected suffix exception, preserve the complete original `exception_t`, including cause, `origin`, `origin2`, `tinst`, and `gva`; use custom0's PC for exception / restart attribution. Either exception path removes the exceptional head internally with `retire_inst_o = 2'b01`, leaves `req_cpu_csr_o.csr_retire = 2'b00`, and relies on recovery to discard the suffix and younger instructions. Neither component retires successfully.

Relevant existing signals and required routing:

- `instruction_to_commit_i[0].ex_valid`, `[1].ex_valid`: prefix and load-suffix exception flags; suffix selection requires confirmed pair identity.
- `ex_gl_out_int` / `exception_gl_i`: GL's retained full exception record, supplying the selected prefix or load exception.
- `exception_mem_commit_int` / `exception_mem_commit_i`: existing store-commit exception channel. Qualify a suffix-store exception against the confirmed pair and the returning store's GL index (`mem_gl_index_int`).
- `commit_xcpt`, `commit_xcpt_cause`: update datapath's commit-exception selection together with `csr_interface`'s exception-payload selection. Control-unit recovery and CSR trap reporting must use the same selected exception.
- `req_cpu_csr_o.csr_pc`: already uses slot 0's PC, which is custom0's PC for the pair.

GL retains one full exception in `exception_q`, with its owner in `exception_index_q`, rather than separate full payloads for both slots. Prefix-first priority agrees with retaining the older fault: if both prefix and load fault, retain the prefix's record; otherwise retain the load's record. DFIC-versus-WB arbitration must preserve this rule, including simultaneous reports and GL-index wraparound. Check that the retained payload belongs to the selected entry; a second per-entry exception payload is not required by this priority rule.

An ordinary memory fault must also let DFIC resolve or terminate any now-inapplicable metadata work so custom0 cannot remain unready forever. DFIC may consume fault notifications for that purpose, but does not reclassify the memory fault as a prefix exception. A nonexceptional prefix may release its suffix's fault only after its own outcome is settled; an already-established prefix fault wins. Detailed cancellation and recovery handling remains separate.

### Load-Pair Retirement

Successful retirement always removes custom0 and its associated load together in one cycle. custom0 must be in slot 0 and its ready load in slot 1. Their shared nonzero token identifies the pair; sequential PC and request-class validation has already happened during association.

Relevant signals:
- `instruction_to_commit_i[0].valid`, `[1].valid`: both entries are ready and exposed by GL.
- `instruction_to_commit_i[0].dfi_token`, `[1].dfi_token`: proposed shared token metadata used to confirm association.
- `instruction_to_commit_i[1].mem_type`: identifies the suffix as a `LOAD`.
- `instruction_to_commit_i[0].ex_valid`, `[1].ex_valid`, `commit_xcpt_i`: prefix and load exception state checked by the common pair-exception decision before successful release.
- `mem_commit_stall_i`: preserve the existing retirement stall condition.

Successful load-pair release condition (a proposed combinational predicate in `csr_interface`, not additional GL state):

```systemverilog
load_pair_ready =
    instruction_to_commit_i[0].valid &&
    (instruction_to_commit_i[0].instr_type == DFI_PREFIX) &&
    instruction_to_commit_i[1].valid &&
    (instruction_to_commit_i[1].mem_type == LOAD) &&
    (instruction_to_commit_i[0].dfi_token != '0) &&
    (instruction_to_commit_i[0].dfi_token ==
     instruction_to_commit_i[1].dfi_token) &&
    !instruction_to_commit_i[0].ex_valid &&
    !instruction_to_commit_i[1].ex_valid &&
    !commit_xcpt_i &&
    !mem_commit_stall_i;
```

After pair-exception handling, a successful condition inside the slot-0 custom0 branch sets both `retire_inst_o` and `req_cpu_csr_o.csr_retire` to `2'b11`. Otherwise keep both at `2'b00`. GL already consumes both retire bits and advances past both entries; no change to its head-selection logic is required.

| Head situation | Action |
|---|---|
| custom0 has a final exception | Existing head-exception path; neither instruction retires successfully. |
| Nonexceptional custom0 and matching faulting load | Select the load exception at retirement, attributed to custom0's PC; no successful retirement. |
| Nonexceptional custom0 and matching ready load; neither faults and no retirement stall | Retire both in the same cycle with `2'b11`. |
| Matching load not ready, missing, or pair identity not established | Keep custom0 blocked. |

The load keeps its own ordinary exception in GL. The common pair-exception decision selects that fault only when custom0 has no exception; DFIC does not mirror the load exception into the prefix.

### Store-Pair Retirement

Use the existing store memory-commit process for the suffix in slot 1, while custom0 remains in slot 0. `commit_store_or_amo_int[1]` already supplies slot-1 commit permission to the memory unit. With a prefix in slot 0, `commit_cu_int.gl_index` already selects the slot-1 GL index.

Keep the ordinary store-retirement path unchanged. In the dedicated custom0 branch, first confirm the suffix is a `STORE` with the same nonzero token and apply the common pair-exception decision. For a nonfaulting pair, reuse the existing store wait and matching-index checks for slot 1:

```systemverilog
// Proposed pair-specific wait, computed in datapath.
pair_store_wait =
    mem_commit_stall_int |
    (commit_store_or_amo_int[1] &
     ((commit_cu_int.gl_index != mem_gl_index_int) |
      !mem_commit_store_or_amo_int));
```

Supply this pair-specific wait to `csr_interface`; its existing `mem_commit_stall_i` alone does not include the slot-1 matching-index guard. While the store is waiting, keep both retirement masks at `2'b00`. When the matching store satisfies the existing commit condition and neither component has a selected exception, set both masks to `2'b11`. This pair-specific branch bypasses the ordinary `commit_2_blocked` restriction on a slot-1 store only for the confirmed protected pair and preserves other applicable retirement stalls.

A store commit exception has priority over successful pair release, even when custom0's readiness bit was set earlier. The exception is selected directly from the memory-commit channel; there is no DFIC exception-copy round trip. Preserve existing store-commit semantics; RDTable-update commitment and detailed store recovery are separate topics.

## 6. Exceptions and Interrupts

### Interrupt Attachment at GL Allocation

The CSR block `csr_bsc` produces `resp_csr_cpu_i.csr_interrupt` and `resp_csr_cpu_i.csr_interrupt_cause`. Attachment happens in datapath during RR / GL allocation, rather than in `csr_interface`: the current RTL marks the allocating instruction exceptional, supplies its exception payload to GL, and suppresses its execution. The interrupt is taken when that instruction reaches the retirement exception path.

Allow attachment to an ordinary instruction or a custom0 prefix, but not to the successfully associated protected memory suffix. A prefix represents a safe boundary before the protected operation; its PC is the restart PC. Blocking every nonzero-token instruction would also block prefixes and could indefinitely defer an interrupt in a stream consisting only of protected pairs.

Use the token prepared for the current allocation candidate, before the clock edge. The incoming `stage_ir_rr_q.instr` suffix has not yet received its token; successful association selects `dfi_pending_q.token` for the combinational metadata sent to both GL and RR–EX.

Proposed combinational helpers, not existing RTL signals:

```systemverilog
// Initial protected memory scope: LOAD and STORE.
candidate_is_memory =
    (stage_ir_rr_q.instr.mem_type == LOAD) ||
    (stage_ir_rr_q.instr.mem_type == STORE);

protected_suffix = candidate_is_memory && (prepared_dfi_token != '0);
attach_interrupt = resp_csr_cpu_i.csr_interrupt && !protected_suffix;
```

`prepared_dfi_token` denotes the token selected for the candidate before interrupt handling: the pending token on successful suffix association, an offered token for normal prefix admission, or zero for an unprotected instruction. Derive this selection from the candidate and pending state independently of the interrupt-modified execution `valid` / `ex_valid` signals, avoiding a combinational dependency loop. If an interrupt attaches to a prefix, override its final GL and RR–EX token to zero and skip DFI admission.

Relevant existing signals and required changes:

- `resp_csr_cpu_i.csr_interrupt`, `.csr_interrupt_cause`: CSR interrupt eligibility and cause; keep the original CSR pending / enable logic.
- `stage_ir_rr_q.instr.mem_type`, `.instr_type`, `.pc`, and `dfi_pending_q.token`: identify the current candidate and select its prepared token. `dfi_pending_q` and the token fields are proposed additions from the allocation design.
- `instruction_decode_gl.ex_valid`, `ex_gl_in_int`, and `interrupt_ex.valid`: use the filtered `attach_interrupt` for interrupt attachment. Preserve the existing precedence of `stage_ir_rr_q.ex` over an interrupt when selecting the GL exception payload; `interrupt_ex.cause` still comes from `.csr_interrupt_cause`.
- `stage_rr_exe_d.instr.valid`, `.ex_valid`: use the same filtered interrupt signal in their existing interrupt-dependent controls. Masking GL attachment alone would leave execution disabled for the protected suffix.
- `instruction_decode_gl.exception`: update the interrupt selection under `SIM_COMMIT_LOG` consistently with the functional GL exception path.
- `instruction_decode_gl.dfi_token`, `stage_rr_exe_d.instr.dfi_token`: capture the same final token. A prefix carrying an attached interrupt uses zero; a protected suffix retains its associated nonzero token.
- `alloc_fire_o`: accepted GL allocation qualifies capture. Do not treat interrupt eligibility during a stall or flush as an accepted allocation or a DFI state update.

For a prefix carrying an attached interrupt, reserve no DFIC token, issue no `dfi_req`, and create no pending suffix association. Bypass DFI token / request-capacity stalls for this candidate. GL's existing allocation exception handling makes it ready without waiting for DFIC; preserve the slot-1 custom0 retirement block so it takes its exception from slot 0. Recovery restarts execution at the prefix PC.

For a protected suffix, suppress only interrupt attachment. Preserve its ordinary exceptions and normal execution, pairing, and retirement behavior. A still-pending interrupt can attach to a subsequent ordinary instruction or prefix; it cannot split successful pair retirement.

### Deferred Interrupt Eligibility

Deferring attachment does not acknowledge or clear an interrupt. `csr_bsc` recomputes eligibility from pending sources, enable bits, and privilege state; `csr_interrupt` is not a ready/valid event held until allocation accepts it. The interrupt remains eligible while its source remains pending and enabled, but the current output is not guaranteed to remain asserted after its source disappears. Preserve this existing pending-source behavior rather than assuming an independent sticky request queue.

## 7. Flush and Recovery

### GL Token Cancellation Mask

Keep flush-boundary calculations inside GL and report discarded protected operations to DFIC by token. A GL index identifies a current occupied slot; it can be reused while old cancelled DFIC work is still draining. The token identifies the operation across that reuse.

Add a proposed GL output:

```systemverilog
logic [63:0] dfi_kill_mask_o;
```

Each bit represents its token value: `dfi_kill_mask_o[t] == 1` cancels the protected operation with token `t`. Six-bit tokens require 64 mask bits. Bit 0 remains clear because token 0 denotes an unprotected instruction. An all-zero mask means no cancellations. Multiple set bits report multiple victims together; for example, cancelling tokens 7 and 3 produces `64'h0000_0000_0000_0088`. DFIC checks the set-bit positions rather than decoding a list of packed token values.

Generate the mask from occupied prefix entries being discarded by the existing recovery event:

- `flush_i` with `flush_index_i`: cancel prefixes younger than the boundary instruction. The boundary instruction itself survives; the current GL sets its new tail to `flush_index_i + 1`.
- `flush_commit_i`: cancel every prefix still occupying GL. This takes priority over a simultaneous partial flush, as in the existing GL recovery logic.
- No GL flush: drive the mask to zero. A front-end-only redirect does not by itself cancel an allocated prefix.

Use `head` and `num` to identify occupied entries, and compare age in circular GL order for partial recovery. Do not use a plain numerical `index > flush_index_i` comparison across wraparound. Do not use `valid_bit_q` as an occupancy mask: it represents readiness, and unready prefixes must also be cancelled. Ignore stale contents outside the occupied range.

For each discarded `DFI_PREFIX` with a nonzero token, set the corresponding mask bit. Inspect prefixes only; their associated memory instructions share the token and do not need separate notifications. Capture the victim mask from the pre-flush entries at the recovery edge, before their slots can be reused. DFIC must receive the complete mask, including every set bit, for each recovery event.

### Cancellation and Token Lifetime

DFIC marks all indicated operations killed and prevents their results from completing a GL entry or authorizing speculative RDTable updates. Cancellation wins over simultaneous PA reports or DFI completions for the same killed token; surviving operations continue normally. Suppress killed completions at the GL update boundary as well, so a late result cannot modify a reused slot.

A cancellation notification does not immediately return a token to the free pool. Retain ownership until outstanding processor and metadata references have drained or have been definitively cancelled. Consume and discard late results belonging to the killed operation before token reuse. Already-retired store metadata updates survive speculative recovery independently of discarded GL entries.

If a cancelled prefix owns `dfi_pending_q`, clear that pending association; preserve the pending state of a surviving prefix. Apply the same token cancellation to registered PA reports for discarded operations.

A faulting prefix removed during exception handling needs a separate terminal notification carrying its identity. The subsequent full-flush mask covers only prefixes still occupying GL and cannot rediscover that removed entry. Successful retirement likewise remains a separate operation-lifetime event, not a cancellation.
