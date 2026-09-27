# Harness evolution quickstart

This example is the smallest full run of the harness evolution mechanism: the agent harness configuration is a composition tree of nodes, a proposal is one gated tree mutation, and a winning mutation publishes as a versioned artifact any client can pull. The proposer is the served model itself: it reads the current skill nodes and its own failing requests, each with the score and feedback its report carried, and proposes one skill mutation as a strict JSON object. `evaluate` grades real headless episodes on a small fixed task set by exact final answer, so a proposal only publishes when it makes previously failing tasks pass their episodes.

The pinned paper reproduction built on this mechanism lives at `recipes/skillclaw/`.
## Directory layout

```text
evolve-your-harness/
  README.md      this file: what the tutorial shows, how to run it, what it saw
  evolve-your-harness.ipynb
                 the same pass cell by cell, with the service managed as a
                 subprocess from the notebook
  run.sh         copies the recipe config out of the serve file, starts
                 reef serve, waits for /healthz, runs run.py
  run.py         the loop, written out: record -> report -> evolve -> pull
  configs/
    serve.yaml   the stack (reef service) plus the harness_evolve recipe
                 sections used by the self-contained ./run.sh pi demo
    serve-native.yaml
                 the ./run.sh native variant: the loop's own tools, hook
                 and graph are the seed, so the proposer can change them
    deployment.yaml
                 the minimal long-running pi deployment used by the root
                 README; run it directly with reef serve -c
  harness/       the method package the serve files' dotted refs name
    evolution.py the method: propose (self proposer over failures, skill
                 nodes only) and evaluate (exact last-line answer grading)
    native_evolution.py
                 the native variant: propose over skills, tools, hooks,
                 the loop graph and its agents, evaluate over the native
                 trajectory; the grader is shared
    materialize_recipe.py
                 copies a serve file's recipe sections where the recipe
                 registry reads them; run.sh and the notebook call it
    probe_proposals.py
                 samples the native proposer over one failing task and
                 counts what admission lets through, by kind
    replay.py    writes work/replay.html from a run's files: the release
                 chain, the loop graph replayed from a session, the session
                 event log and the process timeline
  pyproject.toml makes harness/ an installable package
```

## Setup (once)

The client side needs only the stdlib SDK; the service side runs from the repo checkout via `run.sh`:

```bash
pip install reef-client
```

