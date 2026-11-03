# Capstone Track: Applied Data Science

**Final Sprint | Due before Final Presentations**

## Overview

Harborlight Logistics runs `statustracker` for its Operations Support Desk. Dana Okafor, Director of Operations, has to decide before the next quarterly business review whether the Support Desk (six agents today) needs more people. Priya Nair, Head of Support Desk, says her agents are drowning. Dana has heard that before and needs numbers: are incidents growing, how long do they take to resolve, and which kinds of incidents keep the desk busiest?

The Platform Team (Sam Whitfield, lead) has the data but has never analyzed it. Your team plays the data analysts. You pull incident data from the places it already lives (the Postgres `incidents` table, the Flask API, and Prometheus), turn it into clean CSV files, charts, and a short written report, and give Dana an evidence-based staffing recommendation. Everything must run on the existing single server with no new paid services.

After completing this track, you will have a repeatable, scripted analysis pipeline that goes from raw incident data to CSV tables, PNG charts, and a Markdown report with a staffing recommendation, plus the documentation another team could use to rerun it.

## The Business Problem

**Who is asking:** Dana Okafor (business sponsor) and Priya Nair (Support Desk). Sam Whitfield is your technical contact.

**Pain today:** Staffing arguments are made from gut feeling. The `incidents` table has `id`, `title`, `status`, `description`, and `created_at`, and nothing else. There is no category column and no resolved time. The seed generator (`week-2/app/seed.py`) only produces a status, a title from a short list, and a random `created_at` over 180 days. Your team must decide how to get categories and resolution times (for example, derive a category from the title, extend the generator in your own copy so it also writes realistic categories and resolved times, or both). You must document the choice and its limits honestly. Data that is uniformly random will not produce a trend worth reporting, so think about what your data needs to look like for the question to be answerable.

**Constraints:**
- Budget: no new paid services; everything runs on the existing single server.
- Compliance: Elena Brandt (Compliance Officer) requires that any data you share contains no real customer names or personal data. The data here is synthetic; say so in your report.
- Deadline: before the next quarterly business review.

