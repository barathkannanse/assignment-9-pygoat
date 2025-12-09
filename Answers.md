# Answers to Part 3

Add your answers to the questions in Part 3, Step 2 below. 

## Vulernability Remediation:
### Vulnerability 1:
1. Which package or library are you addressing?
   - We are focusing on the PyYAML library that the project uses for parsing YAML data. PyYAML is convenient, but older versions have a known remote-code-execution issue if they are used with unsafe loaders on untrusted input.

2. Which CVE is linked to this vulnerability?
   - The issue being  addressed is CVE-2017-18342. This CVE describes how older PyYAML releases allow `yaml.load()` to construct arbitrary Python objects while it parses YAML. If the YAML content comes from an attacker, they can abuse that behavior to trigger code execution on the server.

3. What remediation steps do you suggest?
   - Upgrade the dependency: Update PyYAML to a patched release in `requirements.txt`, then rebuild the Docker image so the container no longer ships with the vulnerable version.  
   - Use safe loaders: In the application code, make sure only `yaml.safe_load()` or other restricted loaders are used for any YAML that could possibly come from outside the application. These loaders only deserialize simple types and do not allow arbitrary object construction.  
   - Re-run the scan: After upgrading and changing the loader usage, run the CI pipeline again and confirm that the PyYAML CVE is no longer reported in the Trivy results.

### Vulnerability 2:
1. Which vulnerability are you addressing?
   - For the second example we are looking at the `requests` HTTP client library, which the project uses to make outbound HTTP calls. Some versions of `requests` mishandle redirects in a way that can leak authentication headers to the wrong host, which turns into a credential-exposure problem.

2. Which CVE is linked to this vulnerability?  
   - The relevant issue is CVE-2018-18074. In affected versions, if a request with an `Authorization` header is sent to one host and the response causes a redirect to a different host, `requests` may forward that same `Authorization` header along with the redirected request. That means credentials intended for one server can accidentally be sent to another server, possibly under an attacker’s control.

3. What remediation steps do you suggest?  
   - Upgrade `requests` to a fixed version: Bump the `requests` dependency in `requirements.txt` to a release where this behavior is corrected, then rebuild the container so the application is running with the patched library.  
   - Be strict about redirects and headers: Review any code that follows redirects automatically, especially when using session objects. Avoid blindly forwarding sensitive headers across hosts. If needed, strip those headers when the redirect target is a different domain.  
   - Re-scan after the change: Once the dependency is upgraded and redirect handling is tightened, re-run the vulnerability scan to verify that the CVE no longer appears in the report and that there are no new high-severity issues introduced by the upgrade.
