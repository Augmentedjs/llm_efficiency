# LLM Improvement and Efficiency Through Deterministic Offload

## Executive summary

The objective of this approach is not merely to make individual LLM
calls faster or cheaper. It is to explore application architectures that
systematically reduce how much **model capability, context, generation,
and reasoning** are required to solve a problem successfully.

The central strategy is to assign each kind of work to the part of the
system best suited to perform it. Deterministic software selects
relevant context, enforces rules, manages state, validates results,
renders repeatable outputs, and performs mechanical transformations. The
LLM is reserved for the smaller set of decisions that genuinely benefit
from language understanding, ambiguity resolution, synthesis, or
semantic reasoning.

This changes the optimization question from:

> How can we make the model perform this entire workflow more
> efficiently?

to:

> What is the minimum amount of probabilistic semantic computation
> required to solve this application task correctly?

Reducing that semantic responsibility can reduce inference time, token
usage, computational requirements, model-size requirements, operating
expense, and implementation complexity while increasing predictability,
testability, reproducibility, and control.

The goal is therefore not simply to use a smaller model. It is to
**design the application so that it requires less AI computation to
produce each useful result**.

## Architectural thesis

> **LLMs should be semantic engines, not application engines.**

The governing rule is:

> **Ask the LLM for the smallest semantic decision that cannot be
> produced deterministically. Assemble, validate, render, and persist
> everything else in code.**

This can be summarized more simply for a general audience:

> **Ask the AI to do less, so it can do the important part better.**

The approach treats model capability as a resource that should be spent
deliberately. If an operation can be calculated, looked up, validated,
transformed, or rendered exactly by software, there is usually little
value in asking a probabilistic model to perform it.

## Research hypothesis

The architectural hypothesis is:

> **As deterministic software takes responsibility for work that does
> not require semantic judgment, the amount of computation required from
> the LLM should decrease while overall system reliability and output
> quality remain constant or improve.**

This creates several measurable expectations:

  Expected reduction                   Quality or control to preserve/improve
  ------------------------------------ ----------------------------------------
  Input context and tokens             Relevance and grounding
  Generated output and tokens          Acceptance without editing
  Reasoning workload                   Semantic decision quality
  Number of inference calls            End-to-end task completion
  Required model capability / size     Accuracy and usefulness
  Inference latency                    User-perceived responsiveness
  Compute and operating expense        Reliability
  Probabilistic application behavior   Determinism and reproducibility

A particularly important proposition is that **architecture changes the
minimum model capability required for a task**. A workflow that appears
to require a large model when the model owns the entire process may be
solvable by a substantially smaller model when context selection, state,
validation, orchestration, and rendering are moved into deterministic
software.

Model-size reduction should therefore be treated as an **experimental
result**, not an assumption.

## Minimum Semantic Computation

For discussion purposes, this document refers to the target of this
optimization as **Minimum Semantic Computation (MSC)**:

> **The minimum amount of probabilistic semantic computation required to
> produce an acceptable application result.**

MSC is not necessarily the smallest possible model or the fewest
possible tokens. The useful minimum is determined by the complete system
outcome. An optimization that reduces tokens but increases retries,
manual correction, unsupported conclusions, or rejected results has not
necessarily reduced the true cost of the task.

The more meaningful unit is therefore often:

> **Compute, cost, or latency per accepted result---not merely per model
> call.**

The App / Domain API, Semantic Pipeline, and Templating are
architectural mechanisms for moving the system toward this minimum while
retaining explicit quality gates.

## Visual model of the approach

The architecture can be understood as a deliberate narrowing of the
probabilistic portion of the system.

``` mermaid
flowchart LR
    A["Application problem"] --> B["Deterministic context selection"]
    B --> C["Small semantic problem"]
    C --> D["LLM"]
    D --> E["Minimal semantic result"]
    E --> F["Deterministic validation"]
    F --> G["Domain assembly"]
    G --> H["Semantic Pipeline"]
    H --> I["Templating"]
    I --> J["Durable result"]

    B -. "rules, state, retrieval" .-> F
```

The important boundary is not between "AI software" and "traditional
software." It is between work that requires **probabilistic semantic
judgment** and work that can be performed **deterministically**.

