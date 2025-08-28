# Official Customization & Extension Approach in ERPNext/Frappe

## 🔍 Research Findings

### Core APIs and Functions
- ERPNext/Frappe provides built-in mapping functions such as:  
  `frappe.model.open_mapped_doc` and `frappe.model.mapper.get_mapped_doc`.
- These should be reused whenever possible to keep logic consistent with core.
- `get_mapped_doc` automatically copies fields with identical **fieldnames** across Doctypes.
- For fields with different names, extend mapping via `field_map` or `postprocess`.

### Button Customization
- Avoid hiding entire menus with raw DOM/CSS selectors.
- Use the official API to remove specific buttons:
  ```js
  frm.remove_custom_button(__('Unwanted Option'), __('Create'));
  ```
- Add custom buttons using:
  ```js
  frm.add_custom_button(__('Custom Action'), () => {
      frappe.model.open_mapped_doc({
          method: "path.to.whitelisted.method",
          frm: frm
      });
  }, __('Create'));
  ```

### Custom Field Mapping
- **Option 1 (No Code):** Use the same `fieldname` on both source and target Doctypes → values auto-copied.
- **Option 2 (Code):** Wrap or extend core mapping functions for custom logic.

Example wrapper:
```python
import frappe
from some_app.some_module import core_function as base_function

@frappe.whitelist()
def custom_mapped_doc(source_name, target_doc=None):
    doc = base_function(source_name, target_doc)
    source = frappe.get_doc("Source Doctype", source_name)

    # Example of mapping fields with different names
    doc.custom_field_x = source.other_field_y
    doc.segment_type = source.custom_segment

    return doc
```

### Hooks for Extension
Use `hooks.py` to override whitelisted methods cleanly:
```python
override_whitelisted_methods = {
    "path.to.core.method": "your_app.api.custom_mapped_doc"
}
```

### Why Reuse Core APIs?
- **Upgrade-safe:** Core changes automatically flow through.  
- **Avoid duplication:** No need to replicate validations and defaults.  
- **Maintainable:** Only your custom logic is added, core remains untouched.

---

## 🛠️ Recommended Development Approach

1. **Hide or Adjust UI Actions**  
   - Remove only what is not required via `frm.remove_custom_button`.
   - Provide clear, role-based actions via `frm.add_custom_button`.

2. **Use Built-in Mapping Functions**  
   - Prefer `frappe.model.open_mapped_doc` and core APIs.
   - Avoid writing duplicate insert/create logic unless absolutely necessary.

3. **Handle Custom Fields Properly**  
   - Same `fieldname` → auto-mapped.
   - Different names → add lightweight wrapper/postprocess logic.

4. **Maintain Upgrade Safety**  
   - Use overrides via hooks rather than editing core files.
   - Keep business logic extensions minimal.

---

## ✅ Quick Test Plan Template

1. **Custom Field Mapping**  
   - Add values on source doc → verify copied to target doc.

2. **UI Behavior**  
   - Check only required buttons/options are available.  
   - Verify new custom button opens/creates the correct document.

3. **Role & Permission Matrix**  
   - Ensure visibility/actions match user roles.

4. **End-to-End Flow**  
   - Simulate complete document lifecycle.  
   - Verify integrity of data across mapped doctypes.

---

## 📅 Summary

- Use **existing ERPNext/Frappe APIs** (`open_mapped_doc`, `get_mapped_doc`) for creation/mapping flows.  
- **Avoid duplicating** core logic; extend only where necessary.  
- **Map custom fields** via same fieldnames or wrapper functions.  
- **Modify UI** with `frm.remove_custom_button` and `frm.add_custom_button`.  
- **Use hooks** to extend/override safely.  

This approach ensures customizations are **optimistic, efficient, and upgrade-safe** for any ERPNext/Frappe development.
