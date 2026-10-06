# github.com/kimjooyoon/gooo-library-starters

This starter is a small Gooo package graph. The core package declares
Normalize(Integer) -> Integer and Clamp(Integer) -> Integer; the app package
imports core and exposes Main. The explicit chain increments an input, clamps
the result to the closed interval 0..10, and returns it. Normalize's
`assembling` block declares complete two-hole assignments: a base expression
and a step. Two assignments that pass the finite examples express the same
behavior with different structure. This lets an optional decision model choose
between compiler-checked alternatives. The candidates and their finite examples
live in Gooo source, so the workspace executes without a separate body-plan
JSON file.

Install the Gooo CLI and run these commands from this directory:

~~~sh
go install github.com/kimjooyoon/meta-ontology-go/cmd/gooo@e83f5e078ad476aab1ef22d7fc59998dc5bf6356
gooo package resolve gooo.workspace.json
gooo package execute --json --cases cases.json gooo.workspace.json
~~~

To use the local compact model for the source-declared assignment:

~~~sh
gooo package execute --json --cases cases.json --tiny-model path/to/model.json \
  gooo.workspace.json
~~~

`--tiny-model` loads one local compact model for the sequential activity fills.
This repository does not bundle model weights; supply a compatible local model
file. It cannot be combined with `GOOO_LAYA_URL` or `GOOO_LAYA_API_KEY`. Without it, Laya is
used when configured; otherwise Gooo follows the deterministic declared
candidate order. Laya may select only one of the complete assignments in the
Gooo source. The compact model maps a supported root operation in Normalize's
first hole to a distinct assignment; it does not jointly reason over later
holes or invent fills.

The execute command follows both declared bindings, fills Normalize's two
holes, compiles the generated Go, and runs three cases through the whole chain.
The cases cover values below, inside, and above the clamp range. Gooo scores
each candidate before emission, checks the selected body, and reports a
replayable receipt with finite observed accuracy. These examples do not prove
behavior for all integer inputs. Unsupported or ambiguous compact-model plan
shapes produce a diagnostic.

The workspace manifest records package imports and the public entry. Package
resolve prints the deterministic graph receipt; package execute adds generated
native execution and a replayable receipt for the chosen body fill.

## Optional Laya decision service

Laya can choose between the complete assignments already written in Gooo. It
does not generate source code. Gooo filters candidates through its declared
tests, asks Laya to choose among the remaining alternatives, then typechecks,
replays, and executes the chosen body. Without a configured model service, the
compiler uses its deterministic candidate order.

For a local Laya install, download only the multilingual checkpoint and start
the official server on loopback. The multilingual model supports Korean and
English; see the [Laya model card](https://huggingface.co/convaiinnovations/laya)
for its requirements and license.

~~~sh
hf download convaiinnovations/laya --repo-type model --include 'multilingual/*' \
  --local-dir /path/to/laya-model
~~~

Configure the server to load only that checkpoint. Then set the endpoint and
run the workspace:

~~~sh
GOOO_LAYA_URL=http://127.0.0.1:8787/v1/systemone \
  gooo package execute --json --cases cases.json gooo.workspace.json
~~~

The decision is synchronous and happens after candidate scoring, before final
emission. The JSON report includes the selected candidate, model revision,
candidate scores, request latency, and finite-case results. In one local Apple
M4 run, Gooo retained two of the four source-declared assignments after its
finite tests; Laya selected `add_one`. The first request took about 999 ms and
a subsequent warm request took 209 ms. The three body examples scored 3/3 and
the generated package graph executed 9/9 finite observations. This is one
machine's measurement, not a speed or correctness guarantee. The loaded model
reached about 1.13 GiB RSS after inference. An instantaneous CPU sample was
about 0.2%; GPU utilization was not measured.

The local Laya runtime also warned that this checkpoint contains invalid
temperature settings and used its fallback temperature. It explicitly marked
the resulting confidence values as uncalibrated. Treat the choice as a ranking
signal only: Gooo's finite tests and type checks decide whether a candidate can
proceed. The decision receipt records the checkpoint revision and returned
probabilities for inspection; those probabilities are not calibrated
confidence estimates.
