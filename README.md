import os
import sys
import requests

USERNAME = os.environ.get("GITHUB_REPOSITORY_OWNER", "NITISH28111")
TOKEN = os.environ.get("GH_TOKEN")
README_PATH = "README.md"
START = "<!-- PROJECTS:START -->
| Project | Description | Language | ⭐ Stars | Updated |
|---|---|---|---|---|
| **[Signature-Carving-and-Image-Restorer](https://github.com/NITISH28111/Signature-Carving-and-Image-Restorer)** | Recovers deleted JPEG/PNG images from raw disk sectors via signature-based carving, then classifies and restores degraded (blurry/noisy) images using a fine-tuned ResNet-50 + MPRNet — all through a unified PyQt5 GUI for digital forensics and personal data recovery. | `Python` | 0 | 2026-07-10 |
| **[monoped-rl](https://github.com/NITISH28111/monoped-rl)** | Reinforcement learning with SAC and D4PG for monopod robot hopping, balance, and forward movement in ROS/Gazebo. | `Python` | 0 | 2026-07-04 |
<!-- PROJECTS:END -->"

headers = {"Accept": "application/vnd.github+json"}
if TOKEN:
    headers["Authorization"] = f"Bearer {TOKEN}"

def fetch_repos():
    repos = []
    page = 1
    while True:
        r = requests.get(
            f"https://api.github.com/users/{USERNAME}/repos",
            headers=headers,
            params={"per_page": 100, "page": page, "sort": "updated"},
        )
        r.raise_for_status()
        batch = r.json()
        if not batch:
            break
        repos.extend(batch)
        page += 1
    # skip only the profile repo itself (username/username) — forks are included
    repos = [
        r for r in repos
        if r["name"].lower() != USERNAME.lower()
    ]
    return repos

def build_table(repos):
    if not repos:
        return "_No public repositories found._"

    header = "| Project | Description | Language |\n|---|---|---|\n"
    rows = []
    for r in repos:
        name = r["name"]
        url = r["html_url"]
        desc = (r.get("description") or "—").replace("|", "-")
        lang = r.get("language") or "—"
        rows.append(f"| **[{name}]({url})** | {desc} | `{lang}` |")
    return header + "\n".join(rows)

def update_readme(table_md):
    with open(README_PATH, "r", encoding="utf-8") as f:
        content = f.read()

    if START not in content or END not in content:
        print("Markers not found in README.md — aborting.")
        sys.exit(1)

    before = content.split(START)[0]
    after = content.split(END)[1]
    new_content = f"{before}{START}\n{table_md}\n{END}{after}"

    with open(README_PATH, "w", encoding="utf-8") as f:
        f.write(new_content)

if __name__ == "__main__":
    repos = fetch_repos()
    table = build_table(repos)
    update_readme(table)
    print(f"Updated README with {len(repos)} repos.")
