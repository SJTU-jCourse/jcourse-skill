---
name: jcourse
description: 在 agent 中使用 jCourse 选课社区 API。适用于搜索课程或教师、查看课程详情、总结评价、比较课程、关注或忽略课程、管理选课记录、创建或更新课程评价、给评价投票，以及通过 HTTP 操作 jCourse 后端的场景。
---

# jCourse

使用这个 skill 将 agent 作为 jCourse 选课社区助手。回答课程推荐、评价总结和选课判断时，优先调用 API 获取课程、教师、评分、开课学期和评价数据，不要凭印象猜测。

## 连接方式

默认服务地址是 `https://course.sjtu.plus`。

默认 API 根地址是 `https://course.sjtu.plus/api`。如果用户提供了其他部署地址，改用用户提供的地址；除非用户给出的地址已经以 `/api` 结尾，否则 API 根地址为 `${base_url}/api`。

如果用户没有提供用户 API Key，先向用户索取。

使用用户 API Key 时带上：

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

使用 API Key 时，非 GET 请求会跳过 CSRF。除非用户明确提供 cookie 并要求使用浏览器会话，否则不要走浏览器会话和 CSRF 流程。

处理个性化任务前，先用 `GET /api/auth/me` 验证认证状态。如果返回 401，向用户索取有效的用户 API Key。

## 工作流程

1. 判断任务是只读还是会修改数据。创建、更新、删除、投票、关注、忽略或登记选课前，先向用户确认。
2. 做课程推荐时，先广泛搜索，再查看最相关课程的详情和评价。不要只按评分均值排序；同时考虑评分人数、评分 score、评价新旧、教师、开课学期、院系、学分和用户约束。
3. 总结评价时，把 API 返回的事实和 agent 的推断区分开。评价数量少时说明样本量有限。
4. 执行课程相关操作前，先把课程名称解析为明确的 `course_id`，并向用户展示匹配到的课程代码、课程名和教师。
5. 需要 `user_id` 的用户数据任务，先调用 `/auth/me` 获取当前用户。

## 常用任务

- 搜索课程：`GET /api/course/?q=<keyword>&page=1&page_size=10`。
- 查看课程：`GET /api/course/{course_id}`；认证后会包含开课学期、教师组、相关课程、`notification_level`、`my_enrollments` 和 `my_review`。
- 阅读课程评价：`GET /api/course/{course_id}/review?order_by=like_count&page_size=20`，必要时再调用 `GET /api/course/{course_id}/review/filter`。
- 查找教师：`GET /api/teacher/?q=<name>`，再调用 `GET /api/teacher/{teacher_id}/course` 查看该教师课程。
- 关注或忽略课程：`POST /api/course/{course_id}/notification`，`{"level":1}` 为关注，`{"level":2}` 为忽略，`{"level":0}` 为恢复普通状态。
- 创建评价：`POST /api/review/`，字段包括 `course_id`、`rating`、`content`，以及可选的 `semester` 和 `score`。
- 给评价投票：`POST /api/review/{review_id}/vote`，`vote_type` 为 `1`、`-1` 或 `0`。

## 参考资料

需要精确端点、查询参数、JSON 结构或错误处理方式时，读取 [references/api_zh_CN.md](references/api_zh_CN.md)。
