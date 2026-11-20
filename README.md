# Capstone Track: Applied Data Science
 
**Final Sprint | Due before Final Presentations**

> **New to this track? Start here.** The shared [INET 4031 Capstone Guide](https://github.com/INET4031-Labs/inet4031-capstone-guide) explains how to work through any track on your own. Feeling lost in the first few days is normal for a capstone: read the guide's Start Here page, run the Environment Checkpoint below, and take the next small step.
>
> | I need... | Go to |
> |---|---|
> | A reading order and what to do in my first session | [START-HERE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/START-HERE.md) |
> | What I may change in the app, and how to propose my own criterion | [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) |
> | Local VM: how my k3s and OpenTofu setup differs from this README | [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) |
> | What to prioritize in Weeks 11 to 14 | [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md) and the Sprint Timeline below |
> | Plain-language explanations of this track's concepts | [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) |
> | A cheat sheet for each tool in this track | [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md) |
> | Which file proves which success criterion | [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md) |
> | How the Week 13 rebuild works | [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md) and the Rebuild section below |
> | Help when something is broken | [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) |
> | How to use AI tools well on this project | [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md) |
>
> **Local VM students:** much of this README describes the Shared VM setup (k3d, team numbers, `main`, `kubectl apply`). Your setup is a single-node k3s cluster that changes through OpenTofu and your deploy script. Look for the **Local VM** notes in each section, and keep [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) open while you work.
 
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
 
Everything from earlier weeks must still work: the Week 2 Compose stack with Postgres 15 and the Flask API, the Week 3 k3d cluster, and the Week 5 Prometheus. The incidents you seeded in earlier labs are a starting point but not enough on their own (see The Business Problem).

> **Local VM (Ubuntu, individual):** your data lives in the `tickets` table (same columns as `incidents`) in the `db` pod on your k3s cluster, managed by OpenTofu since Week 5. Your Compose stack has been stopped since Week 4, there is no `seed.py`, and Prometheus is from Week 6. Week 10's 10,000 `Seeded ticket` rows, if you kept them, span about a week and have no categories, so criterion 1 needs a generator of your own under `capstone/data-science/`.
 
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
 
**Local VM:** use this instead (`sql.sh` is your Week 10 helper, `pq.sh` your Week 6 one):

```bash
curl -s http://$(kubectl get service web -o jsonpath='{.spec.clusterIP}'):8080/health
./week-10/sql.sh "SELECT COUNT(*), MIN(created_at), MAX(created_at) FROM tickets"
kubectl get nodes
kubectl get pods -A
# In its own terminal:
kubectl port-forward --namespace monitoring service/prometheus-server 9090:80
# Then:
./week-6/pq.sh 'up'
```

You should see one line per target ending in `-> 1`. (`pq.sh` is the Week 5 helper on the Shared VM; for raw JSON, `curl -s "$PROM_URL/api/v1/query" --data-urlencode 'query=up' | jq .status` returns `"success"`.) If the port-forward dies or hangs: `pkill -u $USER -f "kubectl port-forward"` and start it again. (The Compose port is 8080 + your team number, so team 3 uses 8083.)
 
## Team Port Assignments
 
This track needs no new ports. You use only the ports you already have:
 
- Compose app: 8080 + team number
- Kubernetes (k3d load balancer): 8090 + team number
- Prometheus port-forward: 9090 + team number
> **Shared VM (RHEL, teams):** Use only your own team's ports. Do not send traffic to another team's ports.
 
> **Local VM (Ubuntu, individual):** There is no team number. The app is on port 8080 (the `web` Service, or `kubectl port-forward --address 0.0.0.0 service/web 8080:8080` for your browser at http://localhost:8080) and Prometheus on 9090.
 
## Tools Available
 
| Tool | What it's for | Verify (both OSs) | Ubuntu install |
| ---- | ------------- | ----------------- | -------------- |
| [Python 3 + venv](https://docs.python.org/3/library/venv.html) | Run your analysis scripts | `python3 --version` | `sudo apt install -y python3 python3-venv` |
| [pandas](https://pandas.pydata.org/docs/) | Tables, grouping, resampling | `~/data-science-venv/bin/python -c "import pandas; print(pandas.__version__)"` | `~/data-science-venv/bin/pip install pandas` |
| [matplotlib](https://matplotlib.org/stable/) | Charts saved as PNG (use the `Agg` backend; no display needed) | `~/data-science-venv/bin/python -c "import matplotlib; print(matplotlib.__version__)"` | `~/data-science-venv/bin/pip install matplotlib` |
| [scipy](https://docs.scipy.org/doc/scipy/) | Statistical tests and regression | `~/data-science-venv/bin/python -c "import scipy; print(scipy.__version__)"` | `~/data-science-venv/bin/pip install scipy` |
| [csvkit](https://csvkit.readthedocs.io/) | Inspect and slice CSV files (`csvstat`, `csvlook`, `csvcut`) | `~/data-science-venv/bin/csvstat --version` | `~/data-science-venv/bin/pip install csvkit` |
| [psql (PostgreSQL client)](https://www.postgresql.org/docs/current/app-psql.html) | Query Postgres directly | `psql --version` | `sudo apt install -y postgresql-client` |
| [jq](https://jqlang.org/manual/) | Reshape JSON from the API and Prometheus | `jq --version` | `sudo apt install -y jq` |
| [DuckDB CLI](https://duckdb.org/docs/) | Run SQL directly on CSV files | `duckdb --version` | See the single-binary note below |
 
> **Shared VM (RHEL, teams):** The system tools in this table (everything except Python packages in your own virtualenv) are pre-installed by the TA. Run the Verify command for each tool. If a command is not found, post in the course channel or tell the TA or professor before continuing. Do not install anything system-wide.
 
> **Local VM (Ubuntu, individual):** Your VM runs Ubuntu Server 26.04.1. You install tools yourself. Run the command in the last column, then the Verify command. The `psql` client is optional for you: the database runs in the `db` pod, and `./week-10/sql.sh` or `kubectl exec -i deploy/db -- psql ...` reach it without a local client.
 
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
curl -fLO https://github.com/duckdb/duckdb/releases/latest/download/duckdb-cli-linux-amd64.tar.gz
tar -xzf duckdb-cli-linux-amd64.tar.gz
sudo install -m 0755 "$(find . -type f -name duckdb | head -n 1)" /usr/local/bin/
```

If the download returns "404 Not Found", DuckDB renamed its release file again (it changed from `duckdb_cli-linux-amd64.zip` to `duckdb-cli-linux-amd64.tar.gz` in version 2.0). Open the [releases page](https://github.com/duckdb/duckdb/releases/latest), copy the Linux CLI file name (not the `-musl` one), and adjust the command.
 
On RHEL the TA installs it; you never do.
 
## Example Approaches
 
These are examples, not requirements. Combine them, change them, or ignore them.
 
**Approach A: SQL first.** Decide how categories and resolution times exist in Postgres (a derived column, a view, or an extended seed script), export with `psql` to CSV, and do most of the aggregation in SQL or DuckDB. The Python layer only draws charts. This demonstrates that a database can do most of the analytical work and that the SQL is easy to review.
 
**Approach B: Python pipeline.** One script pulls rows from `/incidents` (or from `psql`), loads them into pandas, cleans them, and writes every CSV and PNG in one run. This demonstrates repeatability: one command regenerates everything, and the script is the documentation.
 
**Approach C: Three-source triangulation.** Compare incident volume from the table with request counts from the API's Prometheus metrics (`/metrics`, scraped by Prometheus) to see whether system load and incident load move together, then use `scipy` to test it. This demonstrates how independent sources can confirm or contradict each other, and the limits of Prometheus data, which only goes back as far as your Prometheus has been running.
 
Optional extension: expose a small read-only summary endpoint (for example, JSON totals) from a separate small pod. This is not required. If you do it, see the Rules of the Road.

> **Local VM:** your app has no JSON `/incidents` endpoint and no `/metrics` endpoint, so Approach B pulls from the database, and Approach C has no request counts from the app. Prometheus does have your Week 9 database exporter metrics and container CPU and memory, which you can compare with ticket volume. Adding a JSON or `/metrics` endpoint to the app is a good [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) change if you want it.
 
## Track Tips and Common Trip-Ups

These notes come from where students usually get stuck. They do not change the success criteria; they help you reach them. Concepts are explained in [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) and commands in [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md). If a flag here does not work on your installed version, check the tool's `--help`.

**The pipeline at a glance**

Every approach above passes through the same stages. Knowing them helps you split the work and name your scripts:

1. **Generate:** a script with a fixed random seed writes synthetic incidents.
2. **Load:** put them in the database, or keep them as a CSV if you analyze outside the database (say which in `DATA.md`).
3. **Extract:** pull the rows you analyze into a CSV in your repo.
4. **Check quality:** count rows, nulls, and the date range, and write `data_quality.csv`.
5. **Aggregate:** produce the three analysis CSVs.
6. **Chart:** produce the three PNGs.
7. **Test:** run your statistical check and save the number.
8. **Write:** `report.md` for Dana.

Your "one command" for criterion 2 runs stages 3 to 7 at least. Steps 1 and 2 can be separate, as long as the README says how to run them.

**The data decision (criterion 1)**

- **What one generated row could look like.** Think in columns before you think in code. A row needs a title, a category, a status, a `created_at`, and, for closed rows, a resolution time (as a `resolved_at` timestamp or a duration). Then decide how each column behaves:
  - *Category:* a handful of categories that fit a logistics company (for example scanners and handhelds, the dispatch app, printers and labels, logins and access, network), each with its own list of titles so the title hints at the category without giving it away every time.
  - *`created_at`:* spread over more than 90 days, with more incidents on weekdays and during working hours, and a small week-over-week growth if you want a trend to find.
  - *Resolution time:* skewed, not uniform. Most incidents close in a few hours, a few take days. Each category gets a different typical value. A search for "lognormal distribution" will show you one common way to draw numbers with that shape.
  - *Status:* older incidents are mostly closed; the most recent ones are more likely to be open or in progress, which is what a real queue looks like.
- Write these choices in `DATA.md` before you write the generator. That list is your plan, and later it is the "what we planted" section of your report.

- Your analysis can only find patterns your data contains. A useful approach is to **plant a signal and then recover it**: build realistic structure into your generator (volume that grows over the weeks, busier weekdays and hours, categories with different resolution times and long tails), then check that your analysis finds it. Say plainly in `DATA.md` and the report what you planted. That is honest synthetic-data practice.
- Reseeding appends rows. Pick one clean approach and document it, for example loading into a fresh table, or clearing your own team's table before a full reload. Record the exact command and the row counts before and after.

**Analysis (criteria 2 to 4)**

- Pull data to a CSV from the host so the file lands in your repo, not inside the container: `docker compose -f week-2/docker-compose.yml exec -T db psql -U appuser -d statustracker --csv -c "SELECT ..." > capstone/data-science/output/raw.csv`.
- Call `matplotlib.use("Agg")` before importing `pyplot`, since the VM has no display. Every chart needs a title, labeled axes, and units.
- Resolution times are usually skewed, so compare medians and the 90th percentile rather than means. For a trend, `scipy.stats.linregress` gives a slope and p-value. For comparing two categories' resolution times, a rank-based test such as `scipy.stats.mannwhitneyu` suits skewed data better than a t-test.
- Write the plain-English sentence for your statistical check as if Dana will read it: what the number says, and what it does not say.

**The staffing recommendation (criterion 5)**

- Resolution time is how long an incident stays open, not how long an agent works on it. A staffing number needs an assumption about handling time. One simple, transparent model: agents needed in an hour = (incidents arriving per hour x hours of agent work per incident) / share of the hour an agent spends on incidents. State every input, show the arithmetic, and test the 20% growth case for discussion question 5.
- **You will find Erlang C.** Searching for call center or service desk staffing leads to the Erlang C formula and workforce-management tools. That is what real support teams use, and you may mention it in your report or discussion answers. The simple model above is enough for criterion 5; do not spend the sprint implementing queueing theory.
- Prometheus only has data from the time your Prometheus has been running. If you use Approach C, say what time range it covers.

**The report (criterion 5)**

Dana is not technical, has five minutes, and needs a number. One outline that works (section names only; the content is yours):

1. **The question:** one or two sentences in Dana's words.
2. **What we found:** three findings, each pointing to one of your charts.
3. **How we got the number:** the staffing arithmetic, with every input shown.
4. **Assumptions and limits:** handling time, synthetic data and what you planted, what would change the answer.
5. **If volume grows 20%:** the same arithmetic with the higher volume.
6. **Recommendation:** the number or range of agents, and which hours or categories need the coverage.

The criterion asks the report to end with the recommendation. You may also state it in one line at the very top so Dana sees it first.

**Local VM notes**

- **Export from the `db` pod to a file on the VM:** `kubectl exec -i deploy/db -- sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" --csv -c "SELECT ..."' > capstone/data-science/output/raw.csv`. The single quotes keep `$POSTGRES_USER` and `$POSTGRES_DB` for the pod's own environment.
- **Load generated rows the same way in reverse:** `kubectl exec -i deploy/db -- sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "\copy tickets (title, description, status, created_at) FROM STDIN WITH CSV HEADER"' < capstone/data-science/data/generated.csv`. Add columns first if your data needs them (see [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) for safe schema changes).
- **Back up before a bulk load.** Run your Week 9 `./week-9/backup.sh` first, so you can restore if a load goes wrong.
- **Or analyze outside the database.** Your generator can write CSV files that pandas or DuckDB read directly. Criterion 1 is about the dataset you analyze, so say in `DATA.md` which path you chose and why.

## Rules of the Road
 
- **No sudo on RHEL.** Students never use `sudo` on RHEL for anything. Run `k3d`, `kubectl`, `helm`, and `k6` without sudo (it cannot find them in `/usr/local/bin`). Always pass `-c <your cluster name>` to `k3d image import`; other `k3d cluster` commands take your cluster name as an argument (for example `k3d cluster list`).
- **Port-forwards.** Run `kubectl port-forward` in the foreground in its own terminal. Never background it with `&`. If something is stuck: `pkill -u $USER -f "kubectl port-forward"`.
- **Your own system only.** Query only your own team's database, API, cluster, and Prometheus port. Do not scan or probe the VM or the campus network, and do not send traffic to other teams' ports.
- **Resources are shared.** About six teams share one VM. Analysis must finish in seconds to a few minutes on CPU. No Grafana, Spark, Elasticsearch, Kafka, PyTorch/TensorFlow, or always-on JupyterLab. Run analysis scripts on the host from your venv and let them exit; do not leave them running. Aggregate in SQL or read in chunks rather than loading something huge.
- **Memory budget.** Keep anything you deploy to the cluster under 512Mi total in limits (analysis runs on the host; only the optional endpoint would be deployed). Every pod must have memory `requests` and `limits`. Check with `kubectl top pods -A` and `free -h`. If `kubectl top` errors, tell the TA, and use `kubectl describe pod <name>` and `free -h` meanwhile.
- **Seeding changes data.** Re-running a seed script appends rows. You may change the generator and the app (see [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md)): commit every change, keep it safe to run twice, and record the exact command that produced the data you analyzed, and the row count before and after.
- **If you build an image (optional extension).** Tag it `<name>:team<N>` (the Docker daemon is shared, so other tags collide) and import with `k3d image import <name>:team<N> -c <your cluster name>`. Build with `docker build --network host -t <name>:team<N> <dir>`, because container addresses on the campus network cannot reach the internet. Do not use `build: network: host` in Compose. The VM uses `umask 077`, so edited files become mode 600; if your Dockerfile runs as a non-root user, use `COPY --chmod=644 ...` (or `chmod 644` the files before building) or the container cannot read them.
- **New pods need a NetworkPolicy.** The `default` namespace has a default-deny ingress policy, so a new pod that receives traffic needs its own allow policy or requests silently hang. You write it; start from the existing nginx-to-flask policy in `manifests/`.
- **Firewall.** Never disable firewalld or edit `/etc/docker/daemon.json`.
- **Git.** On RHEL, commit on a branch: `git switch -c <branch>`, `git push -u origin <branch>`, open a PR, wait for build-and-push, then merge. Never push to main.
- **No real data.** All data is synthetic. Do not put real names, emails, or customer details into the dataset.

### Local VM (Ubuntu, individual) rules

The rules above that mention other teams, team numbers, k3d, `--network host`, firewalld, `/etc/docker/daemon.json`, `umask 077`, or [PREDICT ONLY] describe the Shared VM. On your own VM these apply instead:

- **Change the cluster through OpenTofu.** Anything that should last goes in `week-5/*.tf`, through a pull request, the Deploy gate, and `week-7/deploy.sh`. Throwaway test pods may use `kubectl apply`, and you delete them afterward. A change made with `kubectl` to an object OpenTofu manages is undone by the next deploy.
- **The gate checks your additions.** Every container you add in `week-5/` needs CPU and memory requests and limits and must not allow privilege escalation, or the `security scan` job fails and the ruleset blocks the merge.
- **New image, new tag.** Build a new tag (for example `:v1`, `:v2`), import it with `docker save` and `sudo k3s ctr images import` **before** the pull request that uses it, and never overwrite the tag that is running.
- **`master` and `protect-master` stay.** Work on branches and merge through pull requests. Do not delete the ruleset, remove its checks, or add a bypass.
- **No default-deny.** Only the `db` pods are protected (by `db-clients`). A pod you add accepts traffic from anywhere until a NetworkPolicy selects it.
- **sudo** is for package installs, `sudo install` of single binaries, and `sudo k3s ctr`. Run `kubectl`, `helm`, `tofu`, and `k6` as your normal user.
- **Memory budget still applies:** anything you deploy (only the optional endpoint) stays under 512Mi of limits. Analysis runs on the VM from your venv and exits.
- **The optional endpoint** ships through `week-5/` like any other service, with its own image tag imported into k3s and a NetworkPolicy if you want to limit who reaches it.

## How to Run This Sprint
 
### Sprint Timeline (Weeks 11 to 14)

Both sections follow the same four weeks. The planning steps below still apply; this table says where your work should be at the end of each week. The full plan is in [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md).

| Week | By the end of the week |
|---|---|
| 11 Kickoff | Environment Checkpoint saved. Data decision written in `DATA.md` (how categories and resolution times exist, and their limits). Dataset meets criterion 1. `data_quality.csv` committed. QA test cases drafted. |
| 12 Build | Three analysis CSVs, three labeled charts, and one statistical check, all regenerated by one command. |
| 13 Consolidate and rebuild | `report.md` with the staffing recommendation and assumptions, RUNBOOK and ARCHITECTURE, rebuild, and gap log. |
| 14 Present | Rebuild presentation (live on the Shared VM, recorded on the Local VM), running the pipeline from one command and reading the recommendation aloud. |

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
 
You follow the same Sprint Timeline as the Shared VM teams, alone.

- **Week 11 milestone:** Environment Checkpoint saved, data decision written in `DATA.md`, dataset meets criterion 1, `data_quality.csv` committed, test cases drafted.
- **Week 12 milestone:** three analysis CSVs, three labeled charts, and one statistical check, all regenerated by one command.
- **Week 13:** `report.md` with the staffing recommendation and assumptions, RUNBOOK, ARCHITECTURE, and README section, then rebuild on a new VirtualBox VM and keep a gap log (see the Rebuild section).
- **Week 14:** recorded presentation of your rebuild.
- Write your own test cases in the same `docs/qa-report-final.md` file (you sign as both author and tester) before you consider each piece finished.
- A short reflection (the Individual Reflection section of `docs/final-sprint-retrospective.md`) replaces the team retrospective.
- If you need a browser, run `kubectl port-forward --address 0.0.0.0 service/web 8080:8080` and open http://localhost:8080. Ports are the same as above without a team number.

## Rebuild (Weeks 13 and 14)

Your work has to come back up from your repository alone. Start from a clean VM, clone your repo, and follow only your README, RUNBOOK, and scripts. Every step that fails or that your README forgot is a gap: write it in `capstone/data-science/REBUILD-GAPS.md`, fix it by pull request, and update the README. The full contract, the "what lives outside git" checklist, and the gap log template are in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md).

> **Shared VM (RHEL, teams):** In Week 13 your team takes a turn in a queue to test the rebuild on a separate VM, which is wiped between teams. In Week 14 you present live: the tools are pre-installed, and your team clones the repo and brings the app up from your README and setup instructions. There is no sudo on the rebuild or presentation VM, so every step your README asks someone to run must work without it. Your Ansible playbooks and any [PREDICT ONLY] code stay reference-only: do not run them, and make sure bringing the app up does not depend on them. Recreate your k3d cluster with the same `--subnet 100.97.<team>.0/24` flag you used before.

> **Local VM (Ubuntu, individual):** In Week 13 create a new VM in VirtualBox using the same steps you used for your original VM, clone your repo, and follow your README. Keep the original VM until the rebuild works. In Week 14 you present a recording of the rebuild. Start from the "Rebuild on a fresh VM" sections you wrote in your Week 7, 8, and 9 READMEs (k3s, OpenTofu variables, image import, deploy and backup cron lines), then add everything this track added. The full Local VM order is in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md). You have sudo on your own VM; every step that needs it must be in your README.

**Proof of life for this track:** one command regenerates every CSV and PNG; row counts match `data_quality.csv`.

**Gaps this track usually finds:**
- The database starts empty on a new VM, and the seed command (or whether it is safe to run twice) is not documented.
- The analysis depends on a CSV that was exported by hand and never committed or scripted.
- Prometheus history does not exist on a fresh cluster, so Approach C results cannot be reproduced without an export or an explanation.
- Local VM: the generated rows were loaded into the old VM's database by hand, and the load command (or the restore from a backup) is not in the README.

## Deliverables
 
- Implementation committed under `capstone/data-science/` in your repo (merged via PR on the RHEL section)
- `capstone/data-science/RUNBOOK.md`: how to operate, check, and recover the implementation
- `capstone/data-science/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages): the tool versions you actually used (see Tools Available)
- `capstone/data-science/ARCHITECTURE.md` with an architecture diagram (Mermaid OK) showing VM, cluster, app, and new components
- A "How to run this" section added to the repo's top-level README so someone who clones the repo can bring the track implementation up
- `docs/qa-report-final.md` with your own test cases and results
- `capstone/data-science/REBUILD-GAPS.md`: the gap log from your Week 13 rebuild (template in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md))
- Shared VM: `docs/final-sprint-retrospective.md`; Local VM: a short reflection (a section in the retrospective file named "Individual Reflection")
- Final presentation/demo: 8-10 minutes for a team on RHEL, 5 minutes individual on Ubuntu. Show the data decision and quality check, run the pipeline from one command in the terminal, show the CSVs and charts, and read the staffing recommendation aloud. The RHEL demo is CLI-driven.
- Week 14 rebuild presentation: live on the Shared VM (clone the repo and bring the app up from your README), recorded on the Local VM. See the Rebuild section.
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

**Before you ask:** work through [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) for about 30 minutes, then ask with the command, its full output, what you expected, and what you tried.

**Using AI:** AI assistants are welcome as a tool for learning how these tools work and how enterprises use them. Do not rely on them to produce your work: understand every line you commit, verify AI answers against the tool's documentation, never paste AI output into evidence files, and never paste secrets into an AI tool. See [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md).
