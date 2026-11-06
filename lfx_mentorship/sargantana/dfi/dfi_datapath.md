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

Generate one DFI request when a custom0 prefix successfully allocates a Graduation List entry. DFIC allocates a token from its free pool and captures the request at the same event.

Tokens are six bits wide. Token `0` is reserved for normal operations including unprotected memory operations; values `1..63` identify protected memory operations. The prefix and its associated memory instruction share one token, distinct from the static 16-bit identifier. DFIC returns the token to the free pool after successful DFI processing and release of outstanding references. Pair association is described below; retirement and detailed recovery handling remain to be discussed.

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
    `alloc_fire_o && (stage_rr_exe_d.instr.instr_type == DFI_PREFIX)`
#### Request payload:

- `dfi_req.token = stage_rr_exe_d.instr.dfi_token`.
- `dfi_req.imm = stage_rr_exe_d.instr.imm`.
- `dfi_req.gl_index = stage_rr_exe_d.gl_index`; DFIC retains this prefix location alongside the token for completion reporting.

Generate the request from the allocation event, not the RR–EX register’s held valid bit. Stalls therefore do not generate duplicate requests, and allocations suppressed by a flush generate no request.

Read the combinational `_d` metadata to form the request in the allocation cycle; do not delay request generation until the registered `_q` output. Prefix recognition, pair checks, token selection, and pending next-state preparation start from `stage_ir_rr_q.instr` in parallel with operand reads. Accepted GL allocation qualifies request capture and the pending-state update at the clock edge.

Prefix allocation, token reservation, and request capture must succeed together. Require `dfi_alloc.valid` for prefix allocation; also backpressure allocation if DFIC cannot capture the request. The `dfi_alloc` structure contains only valid and token; request-capacity backpressure remains part of the interface handling. A later token reply is not required: DFIC offers the token before the accepting edge.

DFIC holds the assigned value on `dfi_alloc.token` until it accepts the pair outcome for that prefix. The datapath also captures the token in `dfi_pending_q.token` and uses that pending copy when tagging the suffix. Successful and malformed pair outcomes both acknowledge the held token; advancing the offered token does not release the previous token back to the free pool.

Attach a proposed `dfi_token_t dfi_token` field to `instr_entry_t`, with `dfi_token_t` defined as `logic [5:0]` in `drac_pkg.sv`. Initialize it to zero before allocation and assign the selected token to `stage_rr_exe_d.instr.dfi_token`. Add the field separately to `gl_instruction_t`; drive `instruction_decode_gl.dfi_token` from that same prepared `_d` token so GL and RR–EX capture identical metadata. Preserve the existing separate GL and execution valid / exception controls. The opcode enum `instr_type_t` remains unchanged by this metadata addition.

## 4. Graduation List and Retirement
### GL Entry Handling

Pair association belongs to the datapath. GL stores custom0 and its associated memory instruction as separate entries carrying the shared token; no additional per-entry DFI tracking state is required. Keep the existing GL head-output logic unchanged.

DFIC is responsible for establishing custom0's final exception state **PRIOR to setting its `valid_bit_q` readiness bit**. Ordinary memory exceptions are inherited by custom0 through DFIC; retirement does not copy exceptions from the memory instruction.

- For a load prefix, nonexceptional readiness means that the DFI check and the load's ordinary fault outcome are settled. A successful metadata check alone must not expose a ready prefix while its memory-exception state is unresolved.
- If the load faults, DFIC reports the original exception against the prefix. Preserve its cause and fault information; the prefix GL entry supplies the exception / restart PC.
- DFIC also reports DFI-check failure or malformed pairing against custom0. Exception-priority rules remain to be discussed.
- On any final fault, GL records the prefix's exception state and makes it ready for the existing exception-handling path. It need not wait for successful load completion.
- The memory instruction retains its existing readiness and exception mechanism: loads become ready through normal writeback, while stores are marked ready at allocation.
- Normal execution writeback must not independently complete custom0. Store-prefix readiness and retirement details will be discussed separately.

Relevant existing signals:
- `valid_bit_q`: per-entry readiness, controlled by DFIC for custom0.
- `entries_q[index].ex_valid`: prefix exception flag established before it is exposed as ready.
- `exception_q`, `exception_index_q`: retained exception details and owning GL entry; DFIC's prefix-fault update participates in this existing mechanism.
- `instruction_o`: existing ready-qualified head entries supplied to retirement.

### Default custom0 Retirement Block

Implement the retirement decisions in `csr_interface.sv`. custom0 cannot retire from slot 1, including when it carries an exception: an older instruction may retire alone, moving custom0 to slot 0. Add the slot-1 `DFI_PREFIX` condition to the existing `commit_2_blocked` expression.

In slot 0, preserve the existing head-exception branch first. A nonexceptional custom0 remains blocked unless an explicitly supported pair-release condition succeeds. Load-pair release is defined below; store-pair release remains to be discussed.

