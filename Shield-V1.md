# 🛡️ Lab: Shield-V1 Modular Security Suite

**Date:** 2026-05-02 **Environment:** Parrot OS (Guest) via KVM/QEMU on Pop!_OS (Host) **Objective:** Develop a portable, modular toolkit for automated log analysis and network integrity verification.

---

## 🏗️ Architecture Overview

The toolkit uses a **Modular Design Pattern**, separating specific detection logic into "Micro-Tools" that are orchestrated by a central wrapper script.

### 📁 Directory Structure

- **Host Path:** `/home/deepwarden/VM_Shares/` (Persistent)
    
- **Guest Mount:** `/mnt/host_share/` (Virtio-9p)
    
- **Execution Path:** `~/` (Local VM Home)
    

---

## 🛠️ The Modules

### 1. Brute Force Detection (`find-brute`)

Identifies high-frequency IP hits that may indicate automated scanning or credential stuffing.

Bash

```
#!/bin/bash
echo "--- [MODULE] BRUTE FORCE DETECTION ---"
awk '{print $3}' "$1" | sort | uniq -c | sort -nr | head -n 5
```

### 2. Error Code Analysis (`find-errors`)

Monitors for 403 (Forbidden) and 500 (Server Error) codes to detect unauthorized access attempts or exploit success.

Bash

```
#!/bin/bash
echo "--- [MODULE] 4xx/5xx ERROR ANALYSIS ---"
grep -E " 40[0-9] | 50[0-9] " "$1" | awk '{print $7, $3, $5}' | sort | uniq -c | sort -nr | head -n 5
```

### 3. Payload Exfiltration Check (`check-payload`)

Flags unusually large responses which could indicate data exfiltration.

Bash

```
#!/bin/bash
echo "--- [MODULE] HIGH-PAYLOAD DETECTION (EXFIL) ---"
awk '$6 > 400 {print "SIZE:", $6, "bytes | IP:", $3, " | PATH:", $5}' "$1" | sort -nr | head -n 5
```

### 4. ARP Integrity Check (`scan-arp-poison`)

Verifies Layer 2 stability by checking for MAC addresses claiming multiple IP addresses.

Bash

```
#!/bin/bash
echo "--- [MODULE] ARP CACHE ANALYSIS ---"
arp -n | awk '{print $3}' | sort | uniq -c | awk '$1 > 1 {print "SUSPICIOUS: MAC", $2, "is claiming multiple IPs!"}'
```

### 5. Route Integrity Check (`check-route`)

Ensures the Default Gateway hasn't been spoofed or redirected.

Bash

```
#!/bin/bash
EXPECTED_GW="192.168.122.1"
CURRENT_GW=$(ip route | grep default | awk '{print $3}')
echo "--- [MODULE] ROUTE INTEGRITY CHECK ---"
if [ "$CURRENT_GW" == "$EXPECTED_GW" ]; then
    echo "OK: Default Gateway is correct ($CURRENT_GW)."
else
    echo "⚠️ ALERT: ROUTE MANIPULATION DETECTED!"
fi
```

---

## 🔗 The Orchestrator (`daily_audit.sh`)

This wrapper script aggregates the findings into a single report.

Bash

```
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

bash ./find-brute "$LOG"
echo ""
bash ./find-errors "$LOG"
echo ""
bash ./check-payload "$LOG"
echo ""
bash ./scan-arp-poison
echo ""
bash ./check-route

echo "=========================================="
```

---

## 📖 Key Takeaways & Forensics Concepts

> [!TIP] The Dot (`.`) Operator In Linux, `cp [Source] .` uses the dot as a relative path representing the **current working directory**.

> [!IMPORTANT] East-West Security Internal threats (like `192.168.1.50`) often bypass perimeter firewalls. By monitoring **ARP tables** and **Internal Logs**, we gain visibility into traffic that never reaches the Default Gateway.

> [!ABSTRACT] Baseline Validation Security analysis is the study of **deviations**. "No news is good news" only when the tool has been validated against a known baseline.