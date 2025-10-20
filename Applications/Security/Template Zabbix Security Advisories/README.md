# CVE Monitoring for Zabbix

This template is for Zabbix version: 6.0  
Also available for: 7.4 7.2 7.0 6.4 6.2 

CVE Monitoring
Overview
Template to discover and monitor Zabbix-related CVEs using a single master JSON item. The template performs Low-Level Discovery (LLD) from a master JSON payload, extracts severity via preprocessing, creates dependent items per CVE, provides trigger prototypes for High/Critical severities, includes a master-item health trigger, and supports excluding handled CVEs via a template macro.

Requirements
Zabbix version: 6.0 and higher.

Tested versions
This template has been tested on:
- Zabbix 6.x, 7.x (import and validation recommended on your target version)

Configuration
Zabbix should be configured according to the Templates out of the box instructions.

Setup
1. Ensure you have a host to link the template and a master collector that returns JSON consumable by the discovery rule.
2. If using the provided Python collector, install Python 3.8+ and required packages (see Master collector requirements).
3. Import and link the template (see Import section).

Set the master collector endpoint or script on the host and configure template macros as needed.

Macros used
Name — Description — Default
{$CVE_IGNORE} — Pattern of CVE identifiers to exclude from discovery — ^()$
{$ZBXVERSION} — Optional macro used in master item key examples — (none)

Discovery rule and produced items
Discovery rule
- Source: master JSON item (text)
- LLD macros produced: {#CVENUMBER}, {#ZABBIXID}, {#ZABBIXSEVERITY}
- Discovery filter: {#CVENUMBER} does not match {$CVE_IGNORE}

Item prototypes created by discovery (examples)
- CVE {#CVENUMBER}: Severity — Dependent item, preprocessed to store values like Critical/High/Medium
- CVE {#CVENUMBER}: Description / Link — Dependent item, raw text
- CVE {#CVENUMBER}: First seen — Dependent item, timestamp
- Additional dependent items as needed (references, solution status)

Template-level items
- Master item (external/script or HTTP agent) that returns the raw JSON used by LLD
  - Key (external script): getCve.py[{$ZBXVERSION}]
  - Type of information: Text
  - Recommended update interval: 1d

Preprocessing notes
- Severity extraction example:
  1. JSONPath: $.data[?(@.ZABBIXID=="{#ZABBIXID}")].ZABBIXSEVERITY
  2. Regex cleanup: ^\["([^"]+)"\]$ → output: \1
- Test preprocessing steps in the item Test dialog to confirm final values (e.g., Critical)

Triggers
Trigger prototypes (discovery)
- CVE severity (prototype)
  - Description: Fires on High or Critical severity
  - Example expression (prototype context): last(/Template/cve.severity[{#ZABBIXID}])="Critical" or last(/Template/cve.severity[{#ZABBIXID}])="High"

Template-level triggers
- Master item failure — please check CVE data collection
  - Expression example: {Template:getCve.py[{$ZBXVERSION}].nodata(10m)}=1
  - Severity: Warning (adjust as needed)

Import and configuration (quick steps)
1. Import template: Configuration → Templates → Import → upload XML/YAML.  
2. Link template to host: Configuration → Hosts → select host → Templates → Link new template.  
3. Create master item on that host:
   - Key example (external script): getCve.py[{$ZBXVERSION}] or HTTP agent key matching your collector
   - Type: External script / HTTP agent / Zabbix agent (match your collector)
   - Type of information: Text
   - Interval: 300s (recommended)
   - Ensure the item returns valid JSON with a top-level `data` array and required fields.
4. Set macro {$CVE_IGNORE} on the template (default ^()$). To ignore handled CVEs add identifiers: (CVE-2022-23131|CVE-2025-27237).
5. Run discovery (wait next LLD or force run) and confirm items and triggers are created.

Master collector (Python) requirements
- Python 3.8+ installed on the host that runs the collector (or environment reachable by Zabbix HTTP agent).  
- Required Python packages:
  - requests
- Collector expectations:
  - Outputs valid UTF-8 JSON to stdout (external script) or HTTP response body (HTTP agent).
  - JSON structure: top-level object with `data` array; each element must include keys used by LLD: CVE id, ZABBIXID, ZABBIXSEVERITY, description, timestamp.
  - On failure, return empty output or a clearly detectable error so the master-item failure trigger can fire.

Testing checklist
- Master item returns valid JSON (use Test button).  
- Severity preprocessing returns plain value (e.g., Critical).  
- LLD creates items with {#CVENUMBER} and {#ZABBIXID}.  
- Adding a CVE to {$CVE_IGNORE} prevents that CVE from being discovered on the next LLD run.  
- Stopping the collector triggers the master-item failure alert (nodata(10m)).

Files included in this template package
- getCve.py
- template-cve-monitoring.xml (exported template)  
- README.md (this file)  

Feedback
Please report any issues with the template at https://support.zabbix.com or via the Zabbix community repository PR/discussion.

Notes
- Remove any credentials or host-specific identifiers from the exported XML before publishing.  
- Validate import on a clean Zabbix instance before submitting a Pull Request.
