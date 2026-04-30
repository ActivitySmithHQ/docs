---
title: "Live Activity Stream"
description: "Use stream updates when you want ActivitySmith to manage the Live Activity for you."
og:title: "Live Activity Stream | ActivitySmith"
og:description: "Use stream updates when you want ActivitySmith to manage the Live Activity for you."
---

Send the latest state for a stable `stream_key`, and ActivitySmith handles the
rest for you:

- if the Live Activity does not exist yet, ActivitySmith starts it
- if it already exists, ActivitySmith updates it

You don't need to store `activity_id` or manage the lifecycle yourself.

You can read about real world use case example in the following blog post: [How I Track VPS System Health on My Lock Screen with a Cron Job](https://activitysmith.com/blog/track-vps-system-health-on-your-lock-screen-from-a-cron-job).

## Two approaches to run Live Activities

### Simple: stream updates

Use stream updates when you want the easiest, stateless flow:

- `PUT /live-activity/stream/:stream_key`
- `DELETE /live-activity/stream/:stream_key`

This is a good fit for:

- cron jobs
- scheduled tasks
- CI workflows
- monitoring jobs
- background workers

### Advanced: full lifecycle control

Use the lifecycle endpoints when you want to manage one specific Live Activity
instance yourself:

- `POST /live-activity/start`
- `POST /live-activity/update`
- `POST /live-activity/end`

With this approach, you save the returned `activity_id` and reuse it for later
updates and the final end call.

## What is a `stream_key`?

A `stream_key` is a stable name for one ongoing process.

Examples:

- `prod-web-1`
- `deployment-main`
- `nightly-backup`
- `ev-charging`

Use one `stream_key` for one system, workflow, or process.

Allowed characters:

- letters
- numbers
- `_`
- `-`

## Example

```bash
curl -X PUT https://activitysmith.com/api/live-activity/stream/prod-web-1 \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_state": {
      "title": "Server Health",
      "subtitle": "prod-web-1",
      "type": "metrics",
      "metrics": [
        {
          "label": "CPU",
          "value": 24,
          "unit": "%"
        },
        {
          "label": "MEM",
          "value": 61,
          "unit": "%"
        }
      ]
    }
  }'
```

Call the same endpoint again with the same `stream_key` whenever the state
changes.

## Ending a stream

Use `DELETE` when the tracked process is finished and you no longer want the
Live Activity on devices.

```bash
curl -X DELETE https://activitysmith.com/api/live-activity/stream/prod-web-1 \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_state": {
      "title": "Server Health",
      "subtitle": "prod-web-1",
      "type": "metrics",
      "metrics": [
        {
          "label": "CPU",
          "value": 7,
          "unit": "%"
        },
        {
          "label": "MEM",
          "value": 38,
          "unit": "%"
        }
      ]
    }
  }'
```

`content_state` is optional here. Include it if you want to end the stream
with a final state.

If you later send another `PUT` request with the same `stream_key`,
ActivitySmith starts a new Live Activity for that stream again.

This way you can keep a live activity running on your lock screen for as long as you want, as opposed to the iOS limit of 8 hours. The battery usage is noticeable but not significant.

## Stream responses

Stream responses include an `operation` field:

- `started`: ActivitySmith started a new Live Activity for this `stream_key`
- `updated`: ActivitySmith updated the current Live Activity
- `rotated`: ActivitySmith ended the previous Live Activity and started a new one
- `noop`: the incoming state matched the current state, so no update was sent
- `paused`: the stream is paused, so no Live Activity was started or updated
- `ended`: returned by `DELETE /live-activity/stream/:stream_key`

## Which approach should you choose?

Choose stream updates if:

- you want the easiest setup
- you only know the latest state
- you do not want to store `activity_id`
- you are sending repeated updates from scripts, cron, CI, or workers

Choose lifecycle endpoints if:

- you want to explicitly decide when to start, update, and end
- you already store `activity_id`
- your application wants direct control over one specific Live Activity instance
