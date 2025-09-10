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


# Cron Jobs

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
