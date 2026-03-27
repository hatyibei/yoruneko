# Yoruneko

Claude Code Schedule 機能に特化した Skill。`/loop`・Desktop Scheduled Tasks・GitHub Actions schedule trigger を使った自律スケジュールタスクの設計・セットアップ・運用をガイドする。

## What This Does

- **方式選定**: 要件から最適な Schedule 方式を判定
- **CLAUDE.md 設計**: スケジュール用 CLAUDE.md のテンプレートと設計パターン提供
- **エスカレーション設計**: 人間に返すべき判断の境界線定義
- **通知設計**: Slack / Discord / メール通知の設計ガイド
- **アンチパターン集**: よくある失敗パターンと対処法

## Installation

### Claude Code CLI / Desktop

```bash
git clone https://github.com/hatyibei/yoruneko.git ~/.claude/skills/yoruneko
```

Or copy into an existing skills directory:

```bash
git clone https://github.com/hatyibei/yoruneko.git
cp -r yoruneko ~/.claude/skills/yoruneko
```

### Claude.ai Browser (Skills UI)

1. Open Claude.ai > Settings > Customize > Skills
2. Copy the contents of `SKILL.md` and paste as a new skill
3. Note: Browser-only installation provides the core design workflow. Templates are available in the full CLI installation.

## Usage

Claude Code will activate this skill automatically when you discuss scheduling or autonomous tasks. You can also invoke it directly:

**Examples:**
- "毎朝9時にコードレビューを自動化したい"
- "CS対応を自動化するスケジュールタスクを設計して"
- "GitHub Actions で定期的にレポートを生成したい"
- "Help me set up a scheduled monitoring task"
- "/loop と Desktop Scheduled Tasks どっちを使うべき?"

## File Structure

```
yoruneko/
├── SKILL.md                         # Core skill (method selection + design workflow)
├── templates/
│   ├── claude-md/
│   │   ├── cs-automation.md         # CS support automation template
│   │   ├── code-review.md          # Automated code review template
│   │   ├── monitoring.md           # Infrastructure monitoring template
│   │   ├── reporting.md            # Report generation template
│   │   └── custom.md               # Generic template (start here)
│   ├── api/
│   │   ├── nextjs-vercel.md        # Next.js + Vercel API pattern
│   │   └── generic-webhook.md      # Generic webhook pattern
│   └── github-actions/
│       └── schedule-trigger.yml    # GitHub Actions workflow template
├── patterns/
│   ├── DECISION_MATRIX.md          # /loop vs Desktop vs GitHub Actions comparison
│   ├── ESCALATION.md               # Escalation design guide
│   ├── NOTIFICATION.md             # Notification design (Slack/Discord/email)
│   └── ANTI_PATTERNS.md            # Common mistakes and how to avoid them
└── README.md
```

## Templates

| Template | Use Case |
|---|---|
| `cs-automation.md` | Support ticket triage, response drafting, escalation |
| `code-review.md` | Automated PR review with security and quality checks |
| `monitoring.md` | Health checks, anomaly detection, incident reporting |
| `reporting.md` | Daily/weekly report generation and distribution |
| `custom.md` | Starting point for any other scheduled task |

## Schedule Methods at a Glance

| | `/loop` | Desktop Tasks | GitHub Actions |
|---|---|---|---|
| Persistence | Session only | Permanent | Permanent |
| PC Required | Yes | Yes | No |
| OS | All | macOS/Win | All |
| Best For | Quick polls | Local tasks | 24/7 automation |

## Quality Evaluation

Evaluated against [Anthropic skill-creator](https://github.com/anthropics/skills) criteria (5 dimensions):

| Dimension | Result | Details |
|---|---|---|
| Frontmatter Completeness | PASS | `name` (6 chars, kebab-case) + `description` (479 chars) both valid |
| Description Quality | PASS | 3 methods explicitly named, 5 use cases listed, "pushy" trigger phrasing |
| Line Count Compliance | PASS | SKILL.md: 121 lines (limit: 500) |
| WHY Explanations | PASS | All 7 CLAUDE.md sections include rationale; method selection includes judgment basis |
| Eval Pass Rate | NOT TESTED | Requires skill-creator eval mode with eval.json test cases |

### Notes
- Progressive disclosure: frontmatter ~40 tokens, full SKILL.md <5k tokens, templates loaded on demand
- Browser-only graceful degradation: SKILL.md is self-sufficient without template files
- Evaluated 2026-03-27 against skill-creator v2 standards

## License

MIT
