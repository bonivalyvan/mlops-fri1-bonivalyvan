# Week 2, Labs 3–4: dependency lock, Docker, clean rerun, tested prediction API

## Student / team

**Student:** Yvan Bonival FEUGANG (`bonivalyvan`)  
**Team:** Vanta — Jeromsan JUDES RAMESH, Merwane KHELOUFI and Yvan FEUGANG  
**Week:** 2, Lab 3. Lab 4 (prediction API) is not part of this work and remains
for a later task.

## Links to code and pull requests

- Team repository: https://github.com/Jeromsan/mlops-fri1-vanta-bike-demand
- Lab 3 pull request: https://github.com/Jeromsan/mlops-fri1-vanta-bike-demand/pull/2
- PR head / image source commit: `df79631140a982233ee5b2e324d88f7887db74e3`
- PR #2 was merged into `main` on 2026-10-16. The infrastructure check passed
  on the PR head.

## Test and execution results (commands, revision, observed output)

### Locked dependencies and clean image build

The committed `uv.lock` was not changed. `uv lock --check` passed; its SHA-256
is `8d84120008104540172043b5dc103a83a3827f42dc8f0f369f0367e32e8c7462`.
Separate disposable-copy checks confirmed that a missing or stale lock is
rejected without regenerating it.

The image was built without cache from a clean checkout of commit
`df79631140a982233ee5b2e324d88f7887db74e3`:

```text
docker build --no-cache --progress=plain --build-arg SOURCE_COMMIT=df79631140a982233ee5b2e324d88f7887db74e3 -t bike-demand:lab3 <clean-checkout>
Result: exit 0
Image ID: sha256:8ace904866a3c1122d13b8ac4a266ae42b243d42e315c80fc5398b04a661d540
```

The Dockerfile pins Python `3.12.11-slim-bookworm` and uv `0.12.23`. It
installs from the committed lock with `--locked`, excludes `.git`, local
environments and generated outputs, and receives the source commit through
`SOURCE_COMMIT`.

On a disposable copy of that commit with only `uv.lock` removed, the build
failed as intended at `COPY pyproject.toml uv.lock README.md ./` with
`"/uv.lock": not found` (exit 1). The working lock was left intact.

### Container tests

```text
docker run --rm --network none --entrypoint uv bike-demand:lab3 run --locked pytest -q
49 passed, 1 warning in 150.06s
```

The warning is an MLflow/SQLAlchemy deprecation warning; no tests failed. The
container ran without network access.

### Baseline rerun

```text
docker run --name bike-demand-lab3 --network none bike-demand:lab3
Result: exit 0
```

The run used the unchanged frozen dataset (SHA-256
`e03de4ee4ef4dc376ac6e04bf829673c6269e8eba5c60fa121640fa2f829504f`) and the
fixed Lab 2 configuration. Validation results:

| Predictor | Validation MAE (rentals/hour) | MLflow run ID |
| --- | ---: | --- |
| Training mean | 154.23179306864967 | `3f30d79c5af2496f8649a5e5abc4b3bb` |
| Random Forest | 86.84685685591091 | `d05587ceb68d4030abeb95988fa3f599` |

Model version: `baseline-9bc4824e1d60`. Model SHA-256:
`d24c050e3631d5b4c2feb8cb571bef6382d8c975c42a5c11edd28de78a0b8c07`.
Readback succeeded: the metadata and logged model bytes matched, the reloaded
validation MAE matched, and the maximum prediction delta was `0.0`.

The metrics exactly match those recorded for Lab 2. The train, validation and
held-test partitions were unchanged: 8,645 / 4,358 / 4,376 rows respectively.
The held-test rows were counted only; no test metrics were computed. Because
the image excludes Git metadata, `working_tree_dirty` is `null`; the build
argument records the source revision explicitly.

## Screenshots

No Lab 3 screenshots were captured for this report. Before submitting, add
2–4 screenshots under `reports/images/` in the personal repository and embed
them here. Useful evidence would be the successful container test output,
baseline MAEs/run IDs, clean image build, and the PR check. Do not present a
screenshot as a teammate review or non-author rerun.

## My contribution

My traceable GitHub contribution was integrating the Lab 3 pull request into
`main` as `bonivalyvan`. The team report attributes implementation and command
execution to Jeromsan. I therefore do not claim to have authored the Docker
changes or personally performed the recorded test and baseline commands.
Independent review and a rerun by a non-author are still outstanding.

## Decisions and rationale

- Pin the Python and uv image tags and install dependencies with `uv sync
  --locked`, so an image build cannot silently repair or refresh the lock.
- Copy dependency manifests before application files to keep dependency
  installation separate from source changes.
- Keep `.git`, local virtual environments, secrets and generated model,
  tracking and report outputs out of the image context.
- Disable networking for both container tests and the baseline run; the
  frozen dataset and locked packages are already present in the image.
- Pass `SOURCE_COMMIT` into the image because Git metadata is excluded, while
  still linking the observed metrics to the source revision.

## Review and response

PR #2 was merged and its infrastructure check passed. GitHub records an
approval from the PR author/repository owner, not an independent teammate
review. There are no review threads or actionable reviewer observations, so
there is no reviewer finding or author response to report. A peer review should
still be requested and recorded.

## Blockers / missing evidence

- Ask a teammate to rebuild and rerun from the README, then record the
  teammate, commit, image ID and observed result.
- Obtain and document an independent review, one actionable finding and the
  author's response/rerun.
- Add the required 2–4 personal-report screenshots.
- Lab 4 prediction API work has not been performed.

## Assistance and recovery acknowledgement

Copilot assisted with implementation/debugging and preparation of the evidence
and this draft. The team report attributes the technical implementation and
command execution to Jeromsan. No instructor recovery function was used.
