# CATALOG_CLIENT_SCRIPT

# Catalog Client Script - Templates and Examples

This knowledge source provides working code templates for ServiceNow Catalog Client Scripts. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

## onLoad — Initial Setup

```typescript
import { CatalogClientScript } from "@servicenow/sdk/core";
import { laptopRequest } from "../catalog-items/laptop-request.now";

CatalogClientScript({
  $id: Now.ID["laptop_onload"],
  name: "Laptop Request - OnLoad",
  script: Now.include("../../client/laptop-onload.js"),
  type: "onLoad",
  catalogItem: laptopRequest,
  active: true,
  appliesOnCatalogItemView: true
});
```

**laptop-onload.js:**

```javascript
function onLoad() {
  // Set initial field states
  g_form.setReadOnly("estimated_cost", true);
  g_form.setValue("estimated_cost", "$0");
  g_form.setMandatory("justification", true);
}
```

---

## onChange — Dynamic Field Behavior

```typescript
CatalogClientScript({
  $id: Now.ID["laptop_type_change"],
  name: "Laptop Type - onChange",
  script: Now.include("../../client/laptop-type-change.js"),
  type: "onChange",
  catalogItem: laptopRequest,
  variableName: laptopRequest.variables.laptopType,
  active: true
});
```

**laptop-type-change.js:**

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return; // Always guard against initial load

  // Show/hide accessories based on selection
  if (newValue === "developer") {
    g_form.setDisplay("accessories", true);
  } else {
    g_form.setDisplay("accessories", false);
    g_form.clearValue("accessories");
  }
}
```

---

## onSubmit — Form Validation

```typescript
CatalogClientScript({
  $id: Now.ID["laptop_validation"],
  name: "Laptop Request - Validation",
  script: Now.include("../../client/laptop-validation.js"),
  type: "onSubmit",
  catalogItem: laptopRequest,
  active: true
});
```

**laptop-validation.js:**

```javascript
function onSubmit() {
  var justification = (g_form.getValue("justification") || "").trim();

  if (justification.length < 20) {
    g_form.showFieldMsg(
      "justification",
      "Please provide at least 20 characters.",
      "error",
      true
    );
    g_form.addErrorMessage("Justification is too short.");
    return false;
  }

  return true;
}
```

---

## onChange with GlideAjax — Server-Side Lookup

```typescript
CatalogClientScript({
  $id: Now.ID["asset_tag_lookup"],
  name: "Asset Tag - Warranty Lookup",
  script: Now.include("../../client/asset-tag-lookup.js"),
  type: "onChange",
  catalogItem: equipmentRepairItem,
  variableName: equipmentRepairItem.variables.asset_tag,
  active: true
});
```

**asset-tag-lookup.js:**

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;

  if (!newValue) {
    g_form.clearValue("warranty_status");
    return;
  }

  var ga = new GlideAjax("global.AssetLookupUtil");
  ga.addParam("sysparm_name", "getWarrantyStatus");
  ga.addParam("sysparm_asset_tag", newValue);
  ga.getXMLAnswer(function (response) {
    if (!response) return;
    var info = JSON.parse(response);
    g_form.setValue("warranty_status", info.status);
  });
}
```

---

## Script on Variable Set

Scripts scoped to a variable set apply to **all** catalog items using that set:

```typescript
import { requesterInfoSet } from "./variable-sets/requester-info-set.now";

CatalogClientScript({
  $id: Now.ID["department_change_script"],
  name: "Department Change - Clear Manager",
  type: "onChange",
  variableSet: requesterInfoSet,
  appliesTo: "set",
  variableName: requesterInfoSet.variables.department,
  script: Now.include("../../client/department-change.js"),
  active: true,
  uiType: "all"
});
```

**department-change.js:**

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;
  g_form.clearValue("manager");
  if (!newValue) return;
  g_form.showFieldMsg(
    "manager",
    "Please select a manager from the new department",
    "info",
    false
  );
}
```

---

## onChange — Auto-Fill from Server (GlideAjax)

```typescript
CatalogClientScript({
  $id: Now.ID["requested_for_change"],
  name: "Requested For - Auto Fill Details",
  type: "onChange",
  variableSet: requesterInfoSet,
  appliesTo: "set",
  variableName: requesterInfoSet.variables.requested_for,
  script: Now.include("../../client/requested-for-change.js"),
  active: true,
  uiType: "all"
});
```

**requested-for-change.js:**

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;

  // Clear all requester-related fields
  g_form.clearValue("email");
  g_form.clearValue("company");
  g_form.clearValue("department");
  g_form.clearValue("manager");
  g_form.clearValue("location");

  if (!newValue) return;

  // Fetch user details from server via Script Include
  var ga = new GlideAjax("global.RequesterLookupUtil");
  ga.addParam("sysparm_name", "getUserDetails");
  ga.addParam("sysparm_user_id", newValue);
  ga.getXMLAnswer(function (response) {
    if (!response) return;

    var userInfo = JSON.parse(response);
    if (userInfo.email) g_form.setValue("email", userInfo.email);
    if (userInfo.company) g_form.setValue("company", userInfo.company);
    if (userInfo.department) g_form.setValue("department", userInfo.department);
    if (userInfo.manager) g_form.setValue("manager", userInfo.manager);
    if (userInfo.location) g_form.setValue("location", userInfo.location);
  });
}
```

