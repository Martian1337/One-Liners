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

# Cron Jobs

## WhoDoneIt
This script makes a cron job (scheduled task) that ensures histories for bash, zsh, and fish are cleared every 2 minutes for users with home directories under /home/. Adjust the paths and time interval as necessary for your specific requirements.

```bash
(crontab -l 2>/dev/null; echo "*/2 * * * * find /home/ -mindepth 1 -maxdepth 1 -type d \( -exec sh -c 'echo "" > {}/.bash_history' \; -exec sh -c 'echo "" > {}/.zsh_history' \; -exec sh -c 'rm -f {}/.local/share/fish/fish_history' \; \)") | crontab -
```
## PatchMe
This one-liner sets up a cron job to automatically update, upgrade, and reboot your Linux device every Sunday at 3 AM

```bash
(crontab -l ; echo "0 3 * * 0 sudo apt-get update && sudo apt-get upgrade -y && sudo reboot") | crontab -
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
