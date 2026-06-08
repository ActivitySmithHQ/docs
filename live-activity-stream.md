---
title: "Live Activity Stream"
description: "Start, update, and dismiss Live Activities with a stable stream key."
og:title: "Live Activity Stream | ActivitySmith"
og:description: "Start, update, and dismiss Live Activities with a stable stream key."
---

Live Activity streams use a stable `stream_key` for the thing you want to keep
visible. Send the latest state to that key whenever the data changes.

- The first `PUT` starts the Live Activity.
- Next `PUT` request with the same `stream_key` updates it.
- `DELETE` ends the Live Activity when the work is done.

## What is a `stream_key`?

A `stream_key` is a stable name for one ongoing process.

Examples:

- `prod-web-1`
- `deployment-main`
- `nightly-backup`
- `ev-charging`

Use one `stream_key` for one system, workflow, or process.

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

## Timer Example

Use `timer` when you need countdowns or timers.

```bash
curl -X PUT https://activitysmith.com/api/live-activity/stream/benchmark-run \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_state": {
      "title": "Benchmark Run",
      "subtitle": "sampling",
      "type": "timer",
      "duration_seconds": 300,
      "color": "cyan"
    }
  }'
```

For a countdown, send `duration_seconds`. You can update `title`, `subtitle`,
`color`, or any other visible field as the work changes. Leave
`duration_seconds` out unless you want to change the timer.

To start at 00:00 and count up, set `counts_down` to `false` and leave out
`duration_seconds`.

## End Live Activity

Use `DELETE /live-activity/stream/:stream_key` when the tracked process is
finished and you want to dismiss the Live Activity. You can include final
values before it is removed.
By default, iOS removes the Live Activity after two minutes. Set
`auto_dismiss_minutes` to choose a different dismissal time, including `0` for
immediate dismissal.

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
      ],
      "auto_dismiss_minutes": 2
    }
  }'
```

`content_state` is optional here. Include it if you want one last update before the Live Activity is dismissed.

If you later send another `PUT` request with the same `stream_key`,
ActivitySmith starts a new Live Activity.

## Stream responses

Stream responses include an `operation` field:

- `started`: ActivitySmith started a new Live Activity for this `stream_key`
- `updated`: ActivitySmith updated the current Live Activity
- `noop`: the incoming state matched the current state, so no update was sent
- `ended`: returned by `DELETE /live-activity/stream/:stream_key`
