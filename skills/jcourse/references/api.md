# jCourse API Reference

## Contents

- Authentication and request rules
- Pagination and query encoding
- Course APIs
- Review APIs
- Teacher APIs
- Enrollment APIs
- User, settings, API key, point, and announcement APIs
- Response shapes
- Error handling
- Example agent flows

## Authentication and Request Rules

Default service URL: `https://course.sjtu.plus`.

Default API root: `https://course.sjtu.plus/api`.

Base path on any deployment: `/api`.

Preferred agent authentication is a user API key:

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

Bearer API keys are resolved before session auth. User API keys authenticate as their owner. System API keys are only for `/api/ext/*` and admin-created integrations.

CSRF applies to browser-session mutating requests. CSRF is skipped when a valid API key is present, so agents should use API keys.

Verify the key before personalized work:

```http
GET /api/auth/me
```

Response:

```json
{"id":1,"username":"u1","email":"u1@example.edu","role":"user"}
```

## Pagination and Query Encoding

Paginated endpoints return:

```json
{"items":[],"total":0,"page":1,"page_size":20}
```

Defaults are usually `page=1&page_size=20`. Encode array filters by repeating the key:

```text
/api/course/?categories=CORE&categories=ELECTIVE&target_years=2026
```

Boolean query parameters are accepted as `0` or `1` by the frontend client.

## Course APIs

### Get Course Filters

```http
GET /api/course/filter
```

Returns filter counts: `credits`, `departments`, `categories`, `target_years`, `languages`, `semesters`; each item is `{name,count}`.

### List Courses

```http
GET /api/course/?q=&department=&language=&categories=&target_years=&credit=&has_review=&order_by=&ascend=&page=&page_size=
```

Query parameters:

- `q`: text search.
- `department`, `language`, `credit`: exact filters.
- `categories`, `target_years`: repeatable filters.
- `has_review`: `1` or `0`.
- `order_by`: `rating_score`, `rating_count`, or `rating_avg`.
- `ascend`: `1` for ascending, omit or `0` for descending.

List item fields: `id`, `code`, `name`, `credit`, `department`, `language`, `target_years`, `categories`, `main_teacher`, `rating`.

`rating` has `count`, `avg`, `score`, `distribution` where `distribution` is a 5-element array for 1-5 star counts.

### Hot Courses

```http
GET /api/course/hot?period=week&limit=5
```

`period` is `week` or `month`. Response has `period`, `period_key`, and `items` of `{course,score}`. Hot score is activity score, not rating.

### Course Detail

```http
GET /api/course/{course_id}
```

Returns course list fields plus:

- `last_semester`
- `teacher_group`
- `offered_courses`: `{semester,language,target_years,categories}`
- `same_code_courses`
- `same_teacher_courses`
- `notification_level`: `0` normal, `1` followed, `2` ignored
- `my_enrollments` when authenticated and present
- `my_review` when authenticated and present

Use detail before taking an action on a course name.

### Course Review Filters and Trend

```http
GET /api/course/{course_id}/review/filter
GET /api/course/{course_id}/review/trend
```

Filters return available `semesters` and `ratings`. Trend returns `[{"semester":"2025-2026-1","avg":4.2,"count":8}]`.

### Follow or Ignore Course

```http
POST /api/course/{course_id}/notification

{"level":1}
```

Levels: `0` normal, `1` follow, `2` ignore. Confirm with the user before changing it.

### Followed and Ignored Courses

```http
GET /api/course/followed?page=1&page_size=20
GET /api/course/ignored?page=1&page_size=20
```

Supports the course list pagination and ordering parameters.

## Review APIs

### List Reviews Globally

```http
GET /api/review?q=&semester=&rating=&order_by=created_at&ascend=&page=&page_size=
```

`order_by` can be `created_at` or `like_count`. Authenticated global review lists exclude ignored courses.

### List Reviews for a Course

```http
GET /api/course/{course_id}/review?q=&semester=&rating=&order_by=like_count&page=1&page_size=20
```

Use `order_by=like_count` for representative review summaries and `order_by=created_at` for recent sentiment.

### Followed Reviews

```http
GET /api/review/followed?page=1&page_size=20
```

Returns reviews from followed courses.

### Get One Review

```http
GET /api/review/{review_id}
```

Review fields: `id`, `course`, `course_id`, `user_id` when viewable, `semester`, `score`, `rating`, `content`, `moderator_remark`, `vote`, `created_at`, `updated_at`.

`vote` has `like_count`, `dislike_count`, and optionally `my_vote`.

### Create Review

```http
POST /api/review/

{"course_id":123,"semester":"2025-2026-1","rating":5,"score":"A","content":"..."}
```

Required: `course_id`, `rating`, `content`. Optional: `semester`, `score`. Confirm the final text with the user before posting. Sensitive content can return 400.

