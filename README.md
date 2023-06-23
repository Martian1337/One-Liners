# Custom One-Liners for Security Research

## Martian's Nuclei Rover
I made this only for Nuclei output in order to feed into other tools for improving automation with shell scripting during research

```awk '{for(i=1;i<=NF;i++) if ($i ~ /^https?:\/\//) {split($i,a,"/"); print a[3]} else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {print $i}}' nucleifile.txt > output.txt```
- Replace `nucleifile.txt` with your actual filename.

Script explanation:

1. **`for(i=1;i<=NF;i++)`**: This `for` loop traverses every field in each line. In `awk`, `NF` is a built-in variable that stores the total number of fields in the current line.
2. **`if ($i ~ /^https?:\/\//)`**: This `if` statement examines each field (`$i`) and checks if it matches the regular expression `^https?:\/\/`. This expression matches any field starting with 'http://' or 'https://'.
3. **`{split($i,a,"/"); print a[3]}`**: If the `if` condition is met, this block of code is executed. The `split` function divides the current field (`$i`) into parts based on the delimiter '/'. These parts are stored in the array `a`. Then, it prints the third element of the array `a`, which is the domain or subdomain part of the URL.
4. **`else if ($i ~ /^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)`**: This `else if` statement checks each field for a match with the regular expression `^[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`. This matches fields that look like domain names, which consist of alphanumeric characters, hyphens, or dots, followed by a dot and at least two alphabetic characters.
5. **`{print $i}`**: If the `else if` condition is met, this block of code is executed. It directly prints the field without any changes.
