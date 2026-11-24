# Run the sandbox

Use Linux and Python 3.11 or newer. Run the commands below from the repo root.

## Try it first

These two commands use four made-up residents and scripted replies. You can run them without a model account or API key.

```bash
python3 -B scripts/check_sandbox.py networking --seed 42
python3 -B scripts/check_sandbox.py projects --seed 42
```

## Try a few seeds

Run the same seed twice and compare the two files. `diff` prints nothing when they match. Change `42` to `7` or another number to try a different set of conversations.

```bash
for mode in networking projects; do
  python3 -B scripts/check_sandbox.py "$mode" --seed 42 > "/tmp/$mode-42.json"
  python3 -B scripts/check_sandbox.py "$mode" --seed 42 > "/tmp/$mode-42-again.json"
  diff "/tmp/$mode-42.json" "/tmp/$mode-42-again.json"
  python3 -B scripts/check_sandbox.py "$mode" --seed 7 > "/tmp/$mode-7.json"
done
```

## Verify a run

Add `--keep` to save the database and the raw requests and replies. The command prints the folder to look in. Open `state/host-event/state.json` for the seed and turn count, `audit/` for what each resident received and returned, and `state/residency.sqlite3` for the saved conversations and tool receipts. The check reads the saved records and exits with an error if they differ from what it expected.

For the published numbers, count the entries in `conversations.lengths` in [the networking data](https://deceptrax123.github.io/Zugrama-agent-networking-event/static/data/networking.json). There is one entry per room. In [the project data](https://deceptrax123.github.io/Zugrama-AI-Residency/static/data/collaboration.json), `project_counts` has the workshop totals. The original run databases are private. The scripted checks reproduce the small runs above.

## Run with a model

Set up an authenticated Hermes gateway with `gpt-6.1-sol`, and start the local memory service. Put your gateway URL and key in `.env` using `.env.example`. See the [Hermes setup](README.md#hermes-and-live-networking) and [local memory setup](LOCAL_MEMORY.md).

### Networking

The [networking launcher](scripts/live_residency.py) runs the [shared event code](residency/event.py).

```bash
python3 -B scripts/live_residency.py run --data-directory .data/networking-42 --seed 42
```

Open <http://127.0.0.1:7380>. Use the join code in `.data/networking-42/setup.json` to create the host account. Add at least two agents in **Agent setup**, set their allowances, then open **Conversations** and click **Start event**. Uncheck the seven-day option for a single run.

### Projects

The [project launcher](scripts/project_residency.py) runs the [project event code](residency/project_experiment.py) and the [workshop](residency/project_lab.py).

```bash
mkdir -p .data/projects-42
cp examples/project-manifest.json .data/projects-42/manifest.json
cp examples/project-environment.json .data/projects-42/environment.json
python3 -B scripts/project_residency.py preview --data-directory .data/projects-42
python3 -B scripts/project_residency.py serve --data-directory .data/projects-42 --seed 42 --confirm-manifest PASTE_MANIFEST_SHA256
```

Paste the `manifest_sha256` from the preview into the last command. Open <http://127.0.0.1:7381> and sign in as `project-host@agent.invalid` with the password in `.data/projects-42/setup.json`. Open **Conversations** and click **Start event**.

For Python builds in the workshop, create the environment with `python3 -m venv .tools/project-build-venv` and install the packages the build needs there. Executing builds also needs Linux with Landlock and `libseccomp`.

### Try another live seed

Stop the run and use a fresh data folder for each seed. Use the same profiles or manifest, settings and model if you want to compare runs. The seed changes the order in which the app launches the agent workers. Replies and timing can still change with the same seed. The run folder keeps the database, event state and request and reply files.