### Update or Delete Review

```http
PUT /api/review/{review_id}
DELETE /api/review/{review_id}
```

Update body:

```json
{"semester":"2025-2026-1","rating":4,"score":"A-","content":"..."}
```

Only the owner or an admin can delete, depending on backend policy. Confirm before mutating.

### Vote Review

```http
POST /api/review/{review_id}/vote

{"vote_type":1}
```

`vote_type`: `1` like, `-1` dislike, `0` clear vote. Daily vote limits can return 429.

### User Reviews

```http
GET /api/user/{user_id}/review?page=1&page_size=20
```

Requires self or admin.

## Teacher APIs

### Get Teacher Filters

```http
GET /api/teacher/filter
```

Returns `departments` and `titles` filter counts.

### List Teachers

```http
GET /api/teacher/?q=&department=&title=&page=&page_size=
```

Teacher fields: `id`, `code`, `name`, `department`, `title`.

### Teacher Detail and Courses

```http
GET /api/teacher/{teacher_id}
GET /api/teacher/{teacher_id}/course?q=&order_by=rating_score&page=1&page_size=20
```

Teacher courses support course-list filters.

## Enrollment APIs

### My Enrollments

```http
GET /api/course/enrolled
```

Returns `[{"id":1,"course":{...},"semester":"2025-2026-1","created_at":"..."}]`.

### Add or Remove Enrollment

```http
POST /api/course/{course_id}/enrollment
DELETE /api/course/enrollment/{enrollment_id}
```

Create body:

```json
{"semester":"2025-2026-1"}
```

The semester must be offered for that course, unless it equals the course last semester according to backend policy. Confirm before changing enrollment records.

### JAccount Enrollment Sync

```http
GET /api/course/enrollment-sync/start?semester=2025-2026-1
GET /api/course/enrollment-sync/callback?code=...&state=...
```

This is an OAuth-like browser flow. In agent contexts, prefer manual enrollment unless the user explicitly wants to open the sync URL.

## User, Settings, API Key, Point, and Announcement APIs

### Account

```http
GET /api/auth/me
POST /api/auth/logout
```

Public session endpoints also exist: `/api/auth/register/code`, `/api/auth/register`, `/api/auth/login`, `/api/auth/password-reset/code`, `/api/auth/password-reset`. Avoid asking users for passwords in agent workflows; prefer API keys.

### User Settings

```http
GET /api/user/settings
PUT /api/user/settings

{"current_semester":"2025-2026-1"}
```

### API Keys

```http
GET /api/api-key/
POST /api/api-key/
DELETE /api/api-key/{api_key_id}
```

Create body: `{"name":"agent"}`. Creation returns the key string. Treat it as a secret and do not print it back unless the user explicitly requests it.

Admin-only system API keys:

```http
GET /api/admin/api-key/system
POST /api/admin/api-key/system
DELETE /api/admin/api-key/system/{api_key_id}
```

### Points

```http
GET /api/user/{user_id}/point?page=1&page_size=20
```

Returns the user's point total and paginated point records. Requires self or admin.

System integration endpoint:

```http
GET /api/ext/point?email=user@example.edu
```

Requires a system API key.

### Announcements

```http
GET /api/announcement/
```

Returns active announcements: `id`, `title`, `body`, `priority`, `created_at`.

## Response Shapes

### Course List Item

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

### Review

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

## Error Handling

Errors return JSON like:

```json
{"error":"message"}
```

Common status codes:

- `400`: invalid query/body, invalid vote, invalid notification level, sensitive review content.
- `401`: missing/invalid auth or expired session.
- `403`: forbidden, CSRF failure, suspended user, insufficient role.
- `404`: missing course, review, teacher, user, or API key.
- `409`: duplicate or insufficient point balance.
- `429`: rate limit, verification too soon, login locked, daily vote limit.

## Example Agent Flows

### Recommend Courses

1. `GET /api/course/filter` if the user gives broad constraints.
2. `GET /api/course/?q=<topic>&department=<dept>&order_by=rating_score&page_size=10`.
3. For 2-5 candidates, call `GET /api/course/{id}` and `GET /api/course/{id}/review?order_by=like_count&page_size=10`.
4. Summarize tradeoffs with course code/name/teacher/id, rating count, average, offered semesters, and review themes.

### Prepare a Review Draft

1. Resolve the course with `GET /api/course/?q=<name>` and confirm the matched course.
2. Check `GET /api/course/{id}` for `my_review` to avoid duplicate review intent.
3. Draft content locally and ask the user to approve exact text.
4. `POST /api/review/` only after confirmation.

### Follow a Course and Track New Reviews

1. Resolve course with search and detail.
2. Confirm setting `notification_level` to `1`.
3. `POST /api/course/{id}/notification`.
4. Later use `GET /api/review/followed?order_by=created_at&page_size=20`.