Relevant signals:
- `instruction_to_commit_i[0]`, `[1]`: GL head entries visible to `csr_interface`.
- `instruction_to_commit_i[0].instr_type`, `[1].instr_type`: identify custom0 using the proposed `DFI_PREFIX` instruction type.
- `commit_2_blocked`: existing restriction on dual retirement; extend it to block custom0 in slot 1.
- `commit_xcpt_i`: existing head-exception indication; handle before checking pair readiness.
- `retire_inst_o`: mask returned as `retire_inst_gl` to GL's `read_head_i`.
- `req_cpu_csr_o.csr_retire`: successful-retirement mask sent to CSR logic.

With a blocked prefix in slot 0, both masks remain `2'b00`. For a head exception, keep the existing behavior: `retire_inst_o = 2'b01` removes the exceptional head internally, while `req_cpu_csr_o.csr_retire = 2'b00`; this is exception handling, not successful retirement. Recovery discards the remaining pair entry and younger work.

### Load-Pair Retirement

Successful retirement always removes custom0 and its associated load together in one cycle. custom0 must be in slot 0 and its ready load in slot 1. Their shared nonzero token identifies the pair; sequential PC and request-class validation has already happened during association.

Relevant signals:
- `instruction_to_commit_i[0].valid`, `[1].valid`: both entries are ready and exposed by GL.
- `instruction_to_commit_i[0].dfi_token`, `[1].dfi_token`: proposed shared token metadata used to confirm association.
- `instruction_to_commit_i[1].mem_type`: identifies the suffix as a `LOAD`.
- `instruction_to_commit_i[0].ex_valid`, `commit_xcpt_i`: prefix exception state already established through DFIC / GL.
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
    !commit_xcpt_i &&
    !mem_commit_stall_i;
```

Inside the slot-0 custom0 branch, a successful condition sets both `retire_inst_o` and `req_cpu_csr_o.csr_retire` to `2'b11`. Otherwise keep both at `2'b00`. GL already consumes both retire bits and advances past both entries; no change to its head-selection logic is required.

| Head situation | Action |
|---|---|
| custom0 has a final exception | Existing head-exception path; neither instruction retires successfully. |
| Nonexceptional custom0 and matching ready load; no retirement stall | Retire both in the same cycle with `2'b11`. |
| Matching load not ready, missing, or pair identity not established | Keep custom0 blocked. |

Retirement performs no exception inheritance or late exception selection from slot 1. DFIC must complete that work before making custom0 ready. Store retirement and exception-priority decisions are outside this load rule.

## 5. LSQ / Memory Execution
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

`translation_fire = translate_enable && (translate_incoming || num_to_translate != 0) && !flush_i`

`dfi_pa_d.valid = translation_fire && (translated_instr.instr.dfi_token != 0)`

`dfi_pa_d.token = translated_instr.instr.dfi_token`

`dfi_pa_d.pa = translated_instr.data_rs1`

`dfi_pa_d.mem_size = translated_instr.instr.mem_size`

`dfi_pa_d.ex = translated_instr.ex`

On each clock edge, capture `dfi_pa_d` into `dfi_pa_q`. When there is no qualifying protected translation, capture `valid = 0`; do not retain or resend the previous valid report. Reset clears the register. Applicable recovery suppresses discarded reports. The existing LSQ translation writes, counters, pointers, and bypass behavior do not need additional backpressure gating under the always-consuming DFIC assumption.

The report event must cover both queued translations and directly accepted incoming instructions, including `SARG_BYPASS_LSQ`; watching only writes to `control_table` would miss bypassed operations. DTLB misses generate no report; a later accepted translation generates one address or fault report for the supported scalar operation.

On a reported translation fault, DFIC uses the token to locate its stored prefix GL index and returns an ordinary-memory-fault completion for custom0, analogously to malformed-pair reporting. The memory instruction retains its ordinary exception path. Both entries are intended to reflect the pair's failure, with one architectural trap attributed to the prefix PC. Preserve the original exception cause and fault information; the internal DFIC status encoding and exact GL updates remain to be discussed. In particular, current store faults use a separate commit exception path, so marking a store's GL entry is not already automatic.

Faults detected after translation require a later memory-completion notification. They do not come from this translation-only tap and must not be mistaken for another newly translated PA.

Recovery invalidates registered reports belonging to discarded operations. If DFIC already consumed a report, cancel the corresponding speculative work there. A token remains owned until outstanding references have drained; a report must not be applied to a reused token.

PA reporting establishes address availability, not memory completion or retirement. Loads may begin RDTable lookup, subject to the later data/label-ordering contract. Stores retain the address for their eventual label update; address translation does not authorize an RDTable write.

## 6. Store Completion

## 7. Exceptions and Interrupts

## 8. Flush and Recovery

## 9. Interface Summary