---

## Properties

| Property                 | Type    | Description                                                                |
| ------------------------ | ------- | -------------------------------------------------------------------------- |
| $id                      | string  | **Required.** Unique identifier.                                           |
| name                     | string  | **Required.** Name of the script.                                          |
| script                   | string  | Inline script or `Now.include()` reference.                                |
| type                     | string  | `'onLoad'` \| `'onChange'` \| `'onSubmit'`.                                |
| uiType                   | string  | `'desktop'` \| `'mobileOrServicePortal'` \| `'all'`. Default: `'desktop'`. |
| active                   | boolean | Whether the script is enabled. Default: `true`.                            |
| catalogItem              | string  | **Required** if not using variableSet.                                     |
| variableSet              | string  | **Required** if not using catalogItem.                                     |
| appliesTo                | string  | Required if using variableSet. `'item'` or `'set'`. Default: `'item'`.     |
| variableName             | string  | **Required** for onChange. The variable that triggers the script.          |
| appliesOnCatalogItemView | boolean | Applies on catalog item view. Default: `true`.                             |
| appliesOnRequestedItems  | boolean | Applies on requested items. Default: `false`.                              |
| appliesOnCatalogTasks    | boolean | Applies on catalog tasks. Default: `false`.                                |
| appliesOnTargetRecord    | boolean | Applies on target record. Default: `false`.                                |

## Script Types

### onLoad

Runs when the form loads. Use for initial setup (field states, defaults, visibility).

### onChange

Runs when a specific variable changes. **Always guard with `if (isLoading) return;`**.

### onSubmit

Runs on form submission. Return `false` to block. **Avoid GlideAjax here** (async issues).

## g_form API Reference

| Method                                               | Description           |
| ---------------------------------------------------- | --------------------- |
| `getValue(fieldName)`                                | Get variable value    |
| `setValue(fieldName, value)`                         | Set variable value    |
| `setDisplay(fieldName, display)`                     | Show/hide variable    |
| `setMandatory(fieldName, mandatory)`                 | Set mandatory state   |
| `setReadOnly(fieldName, readOnly)`                   | Set read-only state   |
| `clearValue(fieldName)`                              | Clear variable value  |
| `hasField(fieldName)`                                | Check if field exists |
| `showFieldMsg(fieldName, message, type, scrollForm)` | Show field message    |
| `hideFieldMsg(fieldName, clearAll)`                  | Hide field message    |
| `addErrorMessage(message)`                           | Add error message     |
| `clearOptions(fieldName)`                            | Clear select options  |
| `addOption(fieldName, value, label)`                 | Add select option     |

## Catalog Client Script vs Standard Client Script

| Aspect   | Catalog Client Script                      | Standard Client Script |
| -------- | ------------------------------------------ | ---------------------- |
| Scope    | Catalog item or variable set               | Table (e.g., Incident) |
| onChange | Links to a **variable**                    | Links to a **field**   |
| Context  | Catalog ordering, RITM, Catalog Task forms | Table forms            |

## Best Practices

- Use `hasField()` checks when working with variable sets
- Avoid GlideAjax in onSubmit (async issues)
- Use g_form APIs, not direct DOM manipulation
- For variable name conflicts with table fields, use `variables.variable_name`
- Use `Now.include()` for external script files for maintainability

---

## GlideAjax Patterns

### getXMLAnswer — Recommended for Most Cases

Returns the answer string directly to the callback. Cleanest and simplest pattern.

**Callback signature**: `function callback(answer, additionalParam, responseParam)`

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;
  if (!newValue) return;

  var ga = new GlideAjax("GetUserInfo");
  ga.addParam("sysparm_name", "getManagerName");
  ga.addParam("sysparm_user_id", newValue);

  ga.getXMLAnswer(function (answer) {
    if (answer) {
      var result = JSON.parse(answer);
      g_form.setValue("manager", result.manager_sys_id);
    }
  });
}
```

**With additional parameters** (pass context into the callback):

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;

  var ga = new GlideAjax("CatalogUtils");
  ga.addParam("sysparm_name", "getItemPrice");
  ga.addParam("sysparm_item_id", newValue);

  var extraContext = {
    fieldToUpdate: "estimated_cost",
    requestedAt: new Date().toISOString()
  };
  ga.getXMLAnswer(handlePrice, extraContext);
}

function handlePrice(answer, extraContext) {
  if (answer && extraContext) {
    g_form.setValue(extraContext.fieldToUpdate, answer);
  }
}
```

