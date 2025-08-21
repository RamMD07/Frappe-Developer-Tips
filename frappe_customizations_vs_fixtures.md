# Frappe Customizations: Export Customizations vs Fixtures

## 1. Export Customizations (UI button)
- **What it exports:** Only **Custom Fields** + **Property Setters** for the selected DocType.  
- **Output format:** **One JSON per DocType** → very readable and organized.  
- **Best for:** Simple form-level tweaks (fields, labels, reqd, in list view).  
- **Caveat:** Import is **additive** — won’t auto-delete old records.  
- **Fix:** Use patch/reset if you want DB = JSON strictly.  

---

## 2. Fixtures (`hooks.py`)
- **What it exports:** Any DocType records (Custom Fields, Property Setters, Custom Scripts, Print Formats, Workspaces, etc.).  
- **Output format:** **One JSON per fixture Doctype** (e.g., one `custom_field.json` for all doctypes).  
- **Best for:** When you also want to version control Custom Scripts, Print Formats, Workspaces, etc.  
- **Caveat:** Same **additive** behavior → won’t auto-delete old records.  
- **Fix:** Same as above (patch/reset).  

### Example: Filtered Fixtures
```python
fixtures = [
    {
        "doctype": "Custom Field",
        "filters": [["dt", "in", ["Sales Invoice", "Item", "Purchase Order"]]]
    },
    {
        "doctype": "Property Setter",
        "filters": [["doc_type", "in", ["Sales Invoice", "Item", "Purchase Order"]]]
    },
    # Optional extras:
    # {"doctype": "Custom Script", "filters": [["dt","in",["Sales Invoice"]]]},
    # {"doctype": "Print Format", "filters": [["doc_type","in",["Sales Invoice"]]]},
]
```

---

## 3. Key Differences

| Feature                 | Export Customizations | Fixtures |
|--------------------------|----------------------|----------|
| **Granularity**          | Per-DocType JSON     | Per-Doctype-type JSON (e.g. one for all CFs) |
| **Readability**          | ✅ Clear, one file per DocType | ❌ Mixed, needs filters to stay tidy |
| **Supports only**        | Custom Fields + Property Setters | Any DocType (CF, PS, Scripts, Workspaces, etc.) |
| **Auto-deletes old?**    | ❌ No | ❌ No |
| **Recommended for**      | Small projects, form tweaks | Full apps with multiple customization types |

---

## 4. Cleanup Requirement
⚠️ **Neither method deletes old Custom Fields / Property Setters automatically.**

### Solutions:
- One-time cleanup patch:
```python
# your_app/patches/remove_old_customizations.py
import frappe

def execute():
    doctypes_to_reset = ["Sales Invoice", "Item"]
    for dt in doctypes_to_reset:
        frappe.db.delete("Custom Field", {"dt": dt})
        frappe.db.delete("Property Setter", {"doc_type": dt})
    frappe.clear_cache()
```

- Or reset manually:
```bash
bench --site <yoursite> execute frappe.custom.doctype.customize_form.customize_form.reset_to_defaults --kwargs "{'doctype': 'Sales Invoice'}"
bench --site <yoursite> migrate
```

---

## 5. Best Practices
1. **Pick one method** → don’t mix Export Customizations + Fixtures for the same DocTypes.  
2. **Keep `fieldname`s stable** → avoid accidental duplicates.  
3. **Clean once** → delete old DB records, then rely only on JSON.  
4. **Block Customize Form on live** → avoid “live-only” changes.  
5. **Always deploy via code → migrate** → ensures consistency.  

---

✅ **Summary:**  
- Use **Export Customizations** for clean per-DocType JSON (nice for HRMS-like apps).  
- Use **Fixtures** when you need more than fields/setters (scripts, prints, workspaces).  
- In both cases → add a **reset/patch step** to enforce *optimistic behavior* (DB = JSON only).  
