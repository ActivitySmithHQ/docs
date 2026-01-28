# ActivitySmith API Key Quickstart

Push Notifications and iOS Live Activities for your own devices, powered by a single API key.

## Why ActivitySmith

- One API key for push + Live Activities
- Account-level fanout to all of your paired iOS devices

## Before You Start

1. Create an ActivitySmith account.
2. Install the ActivitySmith iOS app on your iPhone/iPad and pair it with your account.
3. Generate an API key in your dashboard (or via `POST /api-keys` using user auth).
4. The ActivitySmith iOS app registers device tokens on your behalf:
   - `push` for standard push notifications
   - `start_live_activity` for starting Live Activities
   - `update_live_activity` for Live Activity updates

## How It Works (High Level)

You create an account, pair the iOS app, then call the API to notify yourself in real time.

- **Push notifications**: Send alerts for events like “New user signed up” or “Deployment finished”.
- **Live Activities**: Start and update a Live Activity (e.g., a GitHub Actions deployment) to see progress on your Lock Screen or Dynamic Island.

## Base URL

- Production: `https://activitysmith.com/api`

## Authentication

Send your API key in the `Authorization` header:

```
Authorization: Bearer <API_KEY>
```

## Quickstart Flow

1. Send a push notification to all of your paired devices.
2. Start a Live Activity and receive an `activity_id`.
3. Update the Live Activity as your backend state changes.
4. End the Live Activity when the experience completes.

---

## Endpoint: Send a Push Notification

**POST** `/push-notification`

Send a push notification to every paired device in your account.

**Body**

- `title` (string, required)
- `message` (string, optional)
- `subtitle` (string, optional)

**Example**

```bash
curl -X POST https://activitysmith.com/api/push-notification \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Order ready",
    "message": "Your order #1842 is ready for pickup",
  }'
```

**Response**

```json
{
  "success": true,
  "devices_notified": 3,
  "users_notified": 1,
  "message": "Push notification sent to all account users",
  "timestamp": "2025-08-12T12:00:00.000Z"
}
```

If your account is solo, `users_notified` will typically be `1`.

---

## Endpoint: Start a Live Activity

**POST** `/live-activity/start`

Starts a Live Activity on all registered devices in your account and returns a secure `activity_id`.

**Body**

- `content_state` (object, required)
- `alert` (object, optional: `{ "title": "...", "body": "..." }`)

**Example**

```bash
curl -X POST https://activitysmith.com/api/live-activity/start \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_state": {
      "title": "Order #1842",
      "subtitle": "Preparing",
      "current_step": 1,
      "type": "segmented_progress",
      "number_of_steps": 4,
      "color": "blue"
    },
    "alert": {
      "title": "Order started",
      "body": "We are preparing your order"
    }
  }'
```

**Response**

```json
{
  "success": true,
  "devices_notified": 3,
  "message": "Live Activity started",
  "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
  "timestamp": "2025-08-12T12:00:00.000Z"
}
```

---

## Endpoint: Update a Live Activity

**POST** `/live-activity/update`

Updates the Live Activity. If the per-activity token is not yet registered by iOS, updates are queued and delivered later.

**Body**

- `activity_id` (string, required)
- `content_state` (object, required)
- `alert` (object, optional)

**Example**

```bash
curl -X POST https://activitysmith.com/api/live-activity/update \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
    "content_state": {
      "title": "Order #1842",
      "subtitle": "Almost done",
      "current_step": 3,
      "type": "segmented_progress",
      "number_of_steps": 4,
      "step_color": "yellow"
    },
    "alert": {
      "title": "Almost ready",
      "body": "Just a few minutes left"
    }
  }'
```

**Response (sent immediately)**

```json
{
  "success": true,
  "message": "Live Activity updated",
  "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
  "devices_sent": 2,
  "devices_queued": 1,
  "timestamp": "2025-08-12T12:04:00.000Z"
}
```

**Response (queued)**

```json
{
  "success": true,
  "message": "Live Activity update queued - waiting for device tokens",
  "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
  "devices_queued": 3,
  "timestamp": "2025-08-12T12:04:00.000Z"
}
```

---

## Endpoint: End a Live Activity

**POST** `/live-activity/end`

Ends the Live Activity and archives its lifecycle.

**Body**

- `activity_id` (string, required)
- `content_state` (object, required)
- `alert` (object, optional)

**Example**

```bash
curl -X POST https://activitysmith.com/api/live-activity/end \
  -H "Authorization: Bearer $ACTIVITYSMITH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
    "content_state": {
      "title": "Order #1842",
      "subtitle": "Ready",
      "current_step": 4,
      "type": "segmented_progress",
      "number_of_steps": 4,
      "step_color": "green"
    },
    "alert": {
      "title": "Ready for pickup",
      "body": "We will hold your order for 15 minutes"
    }
  }'
```

**Response**

```json
{
  "success": true,
  "message": "Live Activity ended",
  "activity_id": "h8QmSyYFTuwOIF6Wh3YcZlzHLhUcr034",
  "devices_sent": 2,
  "devices_queued": 0,
  "timestamp": "2025-08-12T12:10:00.000Z"
}
```

---

## Content State Reference

`content_state` is required for start, update, and end.

**Required fields**

- `title` (string)
- `current_step` (integer, > 0)

**Common optional fields**

- `subtitle` (string)
- `color` (string)
- `auto_dismiss_minutes` (integer, >= 0)

**Type**

- `type` (string, required on start): `progress`, `counter`, `timer`, `countdown`, `segmented_progress`
- If `type` is `segmented_progress`, also include:
  - `number_of_steps` (integer, > 0)
  - `step_color` (string, optional)

## Notes and Best Practices

- API keys are account-scoped. Requests fan out to all account members (you + teammates), not your end users.
- Keep `activity_id` in your system and reuse it for every update/end call.
