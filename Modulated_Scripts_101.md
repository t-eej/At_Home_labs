## 🏗️ Phase 1: Creating the Micro-Modules

Run these commands on your **Host (Pop!_OS)** or create these files in your `VM_Shares` folder. Each script is designed to be a "silent operator" that only outputs data when called.

### 1. `find-brute`

Focuses on volume.

Bash

```
cat << 'EOF' > ~/VM_Shares/find-brute
#!/bin/bash
echo "--- [MODULE] BRUTE FORCE DETECTION ---"
awk '{print $3}' "$1" | sort | uniq -c | sort -nr | head -n 5
EOF
```

### 2. `find-errors`

Focuses on server and client-side failures.

Bash

```
cat << 'EOF' > ~/VM_Shares/find-errors
#!/bin/bash
echo "--- [MODULE] 4xx/5xx ERROR ANALYSIS ---"
grep -E " 40[0-9] | 50[0-9] " "$1" | awk '{print $7, $3, $5}' | sort | uniq -c | sort -nr | head -n 5
EOF
```

### 3. `check-payload`

Focuses on potential data exfiltration (large responses).

Bash

```
cat << 'EOF' > ~/VM_Shares/check-payload
#!/bin/bash
echo "--- [MODULE] HIGH-PAYLOAD DETECTION (EXFIL) ---"
awk '$6 > 400 {print "SIZE:", $6, "bytes | IP:", $3, " | PATH:", $5}' "$1" | sort -nr | head -n 5
EOF
```

---

## 🔗 Phase 2: The Wrapper Script (`daily_audit.sh`)

This script acts as the "Manager." It ensures the modules exist and runs them in sequence.

Bash

```
cat << 'EOF' > ~/VM_Shares/daily_audit.sh
#!/bin/bash
LOG=$1

if [ -z "$LOG" ]; then
    echo "Usage: ./daily_audit.sh <logfile>"
    exit 1
fi

echo "=========================================="
echo "   SHIELD-V1 LOG AUDIT REPORT             "
echo "   Date: $(date)"
echo "=========================================="
echo ""

# Running the modules locally
bash ./find-brute "$LOG"
echo ""
bash ./find-errors "$LOG"
echo ""
bash ./check-payload "$LOG"

echo ""
echo "=========================================="
echo "   AUDIT COMPLETE"
echo "=========================================="
EOF
```

---

## 🧪 Phase 3: Running the Lab in Parrot

Now, jump into your **Parrot VM** terminal and follow these steps to deploy and test.

1. **Sync from Share:**
    
    Bash
    
    ```
    cp /mnt/host_share/find-brute /mnt/host_share/find-errors /mnt/host_share/check-payload /mnt/host_share/daily_audit.sh ~/
    chmod +x ~/find-brute ~/find-errors ~/check-payload ~/daily_audit.sh
    ```
    
2. **Execute the Suite:**
    
    Bash
    
    ```
    ./daily_audit.sh lab_access.log
    ```
    

---

## 📝 Obsidian Lab Report Entry

Copy this into your vault to document the logic behind this setup:

> ### 🛠️ Lab: Modular Bash Analysis Suite
> 
> **Concept:** Decoupling detection logic from reporting logic.
> 
> **Why this matters:**
> 
> - **Scalability:** If I want to add a `sql-injection-check` module later, I just write the script and add one line to `daily_audit.sh`.
>     
> - **Portability:** The `VM_Shares` setup allows these tools to be "OS Agnostic"—I can run these on Kali, Parrot, or Ubuntu without reconfiguration.
>     
> - **Troubleshooting:** Each module can be tested individually. If `check-payload` gives a syntax error, I know exactly where the bug is.
>