You also need an OpenAI-compatible endpoint for the model under test, and for the pi variant the pi coding agent binary on your PATH (serve.yaml's `evolution.binary`). The native variant needs no agent binary: the loop is reef's own.

## Quick start

```bash
./run.sh          # on pi (serve.yaml)
./run.sh native   # on reef's native harness (serve-native.yaml)
./run.sh self     # the same, and the model proposes the change itself through its self tools
```

Every run ends by writing `work/replay.html`: the release chain with each step's verdict and tree diff, the loop graph replayed from a session's events, the session event log and the process timeline. Open it in a browser; `python3 run.py replay` rebuilds it from `work/` at any time.

serve.yaml carries the endpoint (`inference.upstream-url: http://127.0.0.1:8000`, no /v1 suffix) and the model (`inference.upstream-model: qwen3-8b`) as literals; edit them there to point at your own. The recipe takes its model from that same `inference` section, so the proposer, the evolve episodes and served traffic all use the one name, and run.py's `MODEL` must match; a name the endpoint does not serve fails the proposer's call, and the step records `skipped: no proposal`. The one value serve.yaml does not hold is the provider key: `export REEF_UPSTREAM_API_KEY=...` if your endpoint needs one.

## Worker execution

The configs default to `auto`; the single service controller resolves to `uni`.
Worker placement defaults to `execution.evolution.backend: auto` with
`execution.evolution.workers: 1`: one CPU worker uses `uni`; increasing the count selects
spawned `mp` workers. Model inference still runs at the configured upstream
endpoint, so these scorers do not reserve GPUs.

The YAML only needs the worker count; `backend: auto` and resource requests
can be omitted. Local workers are not pinned or limited to one CPU core.

The worker/scorer pool is reused across evaluations and released when its
scenario closes. Each episode still starts a separate harness process in a
fresh temporary directory. This is a fixed-size pool, not autoscaling.
`evolution.executor` is a separate episode-isolation option (`local` or
`sandbox`), not a worker backend selector.

The demo's materializer also preserves optional top-level `execution` and
`executors` sections. For example, choose a reusable worker/resource profile:

```yaml
execution:
  evolution: cpu-pool
executors:
  cpu-pool:
    workers: 8
```

CPU requests do not enforce core limits on local workers. Ray uses CPU/GPU
requests as per-worker scheduling reservations. The old `episode_workers`,
`worker_executor` and `worker_resources` fields are deprecated; remove them
when migrating to the new structure. Conflicting resource values are rejected.

## Keep a deployment running

Pass the model at startup using `REEF_UPSTREAM_MODEL`; no YAML edit is needed. `REEF_PROPOSER_TIMEOUT_S` caps one proposer call in seconds (60 for a failure step and 600 for a request by default) and `REEF_PROPOSER_MAX_TOKENS` its reply (2048 and 65536); raise both for a local thinking model, which answers in minutes and spends the reply budget on its reasoning first.
[deployment.yaml](configs/deployment.yaml) uses the same variable for serving and
evaluation. From the repository root, after the source installation and with `pi`
on PATH, replace the model ID and API key below with your provider's values:

```bash
export REEF_UPSTREAM_URL="https://api.openai.com"  # no /v1 suffix
export REEF_UPSTREAM_MODEL="REPLACE_WITH_YOUR_PROVIDER_MODEL_ID"
export REEF_UPSTREAM_API_KEY="your-openai-api-key"
reef serve -c tutorials/evolve-your-harness/configs/deployment.yaml
```

Use the exact model ID accepted by your provider, not the placeholder above.
An unset or empty `REEF_UPSTREAM_MODEL` is reported at startup.
The deployment resolves this variable in `reef.upstream_model`. The recipe
omits `model.path` and uses the resolved runtime's model; recipe YAML loading
does not read environment variables or expand `${...}` references.

Use your provider's API key, or omit it for a local endpoint without authentication.
`deployment.yaml` keeps Reef running until interrupted. It contains both the
deployment and its named recipe preset; no recipe generation or extra Python
launcher is needed. A YAML anchor shares the model name between the recipe
and the upstream runtime, so inference, proposal, and evaluation use the same model.
Defaults: port `8901`, Reef access token `reef-local`, state under
`work/deployment/`. Use `--reef.port` and `--reef.token` to override the port
and access token. The `run.sh` demo keeps its separate configuration and state.

`data.training_mode` has three values. `deployment.yaml` sets `hybrid`: the deployment keeps learning from failures, and a person asks for a change with `POST /reef/train` (the [manual training API](https://www.yx-sf.com/tech/51036)); the next step that reads it runs it first, with no mode switch. `serve.yaml` and `serve-native.yaml` stay `auto`, the default: the `./run.sh` demo is failure driven, nobody asks there, and an ask is refused. `manual` runs instructions only and never batches on traffic; no file here selects it, because a measurement that wants one step per instruction and no failure driven step between them sets it, in its `data` section or on a running deployment with `POST /reef/scenarios/{scenario}/update`.

Install a harness from this service and submit feedback as shown in the
[root README](../../README.md#harness-evolving-deployment). Adapt `deployment.yaml`'s
tasks and `harness/evolution.py`'s proposer and grader for your workload before
starting the deployment.

If you already installed the harness with the placeholder model, set `REEF_UPSTREAM_MODEL`,
stop and restart `reef serve`, then rerun the harness install command from the root
README before retrying `reef-pi`. Installation writes the model ID into the local
harness configuration.

`deployment.yaml` also sets `evolution.requests: true`, `evolution.version_check: true` and `evolution.review_kinds: [code_extension]`: a session asks for a harness change with `reef-pi evolve "..."`, or `/reefine ...` in the TUI, and needs no mode switch because the deployment runs in `hybrid`; the notice at session start offers the newest release that is not pending; and a win that touches a `code_extension` waits as a pending release, because an evolved extension runs in pi's process with your privileges. A pending release shows only under a promote or a trial install by id. Promote it with `POST /reef/scenarios/{scenario}/promote` and `{"release_id": ...}`, the id of the row marked `pending: true` in `GET /reef/harness/releases`; the curl is under [Promote a pending release](../../docs/user-guide/evolve-your-harness.rst#promote-a-pending-release) in the user guide. `serve.yaml` and `serve-native.yaml` set none of the three: an ask is refused in `auto`, so a seeded command would have nothing to do there.

## Notebook

`evolve-your-harness.ipynb` walks the same pass cell by cell and manages the service as a subprocess, so one kernel holds the whole loop. Set the endpoint, model, and key in its first code cell; the notebook loads serve.yaml, sets `inference.upstream-url` and `inference.upstream-model` to those values, writes the result to `work/serve-notebook.yaml`, and materializes the recipe config from it with `harness/materialize_recipe.py`, the step `run.sh` runs. `run.py` stays the reference implementation of the loop; the notebook mirrors it.

Pick a model that fails at least one task and still writes the strict JSON mutation `propose` expects. A model that passes all three tasks reports no failures, so no evolve step ever runs (the DeepSeek result below); one that cannot author the JSON commits its step as `skipped: no proposal`, visible on the step's row in `GET /reef/harness/releases`. The committed outputs are a full local run with no GPU, the `pi, notebook` row of the results below.

## What one run does

1. `run.sh` copies serve.yaml's recipe sections into `work/recipes/harness_evolve.yaml` and starts Reef. The recipe boots with the seed composition: one starter `answer-style` skill node. The endpoint and the model live only in serve.yaml's `inference` section (`upstream-url`, `upstream-api-key`, `upstream-model`); Reef hands them to `propose` as a model binding and renders them into each evaluation episode, so neither the method nor the published tree ever names them.
2. `run.py` sends each of the three exact-answer coding tasks once through reef inference; reef serves the reply and records the exchange. The reply is graded the same way the evolve gate grades episodes, and the score is reported against the receipt.
3. Every valid scored report batches, and `batch_size: 1` makes each report one evolve step: `propose` sends the failing requests, each with its report's score and feedback, and the current skills back to the same model through `models.served.chat`, which answers with one skill mutation; the candidate and current compositions each run one episode per task; the mutation publishes only on a gate win.
4. `run.py` pulls `GET /reef/harness` and prints the gate metrics and the evolved `SKILL.md` files. Point any pi at the pulled tree, with its model set to Reef, and it carries the learned skill.

## Native variant

`./run.sh native` runs the same loop on reef's own coding agent (`evolution.adapter: native`), whose tools, loop hooks, loop graph and helper agents are composition nodes. Three things differ from the pi run:

- The seed is the loop's shipped `read_file`, `write_file` and `run_bash` tools, its `loop_guard` hook and its `main` graph (`think` to `act` to `think`, `done` on a text reply), named by reference (`reef.harness.runners.native.seed:SEED_NODES`) beside the starter skill, so the proposer sees them as nodes it may retune or replace.
- `harness/native_evolution.py`'s `propose` accepts one mutation on a skill, a `native_tool` (a schema plus a module defining `run(args, workdir) -> str`), a `native_hook` (a module defining `listen(payload, next) -> decision` at `pre_step`, `pre_execute`, `request_error` or `post_execute`) a `native_agent` (a helper with its own prompt, tools and budget that a graph's `subagent` stage calls) or the `native_graph` (the loop's stages and the edges between them, shown to the model as one worked example: the seed loop plus a `check` stage); a node's entry id is its name, so a proposal that reuses a seed tool's name updates that tool. An agent alone changes nothing (only a `subagent` stage runs one), so an agent proposal carries a second mutation that routes the main graph's model text through a new `ask-<name>` subagent stage and a new `answer-<name>` model stage, whose text goes where the model's text went: the agent's reply comes back as a user message, and only a model stage writes the assistant message the grader reads. `evaluate` grades the last `assistant/message` event of the native-jsonl trajectory with the same grader.
- No agent binary is installed: `run.sh native` writes a `reef-native` launcher for this checkout under `work/bin` and puts it on PATH, which is what an installed reef's console script provides.
- The pass runs through the serve form. `run.sh native` pulls the seed tree under `work/tree` (`python3 run.py pull`), starts `reef-native serve --tree work/tree --follow head --poll-interval 5` on it, and `python3 run.py native` sends each task to that process as one turn with `reef-native turn`, grades the reply and reports the score through the wrapper's `report` command, which claims the turn's receipts from `work/captures`. When the gate publishes, the running process mounts the new release between two steps or at once while idle, one `harness/mount` line in `work/tree/native/sessions/serve.jsonl` (or in the open turn's session), with no reinstall and no restart; `run.py` then sends the first task again and prints the stages of the new session, which are the mounted graph's. `work/serve.log` is the process's stderr.

The recorded pass is identical, so the two variants are comparable on the same three tasks; `run.py` prints every evolved skill, tool, hook and graph file from the pulled tree.

## Self tools variant

`./run.sh self` is the native variant with `reef-native serve --self-tools`: the served model gets `harness_inspect`, `harness_try` and `harness_propose`, the built-in tools of the resident process. `python3 run.py self` sends one turn that tells the model it runs on a harness it can read and change and asks it to inspect the tree, propose one change through `harness_propose` that makes its answers end with the integer alone on the last line, and then answer the sieve task. The route admits the proposal into the scenario's inbox; the turn's answer is graded and reported, and the failing report opens a step that claims the proposal before it asks the method. A win publishes, the process mounts the release the model proposed (the commit's `proposal` names the proposal and the session that made it), and the first task runs again on the mounted tree. The prompt names the goal and the shape of a rules entry, not the rule's text: what the model writes is what the gate judges.

## Results

Last updated: 2026-09-20. Every row was measured on the code of the pull request in its Code column, with the tutorial files as they stood there.

The historical measurements below used the previous failure-only `max_score` filter, which has since been removed. The loop under measurement is one fresh scenario through `./run.sh` (pi adapter) or `./run.sh native` (native adapter): the three tasks `[sieve]`, `[fib]` and `[csv]` go through Reef once, each reply is graded 1.0 for the exact answer alone on the last line and 0.0 otherwise, only a 0.0 report batches (`max_score: 0.0`, `batch_size: 1`), and each batched report runs one evolve step whose gate runs the current and the candidate tree once per task. Scores are listed in task order; W / L / T counts the three task pairings of one gate; the model under test is also the proposer.

### Environment

| Setup | Host | Model server | Model | Agent |
|---|---|---|---|---|
| pi, B200 | one B200 node | vllm 0.27.0 | Qwen3-8B as `qwen3-8b`, tensor parallel 1; DeepSeek-V4-Flash-0731 as `deepseek-v4-flash`, tensor parallel 4, fp8 kv cache | pi 0.84.2 |
| pi, notebook | Mac mini | ollama | `qwen2.5:7b` | pi 0.84.2 |
| native | Mac mini | ollama | `qwen2.5:7b` | reef-native from the checkout |
| native + graph | Mac mini | ollama | `qwen2.5:7b` | reef-native from the checkout, after #250 |
| native, serve | Mac mini | ollama | `qwen2.5:7b` | `reef-native serve` from the checkout, #278 with #282 and #284 |
| native, self | Mac mini | ollama | `qwen2.5:7b` | `reef-native serve --self-tools` from the checkout, after #285 |
| pi, ask | Mac mini | ollama | `qwen2.5:7b` | pi 0.84.2 through `reef-pi` from the checkout, the deployment config with `requests: true`, #311 |

### Runs

| Setup | Model | Run | Date | Code | Recorded pass | Proposal | Gate: current | Gate: candidate | W / L / T | Verdict | Launch to verdict (s) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| pi, B200 | deepseek-v4-flash | 1 | 2026-08-17 | #58 | 1.0 / 1.0 / 1.0 | none, nothing batched | - | - | - | no step | 3 [1] |
| pi, B200 | deepseek-v4-flash | 2 | 2026-08-17 | #58 | 1.0 / 1.0 / 1.0 | none, nothing batched | - | - | - | no step | 4 [1] |
| pi, B200 | qwen3-8b | 1 | 2026-08-17 | #58 | 1.0 / 0.0 / 1.0 | `create direct-answer` (skill) | 1.0 / 0.0 / 1.0 | 1.0 / 1.0 / 1.0 | 1 / 0 / 2 | published `0fe30330` | 63 [2] |
| pi, notebook | qwen2.5:7b | 1 | 2026-08-31 | #70 | 1.0 / 0.0 / 1.0 | `update answer-style` (skill) | 0.0 / 0.0 / 0.0 | 1.0 / 0.0 / 0.0 | 1 / 0 / 2 | published `f98266d2` | not recorded [3] |
| pi, notebook | qwen2.5:7b | 2 | 2026-09-20 | #544 | 1.0 / 0.0 / 1.0 | `update answer-style` (skill) | 1.0 [19] | 0.0 [19] | 0 / 2 / 1 | rejected | not recorded [3] |
| native | qwen2.5:7b | 1 | 2026-09-04 | #241 | 1.0 / 0.0 / 1.0 | `create fib-solver` (native_tool) | 0.0 / 0.0 / 1.0 | 0.0 / 1.0 / 0.0 | 1 / 1 / 1 | rejected | 93 [4] |
| native | qwen2.5:7b | 2 | 2026-09-04 | #241 | 1.0 / 0.0 / 1.0 | `create fib-skill` (skill) | 0.0 / 0.0 / 1.0 | 0.0 / 0.0 / 0.0 | 0 / 1 / 2 | rejected | 217 |
| native + graph | qwen2.5:7b | 1 | 2026-09-05 | #258 | 1.0 / 0.0 / 1.0 | `update main` (native_graph) [5] | 1.0 / 0.0 / 0.0 | 1.0 / 0.0 / 0.0 | 0 / 0 / 3 | rejected | 645 |
| native + graph | qwen2.5:7b | 2 | 2026-09-05 | #258 | 0.0 / 0.0 / 1.0 | `update main` (native_graph): a `check` stage | 0.0 / 0.0 / 0.0 | 1.0 / 0.0 / 1.0 | 2 / 0 / 1 | published | 178 [6] |
| native, serve | qwen2.5:7b | 1 | 2026-09-06 | #282 + #284 | 0.0 / 0.0 / 0.0 | `update main` (native_graph): a `check` stage | 0.0 / 0.0 / 0.0 | 0.0 / 1.0 / 0.0 | 1 / 0 / 2 | published, mounted [11] | 320 [12] |
| native, self | qwen2.5:7b | 1 | 2026-09-06 | #285 | 0.0 [13] | `create answer-format`, `create answer-format-rationale` (rules), by the model through `harness_propose` | 0.0 / 0.0 / 0.0 | 0.0 / 1.0 / 1.0 | 2 / 0 / 1 | published, mounted [14] | 124 |
| native, self | qwen2.5:7b | 2 | 2026-09-06 | #287 | 0.0 [15] | `remove answer-style`, `create answer-style` (skill to rules), by the model after the route refused a kind change | 0.0 / 0.0 / 0.0 | 0.0 / 0.0 / 0.0 | 0 / 0 / 3 | rejected | 113 |
| pi, ask | qwen2.5:7b | 1 | 2026-09-06 | #311 | one plain turn [16] | none: the request's reply was dropped [17] | - | - | - | skipped | 207 |
| pi, ask | qwen2.5:7b | 2 | 2026-09-06 | #311 | one plain turn [16] | `create test-first` (skill), by the service proposer from the request | 0.0 / 0.0 / 0.0 | 1.0 / 0.0 / 0.0 | 1 / 0 / 2 | published `7b91b2ca` | 207 [18] |

One run is one sample and no run was repeated, so the rows carry no spread. Gate episodes are stochastic: the seed tree scored `[sieve]` 1.0 on every recorded pass and 0.0 in four of the five gate episodes on the Mac, because with tools in hand the model runs the sieve and then answers in prose, so the integer is not alone on the last line.

### Proposal admission by prompt shape

Samples of the native proposer over the `[fib]` failure, measured 2026-09-05 at #258 with `harness/probe_proposals.py` against ollama at its default sampling settings; one call per sample under the proposer's 120 s and 2048 completion token budget. A sample is parsed when the reply carries a mutation and admitted when that mutation passes its node kind's admission rules, the check the evolve step runs before any episode.

| Prompt shape | Model | Samples | Parsed | Graph | Graph admitted | Tool | Skill | Unparsed |
|---|---|---|---|---|---|---|---|---|
| placeholders (`<stage>`, `<outcome>`) | qwen2.5:7b | 30 | 25 | 11 | 0 [7] | 14 | 0 | 5 |
| worked example (the seed loop plus a `check` stage) | qwen2.5:7b | 30 | 29 | 29 | 23 [8] | 0 | 0 | 1 |
| worked example | qwen3:8b | 15 | 7 | 5 | 5 | 2 | 0 | 8 [9] |
| worked example | qwen3.5:9b | 15 | 5 | 4 | 3 [10] | 0 | 1 | 10 [9] |

The worked example moved graph admission from 0 of 11 to 23 of 29 and removed every tool and skill proposal, which is the trade the tutorial makes; the two newer models write admissible graphs when they answer at all and lose more than half their samples to the budget, which is why the tutorial stays on qwen2.5:7b.

### Reading

- Both pi publishes came from one failing task and one skill proposal, on Qwen3-8B behind vllm and on qwen2.5:7b behind ollama alike.
- On the native loop without the graph node the gate held both proposals back: the tool won `[fib]` and lost `[csv]`, the skill won nothing.
- With the graph node the proposer rewrote the loop instead: a `check` stage that verifies the last line is a plain integer and, on `fail`, returns to `think` with the message `Reply with the final answer as a plain integer alone on the last line.` Run 2 published it on wins at `[sieve]` and `[csv]`; `[fib]` stayed 0.0 on both sides because fib(90) exact is beyond this model. The rewrite is the prompt's worked example reproduced stage for stage, so the proposer chose it rather than invented it; whether this model writes a graph the prompt did not show is the open measurement.

### Published artifacts

The B200 run's skill, as pulled from the published tree:

```text
--- pi-agent/skills/direct-answer/SKILL.md ---
# direct-answer

Enforces answers to be exact plain integers without additional text. For numerical problems requiring precise values (e.g., Fibonacci numbers, large computations), this skill ensures the response contains only the final integer result on the last line, adhering strictly to the format specified in the query.
```

The graph run's `main`, the stage it added and the edges that route through it:

```text
"check": {"kind": "verify", "check": "last_line_integer", "message": "Reply with the final answer as a plain integer alone on the last line."}
{"from": "think", "when": "text", "to": "check"}, {"from": "check", "when": "pass", "to": "done"}, {"from": "check", "when": "fail", "to": "think"}
```

The notebook keeps its own run's outputs cell by cell, the step's verdict included.

### Reproduce

```bash
cd tutorials/evolve-your-harness
# pi rows: configs/serve.yaml names the endpoint and the model. The B200 rows served
#   vllm serve Qwen/Qwen3-8B --enable-auto-tool-choice --tool-call-parser hermes
./run.sh
# native rows: configs/serve-native.yaml, ollama on 127.0.0.1:11434
./run.sh native
# the admission table
python3 harness/probe_proposals.py --samples 30 --model qwen2.5:7b
```

The model name is set in two places, `inference.upstream-model` in the serve file and `MODEL` in `run.py`; every row above had both on the model in its Model column.

### Notes

[1] The recorded pass alone; no report batched and no step ran.
[2] End to end, including the pull of the published tree.
[3] The notebook's committed outputs hold the run cell by cell but no timing.
[4] `current_residue: 16`: on macOS a `run_bash` call that runs Apple's python3 leaves its bytecode cache under the episode's `HOME`; a Linux run reports 0, and residue is counted, never forbidden, unless `evolution.forbid_residue` is set.
[5] This run was measured before a rejected proposal's content was kept, so the rewrite is known as a `main` update and nothing more. Since then the commit record and the `rejected` history that `propose` receives carry each mutation's options, and `evolution.step_record_dir` writes every step's proposer calls, parsed proposal and gate episode logs to disk; a rerun of this table can show the rejected rewrite itself.
[6] The run's second failing report opened a second step, a `fib-optimize` tool, which was still on the gate when `run.py` pulled the published tree and `run.sh` stopped the service.
[7] 9 graphs left out the verify stage's `check` and 2 omitted the `fail` edge.
[8] 6 graphs carried a duplicated edge or an outcome the stage does not have.
[9] qwen3:8b: 4 samples over the 120 s limit and 4 replies with no mutation; qwen3.5:9b: 9 over the limit and 1 with no mutation.
[10] 1 graph had a stage unreachable from `think`.
[11] The serve form. The resident process saw the new head on its next poll; its first manifest read waited behind step 2 and timed out (`harness/mount-failed`), and the mount landed 64 s after the publish, the moment step 3 (the last batched report, no proposal) released the catalog. The same process then ran the first task again on the mounted graph: 11 stages through `check`, answer 4921, score 0.0.
[12] From the first turn's start to the step's commit: three turns (80 s), the proposer (25 s) and the paired episodes. Steps 2 and 3 followed with no proposal, 23 s and 42 s in the proposer.
[13] The self form sends one turn, not the three tasks: the model inspected its tree, proposed the two rules through `harness_propose` (admitted at the route) and answered the sieve task `\boxed{95920}`, score 0.0, reported.
[14] The step claimed the model's proposal (the commit's `proposal` names its id and session), the process mounted the release within one second of the commit, and the sieve task ran again on the 9 entry tree with the model's rules in its prompt: stages entered think, act, think, done, answer `\boxed{9657}`, score 0.0. The gate's win and the served turn's miss are both on the record. Launch to verdict is the first turn's start to the commit.
[15] `./run.sh self` as shipped. The model's first proposal updated the `answer-style` skill into a `rules` entry; the route refused it under the kind rule and said why (remove the entry and create it under the new kind); the model read the reason and proposed the remove plus create pair, admitted. Its own answer put the right number in a sentence, score 0.0. The gate tied all three tasks (both sides 0.0) and rejected; the settled proposal file carries the verdict, and the replay page shows the rejected step beside the seed.
[16] Historical run on #311 before its rebase onto native auto/manual training: `reef-pi -p "Reply with the single word ready."` for one session, so its receipt spools, then `reef-pi evolve "add a skill named test-first that tells you to run the project's tests before you answer any coding question"`, which files the request and reports that receipt with score 0; the step reads the request beside the trace and the service proposer writes the change. The recorded pass is the one turn, not the three tasks.
[17] The model answered the request prompt with a well formed `test-first` skill whose config omitted the name the entry id carries, and the parser refused a named kind without it; the step recorded `skipped: no proposal` and the request settled `skipped`. The parser now takes the entry id as the name.
[18] The same request on the fixed parser: the proposer took 15 s, the candidate won the sieve task and tied the other two, the release published and the request settled `selected`; the commit's `request` names its id, session and text. Launch to verdict is from the ask to the commit, in 5 s polls.
[19] The notebook prints the gate's total scores, not the per task ones; W / L / T carries the per task pairings.

The current ask command uses `POST /reef/train`, which `deployment.yaml` takes in `data.training_mode: hybrid` (`manual` takes it too; `auto` refuses it, and `POST /reef/scenarios/{scenario}/update` switches a running scenario). It needs no inference receipts and leaves the feedback spool intact; commits use `training_request` metrics. Runs [16] to [18] describe the earlier request-store implementation and have not been repeated against this path.

### Known limitations

- `run.py` stops at the first publish, so a run with two failing reports records one verdict.
- A rejected proposal's content is not persisted, so a rejected row can name the mutation's id and op only.
- vllm serves Qwen3-8B for pi only with `--enable-auto-tool-choice --tool-call-parser hermes`; without them it answers pi's `tool_choice: "auto"` requests with a 400, every episode fails, and every gate ties.
- The B200 rows and the Mac rows differ in host and model server, so wall clocks compare within a setup, not across.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/zixun/careers-70645398.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/news/53850)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/xinwen/tag-64802905.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/baogao/link-02321401.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/tech/74297)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/tuiguang/satisfaction-72702579.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/sheji/notification-34499493.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/56532)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/shichang/module-29723880.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/kaifa/collaborate-35642470.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/wiki/23646)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/sheji/ranking-84042050.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/yanjiu/profile-30842295.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/news/22373)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/huodong/calculator-89303668.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/jishu/study-01498157.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/67795)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/wendang/faq-45431365.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/chuangxin/trading-50053266.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/685)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/yunying/progress-30946658.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/jishu/file-97717703.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/wiki/63887)
* [高韧性数据交换通道规约-#024](https://www.ai-hao123.com/zhineng/media-51693246.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/yanjiu/strategy-74468058.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/tech/39140)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/wendang/tag-79307114.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/zixun/personalization-68002206.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/82028)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/gongju/privacy-38322317.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/anli/customer-49825437.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/55656)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/guanjianci/landing-21716823.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/zhinan/experience-51152176.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/19739)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/gongju/message-20489730.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/pingtai/contact-30505891.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/wiki/7194)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/paiming/presentation-66397431.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/shangye/conference-56802308.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/tech/520)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/wendang/responsive-70344367.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/zhineng/premium-81879847.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/tech/23704)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/gongsi/seminar-50244079.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/shangye/technology-46506893.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/news/56770)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/chanpin/database-71531714.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/paiming/reminder-23014235.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/26609)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/guanjianci/widget-66186931.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/shangye/server-14836875.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/17480)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/jiaocheng/productivity-54066116.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/youhua/planning-32284812.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/wiki/27760)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/fenxi/communication-25145909.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/shuju/metric-87980516.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/tech/63357)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/suanfa/profit-92433839.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/zhinan/share-56125923.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/27925)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/hezuo/resolution-25028698.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/pingtai/reporting-00829503.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/news/69212)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/liuliang/coupon-89796688.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/gongsi/home-98138285.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/tech/22553)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/jiaocheng/finance-96149966.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/suanfa/demographic-79327365.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/tech/36637)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/pingtai/tag-66335145.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/gongsi/folder-29718490.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/news/84610)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/xitong/beauty-30746865.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/shuju/analytics-85249938.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/news/48277)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/pingtai/image-03756955.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/pingtai/promotion-78076214.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/wiki/40358)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/yingxiao/customer-87798964.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/shichang/document-72722578.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/42360)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/paiming/navigation-30417503.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/jiaoliu/calculator-56836607.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/93602)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/yunsuan/version-49392504.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/anli/health-89396580.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/tech/28536)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/ziyuan/blog-84543504.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/xinwen/optimization-90019506.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/30116)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/wenzhang/design-80065052.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/wendang/enterprise-62409872.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/tech/6915)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/youhua/economy-55117872.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/jiaocheng/file-63429932.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/17360)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/sheji/loyalty-47396522.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/gongxiang/learning-97072363.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/3245)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/paiming/innovation-49866663.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/shangye/careers-38958940.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/82260)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/jiaoliu/affordable-70549969.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/guanjianci/like-00646827.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/27780)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/xitong/data-86080241.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/guanjianci/entertainment-88172439.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/wiki/72418)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/baogao/team-91629107.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/xinwen/team-07114465.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/3070)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/shuju/workshop-62732381.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/pingtai/entertainment-15072752.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/2850)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/qiye/discovery-11623122.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/shichang/team-35976517.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/news/58016)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/wendang/analysis-62959056.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/zhinan/browser-81154268.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/96786)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/wendang/tag-18050774.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/liuliang/review-33917941.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/87495)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/yingyong/label-16617765.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/zhinan/discovery-41682699.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/news/56277)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/gongsi/landing-62838598.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/fuwu/careers-67585641.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/news/46909)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/suanfa/button-43839016.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/liuliang/settings-94903348.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/wiki/37104)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/gongju/productivity-71368859.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/yunsuan/data-23747352.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/38539)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/guanjianci/conversion-29286721.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/fuwu/conversion-06474268.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/34349)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/suanfa/technology-03166076.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/keji/deal-07358466.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/12060)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/suanfa/image-53220190.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/shangye/user-47030144.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/wiki/24369)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/gongsi/system-25399170.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/guanjianci/kpi-72474750.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/8942)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/zhizhu/community-80629690.html)

</details>