### Responsibility map

``` mermaid
flowchart TB
    U["User / external input"]

    subgraph DET1["Deterministic preparation"]
        R["Task routing"]
        C["Context selection"]
        S["Authoritative state"]
        K["Rules / eligible choices"]
    end

    subgraph SEM["Probabilistic semantic boundary"]
        L["LLM<br/>interpret • classify • synthesize • choose"]
    end

    subgraph DET2["Deterministic completion"]
        V["Validate"]
        A["Assemble domain result"]
        P["Semantic Pipeline"]
        T["Templating"]
        D["Persist / integrate / publish"]
    end

    U --> R
    R --> C
    S --> C
    K --> C
    C --> L
    L --> V
    S --> V
    K --> V
    V --> A
    A --> P
    P --> T
    T --> D
```

This arrangement makes the LLM a specialized semantic component inside a
larger engineered system rather than the owner of the complete
application workflow.

### What the system uses each layer for

  -----------------------------------------------------------------------
  Layer                   Best suited for         Why
  ----------------------- ----------------------- -----------------------
  **App / Domain API**    State, rules,           Exact, fast, testable
                          validation, IDs,        
                          authoritative data,     
                          synchronous assembly    

  **Retrieval / semantic  Selecting relevant      Prevents context
  memory**                knowledge for the       accumulation
                          current task            

  **LLM**                 Interpretation,         Handles meaning and
                          ambiguity resolution,   language
                          synthesis,              
                          classification, bounded 
                          choice                  

  **Semantic Pipeline**   Multi-step              Observable, reusable,
                          orchestration,          composable
                          asynchronous work,      
                          semantic indexing,      
                          integration             

  **Templating**          Rendering files,        Repeatable and diffable
                          configuration,          
                          documents, payloads,    
                          prompts, previews       

  **Persistence /         Durable truth and       Keeps probabilistic
  downstream systems**    execution               output away from direct
                                                  writes
  -----------------------------------------------------------------------

## Core architecture

The generic architecture uses four responsibilities:

-   **App / Domain API** --- synchronous validation, authoritative
    state, domain rules, deterministic assembly, and persistence
    boundaries.
-   **Semantic Pipeline** --- reusable orchestration for asynchronous,
    observable, multi-step semantic and deterministic work.
-   **Templating** --- deterministic rendering of validated data into
    repeatable artifacts.
-   **LLM** --- bounded interpretation, synthesis, ambiguity resolution,
    and semantic choice.

This separation makes model size and reasoning depth a per-task
engineering decision rather than a property of the whole application.

## Execution boundary

``` mermaid
flowchart LR
    A[Developer message] --> B[Phase router]
    B --> C[Bounded context and eligible choices]
    C --> D[8B LLM]
    D --> E[Small schema-constrained semantic response]
    E --> F[App validation and domain assembly]
    F --> G[Reviewable proposal]
    G -->|accepted automation| H[Semantic Pipeline recipe]
    H --> I[Durable artifacts, indexes, previews, or build requests]

    J[Explicit fact mode] --> F
```

The LLM is not the database, serializer, validator, workflow engine,
build system, or source of domain truth. Its response is an advisory
input to a deterministic pipeline.

The deterministic boundary is deliberately split across specialized
owners:

-   **App / Domain API** owns synchronous domain work needed to produce
    a reviewable proposal: evidence checks, filtering, duplicate
    removal, stable keys, phase rules, catalog grounding, compatibility
    attachment, and proposal assembly.
-   **Semantic Pipeline** owns asynchronous recipe execution and
    inspectable durable work: fact-capture artifacts, semantic-memory
    rebuilds, source previews, generated domain artifacts, artifact
    inventories, and downstream integration requests.

Semantic Pipeline does not replace every deterministic operation in
Studio. It is used when the operation should be a reusable,
asynchronous, observable recipe rather than part of an HTTP request.

### Deterministic rendering with Templating

**Templating** owns repeatable rendering from validated structured data
into source files, documents, configuration, prompts, previews,
payloads, and other artifacts. The LLM can decide *what* should be
produced; Templating determines *exactly how* approved data is rendered.
This keeps boilerplate, formatting rules, and artifact structure
deterministic, diffable, testable, and versionable.

