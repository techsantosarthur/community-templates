# CVE Monitoring Template for Zabbix

## Description
Centralized Zabbix template to discover and monitor CVEs affecting Zabbix components. The template uses a single master JSON item for Low-Level Discovery (LLD), extracts severity values with preprocessing, provides trigger prototypes for high/critical severities, and supports excluding already-handled CVEs via a template macro.

## Prerequisites
- Zabbix server **6.0** or later.  
- Python 3.8+ recommended.

## Import instructions
1. In Zabbix frontend go to **Configuration → Templates → Import**.  
2. Select the exported template file (XML or YAML) and import it.  
3. Link the template to the target host: **Configuration → Hosts → select host → Templates → Link new template → choose imported template**.  
4. Create or configure the master item on the same host (see next section).

## Configure master item and macros

### Master item
- **Item key example**: `getCve.py[{$ZBXVERSION}]` or a custom key matching your collector.  
- **Type**: HTTP agent / Zabbix agent / External script depending on your deployment.  
- **Type of information**: **Text**.  
- **Update interval**: recommended **5m**.  
- **Returned payload**: JSON with a top-level `data` array; each discovery object must include LLD keys `{#CVENUMBER}`, `{#ZABBIXID}`, `{#ZABBIXSEVERITY}`.

### Template macro for excluding handled CVEs
- **Macro name**: `{$CVE_IGNORE}`  
- **Default value**: `^()$` (matches nothing; no CVEs ignored).  
- To ignore one CVE: `(CVE-2022-23131)`  
- To ignore multiple CVEs: `(CVE-2022-23131|CVE-2025-27237)`  
- The discovery rule must use a filter: **Macro** `{#CVENUMBER}` **Condition** `does not match` **Value** `{$CVE_IGNORE}`.

## Preprocessing and trigger notes
- Severity preprocessing for the severity item:
  1. **JSONPath**: `$.data[?(@.ZABBIXID=="{#ZABBIXID}")].ZABBIXSEVERITY`  
  2. **Regular expression**: `^\["([^"]+)"\]$` → **Output** `\1`  
- Master item health trigger example:
  - **Name**: `Master item getCve.py failure — please check CVE data collection`  
  - **Expression**: `{Template:getCve.py[{$ZBXVERSION}].nodata(10m)}=1`

## Testing steps
1. Use the **Test** button in the master item to verify the raw JSON payload is returned.  
2. Test the JSONPath preprocessing step and confirm it returns an array like `["Critical"]`.  
3. Test the regular expression preprocessing step and confirm final output is `Critical`.  
4. Run the discovery rule and confirm LLD creates items with `{#CVENUMBER}` and `{#ZABBIXID}`.  
5. Add a CVE identifier to `{$CVE_IGNORE}` and run discovery again to confirm the listed CVE is no longer created.  
6. Verify the master-item failure trigger fires when the master item stops returning data.

## Files to include in the GitHub pull request
- Template export file (XML or YAML) named clearly, e.g., `template-cve-monitoring.xml`.  
- `README.md` (this file) in English.  
- `LICENSE` file (recommended MIT or other OSI-compatible license).  
- `CHANGELOG.md` with at least an initial entry describing version and test validation.  
- Optional: `screenshots/` with images demonstrating preprocessing tests and discovery filter configuration.  
- Optional: example host configuration snippets (plain text) or sample master collector script if you want to provide usage examples.

## Additional notes for PR acceptance
- Ensure the exported XML/YAML does **not** contain any secrets or API keys.  
- Validate the template by importing it into a clean Zabbix instance before opening the PR.  
- In the PR description include supported Zabbix versions, brief install steps, and a short test checklist.
