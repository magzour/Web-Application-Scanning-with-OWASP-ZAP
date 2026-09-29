# Web-Application-Scanning-with-OWASP-ZAP

Here is your complete, professionally formatted GitHub README / Wiki page, organized according to the Web Application Vulnerability Assessment Report structure with clean Markdown tables, callouts, syntax-highlighted code blocks, and all of your original images and instructions preserved.

---

# Web Application Vulnerability Assessment with OWASP ZAP

| Attribute | Details |
| --- | --- |
| **Tool Used** | OWASP ZAP (Zed Attack Proxy) |
| **Assessment Type** | Dynamic Application Security Testing (DAST) / Hybrid (Manual Exploration + Active Scan) |
| **Target Application** | `[http://testphp.vulnweb.com](http://testphp.vulnweb.com)` / `demo.testfire.net` |
| **Platform** | Windows 10/11 |
| **Objective** | Identify, analyze, and document web application vulnerabilities (DAST, CSRF, insecure headers, and mixed content) using industry-standard AppSec methodologies. |

---

## 1. Executive Summary

This project demonstrates a comprehensive Dynamic Application Security Testing (DAST) assessment using **OWASP ZAP**. The objective was to configure a controlled testing environment, intercept and inspect web traffic, conduct automated and manual crawls, and execute active scans against intentionally vulnerable web applications.

Through a hybrid testing approach (manual browser exploration combined with active automated scanning), multiple security weaknesses were successfully discovered, triaged, and documented. Key findings include the **absence of anti-CSRF tokens**, **missing defensive security headers (CSP, HSTS, X-Frame-Options)**, **active mixed-content script inclusion**, and **information leakage via server response banners**.

---

## 2. Environment Setup & Prerequisites

### Required Tools

