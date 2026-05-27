# jCourse Codex Skill

[中文说明](README_zh_CN.md)

This repository publishes the `jcourse` skill for Codex agents. It teaches an agent how to use the jCourse course community API to search courses and teachers, inspect course details, summarize reviews, compare courses, manage follows or ignores, create or update reviews, vote on reviews, and operate other authenticated jCourse workflows over HTTP.

Default service URL: `https://course.sjtu.plus`.

## One-Click Agent Install

Tell Codex:

```text
Use $skill-installer to install https://github.com/SJTU-jCourse/jcourse-skill/tree/main/skills/jcourse
```

After installation, restart Codex so the new skill is loaded.

## Manual Install

If you want to install it yourself, run the Codex skill installer helper from your Codex skills directory:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --url https://github.com/SJTU-jCourse/jcourse-skill/tree/main/skills/jcourse
```

The installer copies the skill into `$CODEX_HOME/skills/jcourse`. If `CODEX_HOME` is not set, this usually means `~/.codex/skills/jcourse`.

Restart Codex after installing.

## Usage

Ask Codex to use the skill explicitly when working with jCourse:

```text
Use $jcourse to search for machine learning courses and summarize the most useful review signals.
```

For personalized or mutating actions, provide a jCourse user API key when Codex asks for it. The skill uses:

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

The skill verifies authentication with `GET /api/auth/me` before personalized work. It should confirm with you before creating, updating, deleting, voting, following, ignoring, or enrolling.

## Repository Layout

```text
skills/jcourse/
├── SKILL.md
├── SKILL_zh_CN.md
├── agents/openai.yaml
└── references/
    ├── api.md
    └── api_zh_CN.md
```

`SKILL.md` is the primary English skill file. `SKILL_zh_CN.md` and `references/api_zh_CN.md` provide Chinese instructions and API reference material.

## Verify

After installing and restarting Codex, use a prompt like:

```text
Use $jcourse to search jCourse for "数据结构" and list the top matching courses with rating count and teacher.
```

For authenticated flows, ask Codex to verify the key first:

```text
Use $jcourse with this API key: <key>. Verify my jCourse identity with /api/auth/me.
```
