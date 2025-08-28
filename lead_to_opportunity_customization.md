# ERPNext CRM Customization: Lead → Opportunity (Optimistic Approach)

## 🔍 Research Findings

### Core APIs and Functions
- ERPNext already provides a built-in method:  
  `erpnext.crm.doctype.lead.lead.make_opportunity`
- This uses `frappe.model.mapper.get_mapped_doc` internally to map fields from Lead → Opportunity.
- `get_mapped_doc` automatically copies fields with the same fieldname on both Doctypes.
- For fields with different names, you can extend the mapping via `field_map` or a postprocess function.

### Button Customization
- Instead of hiding the entire “Create” dropdown with DOM selectors, use:
  ```js
  frm.remove_custom_button(__('Customer'), __('Create'));
  frm.remove_custom_button(__('Quotation'), __('Create'));
  ```
- Then add your own custom button for Opportunity:
  ```js
  frm.add_custom_button(__('Create Opportunity'), () => {
      frappe.model.open_mapped_doc({
          method: "erpnext.crm.doctype.lead.lead.make_opportunity",
          frm: frm
      });
  }, __('Create'));
  ```

### Custom Field Mapping
- **Option 1 (No Code):** Use the same fieldname for custom fields in both Lead and Opportunity → auto-mapped.
- **Option 2 (Code):** Override or wrap the core method to add extra mappings.

Example:
```python
import frappe
from erpnext.crm.doctype.lead.lead import make_opportunity as core_make

@frappe.whitelist()
def make_opportunity_patched(source_name, target_doc=None):
    opp = core_make(source_name, target_doc)
    lead = frappe.get_doc("Lead", source_name)

    # Example: mapping different fieldnames
    opp.custom_priority = lead.lead_priority
    opp.custom_segment = lead.lead_segment

    return opp
```

Hooks:
```python
override_whitelisted_methods = {
    "erpnext.crm.doctype.lead.lead.make_opportunity": "your_app.api.make_opportunity_patched"
}
```

### Why Reuse Core API Instead of Custom Code?
- Upgrade-safe (future ERPNext changes auto-applied).
- Avoids duplicate business logic (validations, defaults).
- Cleaner and more maintainable.

---

## 🛠️ Recommended Approach

1. **Hide unwanted Create buttons**  
   Remove “Customer”, “Quotation” etc., keep only “Opportunity”.

2. **Add your Opportunity button**  
   Use `frappe.model.open_mapped_doc` with `make_opportunity`.

3. **Handle custom fields**  
   - If same fieldnames: no code required.  
   - If different: add lightweight wrapper function.

4. **Do not** auto-create Customer or set Lead status to Converted when creating an Opportunity.

---

## ✅ Quick Test Plan

1. **Lead → Opportunity with same fieldnames**  
   - Fill Lead custom fields  
   - Create Opportunity → values copied correctly

2. **Lead → Opportunity with different fieldnames**  
   - Enable wrapper mapping  
   - Verify mapped correctly

3. **UI Behavior**  
   - Check Create dropdown shows only Opportunity  
   - Button opens mapped Opportunity form

4. **Permissions**  
   - Sales User can use  
   - Other roles behave per matrix

---

## 📅 Summary

- Use `make_opportunity` (ERPNext core API)  
- Hide only unwanted Create options  
- Map custom fields either via same fieldname or wrapper  
- Avoid Customer creation and Lead status changes when only creating Opportunity

This ensures an **optimistic, efficient, and upgrade-safe** CRM customization.
