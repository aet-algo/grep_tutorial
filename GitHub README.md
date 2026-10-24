# 🐧 Linux `grep` Master Tutorial: Beginner to Advanced

A comprehensive, real-world scenario guide to mastering `grep` (*Global Regular Expression Print*) in Linux and Unix environments—covering fundamental searches, regex patterns, multiline contexts, and production shell scripting.

---

## 📖 Table of Contents

- [Overview & Basic Syntax](#-overview--basic-syntax)
- [1. Beginner Scenarios: Core Searching](#1-beginner-scenarios-core-searching)
  - [Scenario 1: Basic Text Search in a File](#scenario-1-basic-text-search-in-a-file)
  - [Scenario 2: Case-Insensitive Search (`-i`)](#scenario-2-case-insensitive-search--i)
  - [Scenario 3: Inverting Matches to Exclude Lines (`-v`)](#scenario-3-inverting-matches-to-exclude-lines--v)
  - [Scenario 4: Matching Exact Whole Words Only (`-w`)](#scenario-4-matching-exact-whole-words-only--w)
- [2. Intermediate Scenarios: Context, Counting & Traversal](#2-intermediate-scenarios-context-counting--traversal)
  - [Scenario 5: Line Numbers in Results (`-n`)](#scenario-5-line-numbers-in-results--n)
  - [Scenario 6: Counting Matching Lines (`-c`)](#scenario-6-counting-matching-lines--c)
  - [Scenario 7: Viewing Surrounding Context (`-A`, `-B`, `-C`)](#scenario-7-viewing-surrounding-context--a--b--c)
  - [Scenario 8: Recursive Directory Traversal (`-r`, `-R`)](#scenario-8-recursive-directory-traversal--r--r)
  - [Scenario 9: Listing Filenames Only (`-l`, `-L`)](#scenario-9-listing-filenames-only--l--l)
  - [Scenario 10: Filtering Piped Command Output](#scenario-10-filtering-piped-command-output)
- [3. Advanced Scenarios: Regular Expressions & Pattern Extraction](#3-advanced-scenarios-regular-expressions--pattern-extraction)
  - [Scenario 11: Extended Regular Expressions (`-E`)](#scenario-11-extended-regular-expressions--e)
  - [Scenario 12: Extracting Matched Patterns Only (`-o`)](#scenario-12-extracting-matched-patterns-only--o)
  - [Scenario 13: Perl-Compatible Regex (`-P`)](#scenario-13-perl-compatible-regex--p)
  - [Scenario 14: Fast Fixed-String Matching (`-F`)](#scenario-14-fast-fixed-string-matching--f)
- [4. Production & Automation Scenarios](#4-production--automation-scenarios)
  - [Scenario 15: Excluding Specific Files and Directories](#scenario-15-excluding-specific-files-and-directories)
  - [Scenario 16: Silent Condition Checks in Scripts (`-q`)](#scenario-16-silent-condition-checks-in-scripts--q)
  - [Scenario 17: Matching Against a Pattern File (`-f`)](#scenario-17-matching-against-a-pattern-file--f)
  - [Scenario 18: High-Performance Search Acceleration](#scenario-18-high-performance-search-acceleration)
- [⚡ Quick Reference Cheat Sheet](#-quick-reference-cheat-sheet)

---

## 🔍 Overview & Basic Syntax

The `grep` command searches named input files for lines containing a match to a specified pattern.

```bash
grep [OPTIONS] "PATTERN" [FILE...]
```

> [!TIP]
> Always wrap your search patterns in double quotes (`"..."`) or single quotes (`'...'`) to prevent the shell from interpreting special characters or expanding wildcards before passing them to `grep`.

---

## 1. Beginner Scenarios: Core Searching

### Scenario 1: Basic Text Search in a File
Search for every line containing the keyword `"failed"` inside an authentication log.

```bash
grep "failed" /var/log/auth.log
```

---

### Scenario 2: Case-Insensitive Search (`-i`)
Match terms regardless of capitalization (`error`, `Error`, `ERROR`, etc.).

```bash
grep -i "error" /var/log/syslog
```

---

### Scenario 3: Inverting Matches to Exclude Lines (`-v`)
Strip out comments (lines starting with `#`) to inspect a configuration file cleanly.

```bash
grep -v "^#" /etc/ssh/sshd_config
```

Combine `-v` filters to strip both comments and empty lines:

```bash
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"
```

---

### Scenario 4: Matching Exact Whole Words Only (`-w`)
Avoid matching substrings. For example, match the standalone word `"is"` without matching `"this"`, `"island"`, or `"issue"`.

```bash
grep -w "is" notes.txt
```

---

## 2. Intermediate Scenarios: Context, Counting & Traversal

### Scenario 5: Line Numbers in Results (`-n`)
Display the exact line numbers where a match occurs.

```bash
grep -n "Listen" /etc/apache2/ports.conf
```

**Sample Output:**
```text
8:Listen 80
17:Listen 443
```

---

### Scenario 6: Counting Matching Lines (`-c`)
Count how many occurrences exist instead of listing the lines themselves.

```bash
grep -c " 404 " /var/log/nginx/access.log
```

---

### Scenario 7: Viewing Surrounding Context (`-A`, `-B`, `-C`)
When debugging crashes or tracing logs, you often need the events that happened immediately before or after an error.

| Flag | Meaning | Description |
|:---:|:---|:---|
| `-B <N>` | **Before** | Print $N$ lines prior to the match |
| `-A <N>` | **After** | Print $N$ lines following the match |
| `-C <N>` | **Context** | Print $N$ lines before and after the match |

```bash
# Print 2 lines before and 4 lines after each NullPointerException
grep -B 2 -A 4 "NullPointerException" application.log

# Print 3 lines of surrounding context on both sides
grep -C 3 "FATAL" system.log
```

---

### Scenario 8: Recursive Directory Traversal (`-r`, `-R`)
Recursively scan all subdirectories for a matching pattern.

```bash
# Standard recursive traversal
grep -r "DATABASE_URL" /var/www/app/

# -R traverses symbolic links to directories as well
grep -R "DATABASE_URL" /var/www/app/
```

---

### Scenario 9: Listing Filenames Only (`-l`, `-L`)
Find which files contain a pattern without cluttering the screen with matching content.

```bash
# -l: List only filenames CONTAINING the match
grep -rl "deprecated_function" ./src/

# -L: List only filenames that DO NOT contain the match
grep -rL "StrictHostKeyChecking" ~/.ssh/
```

---

### Scenario 10: Filtering Piped Command Output
Filter system process tables, hardware logs, or live event streams.

```bash
# Check if nginx processes are running (bracket trick prevents matching the grep process itself)
ps aux | grep "[n]ginx"

# Inspect USB device messages from the kernel ring buffer
dmesg | grep -i "usb"
```

> [!NOTE]
> Wrapping the first character in brackets (`[n]ginx`) makes the search pattern evaluate to `nginx` without matching the literal string `grep [n]ginx` in the process list.

---

## 3. Advanced Scenarios: Regular Expressions & Pattern Extraction

### Scenario 11: Extended Regular Expressions (`-E`)
Use modern regex operators (`|`, `+`, `?`, `{}`) without backslash escaping.

```bash
# Match HTTP 500, 502, or 503 response codes
grep -E "HTTP/1\.[01]\" (500|502|503)" /var/log/nginx/access.log

# Match ISO dates formatted as YYYY-MM-DD
grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}" audit.log
```

---

### Scenario 12: Extracting Matched Patterns Only (`-o`)
Instead of printing entire lines, isolate and print only the exact matching token.

```bash
# Extract all IPv4 addresses from a connection log
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" /var/log/ufw.log
```

---

### Scenario 13: Perl-Compatible Regex (`-P`)
Harness lookarounds, non-greedy matching, and PCRE character classes.

```bash
# Extract values located between HTML <title> tags
grep -Po "(?<=<title>).*?(?=</title>)" index.html

# Extract a JSON string value by matching after the key
grep -Po '(?<="auth_token": ")[^"]*' response.json
```

---

### Scenario 14: Fast Fixed-String Matching (`-F`)
Treat all pattern characters as literal characters rather than regex operators. This prevents syntax errors when searching for symbols like `$`, `*`, `[`, or `.`.

```bash
# Search literally for $config['db.user'] without regex parsing overhead
grep -F "$config['db.user']" config.php
```

---

## 4. Production & Automation Scenarios

### Scenario 15: Excluding Specific Files and Directories
Speed up searches and ignore build artifacts, logs, and version control directories.

```bash
grep -rn "API_SECRET" /var/www/project/ \
  --exclude="*.min.js" \
  --exclude="*.log" \
  --exclude-dir={.git,node_modules,dist,vendor}
```

---

### Scenario 16: Silent Condition Checks in Scripts (`-q`)
Suppress all standard output and standard error. Evaluate execution via the exit status (`0` = match found, `1` = no match).

```bash
#!/usr/bin/env bash

TARGET_USER="deploy"

if grep -q "^${TARGET_USER}:" /etc/passwd; then
    echo "User '${TARGET_USER}' exists."
else
    echo "User '${TARGET_USER}' does not exist."
    exit 1
fi
```

---

### Scenario 17: Matching Against a Pattern File (`-f`)
Search for any match from an external file of patterns (e.g., a blocklist).

```bash
# Search firewall log using a list of malicious IP addresses (one pattern per line)
grep -F -f blocked_ips.txt /var/log/ufw.log
```

---

### Scenario 18: High-Performance Search Acceleration
When searching gigabyte-scale text files, locale parsing (UTF-8 multi-byte decoding) significantly degrades throughput. For ASCII-safe data, override the locale to `C`:

```bash
# Up to 5x-10x speedup on massive log files
LC_ALL=C grep "TRANSACTION_FAILURE" massive_ledger.log
```

---

## ⚡ Quick Reference Cheat Sheet

| Flag | Meaning | Use Case |
|---|---|---|
| `-i` | Ignore case | Case-insensitive matching |
| `-v` | Invert match | Exclude matching lines (e.g., strip comments) |
| `-w` | Whole word | Match exact words rather than substrings |
| `-n` | Line number | Display file line numbers alongside matches |
| `-c` | Count | Count total matching lines |
| `-r` / `-R` | Recursive | Traverse folders recursively (`-R` follows symlinks) |
| `-l` | List files (matched) | Print only names of files with matches |
| `-L` | List files (unmatched) | Print only names of files without matches |
| `-o` | Only matching | Print only the matching substring, not the whole line |
| `-E` | Extended regex | Enable standard modern regex (`\|`, `+`, `?`, `{}`) |
| `-P` | Perl regex | Enable PCRE syntax (lookaheads, lookbehinds, non-greedy) |
| `-F` | Fixed strings | Disable regex; treat search string as literal characters |
| `-q` | Quiet / Silent | Suppress output; check return code in bash scripts |
| `-A <N>` | After context | Print $N$ lines after match |
| `-B <N>` | Before context | Print $N$ lines before match |
| `-C <N>` | Surrounding context | Print $N$ lines before and after match |

---

## 📄 License
This guide is open-source and released under the [MIT License](LICENSE).