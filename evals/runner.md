# Eval Runner

Run the `solana-auditor` skill against benchmark repos and compare output to benchmark ground truth.

## Usage

```
claude "read evals/runner.md and run all benchmarks"
claude "read evals/runner.md and run helius-account-safety"
```

## Setup

Resolve paths, create plugin symlink, get commit hash, and generate timestamp.

```bash
REPO_ROOT="$(git rev-parse --show-toplevel)"
SKILL_DIR="$REPO_ROOT/solana-auditor"
mkdir -p /tmp/audit-plugin/skills && ln -sfn "$SKILL_DIR" /tmp/audit-plugin/skills/solana-auditor
COMMIT=$(git -C "$REPO_ROOT" rev-parse --short=7 HEAD)
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
echo "commit=$COMMIT timestamp=$TIMESTAMP"
```

## Run

Each `.md` file in `evals/benchmarks/` is a benchmark with frontmatter: `repo_url`, `repo_ref` (optional), `contracts_dir` (optional).

For each benchmark, clone repository (skip if already exists at `/tmp/eval-{name}`), create run dir at `evals/results/{name}/{timestamp}-{commit}`, and run the skill in a fresh Claude process.

```bash
BENCHMARKS_DIR="$SKILL_DIR/evals/benchmarks"
RESULTS_DIR="$SKILL_DIR/evals/results"

for BENCH in $(ls "$BENCHMARKS_DIR"/*.md | sort); do
  name=$(basename "$BENCH" .md)
  REPO_URL=$(grep '^repo_url:' "$BENCH" | sed 's/repo_url: *//')
  REPO_REF=$(grep '^repo_ref:' "$BENCH" | sed 's/repo_ref: *//' || true)
  CONTRACTS_DIR=$(grep '^contracts_dir:' "$BENCH" | sed 's/contracts_dir: *//' || true)

  WORK_DIR="/tmp/eval-$name"
  [ -d "$WORK_DIR" ] || git clone --depth 1 "${REPO_URL}" "$WORK_DIR"
  [ -n "$REPO_REF" ] && git -C "$WORK_DIR" fetch --depth 1 origin "$REPO_REF" && git -C "$WORK_DIR" checkout "$REPO_REF"

  TARGET_DIR="$WORK_DIR${CONTRACTS_DIR:+/$CONTRACTS_DIR}"
  RUN_DIR="$RESULTS_DIR/$name/$TIMESTAMP-$COMMIT"

  echo "=== Starting $name ==="
  mkdir -p "$RUN_DIR" && \
  cd "$TARGET_DIR" && \
  claude --print --plugin-dir /tmp/audit-plugin --dangerously-skip-permissions \
    "run solana auditor skill with --log-output" 2>&1 | tee "$RUN_DIR/full-output.txt" && \
  cp "$BENCH" "$RUN_DIR/ground-truth.md"
  echo "=== Finished $name ==="
done
echo "All benchmarks complete."
```

After runs complete, compare each run by reading `evals/compare.md`, then write `summary.md` inside each run directory.
