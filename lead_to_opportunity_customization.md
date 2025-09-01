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

### Link Hidden script (Connections)
```js
setTimeout(() => {
      $('[data-doctype="Prospect"]').closest('.document-link').hide();
    }, 500);
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

## api.py
```pyhton
from erpnext.crm.doctype.lead.lead import make_opportunity as core_make
@frappe.whitelist()
def make_opportunity_patched(source_name, target_doc=None):
    # Call core to build the mapped doc
    opp = core_make(source_name, target_doc)

    # Post-process for differently-named fields
    lead = frappe.get_doc("Lead", source_name)

    # EXAMPLES (rename to your actual fields)
    # If same names exist, these are not needed — only use for different names.
    opp.custom_opportunity_priority = lead.custom_lead_priority
    opp.custom_segment = lead.custom_source_segment
    # Child tables? map here as well

    return opp
```

## hooks.py
```python
override_whitelisted_methods = {
  "erpnext.crm.doctype.lead.lead.make_opportunity": "finerp.api.make_opportunity_patched"
}
```

## lead.js
```js
frappe.ui.form.on('Lead', {
  refresh(frm) {
    frm.page.remove_inner_button(__('Opportunity'), __('Create'));

    frm.add_custom_button(__('Opportunity'), () => {
      const go = () => frappe.model.open_mapped_doc({
        method: "erpnext.crm.doctype.lead.lead.make_opportunity",
        frm
      });
      frm.is_dirty() ? frm.save().then(go) : go();
    }, __('Create'));

    const prune = () => {
      frm.remove_custom_button(__('Customer'), __('Create'));
      frm.remove_custom_button(__('Prospect'), __('Create'));
      frm.remove_custom_button(__('Quotation'), __('Create'));

      if (frm.page && frm.page.remove_inner_button) {
        frm.page.remove_inner_button(__('Customer'), __('Create'));
        frm.page.remove_inner_button(__('Prospect'), __('Create'));
        frm.page.remove_inner_button(__('Quotation'), __('Create'));
      }
    };

    prune();                 
    setTimeout(prune, 120); 
    setTimeout(prune, 400);

    setTimeout(() => {
      frm.remove_custom_button(__('Add to Prospect'), __('Action'));
      if (frm.page && frm.page.remove_inner_button) {
        frm.page.remove_inner_button(__('Add to Prospect'), __('Action'));
      }
    }, 150);
  }
});

```

## lead.js (for quick dom manipulation)
```js
frappe.ui.form.on('Lead', {
  refresh(frm) {

    setTimeout(() => {
      $('[data-doctype="Prospect"]').closest('.document-link').hide();
    }, 500);

    const prune = () => {
      frm.remove_custom_button(__('Customer'), __('Create'));
      frm.remove_custom_button(__('Prospect'), __('Create'));
      frm.remove_custom_button(__('Quotation'), __('Create'));
      if (frm.page && frm.page.remove_inner_button) {
        frm.page.remove_inner_button(__('Customer'), __('Create'));
        frm.page.remove_inner_button(__('Prospect'), __('Create'));
        frm.page.remove_inner_button(__('Quotation'), __('Create'));
      }
      frm.remove_custom_button(__('Add to Prospect'), __('Action'));
      if (frm.page && frm.page.remove_inner_button) {
        frm.page.remove_inner_button(__('Add to Prospect'), __('Action'));
      }
    };

    prune();

    frappe.after_ajax(() => {
      prune();

      if (frm.page && frm.page.remove_inner_button) {
        frm.page.remove_inner_button(__('Opportunity'), __('Create'));
      }
      frm.remove_custom_button(__('Opportunity'), __('Create'));

      frm.add_custom_button(__('Opportunity'), () => {
        const go = () => frappe.model.open_mapped_doc({
          method: 'erpnext.crm.doctype.lead.lead.make_opportunity',
          frm
        });
        frm.is_dirty() ? frm.save().then(go) : go();
      }, __('Create'));
    });

    setTimeout(prune, 50);
    setTimeout(prune, 120);

    if (!frm._createObserver) {
      const node = frm.page?.inner_toolbar?.get?.(0);
      if (node) {
        frm._createObserver = new MutationObserver(prune);
        frm._createObserver.observe(node, { childList: true, subtree: true });
      }
    }
  },

  onhide(frm) {
    if (frm._createObserver) {
      frm._createObserver.disconnect();
      frm._createObserver = null;
    }
  }
});

```

## fetch multiselect field from opportunity, while creating project from quotation
```py
if opp.get("custom_bd_team_members"):
        bd_table = frappe.get_meta("Opportunity").get_field("custom_bd_team_members").options
        if bd_table:
            bd_members = frappe.db.sql("""
                SELECT DISTINCT user 
                FROM `tab{0}` 
                WHERE parent = %s AND parenttype = 'Opportunity'
            """.format(bd_table), opp.name, as_dict=1)
            
            existing_users = set()
            for member in bd_members:
                if member.user and member.user not in existing_users:
                    project.append("custom_bd_team_members", {"user": member.user})
                    existing_users.add(member.user)
```
