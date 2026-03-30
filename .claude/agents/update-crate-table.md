---
name: update-crate-table
description: Creates or updates the crate inventory table in profile/README.md by scanning all public repos in the atom-planet-embrace GitHub organization.
tools: Bash, Read, Edit, Write, Grep
model: sonnet
---

You are an agent that maintains a crate inventory table in `profile/README.md` for the `atom-planet-embrace` GitHub organization.

## Task

Create or update a Markdown table in `profile/README.md` with four columns: **Name**, **Version**, **Status**, and **Description**. The table lists all qualifying Rust crates from public repositories in the `atom-planet-embrace` GitHub organization.

## Step-by-step procedure

### 1. Gather all crate and repo data from GitHub

Run the following Python script to enumerate all repos, find Cargo.toml files, handle workspaces, filter crates, and discover CI workflows. The script outputs JSON to stdout.

```python
import subprocess, json, base64, re, sys, tomllib

ORG = "atom-planet-embrace"

def gh_api(endpoint):
    r = subprocess.run(["gh", "api", endpoint], capture_output=True, text=True)
    if r.returncode != 0:
        return None
    return json.loads(r.stdout)

def gh_api_raw(endpoint):
    r = subprocess.run(["gh", "api", endpoint, "--jq", ".content"], capture_output=True, text=True)
    if r.returncode != 0 or not r.stdout.strip():
        return None
    try:
        return base64.b64decode(r.stdout.strip()).decode("utf-8")
    except Exception:
        return None

def parse_toml(text):
    try:
        return tomllib.loads(text)
    except Exception:
        return None

def pick_workflow(workflows):
    """Pick the best CI workflow from a list of (filename, path) tuples."""
    if not workflows:
        return None
    for name, _ in workflows:
        if "ci" in name.lower():
            return name
    for name, _ in workflows:
        if "rust" in name.lower():
            return name
    for name, _ in workflows:
        if any(k in name.lower() for k in ("main", "build", "test")):
            return name
    return workflows[0][0]

def list_dir(repo, path):
    """List directory entries at a path in a repo."""
    data = gh_api(f"repos/{ORG}/{repo}/contents/{path}")
    if not data or not isinstance(data, list):
        return []
    return [e["name"] for e in data if e.get("type") == "dir"]

# 1. List all public repos
repos_data = json.loads(subprocess.run(
    ["gh", "repo", "list", ORG, "--visibility=public", "--json", "name", "--limit", "100"],
    capture_output=True, text=True
).stdout)
repos = [r["name"] for r in repos_data if r["name"] != ".github"]

results = []

for repo in repos:
    # Fetch root Cargo.toml
    raw = gh_api_raw(f"repos/{ORG}/{repo}/contents/Cargo.toml")
    if raw is None:
        continue
    toml = parse_toml(raw)
    if toml is None:
        continue

    # Discover workflows
    wf_data = gh_api(f"repos/{ORG}/{repo}/actions/workflows")
    active_wfs = []
    if wf_data and "workflows" in wf_data:
        for wf in wf_data["workflows"]:
            if wf.get("state") == "active":
                path = wf.get("path", "")
                filename = path.rsplit("/", 1)[-1] if "/" in path else path
                active_wfs.append((filename, path))
    workflow_file = pick_workflow(active_wfs)

    workspace_desc = None
    if "workspace" in toml:
        # Workspace-level description
        workspace_desc = toml.get("workspace", {}).get("package", {}).get("description")

        members = toml.get("workspace", {}).get("members", [])
        for member_pattern in members:
            if "*" in member_pattern:
                # Glob: list parent directory
                parent = member_pattern.split("*")[0].rstrip("/")
                dirs = list_dir(repo, parent)
                member_paths = [f"{parent}/{d}" for d in dirs]
            else:
                member_paths = [member_pattern]

            for mpath in member_paths:
                mraw = gh_api_raw(f"repos/{ORG}/{repo}/contents/{mpath}/Cargo.toml")
                if mraw is None:
                    continue
                mtoml = parse_toml(mraw)
                if mtoml is None:
                    continue
                pkg = mtoml.get("package", {})
                name = pkg.get("name")
                if not name:
                    continue
                if not (name.startswith("ai-") or name.startswith("ai_")):
                    continue
                desc = pkg.get("description")
                if isinstance(desc, dict) and desc.get("workspace"):
                    desc = workspace_desc
                results.append({
                    "crate": name,
                    "repo": repo,
                    "workflow_file": workflow_file,
                    "description": desc or "",
                })

    if "package" in toml:
        pkg = toml["package"]
        name = pkg.get("name", "")
        if name.startswith("ai-") or name.startswith("ai_"):
            desc = pkg.get("description", "")
            results.append({
                "crate": name,
                "repo": repo,
                "workflow_file": workflow_file,
                "description": desc or "",
            })

print(json.dumps(results, indent=2))
```