* **OWASP ZAP**: [Download Official Installer](https://www.zaproxy.org/download/)
* **Target Sandbox Applications** (Authorized training environments only):
* `[http://testphp.vulnweb.com](http://testphp.vulnweb.com)` (Acunetix Test Site)
* `[https://juice-shop.herokuapp.com](https://juice-shop.herokuapp.com)` (OWASP Juice Shop)
* `[http://demo.testfire.net](http://demo.testfire.net)` (Altoro Mutual Test Site)



> [!WARNING]
> **Rules of Engagement:** Do **not** attempt to scan live websites, client applications, or production environments without explicit, written authorization.

---

### Installation Troubleshooting: Windows SmartScreen & Unsigned Packages

If Windows SmartScreen prevents the `.exe` installer from executing:

1. **Unblock via GUI:** Right-click the `.exe` file, select **Properties**, check **Unblock** at the bottom, and click **Apply**.
2. **Unblock via PowerShell (Run as Administrator):**
```powershell
Unblock-File -Path "C:\Path\To\ZAP_Installer.exe"

```


3. **Install via Windows Package Manager (winget):**
```cmd
winget install --id=ZAP.ZAP -e

```


*Type `Y` when prompted to accept license agreements.*

If SmartScreen continues to block the unsigned binary:

4. **Cross-Platform Package Alternative:**
* Download the cross-platform ZIP package from the [OWASP ZAP Download Page](https://www.zaproxy.org/download/).
* Extract the ZIP archive and execute the startup launcher directly:



---

## 3. Proxy & Browser Integration Configuration

### Step 1: Automated Browser Integration & HUD

1. Launch OWASP ZAP.
2. Select your preferred browser from the quick-start screen and enable the **Heads Up Display (HUD)**.

3. Enter the target application URL (e.g., `[http://testphp.vulnweb.com](http://testphp.vulnweb.com)`) and launch the browser session.

---

### Step 2: Troubleshooting "The provided browser was not found"

If ZAP fails to locate your browser installation automatically:

1. Confirm the target site is online.
2. In ZAP, navigate to: **Tools** > **Options** > **Selenium**.
3. Under **Chrome** > **Binary**, click **Browse** and specify the absolute path to your Chrome executable:
```text
C:\Program Files\Google\Chrome\Application\chrome.exe

```


4. Save the configuration and restart ZAP.

---

### Step 3: Installing the ZAP Root CA Certificate

To allow ZAP to decrypt and inspect HTTPS traffic without SSL/TLS security warnings (`NET::ERR_CERT_AUTHORITY_INVALID`):

1. In ZAP, navigate to: **Tools** > **Options** > **Network** > **Server Certificates**.
2. Click **Save...** to export the root certificate (e.g., `zap_root_ca.cer`).
3. Press `Win + R`, enter `certmgr.msc`, and press **Enter** to open Windows Certificate Manager.
4. Navigate to: **Trusted Root Certification Authorities** > right-click **Certificates** > **All Tasks** > **Import...**.
5. Select the exported `zap_root_ca.cer` file and complete the wizard.

---

### Step 4: Fallback — Manual Browser Proxy Configuration

If the automated launcher fails due to driver conflicts, route an isolated Chrome instance directly through ZAP's local proxy listener (`localhost:8080`):

1. Verify the proxy listener under **Tools** > **Options** > **Network** > **Local Servers/Proxies** (Default: `127.0.0.1:8080`).
2. Open the Windows **Run** dialog (`Win + R`) and run:
```cmd
"C:\Program Files\Google\Chrome\Application\chrome.exe" --proxy-server="http://127.0.0.1:8080" --user-data-dir="%TEMP%\zap-chrome"

```


3. Navigate to your target application in the newly opened browser window.
4. Verify traffic is actively capturing in ZAP's **Sites** tree and **History** tab.

---

## 4. Assessment Methodology & Scan Execution

### Manual Exploration

Navigate across the application while proxied:

* Click interactive elements, navigation menus, and search forms.
* Submit form data to map out endpoints and dynamic parameters in ZAP's site map.

### Active Scan Execution

1. In the **Sites** panel, right-click the target domain.
2. Select **Attack** > **Active Scan**.
3. Keep default scanner policy settings and click **Start Scan**.

---

## 5. Scan Overview & Alert Analysis

Once the Active Scan completes, findings are categorized by severity under the **Alerts** tab:

---

## 6. Key Findings

### 🟠 Medium Severity Findings

| Vulnerability | Count | Description / Evidence | Impact |
| --- | --- | --- | --- |
| **Absence of Anti-CSRF Tokens** | 2 | Missing anti-CSRF tokens on HTML forms (`/feedback.jsp` and `/login.jsp`). | Allows attackers to induce authenticated users into submitting unauthorized state-changing requests. |
| **Content Security Policy (CSP) Header Not Set** | Systemic | The server omits the `Content-Security-Policy` header across all responses. | Eliminates defense-in-depth protection against Cross-Site Scripting (XSS) and client-side data injection. |
| **Missing Anti-Clickjacking Header** | Systemic | Neither `X-Frame-Options` nor CSP `frame-ancestors` directive is configured. | Leaves the application vulnerable to UI redressing and clickjacking inside malicious `<iframe>` tags. |
| **Secure Pages Include Mixed Content (Scripts)** | 1 | An HTTPS page loads an active JavaScript file over unencrypted HTTP: `[http://demo-analytics.testfire.net/urchin.js](http://demo-analytics.testfire.net/urchin.js)`. | An active Man-in-the-Middle (MitM) attacker can alter script contents and hijack the user session. |
| **Sub Resource Integrity (SRI) Missing** | 1 | Externally hosted script elements lack `integrity` hashes. | If the third-party hosting server is compromised, malicious code will run within the application's origin. |

---

### 🟡 Low Severity Findings

| Vulnerability | Count | Description / Evidence | Impact |
| --- | --- | --- | --- |
| **Cross-Domain JavaScript Inclusion** | 1 | Script loaded from an external domain without origin validation. | Increased exposure to supply-chain tampering and unauthorized third-party tracking. |
| **HTTPS Content Available via HTTP** | 1 | Protected application resources are directly accessible over plaintext HTTP. | Sensitive communication can be intercepted by eavesdroppers on untrusted networks. |
| **Secure Pages Include Mixed Content (Passive)** | 1 | Non-executable media and object assets loaded over plaintext HTTP inside TLS sessions. | Leaks URI details and enables asset spoofing in transit. |
| **Server Leaks Version Information** | Systemic | The `Server` header returns banner version strings (`Apache-Coyote/1.1`). | Simplifies attacker reconnaissance and targeting of known CVEs against specific web container versions. |
| **Strict-Transport-Security (HSTS) Missing** | Systemic | Responses do not include the `Strict-Transport-Security` header. | Allows SSL stripping attacks where TLS connections are downgraded to insecure HTTP. |
| **X-Content-Type-Options Header Missing** | Systemic | Responses omit `X-Content-Type-Options: nosniff`. | Browsers may override declared MIME types via MIME-sniffing, leading to unexpected script execution. |

---

### 🔵 Informational Findings

| Category | Count | Notes / Analyst Interpretation |
| --- | --- | --- |
| **Information Disclosure - Suspicious Comments** | 1 | Developer comments inside `/login.jsp` expose internal administrator contact details and phone numbers. |
| **Modern Web Application** | 1 | Application uses non-traditional anchor structures and client-side routing routines. |
| **Re-examine Cache-Control Directives** | Systemic | Cache directives on dynamic pages are missing or permissive, risking local disk caching of authenticated sessions. |

---

## 7. Technical Interpretation

1. **Broken Request Integrity & Session Security:**
* The absence of unique, cryptographically random Anti-CSRF tokens on state-changing endpoints (`/login.jsp`, `/feedback.jsp`) permits cross-site request forgery attacks.


2. **Degraded Client-Side Hardening:**
* The complete absence of defensive response headers (`Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, and `Strict-Transport-Security`) significantly lowers the barrier for XSS exploitation, clickjacking, and connection downgrades.


3. **Transport Layer Vulnerabilities:**
* Loading executable JavaScript over plain HTTP on an HTTPS page breaks end-to-end TLS integrity, rendering the transport layer susceptible to code injection by an active network adversary.


4. **Reconnaissance Surface:**
* Server banner disclosure (`Apache-Coyote/1.1`) and HTML comments disclose implementation stack details, reducing the effort needed for targeted exploitation.



---

## 8. Remediation Guidance

### Immediate Fixes

* **Implement Anti-CSRF Protection:** Ensure all forms and state-altering POST/PUT requests require a cryptographically secure, unpredictable token tied to the user's session (e.g., using the Synchronizer Token Pattern or SameSite cookies).
* **Fix Mixed Content:** Update all script and asset URLs to use relative paths or strict `https://` schemas. Block mixed content using CSP directives:
```http
Content-Security-Policy: block-all-mixed-content; upgrade-insecure-requests;

```


* **Deploy Core Security Headers:**
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';

```


* **Enforce Subresource Integrity (SRI):** When referencing third-party scripts, specify cryptographic hash attributes:
```html
<script src="https://example.com/script.js" integrity="sha384-..." crossorigin="anonymous"></script>

```



### Architectural Improvements

* **Suppress Infrastructure Banners:** Configure web server and reverse proxy settings to strip or genericize the `Server` and `X-Powered-By` response headers.
* **Centralize Security Filters:** Enforce header insertion and CSRF validation at the API gateway or application framework middleware layer to prevent developer oversight on individual routes.
* **Strict HTTPS Redirection:** Configure web servers to unconditionally redirect all HTTP traffic to HTTPS (301 Permanent Redirect) and register the domain with the HSTS preload list.
