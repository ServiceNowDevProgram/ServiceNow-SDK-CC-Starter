# View Rules (sysrule_view)

## Instructions

### CRITICAL - Dependency

1. **View Rules require existing views.** View Rules select WHICH view to display - views must exist first in sys_ui_view table.
2. **Verify Views Exist:** Use `runQuery` on `sys_ui_view` to verify target views exist before creating rules.
3. **Verify Roles:** For role-based rules with scripts, verify roles exist in `sys_user_role` before using `gs.hasRole()`.

### Creating View Rules

**Three switching approaches:**

1. **Device-Based:** Set `device_type` property (`mobile`, `tablet`, `browser`)
2. **Condition-Based:** Set `condition` property with encoded query (MUST end with `^EQ`)
3. **Script-Based:** Set `advanced: true` with custom `script` for complex logic (roles, multi-criteria)

**CRITICAL: Encoded Query Requirements**

When constructing encoded queries, **MUST use backend names, NOT UI labels:**

- Field names: Use element name (e.g., `priority` not `Priority`)
- Field values: Use internal value (e.g., `state=3^EQ` not `state=Closed^EQ`)
- Must end with: `^EQ`

**Why:** Backend names work even if UI labels change for language/localization.

**For detailed syntax and examples:** See `view-rule-conditions.md` for encoded query syntax.

**Activation:**

- Set `active: true` to enable the rule
- Set `overrides_user_preference: true` for automatic switching (overrides manual selection)

## Key concepts

**View Rules (sysrule_view)** - Automatically select which view to display based on conditions. Evaluation order:

1. Active rules only (`active: true`)
2. Device type match
3. Condition satisfied
4. Script execution (if `advanced: true`)
5. First match wins
6. User preference (unless `overrides_user_preference: true`)

**Script Structure:**

```javascript
(function overrideView(view, is_list) {
  // Available: view, is_list, current (forms only), gs, answer
  // Set answer to view name (string) or null
  answer = "view-name";
})(view, is_list);
```

## Best Practices

**Evaluation Priority:** Order rules by specificity using `order` property (lower numbers = higher priority)
**Performance:** Minimize database queries - use `current` record or cached data (`gs.getUser()`)
**Role Hierarchy:** Check highest priority roles first (admin before agent)
**Debugging:** Use `gs.info()` logging in scripts to troubleshoot

## Critical: One Advanced Rule Per Device Type

When multiple advanced (script-based) View Rules share the same `table` AND `device_type`, only the rule with the lowest `order` value is evaluated. Others are skipped entirely — even if the first rule sets `answer = null`.

**Solution:** Combine all role/condition checks into a single script.

## Avoidance

- **Business Rules, Client Scripts, UI Policies** → NEVER use for view switching, ALWAYS use View Rules (sysrule_view)
- **Heavy database queries in scripts** → Use current record or session properties
- **Missing `^EQ` in conditions** → All encoded queries MUST end with `^EQ`
- **Using field labels in conditions** → CRITICAL: Use field names (e.g., `priority`, `assigned_to`), NOT labels (e.g., "Priority", "Assigned to"). Labels cause silent failures.

## Next Steps

Use `load_skill_resource` with skill name "ui-layout-control" and the file path to get references.
Use `get_knowledge_source` with "SYSRULE_VIEW" for complete API documentation

**`references/view-rule-conditions.md`** - Simple declarative rules:

- Device-based switching (mobile/tablet/browser)
- Condition-based switching with encoded queries
- Encoded query syntax and operators
- Common validation and mistakes

**`references/view-rule-advanced.md`** - Complex script-based rules:

- Role-based switching with custom scripts
- Multi-criteria conditional logic
- GlideElement methods and script variables
- Performance optimization
- Debugging and troubleshooting
