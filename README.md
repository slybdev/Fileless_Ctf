
# 🧪 Supply.exe Fileless Malware Investigation

This write-up details a log analysis conducted using Splunk on a Blue Team Labs Online (BTLO) scenario. The investigation uncovered a **fileless malware attack** leveraging **LOLBINs**, **environment variable hijacking**, and **Gzip-compressed payloads**.

---

## 🛠 Tools Used

- **Splunk** (GUI for visual pivoting)
- **Windows Event Logs**
- **Blue Team Labs Online (BTLO)**
- EventIDs: `1`, `3`, `11`

---

## 🔍 Initial Exploration

Loaded the log data into Splunk and visualized Windows EventIDs to identify suspicious activity patterns.

```spl
| timechart count by Event.System.EventID
````

Key Events:

* `EventID=1` → Process Creation
* `EventID=3` → Network Connections
* `EventID=11` → File Creation

---

## ⚠️ Malicious Behavior Observed

### 1. LOLBIN Abuse

* **`mshta.exe`** executed a suspicious `updater.hta` file
* **`powershell.exe`** was used with flags:

  ```powershell
  -nop -w hidden -EncodedCommand
  ```
* Payload was **Base64 + Gzip** encoded
* Decompressed & executed directly in memory
  → No files dropped: **Fileless malware**

---

### 2. Environment Variable Hijacks

#### 🧠 `COMSPEC` Hijack

```powershell
COMSPEC = supply.exe
```

* Overwrote default `cmd.exe` behavior
* Any script or process calling `cmd` ran the attacker's binary (`supply.exe`) instead

#### 🛡️ `__COMPAT_LAYER` Trick

```powershell
set __COMPAT_LAYER=RunAsInvoker
```

* Used to **bypass UAC prompts** silently
* Attacker escalated privileges without triggering alert dialogs

---

## 🌐 Network Activity

Outbound communication observed from:

```
Source: 192.168.1.8 ➝ Destination: 192.168.1.11  
Port: 50598 ➝ 8080
```

Likely C2 (Command & Control) traffic initiated by `supply.exe`.

---

## 📁 Persistence and Stealth

* No obvious initial `.exe` dropped
* Execution occurred fully in memory
* LOLBINs and env vars enabled stealthy persistence
* Attack chain avoided AV detection vectors

---

## 📌 Key Takeaways

* **Fileless attacks** are difficult to detect without proper log monitoring
* Abusing **LOLBINs** (mshta, powershell) remains common in real-world malware
* **Environment variables** like `COMSPEC` and `__COMPAT_LAYER` can be weaponised for persistence and UAC bypass
* Splunk was useful for fast visualization and pivoting in this investigation

---

## 🧠 Lessons Learned

* Don't overlook environment variables in process command lines
* Fileless malware demands **proactive log analysis**, not just reactive detection
* Combine EventID filtering, network log inspection, and memory-based behavior to uncover hidden attacks

---
## Screenshots 

![Screenshot 2025-06-10 191430](https://github.com/user-attachments/assets/7e204f7c-5803-4086-ac76-308863633861)
![Screenshot 2025-06-10 194515](https://github.com/user-attachments/assets/79e55b5f-e230-4f42-bfe8-8168f07fc68a)


![Screenshot 2025-06-10 191856](https://github.com/user-attachments/assets/039869c7-8b5a-4c2a-9821-e972d45fcdf6)

![Screenshot 2025-06-10 193050](https://github.com/user-attachments/assets/56168460-7db1-48f8-80e2-e588040287a1)


![Screenshot 2025-06-10 194108](https://github.com/user-attachments/assets/600445cb-7501-41c7-aada-2c78f9b0b8ee)


![Screenshot 2025-06-10 200615](https://github.com/user-attachments/assets/8645a7e3-fb4c-40df-8a58-98adf61ee93c)
![Screenshot 2025-06-10 205914](https://github.com/user-attachments/assets/e60bb6dc-24ff-4f8e-ae2e-94d8ac49d84e)



---
## 🏷️ Tags

`#FilelessMalware` `#Splunk` `#LogAnalysis` `#BlueTeam` `#LOLBINs` `#DFIR` `#Persistence` `#WindowsInternals` `#SOC`

---


