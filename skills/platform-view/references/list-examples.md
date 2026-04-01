# Additional List Examples

This file contains supplementary examples for list layout configuration.

## Contents

- List layout configuration with Simple Reference
- List layout configuration with Explicit Relationship

---

## List layout configuration with Simple Reference

**NOTE**: When a simple reference field exists (e.g., `table.field`), the relationship `sys_id` is not required. For example, "Project Tasks" can be displayed on the Project form to show all associated tasks by leveraging the implicit relationship between the `project` table and the `project_task` table. The list is configured to appear in the default view of the `project` form's related list.

```javascript
import { List, default_view } from "@servicenow/sdk/core";

List({
  table: "project_task",
  view: default_view,
  parent: "project",
  columns: [
    "assigned_to",
    "short_description",
    "due_date",
    "project",
    "state",
    "task_number",
    "sys_tags"
  ]
});
```

---

## List layout configuration with Explicit Relationship

**NOTE**: When no simple reference field exists (e.g., `table.field`), the relationship `sys_id` is required. For creating custom relationships, use `get_knowledge_source` tool to get the **RELATIONSHIP** knowledge source.

```javascript
import { List, default_view } from "@servicenow/sdk/core";
import { skillMatchedPlayersRelationship } from "../relationships/game_allotment_relationships.now";

export const skillMatchedPlayersList = List({
  table: "sn_sportshub_players",
  view: default_view,
  parent: "sn_sportshub_sports",
  relationship: skillMatchedPlayersRelationship,
  columns: [
    { element: "first_name", position: 0 }, // First Name
    { element: "gender", position: 1 }, // Gender
    { element: "email", position: 2 }, // Email
    { element: "skill_level", position: 3 }, // Skill Level
    { element: "experience_years", position: 4 }, // Experience Years
    { element: "position", position: 5 }, // Position
    { element: "active", position: 6 } // Active status
  ]
});
```
