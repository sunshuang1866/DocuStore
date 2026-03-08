---
name: pr-summary
description: Summarizes merged PRs from a GitCode repository for a given time period. Extracts software names, version numbers, and base OS versions from Dockerfile path conventions. Outputs a structured markdown report. Use when user asks to analyze GitCode repo PRs, summarize software updates, or generate changelog-style reports from a GitCode repository URL.
---

# PR Summary Skill

Analyzes merged Pull Requests from a GitCode repository within a specified time period and generates a structured markdown report.

## When to Use

- User provides a GitCode repository URL and asks for PR analysis
- User wants a summary of software updates merged in a given month/period
- User needs to extract version numbers from a GitCode repo's merge history

## Key Constraints

- **GitCode web UI is a JavaScript SPA** — cannot be scraped directly
- **GitCode API requires authentication** — direct API calls will fail without tokens
- **Solution**: Clone the repository via git and analyze commit history locally

---

## Workflow

### Step 1: Clone the Repository (shallow, to save time)

```bash
git clone --depth=500 <gitcode-repo-url> /tmp/repo-analysis
cd /tmp/repo-analysis
```

Use `--depth=500` to avoid fetching the full history while still capturing recent merges.

### Step 2: Extract Merge Commits for the Target Period

```bash
git log --merges --oneline --after="YYYY-MM-01" --before="YYYY-MM-28" \
  --format="%H %ad %s" --date=short
```

Or for a specific month:
```bash
git log --merges --format="%H|%ad|%s|%ae" --date=format:"%Y-%m-%d" \
  | grep "^.\{40\}|YYYY-MM"
```

### Step 3: For Each Merge Commit, Get Changed Files

```bash
git show --stat <commit-hash> | grep -E "Dockerfile|\.yml|\.md"
```

### Step 4: Extract Software and Version Info from File Paths

openeuler-docker-images uses this directory convention:

```
<Category>/<software>/<version>/<openeuler-version>/Dockerfile
```

Examples:
- `Others/tomcat/11.0.18/24.03-lts-sp3/Dockerfile` → tomcat 11.0.18 on openEuler 24.03-LTS-SP3
- `Cloud/grafana/12.3.2/24.03-lts/Dockerfile` → grafana 12.3.2 on openEuler 24.03-LTS
- `AI/mlflow/3.9.0/24.03-lts-sp3/Dockerfile` → mlflow 3.9.0 on openEuler 24.03-LTS-SP3

**Extraction regex** (Python/bash):
```bash
# From file path: extract category, software, version, openeuler-version
echo "Others/tomcat/11.0.18/24.03-lts-sp3/Dockerfile" \
  | awk -F/ '{print $1, $2, $3, $4}'
# Output: Others tomcat 11.0.18 24.03-lts-sp3
```

### Step 5: Classify PR Type from Commit Message

| Commit message pattern | PR Type |
|---|---|
| `【自动升级】` | 自动升级 (Auto-upgrade) |
| `【Bug修复】` | Bug 修复 (Bug fix) |
| `Add <software>` (new path) | 新增镜像 (New image) |
| `Fix <software>` | Bug 修复 (Bug fix) |

### Step 6: Generate Markdown Report

Save to `uploads/` or user-specified path using the template below.

---

## Output Template

```markdown
# <Repo Name> <YYYY年MM月>更新汇总

> 数据来源：<repo-url>
> 统计周期：<YYYY年MM月>
> 合并 PR 总数：**N 个**

## 概览

| 类型 | 数量 | 涉及软件 |
|---|---|---|
| 新增镜像 | N | ... |
| 自动升级 | N | ... |
| Bug 修复 | N | ... |

**涉及 openEuler 版本**：
| openEuler 版本 | 涉及 PR 数量 |
|---|---|
| 24.03-LTS-SP3 | N |

## 软件更新清单（按分类）

### <Category> 类
| 软件 | 新版本 | openEuler 版本 | PR | 类型 |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## PR 详情列表

| PR | 合并时间 | 标题 | 类型 |
|---|---|---|---|
| !NNNN | YYYY-MM-DD | ... | ... |

## 重要说明

- 新增镜像亮点
- Bug 修复说明
- 活动规律统计

*文档生成时间：YYYY-MM-DD*
```

---

## Delegating to Subagent (Recommended)

For efficiency, delegate the git clone + analysis to a `general-purpose` subagent with this prompt template:

```
Fetch and analyze merged PRs from GitCode repository <URL> merged in <YYYY年MM月>.

Steps:
1. Clone: git clone --depth=500 <URL> /tmp/repo-<slug>
2. Extract merge commits: git log --merges with date filter for YYYY-MM
3. For each merge commit, run: git show --stat <hash>
4. Parse file paths to extract: category, software, version, openeuler-version
5. Classify PR type from commit subject
6. Return structured table: PR#, date, title, category, software, version, openeuler-version, type
```

The subagent returns structured data; the main agent writes the markdown report.

---

## Example Usage

**User**: 查看仓库 https://gitcode.com/openeuler/openeuler-docker-images/ 今年2月合并的PR，总结更新的软件和版本号

**Steps**:
1. Spawn `general-purpose` subagent to clone + analyze git history
2. Parse results into software/version/openeuler-version table
3. Write report to `uploads/<repo-slug>-<YYYY-MM>.md`

---

## Notes

- openeuler-docker-images path convention: `<Category>/<software>/<version>/<oe-version>/Dockerfile`
- GitCode MR numbers appear in merge commit subjects as `!NNNN` (not `#NNNN`)
- `openeuler-ci-bot` is the bot account that performs actual merges; look at commit author in PR title for real contributor
- Auto-upgrades are always batched — infra_team submits many on the same day
- Always clean up: `rm -rf /tmp/repo-<slug>` after analysis