## How token demand was reduced

### 1. Phase-specific prompts replace one universal prompt

Discovery, Analysis, Selection, and Domain each have a purpose-built
input context and response schema. The model no longer receives every
domain object or returns a complete multi-phase proposal when the
current turn only needs a discovery, a catalog choice, or a named domain
subject.

Advanced Synthesis still use the larger general proposal contract. They
are the remaining phases to specialize after they have stable acceptance
fixtures.

  -----------------------------------------------------------------------
  Phase           LLM receives      LLM returns              Output limit
  --------------- ----------------- ----------------- -------------------
  Discovery       Domain identity,  Reflection, next                  700
                  accepted truths,  question, up to   
                  current-phase     four              
                  history, latest   evidence-backed   
                  message, bounded  discoveries,      
                  memory and        selected citation 
                  knowledge         IDs               

  Analysis        Domain identity,  Reflection, next                1,200
                  accepted domain   question, up to   
                  facts,            four compact      
                  current-phase     analysis          
                  history, latest   discoveries,      
                  message, bounded  selected citation 
                  memory and        IDs               
                  knowledge                           

  Selection       Current question, Assistant message                 180
                  active selection  and one catalog   
                  state, at most    ID                
                  four memory hits,                   
                  and only eligible                   
                  systems                             

  Domain          Current question, Assistant                         220
                  bounded           message, subject  
                  active/recent     kind, and subject 
                  subjects, up to   name              
                  four memory                         
                  records, up to                      
                  four                                
                  released-domain                     
                  citations, and                      
                  allowed domain                      
                  categories                          

  Advanced        Full accepted     General semantic                1,600
  Synthesis       design, target    proposal          
                  data, retrieved                     
                  context, and                        
                  catalog                             
                  candidates                          

  General general Latest message    One answer                        700
  assistant       and up to ten                       
  conversation    prior session                       
                  exchanges; no                       
                  domain or world                     
                  retrieval                           

  Domain Domain   Pinned world      Answer, up to                   1,000
  Assistant       identity, latest  eight             
                  message, up to    epistemically     
                  six prior         labeled           
                  exchanges, and up statements,       
                  to eight          citation IDs, and 
                  released-domain   a canon-mode-null 
                  excerpts          candidate fact    
  -----------------------------------------------------------------------

The limits are ceilings, not expected consumption. Strict schemas and
smaller response shapes are intended to make the model finish well below
them.

### 2. Context is selected instead of accumulated

the App bounds conversational and retrieval context before inference:

-   only turns from the active design phase are eligible;
-   only the four most recent active-phase turns are included;
-   semantic domain memory is limited to four records;
-   Discovery sends accepted truths as compact key/summary pairs;
-   Analysis sends only relevant facts and constraints;
-   Selection sends only the eligible catalog subset;
-   Domain sends compact subject summaries rather than the complete
    domain model;
-   Domain bounds active intent to four entries, accepted answers to
    six, existing subjects to twelve, recent accepted subjects to four,
    domain memory to four, and released-domain citations to four;
-   general assistant conversation is session-only and sends at most ten
    prior exchanges;
-   Domain Assistant sends at most six prior exchanges and eight
    retrieved authoritative excerpts;
-   source and design knowledge is retrieved by relevance instead of
    placing entire repositories in the prompt.

This prevents context growth from turning later interactive application
use turns into progressively slower and less focused requests.

### 3. The model returns signals, not durable domain objects

The former general response schema is 6,353 bytes as stored in the
repository. The specialized schemas are materially smaller:

  Contract             Stored schema size   Required top-level fields
  ------------------ -------------------- ---------------------------
  General proposal            6,353 bytes                           9
  Discovery 1                  ,381 bytes                           4
  Analysis 1                   ,379 bytes                           4
  Selection choice              553 bytes                           2
  Domain subject                743 bytes                           3

These byte sizes are structural evidence, not token measurements;
provider tokenization and the runtime-expanded enums affect the actual
prompt. They show the change in responsibility: the model returns a
small creative or selection envelope, and the App expands it into the
complete proposal contract.

