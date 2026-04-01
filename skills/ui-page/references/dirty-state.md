# Dirty State Management

Track unsaved changes to prevent data loss. ServiceNow components use `onXxxSet` event naming with data in `event.detail`.

## Dirty State Pattern with RecordProvider

```tsx
function RecordForm({ recordId }: { recordId: string }) {
  const [formData, setFormData] = useState({});
  const [originalData, setOriginalData] = useState({});
  const [isDirty, setIsDirty] = useState(false);

  useEffect(() => {
    setIsDirty(JSON.stringify(formData) !== JSON.stringify(originalData));
  }, [formData, originalData]);

  // Warn on browser navigation when dirty
  useEffect(() => {
    if (!isDirty) return;
    const handler = e => {
      e.preventDefault();
      e.returnValue = "";
    };
    window.addEventListener("beforeunload", handler);
    return () => window.removeEventListener("beforeunload", handler);
  }, [isDirty]);

  const handleFieldChange = fieldName => event => {
    const newValue = event.detail?.value ?? event.detail;
    setFormData(prev => ({ ...prev, [fieldName]: newValue }));
  };

  const handleRecordLoaded = record => {
    setFormData(record);
    setOriginalData(record);
  };

  return (
    <RecordProvider
      table="sys_user"
      sysId={recordId}
      onRecordLoaded={handleRecordLoaded}
    >
      {isDirty && <div>⚠️ Unsaved changes</div>}
      <RecordField
        field="name"
        onValueSet={handleFieldChange("name")}
      />
      <RecordField
        field="email"
        onValueSet={handleFieldChange("email")}
      />
      <RecordField
        field="phone"
        onValueSet={handleFieldChange("phone")}
      />
      <RecordField
        field="department"
        onValueSet={handleFieldChange("department")}
      />
      <Button
        label="Save"
        variant="primary"
        disabled={!isDirty}
      />
    </RecordProvider>
  );
}
```

## Key Points

- Read component docs with `package_docs` for event names (`onValueSet`, etc.)
- ServiceNow components use `event.detail.value`, not `event.target.value`
- Track dirty state by comparing original vs current data with `JSON.stringify()`
- Warn users before navigation when there are unsaved changes
- Reset dirty state after successful save