**Success criteria (measurable):**
1. The incident dataset you analyze has at least 10,000 rows, covers at least 90 days, and includes a category for every row and a resolution time for every closed row. Evidence: a `data_quality.csv` (row counts, null counts, date range) committed with your analysis.
2. Three analysis outputs exist as CSV files, each reproducible from one command: weekly incident volume, time to resolve (median and 90th percentile, per category), and incident count per category ranked highest to lowest.
3. At least three PNG charts, each with a title, labeled axes, and units: volume trend, resolution-time distribution, and busiest categories.
4. At least one statistical check backed by a number, not just a picture (for example a trend slope with a p-value, or a comparison of two categories' resolution times), with a one-sentence plain-English interpretation.
5. A Markdown report (`report.md`, two pages or less) that ends in a concrete staffing recommendation (a number or range of agents, and which hours or categories need the coverage), with the assumptions stated. Dana should be able to read it without opening any other file.

## Learning Objectives

- Extract data from a relational database, an HTTP API, and a metrics system using command-line tools
- Clean and validate a dataset, and report its limitations honestly
- Compute and interpret summary statistics (volume trends, medians, percentiles) with pandas and scipy
- Produce clear, labeled charts with matplotlib that a non-technical reader can understand
- Turn analysis into a recommendation and state the assumptions behind it
- Make an analysis repeatable with a script and a RUNBOOK

## Prerequisites

Everything from earlier weeks must still work: the Week 2 Compose stack with Postgres 15 and the Flask API, the Week 3 k3d cluster, and the Week 5 Prometheus. The Week 9 seeded data is a starting point but not enough on its own (see The Business Problem).

### Environment Checkpoint

Run these before you plan anything. Replace `<your cluster name>` and `<your Prometheus port>` with your team's values, and `<N>` with your team number.

```bash
# App and database (Compose)
curl -s http://localhost:808<N>/health
docker compose -f week-2/docker-compose.yml exec db psql -U appuser -d statustracker -c "SELECT COUNT(*), MIN(created_at), MAX(created_at) FROM incidents;"

# Cluster
k3d cluster list
kubectl config current-context
kubectl get nodes
kubectl get pods -A

# Prometheus (run the port-forward in its OWN terminal; leave it in the foreground)
kubectl port-forward -n monitoring svc/prometheus-server <your Prometheus port>:80
```

In a second terminal:

```bash
export PROM_URL=http://localhost:<your Prometheus port>
./week-5/pq.sh 'up'
```

You should see one line per target ending in `-> 1`. (`pq.sh` is the Week 5 helper; for raw JSON, `curl -s "$PROM_URL/api/v1/query" --data-urlencode 'query=up' | jq .status` returns `"success"`.) If the port-forward dies or hangs: `pkill -u $USER -f "kubectl port-forward"` and start it again. (The Compose port is 8080 + your team number, so team 3 uses 8083.)

## Team Port Assignments

This track needs no new ports. You use only the ports you already have:

- Compose app: 8080 + team number
- Kubernetes (k3d load balancer): 8090 + team number
- Prometheus port-forward: 9090 + team number

> **Shared VM (RHEL, teams):** Use only your own team's ports. Do not send traffic to another team's ports.

> **Local VM (Ubuntu, individual):** There is no team number. Use 8080 (Compose), 8081 (Kubernetes), and 9090 (Prometheus).

## Tools Available

| Tool | What it's for | Verify (both OSs) | Ubuntu install |
| ---- | ------------- | ----------------- | -------------- |
| Python 3 + venv | Run your analysis scripts | `python3 --version` | `sudo apt install -y python3 python3-venv` |
| pandas | Tables, grouping, resampling | `~/data-science-venv/bin/python -c "import pandas; print(pandas.__version__)"` | `~/data-science-venv/bin/pip install pandas` |
| matplotlib | Charts saved as PNG (use the `Agg` backend; no display needed) | `~/data-science-venv/bin/python -c "import matplotlib; print(matplotlib.__version__)"` | `~/data-science-venv/bin/pip install matplotlib` |
| scipy | Statistical tests and regression | `~/data-science-venv/bin/python -c "import scipy; print(scipy.__version__)"` | `~/data-science-venv/bin/pip install scipy` |
| csvkit | Inspect and slice CSV files (`csvstat`, `csvlook`, `csvcut`) | `~/data-science-venv/bin/csvstat --version` | `~/data-science-venv/bin/pip install csvkit` |
| psql (PostgreSQL client) | Query Postgres directly | `psql --version` | `sudo apt install -y postgresql-client` |
| jq | Reshape JSON from the API and Prometheus | `jq --version` | `sudo apt install -y jq` |
| DuckDB CLI | Run SQL directly on CSV files | `duckdb --version` | See the single-binary note below |

> **Shared VM (RHEL, teams):** The system tools in this table (everything except Python packages in your own virtualenv) are pre-installed by the TA. Run the Verify command for each tool. If a command is not found, post in the course channel or tell the TA or professor before continuing. Do not install anything system-wide.

> **Local VM (Ubuntu, individual):** Your VM runs Ubuntu Server 26.04.1. You install tools yourself. Run the command in the last column, then the Verify command.

**Versions.** This table has no version numbers on purpose: use the current stable release of each tool, so these instructions keep working in future semesters. Record what you actually used. Run each Verify command and save the output to `capstone/data-science/TOOL-VERSIONS.txt`, then commit it. For Python packages, also save `~/data-science-venv/bin/pip freeze > capstone/data-science/requirements.txt` after you install them, so a teammate (or a future you) can rebuild the same environment.

**Create your virtualenv.** You create this yourself. It lives in your own home directory, so no sudo and no system pip are needed.

```bash
python3 -m venv ~/data-science-venv
~/data-science-venv/bin/pip install --upgrade pip
~/data-science-venv/bin/pip install pandas matplotlib scipy csvkit
```

> **Shared VM (RHEL, teams):** Each person who runs the code creates their own virtualenv from the same package list (commit it as `requirements.txt`). Never use `sudo` or system `pip`.

> **Local VM (Ubuntu, individual):** Use the virtualenv above. Do not use `pip install` outside it.

These packages publish prebuilt wheels, so nothing should compile. If `pip` starts compiling something for more than a minute, stop and ask the TA.

**DuckDB CLI (single binary).** On Ubuntu:

```bash
cd "$(mktemp -d)"
curl -fLO https://github.com/duckdb/duckdb/releases/latest/download/duckdb_cli-linux-amd64.zip
unzip -o duckdb_cli-linux-amd64.zip duckdb
sudo install -m 0755 duckdb /usr/local/bin/
```

On RHEL the TA installs it; you never do.

## Example Approaches

These are examples, not requirements. Combine them, change them, or ignore them.

**Approach A: SQL first.** Decide how categories and resolution times exist in Postgres (a derived column, a view, or an extended seed script), export with `psql` to CSV, and do most of the aggregation in SQL or DuckDB. The Python layer only draws charts. This demonstrates that a database can do most of the analytical work and that the SQL is easy to review.

**Approach B: Python pipeline.** One script pulls rows from `/incidents` (or from `psql`), loads them into pandas, cleans them, and writes every CSV and PNG in one run. This demonstrates repeatability: one command regenerates everything, and the script is the documentation.

**Approach C: Three-source triangulation.** Compare incident volume from the table with request counts from the API's Prometheus metrics (`/metrics`, scraped by Prometheus) to see whether system load and incident load move together, then use `scipy` to test it. This demonstrates how independent sources can confirm or contradict each other, and the limits of Prometheus data, which only goes back as far as your Prometheus has been running.

Optional extension: expose a small read-only summary endpoint (for example, JSON totals) from a separate small pod. This is not required. If you do it, see the Rules of the Road.

## Rules of the Road

- **No sudo on RHEL.** Students never use `sudo` on RHEL for anything. Run `k3d`, `kubectl`, `helm`, and `k6` without sudo (it cannot find them in `/usr/local/bin`). Always pass `-c <your cluster name>` to `k3d image import`; other `k3d cluster` commands take your cluster name as an argument (for example `k3d cluster list`).
- **Port-forwards.** Run `kubectl port-forward` in the foreground in its own terminal. Never background it with `&`. If something is stuck: `pkill -u $USER -f "kubectl port-forward"`.
- **Your own system only.** Query only your own team's database, API, cluster, and Prometheus port. Do not scan or probe the VM or the campus network, and do not send traffic to other teams' ports.
- **Resources are shared.** About six teams share one VM. Analysis must finish in seconds to a few minutes on CPU. No Grafana, Spark, Elasticsearch, Kafka, PyTorch/TensorFlow, or always-on JupyterLab. Run analysis scripts on the host from your venv and let them exit; do not leave them running. Aggregate in SQL or read in chunks rather than loading something huge.
- **Memory budget.** Keep anything you deploy to the cluster under 512Mi total in limits (analysis runs on the host; only the optional endpoint would be deployed). Every pod must have memory `requests` and `limits`. Check with `kubectl top pods -A` and `free -h`. If `kubectl top` errors, tell the TA, and use `kubectl describe pod <name>` and `free -h` meanwhile.
- **Seeding changes data.** Re-running a seed script appends rows. If you extend the generator, work in your own copy and keep the original unchanged. Record the exact command that produced the data you analyzed, and the row count before and after.
- **If you build an image (optional extension).** Tag it `<name>:team<N>` (the Docker daemon is shared, so other tags collide) and import with `k3d image import <name>:team<N> -c <your cluster name>`. Build with `docker build --network host -t <name>:team<N> <dir>`, because container addresses on the campus network cannot reach the internet. Do not use `build: network: host` in Compose. The VM uses `umask 077`, so edited files become mode 600; if your Dockerfile runs as a non-root user, use `COPY --chmod=644 ...` (or `chmod 644` the files before building) or the container cannot read them.
- **New pods need a NetworkPolicy.** The `default` namespace has a default-deny ingress policy, so a new pod that receives traffic needs its own allow policy or requests silently hang. You write it; start from the existing nginx-to-flask policy in `manifests/`.
- **Firewall.** Never disable firewalld or edit `/etc/docker/daemon.json`.
- **Git.** On RHEL, commit on a branch: `git switch -c <branch>`, `git push -u origin <branch>`, open a PR, wait for build-and-push, then merge. Never push to main.
- **No real data.** All data is synthetic. Do not put real names, emails, or customer details into the dataset.

## How to Run This Sprint

### Shared VM (RHEL, teams)

1. **Sprint planning.** Turn each success criterion into GitHub issues on your project board (data, analysis, charts, report, docs, QA). Estimate and assign them.
2. **Suggested roles.**
   - Scrum Master: runs standups and the board, keeps the report deadline visible.
   - System Admin: owns the environment (venv, port-forwards, database and Prometheus access, and the optional endpoint and NetworkPolicy).
   - Developers (two): one owns data creation and cleaning, one owns analysis and charts. Review each other's work.
   - QA: writes test cases in `docs/qa-report-final.md` BEFORE implementation is done, then runs them.
3. **Mid-sprint check-in.** Halfway through, meet the TA or professor: which success criteria already have evidence files, which are at risk, and what is your data decision?
4. **Retrospective.** Fill in `docs/final-sprint-retrospective.md` at the end.

### Local VM (Ubuntu, individual)

- **Week 1 milestone:** environment checkpoint passes, data decision made and documented, dataset meets criterion 1, first CSV and chart produced.
- **Week 2 milestone:** all CSVs, charts, the statistical check, report, RUNBOOK, ARCHITECTURE, and README section complete; demo rehearsed.
- Write your own test cases in the same `docs/qa-report-final.md` file (you sign as both author and tester) before you consider each piece finished.
- A short reflection (the Individual Reflection section of `docs/final-sprint-retrospective.md`) replaces the team retrospective.
- If you need a browser, use http://localhost:8081 (forwarded by VS Code). Ports are the same as above without a team number.

## Deliverables

- Implementation committed under `capstone/data-science/` in your repo (merged via PR on the RHEL section)
- `capstone/data-science/RUNBOOK.md`: how to operate, check, and recover the implementation
- `capstone/data-science/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages): the tool versions you actually used (see Tools Available)
- `capstone/data-science/ARCHITECTURE.md` with an architecture diagram (Mermaid OK) showing VM, cluster, app, and new components
- A "How to run this" section added to the repo's top-level README so someone who clones the repo can bring the track implementation up
- `docs/qa-report-final.md` with your own test cases and results
- Shared VM: `docs/final-sprint-retrospective.md`; Local VM: a short reflection (a section in the retrospective file named "Individual Reflection")
- Final presentation/demo: 8-10 minutes for a team on RHEL, 5 minutes individual on Ubuntu. Show the data decision and quality check, run the pipeline from one command in the terminal, show the CSVs and charts, and read the staffing recommendation aloud. The RHEL demo is CLI-driven.

Track-specific:
- `data_quality.csv`, the three analysis CSVs, and the PNG charts under `capstone/data-science/output/`
- `capstone/data-science/report.md` (two pages or less, written for Dana and Priya)
- `capstone/data-science/DATA.md` describing how categories and resolution times were obtained, and their limits
- The script(s) that regenerate every output from one command

## Discussion (add to Google Doc)

1. How did your team get categories and resolution times, and how would a different choice change Dana's staffing decision?
2. Which is more useful for staffing, the median or the 90th percentile resolution time? Why?
3. Your data is synthetic. What would you need to check before trusting the same analysis on real Harborlight data?
4. Does the Prometheus request data tell you anything about incident volume that the table does not? What are its limits?
5. Is your recommendation still sound if volume grows 20% before the next quarterly business review? Show the number.

## Getting Help

Clarifications go to the TA or professor. There is no wiki for this track. If a tool is missing or a command fails, tell the TA before working around it.