### 4. Reasoning is disabled where the domain pipeline already supplies it

Discovery, Analysis, Selection, Domain, general assistant conversation,
and Domain Assistant requests use `/no_think` and
`reasoning_effort: "none"`. They do not ask the local model to generate
a long hidden deliberation before filling a tightly constrained
response.

This is not a blanket claim that reasoning has no value. The application
has already performed the procedural reasoning by selecting the phase,
current question, accepted facts, eligible choices, validation rules,
and output shape. The model is left with the bounded semantic judgment
that benefits from language understanding.

### 5. Some inputs bypass the model completely

**Record facts** mode is authoritative input, not a creative
interpretation request. It bypasses both LLM inference and knowledge
retrieval. The App deterministically segments the authoritative source
text, preserves the caller-selected classifications and provenance,
removes already-recorded facts, and creates a reviewable proposal.

The reusable `capture-domain-facts` recipe runs the same operation
through Semantic Pipeline and writes
`artifacts/design/fact-capture.json`. The LLM cannot make this path
faster or more correct, so it is not called.

## Work removed from the LLM

  -----------------------------------------------------------------------
  Mechanical task         Deterministic owner     Current behavior
  ----------------------- ----------------------- -----------------------
  Durable proposal JSON   App / Domain API        Builds the complete
                                                  proposal from the
                                                  compact model response
                                                  and current phase
                                                  state.

  JSON shape enforcement  Provider structured     Uses strict JSON
                          output plus App         Schema; malformed or
                          validation              truncated output is
                                                  rejected without
                                                  writing a partial turn.

  Decision keys and       App / Domain API        Generates identifiers
  stable IDs                                      from validated domain
                                                  data rather than
                                                  accepting invented IDs.

  Exact and near          App / Domain API        Compares accepted keys,
  duplicate removal                               normalized summaries,
                                                  and semantic token
                                                  overlap before a
                                                  proposal is persisted.

  Canonical fact wording  App / Domain API        Verifies cited evidence
                                                  and promotes the
                                                  complete authoritative
                                                  source sentence rather
                                                  than model paraphrase.

  Citation filtering      App / Domain API        Keeps only IDs that
                                                  were actually supplied
                                                  by retrieval.

  Catalog metadata and    App / Domain API and    Resolves the chosen ID
  compatibility           catalog                 to authoritative name,
                                                  version, ownership,
                                                  provenance,
                                                  dependencies, and
                                                  compatibility.

  Guidance progression    App / Domain API        Advances deterministic
                                                  question state after
                                                  validating the selected
                                                  catalog item or domain
                                                  subject.

  Implementation state    App / Domain API        Expands bounded play
  graph and acceptance                            discoveries into the
  details                                         first
                                                  implementation-scope
                                                  contract.

  Explicit fact           App / Domain API or     Uses the pure
  extraction              Semantic Pipeline       fact-capture domain
                                                  operation without
                                                  inference.

  Semantic-memory         Semantic Pipeline       Batches embedding work,
  indexing                                        upserts bounded
                                                  batches, and writes the
                                                  generation artifact.

  Source previews and     Semantic Pipeline steps Produces inspectable
  generated artifacts     and Templating          changes from recipes
                                                  and templates.

  Artifact inventory and  Semantic Pipeline steps Computes durable
  hashes                  and Templating          inventory metadata from
                                                  real outputs.

  Build request assembly  Semantic Pipeline step  Produces the
                                                  allowlisted Brand
                                                  downstream integration
                                                  request; the downstream
                                                  service owns its
                                                  execution and
                                                  packaging.

  Implementation          App / Domain API        Normalizes reviewed
  milestone IDs,                                  input, constrains
  revisions, and                                  systems to the active
  persistence                                     implementation scope,
                                                  fingerprints current
                                                  state, and writes only
                                                  after explicit
                                                  approval.

  Module-system adapter   App / Domain API and    Resolves the
  composition             Semantic Pipeline       server-owned
                                                  component-version
                                                  recipe; Semantic
                                                  Pipeline renders
                                                  reviewed app-owned
                                                  adapters and source
                                                  previews.

  Conversation            App / Domain API        Atomically appends
  persistence                                     validated exchanges,
                                                  scopes general sessions
                                                  to one user, and pins
                                                  Domain Assistant
                                                  sessions to one domain
                                                  release.

  Lore citation           App / Domain API        Rejects unavailable
  resolution                                      citation IDs and
                                                  expands accepted IDs to
                                                  immutable source
                                                  provenance before
                                                  persistence.
  -----------------------------------------------------------------------