### getXML — Full XML Response

Use when you need to parse multiple attributes or check for errors beyond the answer element.

**Callback signature**: `function callback(response)` — where `response` is the full XMLHttpRequest object.

```javascript
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;

  var ga = new GlideAjax("ValidateRequest");
  ga.addParam("sysparm_name", "checkApproval");
  ga.addParam("sysparm_request_type", newValue);

  ga.getXML(function (response) {
    var answer = response.responseXML.documentElement.getAttribute("answer");
    var error = response.responseXML.documentElement.getAttribute("error");

    if (error) {
      g_form.addErrorMessage("Validation failed: " + error);
    } else {
      var result = JSON.parse(answer);
      g_form.setValue("approver", result.approver);
      g_form.setDisplay("approval_section", result.needs_approval);
    }
  });
}
```

### getXMLWait — Synchronous (Avoid)

Blocks the browser. **Not available in scoped apps.** Causes UI freezes.

```javascript
// ⚠️ NOT RECOMMENDED — freezes the UI
var ga = new GlideAjax("QuickLookup");
ga.addParam("sysparm_name", "getValue");
ga.addParam("sysparm_key", "config_item");
ga.getXMLWait();
var answer = ga.getAnswer();
g_form.setValue("config_value", answer);
```

---

## Script Include Templates

Every `GlideAjax` call requires a corresponding **Script Include** that extends `AbstractAjaxProcessor` and is marked **Client Callable**.

### Multi-Method Script Include

```javascript
var CatalogUtils = Class.create();
CatalogUtils.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
  // Return a simple string
  getItemPrice: function () {
    var itemId = this.getParameter("sysparm_item_id");

    var gr = new GlideRecord("sc_cat_item");
    if (gr.get(itemId)) {
      return gr.getValue("price");
    }
    return "0";
  },

  // Return a JSON object (for multiple values)
  getManagerName: function () {
    var userId = this.getParameter("sysparm_user_id");

    var gr = new GlideRecord("sys_user");
    if (gr.get(userId)) {
      var result = {
        manager_sys_id: gr.getValue("manager"),
        manager_name: gr.getDisplayValue("manager"),
        department: gr.getDisplayValue("department")
      };
      return JSON.stringify(result);
    }
    return JSON.stringify({ error: "User not found" });
  },

  // Validation with boolean-style response
  checkApproval: function () {
    var requestType = this.getParameter("sysparm_request_type");

    var gr = new GlideRecord("sc_cat_item");
    gr.addQuery("sys_class_name", requestType);
    gr.addQuery("approval", "!=", "not required");
    gr.query();

    var result = {
      needs_approval: gr.hasNext(),
      approver: ""
    };

    if (gr.next()) {
      result.approver = gr.getValue("approval_group");
    }
    return JSON.stringify(result);
  },

  type: "CatalogUtils"
});
```

### Script Include with Input Validation

```javascript
getUserInfo: function() {
    var userId = this.getParameter('sysparm_user_id');

    // Validate: check it looks like a sys_id
    if (!userId || userId.length !== 32) {
        return JSON.stringify({ error: 'Invalid user ID' });
    }

    var gr = new GlideRecord('sys_user');
    if (gr.get(userId)) {
        return JSON.stringify({ name: gr.getDisplayValue('name') });
    }
    return JSON.stringify({ error: 'User not found' });
}
```

---

## End-to-End Examples

### Dynamic Options Based on Selection (onChange + getXMLAnswer)

When the user selects a department, dynamically load available categories.

```javascript
// ── Catalog Client Script (onChange on 'department' variable) ──
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;

  g_form.clearOptions("category");
  g_form.addOption("category", "", "-- Select --");

  if (!newValue) return;

  var ga = new GlideAjax("CatalogOptionLoader");
  ga.addParam("sysparm_name", "getCategoriesByDept");
  ga.addParam("sysparm_department", newValue);

  ga.getXMLAnswer(function (answer) {
    if (!answer) return;
    var categories = JSON.parse(answer);
    categories.forEach(function (cat) {
      g_form.addOption("category", cat.value, cat.label);
    });
  });
}
```

