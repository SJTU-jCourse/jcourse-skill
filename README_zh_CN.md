# jCourse Skill

[English](README.md)

这个仓库发布 `jcourse` skill。安装后，agent 可以按 jCourse API 的约定搜索课程和教师、查看课程详情、总结评价、比较课程、关注或忽略课程、管理选课记录、创建或更新评价、给评价投票，以及执行其他需要认证的 jCourse 工作流。

默认服务地址：`https://course.sjtu.plus`。

## 让 Agent 一键安装

把下面这句话发给 Agent：

```text
Use $skill-installer to install https://github.com/SJTU-jCourse/jcourse-skill/tree/main/skills/jcourse
```

安装完成后，重启 Agent 让新 skill 生效。

## 手动安装

也可以从你的 Agent skills 目录直接运行安装脚本：

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --url https://github.com/SJTU-jCourse/jcourse-skill/tree/main/skills/jcourse
```

安装脚本会把 skill 复制到 `$CODEX_HOME/skills/jcourse`。如果没有设置 `CODEX_HOME`，通常会安装到 `~/.codex/skills/jcourse`。

安装后需要重启 Agent。

## 使用方式

处理 jCourse 相关任务时，可以明确要求 Agent 使用这个 skill：

```text
Use $jcourse to search jCourse for "数据结构" and list the top matching courses with rating count and teacher.
```

个性化或会修改数据的操作需要 jCourse 用户 API Key。Agent 会在需要时向你索取，并使用下面的请求头：

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

处理个性化任务前，skill 会先用 `GET /api/auth/me` 验证身份。创建、更新、删除、投票、关注、忽略或登记选课前，agent 应先向你确认。

## 仓库结构

```text
skills/jcourse/
├── SKILL.md
├── SKILL_zh_CN.md
├── agents/openai.yaml
└── references/
    ├── api.md
    └── api_zh_CN.md
```

`SKILL.md` 是英文主 skill 文件。`SKILL_zh_CN.md` 和 `references/api_zh_CN.md` 提供中文说明和 API 参考。

## 验证安装

安装并重启 Agent 后，可以用下面的提示词验证：

```text
Use $jcourse to search jCourse for "数据结构" and list the top matching courses with rating count and teacher.
```

如果要验证认证流程，可以让 Agent 先检查 API Key：

```text
Use $jcourse with this API key: <key>. Verify my jCourse identity with /api/auth/me.
```
