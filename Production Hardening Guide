# Frappe/ERPNext Production Hardening Guide

## 1\. Overview

This guide provides the necessary steps to resolve common information disclosure vulnerabilities found during a Vulnerability Assessment and Penetration Test (VAPT). These issues, including **Stack Trace Exposure**, **Information Disclosure via Detailed Error Messages**, and **Local File System Path Disclosure**, typically arise when a Frappe instance is running in a development-oriented mode in a production environment.

Following these steps will transition your instance to a secure, production-ready state, ensuring that sensitive internal information is not exposed to clients.

## 2\. Resolution Steps

Execute these commands from your Frappe bench directory (e.g., `~/frappe-bench`). Replace `your.site.name` with the actual name of your site.

### Step 1: Disable Developer Mode

Developer mode is the primary source of verbose error messages and stack traces. It must be disabled in production.bash

# Check the current status of developer mode (1 is on, 0 is off)

bench --site your.site.name get-config developer\_mode

# Disable developer mode

bench --site your.site.name set-config developer\_mode 0

````

### Step 2: Disable Error Snapshots

Error snapshots capture detailed state information during an error, which can include sensitive data. This feature should be disabled in production to prevent logging potentially confidential information.[1]

```bash
# Add the disable_error_snapshot flag to your site_config.json
bench --site your.site.name set-config disable_error_snapshot 1
````

### Step 3: Disable Full Error Reports in System Settings

The Frappe UI has a setting that can show detailed errors. This must be disabled to ensure users only see generic error pages.[2]

1.  Log in to your Frappe/ERPNext instance as an Administrator.
2.  Navigate to the **System Settings** page (you can use the awesomebar to search for it).
3.  Find the "Security" section.
4.  Ensure the checkbox labeled **"Show Full Error and Allow Reporting of Issues to the Developer"** is **unchecked**.
5.  Save the settings.

### Step 4: Rebuild Static Assets for Production

To remove sensitive file paths from client-side assets like JavaScript and CSS files, you must rebuild them in production mode.

```bash
# Force a rebuild of all static assets (JS, CSS, etc.)
bench build --force

# Clear the cache and restart the server for changes to take effect
bench --site your.site.name clear-cache
bench restart
```

*Note: For a production setup, you may need to restart supervisor (`sudo supervisorctl restart all`).*

### Step 5: Verify Production Environment

Finally, ensure your instance is running with the production process manager (`supervisor`) and web server (`nginx`), not the development server (`bench start`).[3, 4]

```bash
# Check the status of supervisor processes
sudo supervisorctl status

# If not set up, configure for production (replace [frappe-user] with your user)
sudo bench setup production [frappe-user]
sudo supervisorctl restart all
```

By completing these steps, you will have successfully hardened your Frappe instance, resolved the VAPT findings, and adopted a secure posture for your production environment.

```
```