```javascript
// ── Script Include: CatalogOptionLoader (Client callable = true) ──
var CatalogOptionLoader = Class.create();
CatalogOptionLoader.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    getCategoriesByDept: function () {
      var deptId = this.getParameter("sysparm_department");
      var categories = [];

      var gr = new GlideRecord("sc_category");
      gr.addQuery("department", deptId);
      gr.addQuery("active", true);
      gr.orderBy("title");
      gr.query();

      while (gr.next()) {
        categories.push({
          value: gr.getUniqueValue(),
          label: gr.getValue("title")
        });
      }
      return JSON.stringify(categories);
    },

    type: "CatalogOptionLoader"
  }
);
```

### Form Validation with Server Check (onChange + getXML)

When the user enters an asset tag, validate it exists in the CMDB.

```javascript
// ── Catalog Client Script (onChange on 'asset_tag' variable) ──
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;
  g_form.hideFieldMsg("asset_tag", true);

  if (!newValue) {
    g_form.clearValue("configuration_item");
    return;
  }

  var ga = new GlideAjax("AssetValidator");
  ga.addParam("sysparm_name", "validateAssetTag");
  ga.addParam("sysparm_asset_tag", newValue);

  ga.getXML(function (response) {
    var answer = response.responseXML.documentElement.getAttribute("answer");
    if (!answer) {
      g_form.showFieldMsg(
        "asset_tag",
        "Unable to validate. Try again.",
        "error"
      );
      return;
    }

    var result = JSON.parse(answer);
    if (result.found) {
      g_form.setValue("configuration_item", result.ci_sys_id);
      g_form.showFieldMsg("asset_tag", "Found: " + result.ci_name, "info");
    } else {
      g_form.clearValue("configuration_item");
      g_form.showFieldMsg("asset_tag", "Asset tag not found in CMDB.", "error");
    }
  });
}
```

```javascript
// ── Script Include: AssetValidator (Client callable = true) ──
var AssetValidator = Class.create();
AssetValidator.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
  validateAssetTag: function () {
    var assetTag = this.getParameter("sysparm_asset_tag");

    if (!assetTag) {
      return JSON.stringify({ found: false, error: "No asset tag provided" });
    }

    var gr = new GlideRecord("cmdb_ci");
    gr.addQuery("asset_tag", assetTag);
    gr.setLimit(1);
    gr.query();

    if (gr.next()) {
      return JSON.stringify({
        found: true,
        ci_sys_id: gr.getUniqueValue(),
        ci_name: gr.getDisplayValue("name"),
        ci_class: gr.getDisplayValue("sys_class_name")
      });
    }
    return JSON.stringify({ found: false });
  },

  type: "AssetValidator"
});
```

### onLoad with GlideAjax (Pre-populate Fields)

On form load, fetch the current user's location and department to pre-fill fields.

```javascript
// ── Catalog Client Script (onLoad) ──
function onLoad() {
  var ga = new GlideAjax("UserContextLoader");
  ga.addParam("sysparm_name", "getCurrentUserContext");

  ga.getXMLAnswer(function (answer) {
    if (!answer) return;
    var context = JSON.parse(answer);

    g_form.setValue("requested_for_location", context.location);
    g_form.setValue("requested_for_department", context.department);

    if (context.is_remote) {
      g_form.setDisplay("vpn_access_section", true);
    }
  });
}
```

```javascript
// ── Script Include: UserContextLoader (Client callable = true) ──
var UserContextLoader = Class.create();
UserContextLoader.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    getCurrentUserContext: function () {
      var userId = gs.getUserID();
      var gr = new GlideRecord("sys_user");

      if (gr.get(userId)) {
        return JSON.stringify({
          location: gr.getDisplayValue("location"),
          department: gr.getDisplayValue("department"),
          is_remote: gr.getValue("vip") == "true"
        });
      }
      return JSON.stringify({ location: "", department: "", is_remote: false });
    },

    type: "UserContextLoader"
  }
);
```

---

## Common User Requests Mapping

| User Request                    | Agent Implementation                                |
| ------------------------------- | --------------------------------------------------- |
| "Set defaults on form load"     | Catalog Client Script (onLoad)                      |
| "Show/hide field on selection"  | Catalog Client Script (onChange)                    |
| "Validate form before submit"   | Catalog Client Script (onSubmit)                    |
| "Auto-fill fields on selection" | Catalog Client Script (onChange) with GlideAjax     |
| "Reusable script across items"  | Client Script on Variable Set with appliesTo: "set" |

---

## Quick Reference

### ALWAYS

- Guard onChange scripts with `if (isLoading) return;`
- Use `Now.include(...)` for external script files
- Use strings inside script code (e.g., `g_form.getValue('urgency')`)
- Use `g_form` API for all field manipulations

### NEVER

- Use GlideAjax in onSubmit scripts (async issues)
- Manipulate DOM directly — always use `g_form` API
- Import functions from JavaScript modules in client scripts
