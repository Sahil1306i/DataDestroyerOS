# DataDestroyerOS
# 🔒 Wipe Drive ISO

**Wipe Drive ISO** is a lightweight, bootable operating system designed to **securely erase storage drives** while generating **verifiable logs** of the process.  
It automates the entire disk wiping workflow — from drive detection to secure deletion — using a custom **Bash script**, built on top of a minimal **Linux live environment**.

---

## 🧭 Overview

When sensitive data is deleted through normal means, it can often be recovered using simple recovery tools.  
This poses a serious risk for companies, refurbishers, and individuals who resell or dispose of old drives.

**Wipe Drive ISO** solves this problem by providing:
- A **bootable Linux-based system** that securely wipes drives.
- A **logging mechanism** that records every wipe action.
- A foundation to build **certificate generation and remote log tracking** in the future.

---

## 💡 Motivation

The project originated from the need for a **simple, open-source, and verifiable drive sanitization tool** that does not rely on expensive enterprise software.  

By combining Linux utilities like `shred`, `blkdiscard`, and `nvme-cli` with custom automation logic, the system ensures that:
- Each connected drive is securely overwritten.
- The wiping process is transparent and logged.
- Future versions can generate **certificates of deletion** for compliance.

---

## 🧠 System Architecture

                 |
                 ↓

---

## ⚙️ How It Works

### 1. **Base Operating System**

- The ISO is built using a minimal **Linux live system** (e.g., **SystemRescue** or **Debian Live**).  
- It includes core utilities:
  - `lsblk` — for listing connected storage devices
  - `shred` — for overwriting data securely
  - `blkdiscard` — for SSD/TRIM-based erasure
  - `nvme-cli` — for NVMe drive secure erase

When booted, the system runs entirely in memory and doesn’t touch your host OS or installed drives — ensuring a clean and controlled environment.

---

### 2. **Automation Script (wipe.sh)**

The heart of the project is a custom Bash script named **`wipe.sh`**, which runs automatically or manually after boot.

#### **Script Workflow:**

```bash
#!/bin/bash

LOG_DIR="/var/logs/wipe-logs"
mkdir -p $LOG_DIR

echo "🔍 Scanning for drives..."
drives=$(lsblk -dpno NAME | grep -E "/dev/sd|/dev/nvme")

for drive in $drives; do
    echo "Found: $drive"
    read -p "Do you want to wipe $drive? (y/n): " confirm
    if [ "$confirm" == "y" ]; then
        start_time=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
        echo "⚙️  Wiping $drive..."
        shred -v -n 3 $drive
        end_time=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

        echo "{
          \"device\": \"$drive\",
          \"method\": \"shred (3 passes)\",
          \"start_time\": \"$start_time\",
          \"end_time\": \"$end_time\",
          \"status\": \"Success\"
        }" >> $LOG_DIR/wipe-log.json

        echo "✅ Wipe complete for $drive"
    else
        echo "⏭️ Skipped $drive"
    fi
done