Save the output JSON for use in the next step.

### 2. Fetch version and description data from crates.io

Using the list of crate names from step 1, run the following Python script. Pass the JSON from step 1 via stdin.

```python
import json, sys, time, urllib.request

crates = json.load(sys.stdin)
crate_names = list({c["crate"] for c in crates})

HEADERS = {"User-Agent": "atom-planet-embrace-agent"}
DELAY = 0.1  # 100ms between requests

def fetch_crate(name):
    url = f"https://crates.io/api/v1/crates/{name}"
    req = urllib.request.Request(url, headers=HEADERS)
    try:
        with urllib.request.urlopen(req) as resp:
            data = json.loads(resp.read())
            return data.get("crate", {}).get("max_version", ""), data.get("crate", {}).get("description", "")
    except Exception:
        return "", ""

results = {}

for name in sorted(crate_names):
    # Fetch version for the ai- crate itself
    version, _ = fetch_crate(name)
    time.sleep(DELAY)

    # Fetch description from the upstream crate (strip ai- or ai_ prefix)
    if name.startswith("ai-"):
        upstream = name[3:]
    elif name.startswith("ai_"):
        upstream = name[3:]
    else:
        upstream = name

    _, upstream_desc = fetch_crate(upstream)
    time.sleep(DELAY)

    results[name] = {
        "version": version,
        "upstream_description": upstream_desc,
    }

print(json.dumps(results, indent=2))
```

Save the output JSON.

### 3. Assemble the table

Using the outputs from steps 1 and 2, build the table rows. For each crate:

#### Name
The crate's package name, linked to its repository:
```
[`{crate_name}`](https://github.com/atom-planet-embrace/{repo})
```

#### Version
Use the version from the crates.io output (step 2). If empty, leave the cell blank.

#### Status
If `workflow_file` is not null, construct a badge:
```
[![Build Status](https://github.com/atom-planet-embrace/{repo}/actions/workflows/{workflow_file}/badge.svg)](https://github.com/atom-planet-embrace/{repo}/actions)
```
If null, leave the cell empty.

#### Description
Use the **upstream description** from crates.io (step 2). If that is empty, fall back to the `description` field from the GitHub data (step 1). If both are empty, leave the cell blank.

### 4. Sort rows

Sort all rows alphabetically by crate name, but for sorting purposes strip the `ai-` or `ai_` prefix first. For example, `ai-chrono` sorts as `chrono` and `ai_color_quant` sorts as `color_quant`.

### 5. Update profile/README.md

Insert or replace the table in `profile/README.md`. The table should be placed in the `## Crates` section, positioned between the `## Philosophy` section and the `## Approach to porting` section. If a `## Crates` section already exists, replace its content. If it does not exist, insert it.

The table format:

```markdown
## Crates

| Name | Version | Status | Description |
|------|---------|--------|-------------|
| [`ai-example`](https://github.com/atom-planet-embrace/ai-example) | 1.2.3 | [![Build Status](https://github.com/atom-planet-embrace/ai-example/actions/workflows/rust.yml/badge.svg)](https://github.com/atom-planet-embrace/ai-example/actions) | A short description |
```

### Important notes

- Do not modify any other content in the README besides the `## Crates` section.
- When a workspace Cargo.toml has `workspace.package.description`, individual member crates may inherit it via `description.workspace = true` — in that case use the workspace-level description as the crate's description for the fallback.
