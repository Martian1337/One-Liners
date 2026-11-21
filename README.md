# Custom One-Liners for Security Research

## Martian's Nuclei Rover
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

### Script explanation:

1. **`for(i=1;i<=NF;i++)`**: This `for` loop traverses every field in each line. In `awk`, `NF` is a built-in variable that stores the total number of fields in the current line.
2. **`if ($i ~ /^https?:\/\//)`**: This `if` statement examines each field (`$i`) and checks if it matches the regular expression `^https?:\/\/`. This expression matches any field starting with 'http://' or 'https://'.
3. **`{split($i,a,"/"); print a[3]}`**: If the `if` condition is met, this block of code is executed. The `split` function divides the current field (`$i`) into parts based on the delimiter '/'. These parts are stored in the array `a`. Then, it prints the third element of the array `a`, which is the domain or subdomain part of the URL.
4. **`else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)`**: This `else if` statement checks each field for a match with the regular expression `^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`. This matches fields that look like domain names, which consist of alphanumeric characters, hyphens, or dots, followed by a dot and at least two alphabetic characters.
5. **`{print $i}`**: If the `else if` condition is met, this block of code is executed. It directly prints the field without any changes.

## XSSPipeline

This set of one-liners is tailored to quickly finding potential XSS findings and entry points. Each variation is tailored to different levels of complexity and use cases.

### **1. Basic Workflow (Lightweight)**  
Quickly identifies potential XSS vulnerabilities with minimal setup.  
```bash
echo https://example.com/ | gau | gf xss | uro | Gxss | kxss | tee xss_output.txt
```

### **2. Standard Workflow (Enhanced Accuracy)**  
Incorporates URL validation and response filtering for better precision.  
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent -mc 200 | Gxss | kxss | tee xss_output.txt
```

### **3. Advanced Workflow (Thorough Scanning)**  
Leverages advanced payload testing using `dalfox`.  
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent | dalfox pipe -b collaborator-url | tee xss_output.txt
```

### **4. Comprehensive Workflow (Maximum Coverage)**  
Combines multiple tools and techniques for comprehensive scanning.  
```bash
echo https://example.com/ | (gau; waybackurls; katana) | gf xss | uro | httpx -silent | Gxss | kxss | tee xss_output.txt
```

### **5. Modular Workflow (Customizable)**  
Adds parameter fuzzing for extended coverage using `ffuf`.  
```bash
echo https://example.com/ | gau | gf xss | uro | httpx -silent | ffuf -u FUZZ -w parameters.txt -mc 200 | Gxss | tee xss_output.txt
```
---


# OPSEC Cron Jobs

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

#### Customizing Cron Schedules

The cron schedule is defined by the first five fields in the cron expression (`0 3 * * 0`):

- **Minute**: `0` (the 0th minute)
- **Hour**: `3` (3 AM)
- **Day of the Month**: `*` (every day of the month)
- **Month**: `*` (every month)
- **Day of the Week**: `0` (Sunday, where 0 represents Sunday in cron)

#### Explanation of Cron Fields

- **Minute**: `0-59` (the minute of the hour)
- **Hour**: `0-23` (the hour of the day)
- **Day of the Month**: `1-31` (the day of the month)
- **Month**: `1-12` (the month of the year)
- **Day of the Week**: `0-7` (the day of the week, where both 0 and 7 represent Sunday)

---

# Wazuh
## Rotated Logs Cleanup

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

---

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

---

#### Example Cron Entries

**Time-based (30 days, daily at 01:00)**

```cron
0 1 * * * find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type f -mtime +30 -delete && find /var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall} -type d -empty -delete
```

**Size-based (50 GB cap, daily at 02:00)**

```cron
0 2 * * * MAX=50; DIRS="/var/ossec/logs/{wazuh,alerts,archives,api,cluster,firewall}"; while [ "$(du -sBG $DIRS 2>/dev/null | awk '{s+=$1} END{print s+0}')" -gt "$MAX" ]; do find $DIRS -type f -printf '%T@ %p\n' | sort -n | head -1 | awk '{print $2}' | xargs -r rm -f; done
```

---

# Recon and Subdomain Enumeration

A set of one-liners for mapping attack surface before anything active happens.

### CertScrape
Pulls subdomains straight from certificate transparency logs, no API key needed.

```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

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

---

# URL and Content Discovery

Collect, dedupe, and filter URLs so you spend time on live, interesting endpoints.

### UrlHarvest
Merges passive sources, strips duplicates, and keeps only live 200s.

```bash
echo example.com | (gau; waybackurls; katana -silent) 2>/dev/null | uro | httpx -silent -mc 200 | sort -u > live_urls.txt
```

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

---

# Vulnerability Hunting

Active tests. Only run these against targets you are explicitly authorized to test. Replace `example.com`, `evil.com`, and `YOUR-COLLABORATOR` with your own values.

### RedirectCheck
Tests gf-tagged redirect parameters for open redirect via the Location header.

```bash
cat live_urls.txt | gf redirect | qsreplace 'https://evil.com' | httpx -silent -location -mc 301,302 | grep -i 'evil.com'
```

### LFIProbe
Swaps parameters for `/etc/passwd` and matches a real root entry in the response body.

```bash
cat live_urls.txt | gf lfi | qsreplace '/etc/passwd' | httpx -silent -mr 'root:.*:0:0:'
```

### CORSAudit
Reflects an arbitrary Origin and flags hosts that trust it.

```bash
while read u; do curl -s -I -H 'Origin: https://evil.com' "$u" | grep -qi 'access-control-allow-origin: https://evil.com' && echo "CORS: $u"; done < live_urls.txt
```

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

---

# JavaScript and Secret Analysis

Mine client-side code for keys, tokens, and hidden endpoints.

### JSHunt
Crawls live JS files and greps for likely keys, tokens, and credentials.

```bash
cat live_urls.txt | grep -iE '\.js(\?|$)' | httpx -silent | while read u; do curl -s "$u" | grep -oiE '(api[_-]?key|secret|token|password|authorization|aws_access_key_id|bearer)["'\'' :=]+[A-Za-z0-9._\-]{8,}'; done | sort -u
```

### EndpointPull
Extracts relative and absolute paths referenced inside a JS file for a deeper map.

```bash
curl -s https://example.com/app.js | grep -oiE '["'\''`](/[a-zA-Z0-9_?&=/\-\.]+)["'\''`]' | tr -d '"'\''`' | sort -u
```

---

# Fingerprinting and Hardening Checks

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

---

# General Security Research and Local Triage

Defensive and local checks that pair with the OPSEC and Wazuh sections above.

### JWTPeek
Decodes a JWT payload without verifying it (base64url-safe).

```bash
echo "$JWT" | awk -F. '{print $2}' | tr '_-' '/+' | base64 -d 2>/dev/null | jq
```

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

### KeyLeak
Sweeps a directory tree for private keys and credential material before shipping.

```bash
grep -rlE 'BEGIN (RSA|EC|OPENSSH|DSA|PGP) PRIVATE KEY' . 2>/dev/null
```

---
