# jCourse API 参考

## 目录

- 认证和请求规则
- 分页和查询参数
- 课程 API
- 评价 API
- 教师 API
- 选课记录 API
- 用户、设置、API Key、积分和公告 API
- 响应结构
- 错误处理
- agent 示例流程

## 认证和请求规则

默认服务地址：`https://course.sjtu.plus`。

默认 API 根地址：`https://course.sjtu.plus/api`。

任意部署的基础路径：`/api`。

agent 优先使用用户 API Key：

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

Bearer API Key 会优先于 session 认证解析。用户 API Key 会以该用户身份访问。系统 API Key 仅用于 `/api/ext/*` 和管理员创建的系统集成。

浏览器 session 的非 GET 请求需要 CSRF。有效 API Key 会跳过 CSRF，所以 agent 应使用 API Key。

个性化任务前先验证 API Key：

```http
GET /api/auth/me
```

响应示例：

```json
{"id":1,"username":"u1","email":"u1@example.edu","role":"user"}
```

## 分页和查询参数

分页接口返回：

```json
{"items":[],"total":0,"page":1,"page_size":20}
```

默认通常是 `page=1&page_size=20`。数组筛选参数通过重复 key 编码：

```text
/api/course/?categories=CORE&categories=ELECTIVE&target_years=2026
```

布尔查询参数按前端实现使用 `0` 或 `1`。

## 课程 API

### 获取课程筛选项

```http
GET /api/course/filter
```

返回筛选项计数：`credits`、`departments`、`categories`、`target_years`、`languages`、`semesters`。每项为 `{name,count}`。

### 列出课程

```http
GET /api/course/?q=&department=&language=&categories=&target_years=&credit=&has_review=&order_by=&ascend=&page=&page_size=
```

查询参数：

- `q`：文本搜索。
- `department`、`language`、`credit`：精确筛选。
- `categories`、`target_years`：可重复筛选。
- `has_review`：`1` 或 `0`。
- `order_by`：`rating_score`、`rating_count` 或 `rating_avg`。
- `ascend`：`1` 表示升序，省略或 `0` 表示降序。

课程列表项字段：`id`、`code`、`name`、`credit`、`department`、`language`、`target_years`、`categories`、`main_teacher`、`rating`。

`rating` 包含 `count`、`avg`、`score`、`distribution`，其中 `distribution` 是 1-5 星数量的 5 元数组。

### 热门课程

```http
GET /api/course/hot?period=week&limit=5
```

`period` 为 `week` 或 `month`。响应包含 `period`、`period_key` 和 `items`，其中每项是 `{course,score}`。热门 `score` 是活跃分，不是评分。

### 课程详情

```http
GET /api/course/{course_id}
```

返回课程列表字段，并额外包含：

- `last_semester`
- `teacher_group`
- `offered_courses`：`{semester,language,target_years,categories}`
- `same_code_courses`
- `same_teacher_courses`
- `notification_level`：`0` 普通，`1` 关注，`2` 忽略
- `my_enrollments`：认证后且存在时返回
- `my_review`：认证后且存在时返回

按课程名执行操作前，先调用详情接口确认课程。

### 课程评价筛选和趋势

```http
GET /api/course/{course_id}/review/filter
GET /api/course/{course_id}/review/trend
```

筛选接口返回可用 `semesters` 和 `ratings`。趋势接口返回 `[{"semester":"2025-2026-1","avg":4.2,"count":8}]`。

### 关注或忽略课程

```http
POST /api/course/{course_id}/notification

{"level":1}
```

`level`：`0` 普通，`1` 关注，`2` 忽略。修改前向用户确认。

### 已关注和已忽略课程

```http
GET /api/course/followed?page=1&page_size=20
GET /api/course/ignored?page=1&page_size=20
```

支持课程列表的分页和排序参数。

## 评价 API

### 全站评价列表

```http
GET /api/review?q=&semester=&rating=&order_by=created_at&ascend=&page=&page_size=
```

`order_by` 可为 `created_at` 或 `like_count`。认证用户的全站评价列表会排除已忽略课程。

### 某课程评价列表

```http
GET /api/course/{course_id}/review?q=&semester=&rating=&order_by=like_count&page=1&page_size=20
```

做代表性评价总结时使用 `order_by=like_count`，看近期反馈时使用 `order_by=created_at`。

### 已关注课程的评价

```http
GET /api/review/followed?page=1&page_size=20
```

返回已关注课程的评价。

### 获取单条评价

```http
GET /api/review/{review_id}
```

评价字段：`id`、`course`、`course_id`、可见时的 `user_id`、`semester`、`score`、`rating`、`content`、`moderator_remark`、`vote`、`created_at`、`updated_at`。

`vote` 包含 `like_count`、`dislike_count`，以及可选的 `my_vote`。

### 创建评价

```http
POST /api/review/

{"course_id":123,"semester":"2025-2026-1","rating":5,"score":"A","content":"..."}
```

必填：`course_id`、`rating`、`content`。可选：`semester`、`score`。发布前让用户确认最终文本。敏感内容可能返回 400。

### 更新或删除评价

```http
PUT /api/review/{review_id}
DELETE /api/review/{review_id}
```

更新请求体：

```json
{"semester":"2025-2026-1","rating":4,"score":"A-","content":"..."}
```

删除权限由后端策略控制，通常需要评价所有者或管理员权限。修改前先确认。

### 给评价投票

