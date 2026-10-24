grep (Global Regular Expression Print) searches plain text data sets for lines that match a regular expression or plain-text pattern.
Basic Syntax
grep [OPTIONS] "PATTERN" [FILE...]

1. Beginner Scenarios: Core Searching
Scenario 1: Basic text search in a single file
Find every line containing the word "failed" inside an authentication log.
grep "failed" auth.log

Scenario 2: Case-insensitive search (-i)
Match "error", "Error", "ERROR", or any casing variation.
grep -i "error" application.log

Scenario 3: Invert match to exclude lines (-v)
Display all configuration lines except comment lines (lines starting with #).
grep -v "^#" /etc/ssh/sshd_config

> Combine -v with empty line removal to inspect clean configuration files:
> grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"
> 
> 
Scenario 4: Match exact whole words only (-w)
Find the word "is" without matching "this", "island", or "issue".
grep -w "is" document.txt

2. Intermediate Scenarios: Context, Counting, and Files
Scenario 5: Print line numbers with results (-n)
Locate where a specific configuration directive is defined in a file.
grep -n "Listen" /etc/apache2/ports.conf

Output format: LineNumber:LineContent (e.g., 8:Listen 80).
Scenario 6: Count occurrences instead of displaying lines (-c)
Count how many times a 404 status appears in an access log.
grep -c " 404 " access.log

Scenario 7: View surrounding context around a match (-B, -A, -C)
Debugging crashes often requires seeing what happened right before or after an error.
 * -B <num> (Before): Show lines preceding the match.
 * -A <num> (After): Show lines following the match.
 * -C <num> (Context): Show lines both before and after.
# Show 2 lines before and 4 lines after each "NullPointerException"
grep -B 2 -A 4 "NullPointerException" server.log

# Show 3 lines of surrounding context on both sides
grep -C 3 "CRITICAL" system.log

Scenario 8: Search recursively across directories (-r and -R)
Search for a database credential across an entire project directory.
# -r follows symbolic links; -R does not traverse symlinked directories
grep -r "DB_PASSWORD" /var/www/myproject/

Scenario 9: List only filenames with or without matches (-l and -L)
Identify which configuration files contain a deprecated directive without printing the matching lines.
# -l: List files containing the match
grep -rl "memory_limit" /etc/php/

# -L: List files that DO NOT contain the match
grep -rL "StrictHostKeyChecking" ~/.ssh/

Scenario 10: Piping command output into grep
Filter active processes or inspect system kernel messages in real time.
# Check if nginx is running
ps aux | grep "[n]ginx"

# Inspect USB device events in kernel ring buffer
dmesg | grep -i "usb"

> Tip: Enclosing the first letter of a process name in brackets ([n]ginx) prevents grep from matching its own search process in the ps aux table.
> 
3. Advanced Scenarios: Regular Expressions & Pattern Extraction
Scenario 11: Extended Regular Expressions (-E / egrep)
Use modern regex operators such as alternation (|), one-or-more (+), and grouping (()) without escaping them.
# Match any lines containing "500", "502", or "503" status codes
grep -E "HTTP/1\.[01]\" (500|502|503)" access.log

# Match dates formatted as YYYY-MM-DD
grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}" audit.log

Scenario 12: Extract only the matched portion (-o)
Instead of printing the entire line, extract only the substring that matches the pattern.
# Extract all IPv4 addresses from a log file
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log

Scenario 13: Perl-Compatible Regular Expressions (-P)
Leverage PCRE features like lookaheads, lookbehinds, non-greedy quantifiers, and \d shorthand.
# Extract the value inside <title> tags using non-greedy matching
grep -Po "(?<=<title>).*?(?=</title>)" index.html

# Extract JSON field values (e.g., "token": "xyz123")
grep -Po '(?<="token": ")[^"]*' response.json

Scenario 14: Fast fixed-string search (-F / fgrep)
Treat all characters literally (no regex interpretation). This skips regex parsing overhead and prevents characters like ., *, [, or $ from behaving as wildcards.
# Search for exact literal string "$config['db.host']"
grep -F "$config['db.host']" app.php

4. Production & Scripting Scenarios
Scenario 15: Excluding files and directories in recursive searches
Avoid searching compiled binaries, .git histories, or vendor folders.
grep -rn "API_KEY" /var/www/app/ \
  --exclude="*.min.js" \
  --exclude="*.log" \
  --exclude-dir={.git,node_modules,vendor}

Scenario 16: Silent condition checks in bash scripts (-q)
Run grep without printing to standard output or standard error. Check the exit status code (0 for match found, 1 for no match).
#!/usr/bin/env bash

TARGET_USER="deploy"

if grep -q "^${TARGET_USER}:" /etc/passwd; then
    echo "User ${TARGET_USER} exists."
else
    echo "User ${TARGET_USER} does not exist."
fi

Scenario 17: Multiple search patterns from a file (-f)
Compare a list of suspect IP addresses against firewall logs.
# blocked_ips.txt contains one IP pattern per line
grep -F -f blocked_ips.txt /var/log/ufw.log

Quick Reference Summary
| Flag | Purpose | Common Use Case |
|---|---|---|
| -i | Ignore case | Matching error, ERROR, Error |
| -v | Invert match | Stripping comments or blank lines |
| -w | Exact word match | Matching variable name id without matching width |
| -n | Show line numbers | Finding exact file location of code/config |
| -c | Count matching lines | Summarizing error occurrences |
| -r / -R | Recursive directory search | Searching whole codebases or /etc directories |
| -l | List matching filenames | Identifying impacted files |
| -L | List non-matching filenames | Finding servers missing a setting |
| -o | Print matching segment only | Extracting IPs, URLs, or IDs |
| -E | Extended regex | Using ` |
| -P | PCRE regex | Lookaheads, lookbehinds, non-greedy parsing |
| -F | Fixed literal strings | High-speed searching without escaping regex symbols |
| -q | Quiet/silent mode | Exit status evaluations inside if statements |
| -C N | Context (N lines around) | Crash trace analysis |

