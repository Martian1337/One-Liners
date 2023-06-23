# Custom One-Liners for Security Research

## Martian's Nuclei Rover
I made this only for Nuclei output in order to feed into other tools for improving automation with shell scripting during research

OPTION 1: Run script and direct it to the nuclei file within the system
```bash
awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\//) {split($i,a,"/"); print a[3]} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {print $i}}' nucleifile.txt > output.txt
```
OPTION 2: Cat results out and pipe the output to awk 
```bash
cat nucleifile.txt | awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\//) {split($i,a,"/"); print a[3]} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {print $i}}' > output.txt
```
- Replace `nucleifile.txt` with your actual filename.

### Script explanation:

1. **`for(i=1;i<=NF;i++)`**: This `for` loop traverses every field in each line. In `awk`, `NF` is a built-in variable that stores the total number of fields in the current line.
2. **`if ($i ~ /^https?:\/\//)`**: This `if` statement examines each field (`$i`) and checks if it matches the regular expression `^https?:\/\/`. This expression matches any field starting with 'http://' or 'https://'.
3. **`{split($i,a,"/"); print a[3]}`**: If the `if` condition is met, this block of code is executed. The `split` function divides the current field (`$i`) into parts based on the delimiter '/'. These parts are stored in the array `a`. Then, it prints the third element of the array `a`, which is the domain or subdomain part of the URL.
4. **`else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)`**: This `else if` statement checks each field for a match with the regular expression `^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`. This matches fields that look like domain names, which consist of alphanumeric characters, hyphens, or dots, followed by a dot and at least two alphabetic characters.
5. **`{print $i}`**: If the `else if` condition is met, this block of code is executed. It directly prints the field without any changes.


## Nuclei Endpoint Extractor
This script is for retrieving MOST of the full URLs from Nuclei results so that the description field are removed for follow-on parsing of the URLs with endpoints included

```bash
awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\/[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/) {print $i} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(\/[a-zA-Z0-9._-]+)*(\/?)$/) {endpoint = substr($0, index($0,$i)); print "https://" endpoint}}' nucleifile.txt > output.txt
```
- Replace `nucleifile.txt` with your actual filename.

### Script explanation:

1. **`for(i=1;i<=NF;i++)`**: This `for` loop iterates through each field in a line. `NF` is an `awk` built-in variable that holds the total number of fields in the current line.
2. **`if ($i ~ /^https?:\/\/[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/)`**: This `if` statement checks if a field (`$i`) matches the regular expression `^https?:\/\/[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`, which identifies URLs starting with 'http://' or 'https://' and having a valid domain/subdomain pattern.
3. **`print $i`**: If the `if` condition is satisfied, this statement prints the field as it is, representing a complete URL.
4. **`else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(\/[a-zA-Z0-9._-]+)*(\/?)$/)`**: This `else if` statement checks if a field matches the regular expression `^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(\/[a-zA-Z0-9._-]+)*(\/?)$`, which identifies domain/subdomain patterns that can have optional paths or endpoint URLs.
5. **`endpoint = substr($0, index($0,$i))`**: If the `else if` condition is met, this line extracts the endpoint part from the original line (`$0`) based on the position where the domain/subdomain starts. It assigns the extracted endpoint to the variable `endpoint`.
6. **`print "https://" endpoint`**: This line prepends 'https://' to the extracted endpoint and prints the complete URL.