The model still emits JSON because a small machine-readable envelope is
safer than parsing prose. The optimization is not "no JSON from the
model"; it is "only the minimum JSON needed to communicate the semantic
result."

## Why quality improved

Reducing tokens is useful only if it preserves or improves interactive
application use quality. The current split improves quality through
several independent mechanisms:

1.  **One task per turn.** The model is not simultaneously interviewing,
    inventing identifiers, normalizing data, deduplicating history,
    calculating compatibility, and formatting a large object.
2.  **Authoritative evidence.** New Discovery and Analysis discoveries
    must be supported by the latest user or domain input. the App
    verifies that evidence before using it.
3.  **Constrained choices.** Selection can select only an eligible
    catalog ID; Domain can select only allowed domain categories and
    valid active-subject state.
4.  **Deterministic rejection.** Unknown catalog items, invalid
    citations, unchanged specification updates, duplicate decisions,
    malformed JSON, and truncated responses do not silently enter the
    domain.
5.  **Stable domain semantics.** Canonical facts, IDs, provenance,
    compatibility, and phase progression do not vary with model
    temperature or wording.
6.  **Review remains the write boundary.** A successful model response
    creates a proposal. It does not directly modify accepted design
    truth, source files, or builds.

The practical result is that the 8B model can focus on language-level
interpretation and guided choice---the part it performs well---while
code handles operations for which determinism is both faster and more
accurate.

## Architectural effect of reducing LLM responsibility

The intended optimization is not merely fewer tokens. Each
responsibility removed from the model can reduce several kinds of system
demand at once.

``` mermaid
flowchart LR
    A["Reduce LLM responsibility"] --> B["Smaller context"]
    A --> C["Smaller output"]
    A --> D["Less procedural reasoning"]
    A --> E["Fewer inference calls"]
    A --> F["Lower capability requirement"]

    B --> G["Lower latency / compute"]
    C --> G
    D --> G
    E --> H["Lower operating cost"]
    F --> H

    A --> I["More deterministic ownership"]
    I --> J["More testability"]
    I --> K["More reproducibility"]
    I --> L["More predictable failure modes"]
```

The desired result is a **smaller probabilistic surface area** without
reducing application capability.

### Conventional versus bounded semantic architecture

``` mermaid
flowchart TB
    subgraph CONV["LLM-heavy approach"]
        A1["Large accumulated context"] --> A2["LLM interprets + reasons + validates + formats + reconstructs state"]
        A2 --> A3["Large generated object"]
        A3 --> A4["Application attempts to consume result"]
    end

    subgraph BOUND["Bounded semantic approach"]
        B1["Application selects relevant context"] --> B2["LLM performs bounded semantic judgment"]
        B2 --> B3["Small semantic response"]
        B3 --> B4["Application validates + assembles"]
        B4 --> B5["Semantic Pipeline + Templating"]
        B5 --> B6["Durable result"]
    end
```

The second architecture does not assume that the LLM is less capable. It
simply avoids spending model capability on responsibilities for which
deterministic software is better suited.

## Optimization dimensions

The approach can be evaluated across six related dimensions:

1.  **Less context** --- provide only information relevant to the
    current semantic decision rather than accumulated application
    history.
2.  **Less generation** --- ask for the semantic delta or decision
    rather than a complete durable application object.
3.  **Less reasoning** --- do not ask the model to reconstruct
    procedural conclusions already determined by application state and
    rules.
4.  **Fewer calls** --- bypass inference entirely when the input already
    contains everything needed for deterministic processing.
5.  **Lower model requirements** --- test whether a more bounded task
    can meet the same acceptance threshold with a smaller or less
    expensive model.
