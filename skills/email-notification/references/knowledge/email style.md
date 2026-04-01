# Email Style

# ServiceNow Email Styles - Record API Reference

Create email styles using the Record API to define reusable CSS styling for email notifications.

### Data Properties

| Field   | Type     | Required | Default | Description                                         |
| ------- | -------- | -------- | ------- | --------------------------------------------------- |
| `name`  | `String` | Yes      | -       | Name of the style (max length: 100, must be unique) |
| `style` | `HTML`   | No       | -       | style content (max length: 65000)                   |

## Implementation Example

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

// Create an email style
const incidentStyle = Record({
  $id: Now.ID["incident_email_style"],
  table: "sysevent_email_style",
  data: {
    name: "Incident Email Style",
    style: "<p>let body &#61; &#39;Hello&#39;<br />\${body}</p>"
  }
});
```
