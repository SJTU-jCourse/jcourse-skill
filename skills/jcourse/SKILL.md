---
name: jcourse
description: Work with the jCourse course community API from an agent. Use when a user wants an agent to search courses or teachers, inspect course detail, summarize reviews, compare courses, follow or ignore courses, manage course enrollments, create or update course reviews, vote on reviews, or otherwise operate the jCourse backend over HTTP.
---

# jCourse

Use this skill to operate jCourse as a course-selection community assistant. Prefer API-backed answers over guesses: fetch courses, teachers, ratings, offered semesters, and reviews before giving recommendations or taking actions.

## Connection

Use `https://course.sjtu.plus` as the default jCourse service URL. The default API root is `https://course.sjtu.plus/api`. If the user provides another deployment URL, use that instead; the API root is `${base_url}/api` unless the user already gives a URL ending in `/api`.

Ask the user for a user API key if they did not provide one.

Use user API keys with:

```http
Authorization: Bearer <api_key>
Content-Type: application/json
```

With an API key, non-GET requests bypass CSRF. Do not use browser-session CSRF flows unless the user explicitly provides cookies and asks for that path.

Before doing personalized work, verify auth with `GET /api/auth/me`. If it returns 401, ask for a valid user API key.

## Workflow

1. Determine whether the task is read-only or mutating. Confirm before creating, updating, deleting, voting, following, ignoring, or enrolling.
2. For course recommendations, search broadly first, then inspect the most relevant course details and reviews. Do not rank only by rating average; include rating count, rating score, review recency, teacher, semester availability, department, credit, and fit to user constraints.
3. For review summaries, separate factual API data from inferred judgment. Mention sample size when review count is small.
4. For course actions, resolve names to exact `course_id` and show the matched course code/name/teacher before executing.
5. For user-owned data, use `/auth/me` first when a `user_id` is required.

## Common Tasks

- Search courses: `GET /api/course/?q=<keyword>&page=1&page_size=10`.
- Inspect course: `GET /api/course/{course_id}`; includes offered semesters, teacher group, related courses, `notification_level`, `my_enrollments`, and `my_review` when authenticated.
- Read course reviews: `GET /api/course/{course_id}/review?order_by=like_count&page_size=20` and optionally `GET /api/course/{course_id}/review/filter`.
- Find teachers: `GET /api/teacher/?q=<name>` then `GET /api/teacher/{teacher_id}/course`.
- Follow or ignore a course: `POST /api/course/{course_id}/notification` with `{"level":1}` to follow, `{"level":2}` to ignore, `{"level":0}` to reset.
- Create a review: `POST /api/review/` with `course_id`, `rating`, `content`, and optional `semester` and `score`.
- Vote on a review: `POST /api/review/{review_id}/vote` with `vote_type` `1`, `-1`, or `0`.

## References

Read [references/api.md](references/api.md) when you need exact endpoint lists, query parameters, JSON shapes, or error handling.