6.  **More determinism** --- move validation, state transitions,
    transformation, orchestration, rendering, and persistence into
    conventional software.

These dimensions should not be optimized independently of quality. The
target is the smallest semantic workload that still meets defined
acceptance, grounding, reliability, and usefulness thresholds.

## Evidence from a working implementation

This architecture has been applied successfully in a real application
implementation. The implementation itself is private, but its observed
behavior provides practical evidence that the research direction is
viable.

In that implementation, a local 8B model became practical for
interactive use after semantic responsibilities were narrowed and
deterministic responsibilities were moved into application code and
reusable pipeline operations. Developer-observed reasoning turns
improved from more than 2.5 minutes to approximately 35--40 seconds
while producing more focused proposals.

Using 150 seconds as a conservative baseline, this represents an
observed wall-clock improvement of approximately **3.75--4.29×**, or a
**73--77% reduction**.

These observations should not be treated as a universal benchmark or as
proof that every application will achieve the same improvement. They are
an implementation result that motivates controlled experimentation. The
measurement contract below is intended to separate architectural effects
from model residency, hardware, prompt size, output size, generation
speed, and other variables.

The implementation details can remain proprietary while the
architectural hypothesis, measurement methodology, and aggregate results
are independently discussable and reproducible in other domains.

## Experimental model

The research can be evaluated by progressively reducing the amount of
responsibility assigned to the model while measuring whether application
quality is preserved.

``` mermaid
flowchart LR
    A["1. General LLM workflow"] --> B["2. Bounded context"]
    B --> C["3. Minimal response contract"]
    C --> D["4. Deterministic assembly"]
    D --> E["5. Semantic Pipeline + Templating"]
    E --> F["6. No-LLM paths where possible"]

    A -. "measure" .-> M["Tokens • latency • compute • cost<br/>acceptance • accuracy • reliability"]
    B -. "measure" .-> M
    C -. "measure" .-> M
    D -. "measure" .-> M
    E -. "measure" .-> M
    F -. "measure" .-> M
```

The central experimental question is whether the system can move from
left to right while maintaining or improving the quality thresholds that
matter to the application.

### Minimum Semantic Computation curve

The conceptual target can also be represented as a search for the lowest
semantic workload that still satisfies the required quality threshold:

``` text
Application
quality
  ^
  |                         ───────────── acceptable quality
  |                    ____/
  |                ___/
  |             __/
  |          __/
  |_________/________________________________> Semantic computation
           ^
           |
           Minimum Semantic Computation (MSC)
           that satisfies the quality threshold
```

This is a conceptual model rather than a claim that the relationship
will always form a smooth curve. Real systems may have capability
thresholds, model-specific discontinuities, and task-dependent behavior.
The purpose of benchmarking is to find the practical minimum
empirically.

### Measurement loop

``` mermaid
flowchart LR
    H["Hypothesis"] --> I["Architecture variant"]
    I --> R["Run fixed fixtures"]
    R --> M["Measure"]
    M --> Q{"Quality threshold met?"}
    Q -->|Yes| O["Reduce semantic workload further"]
    O --> I
    Q -->|No| P["Previous acceptable configuration"]
    P --> C["Candidate MSC boundary"]
```

This turns model selection and prompt sizing into an engineering
qualification process rather than an intuition-driven choice.

## Performance measurement contract

the App currently records `responseDurationMs` on assistant turns and
displays the duration in the application UI. That is useful wall-clock
evidence, but it is not enough by itself to attribute improvements to
prompt size, output size, model residency, or generation speed.

A controlled comparison should record the following for every fixture:

-   commit, phase, prompt/contract version, model name, quantization,
    runtime, and host;
-   cold or warm model state, context window, GPU residency, and
    concurrent load;
-   exact input tokens, output tokens, time to first token, generation
    rate, and total `responseDurationMs`;
-   schema-valid-on-first-attempt rate and timeout/truncation rate;
-   proposal acceptance without editing;
-   duplicate, unsupported-discovery, invalid-citation, and
    catalog-drift rates;
-   the same fixed user or domain input and accepted domain snapshot for
    every candidate configuration.