```http
POST /api/review/{review_id}/vote

{"vote_type":1}
```

`vote_type`：`1` 赞同，`-1` 反对，`0` 取消投票。触发每日投票上限时返回 429。

### 用户评价

```http
GET /api/user/{user_id}/review?page=1&page_size=20
```

需要本人或管理员权限。

## 教师 API

### 获取教师筛选项

```http
GET /api/teacher/filter
```

返回 `departments` 和 `titles` 的筛选项计数。

### 列出教师

```http
GET /api/teacher/?q=&department=&title=&page=&page_size=
```

教师字段：`id`、`code`、`name`、`department`、`title`。

### 教师详情和教师课程

```http
GET /api/teacher/{teacher_id}
GET /api/teacher/{teacher_id}/course?q=&order_by=rating_score&page=1&page_size=20
```

教师课程接口支持课程列表筛选参数。

## 选课记录 API

### 我的选课记录

```http
GET /api/course/enrolled
```

返回 `[{"id":1,"course":{...},"semester":"2025-2026-1","created_at":"..."}]`。

### 添加或删除选课记录

```http
POST /api/course/{course_id}/enrollment
DELETE /api/course/enrollment/{enrollment_id}
```

创建请求体：

```json
{"semester":"2025-2026-1"}
```

学期必须是该课程开课学期，或满足后端对最近开课学期的策略。修改选课记录前先确认。

### JAccount 选课同步

```http
GET /api/course/enrollment-sync/start?semester=2025-2026-1
GET /api/course/enrollment-sync/callback?code=...&state=...
```

这是依赖浏览器 session 状态的 OAuth 类跳转流程。agent 场景优先使用手动选课记录；只有用户明确要求打开同步链接时再使用。

## 用户、设置、API Key、积分和公告 API

### 账户

```http
GET /api/auth/me
POST /api/auth/logout
```

公开 session 端点包括：`/api/auth/register/code`、`/api/auth/register`、`/api/auth/login`、`/api/auth/password-reset/code`、`/api/auth/password-reset`。agent 流程避免索取用户密码，优先使用 API Key。

### 用户设置

```http
GET /api/user/settings
PUT /api/user/settings

{"current_semester":"2025-2026-1"}
```

### API Key

```http
GET /api/api-key/
POST /api/api-key/
DELETE /api/api-key/{api_key_id}
```

创建请求体：`{"name":"agent"}`。创建接口会返回 key 字符串。把它视作密钥，不要主动回显，除非用户明确要求。

管理员系统 API Key：

```http
GET /api/admin/api-key/system
POST /api/admin/api-key/system
DELETE /api/admin/api-key/system/{api_key_id}
```

### 积分

```http
GET /api/user/{user_id}/point?page=1&page_size=20
```

返回用户积分总额和分页积分记录。需要本人或管理员权限。

系统集成查询端点：

```http
GET /api/ext/point?email=user@example.edu
```

需要系统 API Key。

### 公告

```http
GET /api/announcement/
```

返回有效公告：`id`、`title`、`body`、`priority`、`created_at`。

## 响应结构

### 课程列表项

```json
{
  "id": 123,
  "code": "CS101",
  "name": "Data Structures",
  "credit": 3,
  "department": "Computer Science",
  "language": "zh",
  "target_years": ["2026"],
  "categories": ["core"],
  "main_teacher": {"id": 10, "code": "T001", "name": "Teacher", "department": "CS", "title": "Professor"},
  "rating": {"count": 12, "avg": 4.3, "score": 4.1, "distribution": [0, 1, 2, 4, 5]}
}
```

### 评价

```json
{
  "id": 55,
  "course_id": 123,
  "semester": "2025-2026-1",
  "score": "A",
  "rating": 5,
  "content": "...",
  "moderator_remark": "",
  "vote": {"like_count": 3, "dislike_count": 0, "my_vote": 1},
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

## 错误处理

错误响应形如：

```json
{"error":"message"}
```

常见状态码：

- `400`：查询或请求体无效、投票值无效、通知等级无效、评价内容敏感。
- `401`：缺少认证、认证无效或 session 失效。
- `403`：无权限、CSRF 失败、用户被封禁、角色不足。
- `404`：课程、评价、教师、用户或 API Key 不存在。
- `409`：重复或积分余额不足。
- `429`：限流、验证码发送过快、登录锁定、每日投票上限。

## agent 示例流程

### 推荐课程

1. 用户约束较宽时，先调用 `GET /api/course/filter`。
2. 调用 `GET /api/course/?q=<topic>&department=<dept>&order_by=rating_score&page_size=10`。
3. 对 2-5 个候选课程调用 `GET /api/course/{id}` 和 `GET /api/course/{id}/review?order_by=like_count&page_size=10`。
4. 总结时列出课程代码、课程名、教师、课程 ID、评分人数、均分、开课学期和评价主题。

### 准备评价草稿

1. 用 `GET /api/course/?q=<name>` 匹配课程，并向用户确认。
2. 用 `GET /api/course/{id}` 查看 `my_review`，避免重复评价意图。
3. 先在本地起草评价，并让用户确认完整文本。
4. 用户确认后再 `POST /api/review/`。

### 关注课程并追踪新评价

1. 搜索并查看课程详情。
2. 向用户确认将 `notification_level` 设置为 `1`。
3. 调用 `POST /api/course/{id}/notification`。
4. 后续用 `GET /api/review/followed?order_by=created_at&page_size=20` 查看关注课程的新评价。
