# Email Digest Interval

# ServiceNow Email Digest Intervals - Record API Reference

Create email digest intervals using the Record API to define time periods for grouping multiple notifications into summary emails.

### Data Properties

| Field      | Type             | Required | Default | Description                                                   |
| ---------- | ---------------- | -------- | ------- | ------------------------------------------------------------- |
| `name`     | `TranslatedText` | Yes      | -       | Name of the digest interval (max length: 100, must be unique) |
| `interval` | `GlideDuration`  | Yes      | -       | Duration of the interval in GlideDuration format              |

### GlideDuration Format

The interval field uses GlideDuration format: `YYYY-MM-DD HH:MM:SS`

**Note:** The interval is specified in terms of time elapsed since the epoch (January 1, 1970, 00:00:00 UTC). This format is used to represent durations in a consistent manner.

**Common Intervals:**

- **Hourly**: `1970-01-01 01:00:00`
- **Daily**: `1970-01-02 00:00:00` (1day)
- **Weekly**: `1970-01-08 00:00:00` (7 days)

**Critical Note:**

Email digest intervals are restricted to be from one hour to one week (inclusive).

## Example

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

// Daily digest interval
const dailyInterval = Record({
  $id: Now.ID["daily_digest_interval"],
  table: "sys_email_digest_interval",
  data: {
    name: "Daily Summary",
    interval: "1970-01-08 00:00:00" // 7 day
  }
});

// Critical incident digest (15 minutes)
const queryCritical = Record({
  $id: Now.ID["query_critical_interval"],
  table: "sys_email_digest_interval",
  query: { name: "Critical Incident Digest" }
});

// Hourly digest interval
const hourlyInterval = Record({
  $id: Now.ID["hourly_digest_interval"],
  table: "sys_email_digest_interval",
  data: {
    name: "Hourly Digest",
    interval: "1970-01-01 01:00:00" // 1 hour
  }
});
```
