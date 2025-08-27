# Migrating CrowdStrike Endpoints with No MDM

Recently, I had to migrate endpoints from a subsidiary’s CrowdStrike tenant to the parent company’s tenant, both with different CIDs.

### The Challenge

The subsidiary had no MDM solution (like Intune or SCCM), so pushing scripts was not an option. The only agent installed was the **CrowdStrike Falcon** sensor itself.

### The Issue I Faced

Migration requires uninstalling the old sensor and installing the new one. Running the official `falcon_windows_migrate.ps1` script directly from a **Real-Time Response (RTR)** session would fail because as soon as the old sensor uninstalled, the RTR session would die. I needed a way to launch the migration script detached from the RTR session so it would continue running even after the session ended.

---

### The Solution

The fix was simple: wrap the migration PowerShell script in a batch file that uses `start powershell.exe` to spawn a new, detached process. This ensures the migration continues in the background even after the RTR session disconnects.

> **Note:** The `start` command is key here. It launches the PowerShell script in a new, independent process, so it doesn't terminate when the parent RTR session closes.

---

### How I Deployed It

Since RTR was the only management tool on these hosts, I used it to deploy and run the batch file:

1.  **Push the BAT file** onto the endpoint via RTR:
    `put crowdstrike-migration.bat`

2.  **Run it** (this spawns the PowerShell script in the background and survives the sensor uninstall):
    `run C:\crowdstrike-migration.bat`

At this point, the RTR session disconnects, as expected, but the migration process continues to run in the background.

---

### Automating via SOAR

If you have **Falcon Fusion** or other **SOAR** (Security Orchestration, Automation, and Response) workflows, you can automate this process for a hands-off approach:

1.  **Push** the `crowdstrike-migration.bat` file to the CrowdStrike cloud storage.
2.  Use a SOAR workflow to **automatically issue the `put` and `run` commands** as new hosts come online.

This way, you don't need to manually manage each host; the migration happens automatically as they check in.

---

Hopefully, this helps anyone else who finds themselves in a similar situation, stuck migrating between tenants with **only the Falcon sensor installed** and no other management tools.