At minimum, run each fixture once cold and five times warm. Report
median and p95 warm duration separately from cold-start duration.
Compare these variants:

1.  the historical general proposal prompt and schema;
2.  the phase-specific prompt and schema with deterministic App
    assembly;
3.  the phase-specific path plus Semantic Pipeline for durable
    automation;
4.  explicit fact capture with no LLM call.

Token counts should come from the inference provider's usage metadata or
its exact tokenizer. Character or schema byte counts must not be
reported as token counts.

## Current optimization backlog

The specialized paths reveal the next bounded improvements:

-   **Specialize Advanced Synthesis.** They still use the large general
    proposal schema and broader context. Their minimal semantic outputs
    should be derived from acceptance fixtures before reducing the
    contract.
-   **Persist provider usage.** Store input tokens, output tokens, time
    to first token when available, finish reason, model identity, and
    warm/cold state next to `responseDurationMs`.
-   **Tune the legacy timeout from evidence.** The five-minute request
    timeout protects cold inference but should not define expected
    interactive latency.
-   **Add quality gates to model qualification.** A faster model is
    qualified only when it also meets schema-validity, grounding,
    duplicate, and acceptance thresholds.
-   **Benchmark the new conversation surfaces.** General general
    assistant and Domain Assistant have bounded contracts, but their
    700- and 1,000-token ceilings should be evaluated independently from
    task-specific turns because their quality criteria differ.

## Implementation evidence and review checklist

A production implementation should make these architectural claims
inspectable rather than relying on prompt documentation alone. Useful
evidence includes:

-   task-specific prompt builders and strict response schemas;
-   bounded history, memory, retrieval, and catalog-selection logic;
-   deterministic fact-capture or no-LLM paths;
-   duplicate removal, citation/reference filtering, and catalog
    grounding;
-   Semantic Pipeline recipes for asynchronous or reusable workflows;
-   Templating definitions for deterministic artifact rendering;
-   provider usage telemetry for tokens, latency, finish reason, and
    model identity;
-   tests for request shape, output limits, reasoning settings,
    deterministic assembly, rejection behavior, and workflow
    progression.

The key architectural test is simple: **can the system explain which
decisions require semantic inference, and demonstrate that everything
else is owned by deterministic code?**

## Presentation summary

``` mermaid
flowchart LR
    A["KNOW<br/>What is relevant?"] --> B["THINK<br/>What requires semantic judgment?"]
    B --> C["VERIFY<br/>Is the result valid?"]
    C --> D["DO<br/>Execute deterministically"]

    A --- A1["App + retrieval"]
    B --- B1["LLM"]
    C --- C1["App / Domain API"]
    D --- D1["Semantic Pipeline + Templating"]
```

**Design objective:** keep the **THINK** stage as small as practical
while preserving the quality of the complete **KNOW → THINK → VERIFY →
DO** system.

> **Use AI where meaning must be understood. Use deterministic software
> everywhere meaning has already been decided.**

## Conclusion

The objective of this architecture is not to minimize AI usage for its
own sake. It is to **maximize the value of every inference**.

Semantic interpretation, ambiguity resolution, synthesis, and reasoning
remain with the LLM where they provide an advantage. Deterministic
operations remain with software where they are faster, cheaper, exact,
testable, and reproducible. The Semantic Pipeline coordinates reusable
work, while Templating converts validated structured data into
repeatable artifacts.

As these responsibilities become more precisely separated, increasingly
sophisticated AI-assisted applications may be achievable with smaller
contexts, less generation, fewer calls, less reasoning, and potentially
smaller models.

The key research question is therefore not:

> How large a model can the application use?

It is:

> **How little probabilistic semantic computation does the application
> actually need?**

That question can be answered scientifically by measuring not only
tokens and latency, but acceptance rate, grounding, reliability,
determinism, resource consumption, and cost per accepted result.

A successful architecture should demonstrate that reducing LLM
responsibility reduces time, compute, expense, and model requirements
**without transferring those savings into lower-quality outcomes or
greater human correction**.

In short:

> **Use AI where meaning must be understood. Use deterministic software
> everywhere meaning has already been decided.**
