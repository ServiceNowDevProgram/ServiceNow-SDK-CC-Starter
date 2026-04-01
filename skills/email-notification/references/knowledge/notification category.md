# Notification Category

The Notification Category API allows you to create notification categories [sys_notification_category] to organize and group email notifications.

## Notification Category object

Create notification categories using the Record API to organize email notifications into logical groups.

Properties

| Name                   | Type             | Description                                                                                                                                                                                                           |
| ---------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $id                    | String or Number | Required. A unique ID for the metadata object provided in the following format, where `<value>` is a string or number. `$id: Now.ID[<value>]` When you build the application, this ID is hashed into a unique sys_ID. |
| table                  | String           | Required. Must be `'sys_notification_category'`.                                                                                                                                                                      |
| data                   | Object           | Required. Contains the category properties.                                                                                                                                                                           |
| data.name              | String           | Required. Category name (max length: 32, must be unique).                                                                                                                                                             |
| data.short_description | String           | Optional. Short description (max length: 1000).                                                                                                                                                                       |

### Example

```javascript
import { Record } from "@servicenow/sdk/core";

const incidentCategory = Record({
  $id: Now.ID["incident_notification_category"],
  table: "sys_notification_category",
  data: {
    name: "Incident Notifications",
    short_description: "Notifications for incident updates"
  }
});
```
