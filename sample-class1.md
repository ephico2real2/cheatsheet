Below is an example of how you can set up a Python script and corresponding systemd service and timer unit files using your specified paths. In this example, the Python script is located at:

```
/usr/local/dotmobi-tools/scripts/system-monitor.py
```

and it logs its output to a log file in the folder:

```
/usr/local/api/logs
```

We'll also demonstrate how to pass an environment variable (in this case, the log directory) to the script via the systemd service.

---

## 1. Create the Python Script

Create the file `/usr/local/dotmobi-tools/scripts/system-monitor.py` with the following content:

```python
#!/usr/bin/env python3
import os
import sys
import logging
from datetime import datetime

# Use the environment variable SYSTEM_MONITOR_LOG_DIR if set; otherwise, default to /usr/local/api/logs.
log_dir = os.environ.get("SYSTEM_MONITOR_LOG_DIR", "/usr/local/api/logs")
# Ensure the directory exists.
os.makedirs(log_dir, exist_ok=True)
# Define the log file path by appending a filename to the directory.
log_file = os.path.join(log_dir, "system-monitor.log")

# Configure the logger.
logger = logging.getLogger("SystemMonitor")
logger.setLevel(logging.INFO)

# Create a formatter that includes a timestamp.
formatter = logging.Formatter(
    fmt='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# File handler for logging to the file (appends by default).
file_handler = logging.FileHandler(log_file)
file_handler.setLevel(logging.INFO)
file_handler.setFormatter(formatter)

# Console handler for logging to the console.
console_handler = logging.StreamHandler(sys.stdout)
console_handler.setLevel(logging.INFO)
console_handler.setFormatter(formatter)

# Add both handlers to the logger.
logger.addHandler(file_handler)
logger.addHandler(console_handler)

# Obtain the current OS time.
current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
# Log the message, including the current time.
logger.info(f"System Monitor executed {current_time}")

```

Make the script executable:

```bash
sudo chmod +x /usr/local/dotmobi-tools/scripts/system-monitor.py
```

---

## 2. Create the Systemd Service Unit File

Create a file named `/etc/systemd/system/system-monitor.service` with the following content:

```ini
[Unit]
Description=System Monitor Service (Python)

[Service]
Type=oneshot
# Set the environment variable so the script knows where to log.
Environment=SYSTEM_MONITOR_LOG_DIR=/usr/local/api/logs
ExecStart=/usr/bin/python3 /usr/local/dotmobi-tools/scripts/system-monitor.py
```

**Explanation:**

- **Type=oneshot:** The service runs the Python script once and then exits.
- **Environment=...:** This line sets the `SYSTEM_MONITOR_LOG_DIR` variable for the script.
- **ExecStart=...:** Calls the system Python 3 interpreter with your script.

---

## 3. Create the Systemd Timer Unit File

Create a file named `/etc/systemd/system/system-monitor.timer` with the following content:

```ini
[Unit]
Description=Timer to trigger the System Monitor Service every minute

[Timer]
# Run the service every minute.
OnCalendar=*:0/1
Persistent=true

[Install]
WantedBy=timers.target
```

**Explanation:**

- **OnCalendar=*:0/1:** Schedules the timer to trigger the service every minute. (You can adjust the schedule as needed.)
- **Persistent=true:** If the system is off when a timer event was supposed to occur, the service will run as soon as possible after boot.

---

## 4. Reload systemd, Start, and Enable the Timer

After creating or modifying these unit files, reload the systemd daemon to register them:

```bash
sudo systemctl daemon-reload
```

Start the timer immediately:

```bash
sudo systemctl start system-monitor.timer
```

If you want the timer to start automatically on boot, enable it:

```bash
sudo systemctl enable system-monitor.timer
```

---

## 5. Verifying the Setup

- **Check the Timer Status:**

  ```bash
  sudo systemctl status system-monitor.timer
  ```

- **Monitor the Log File:**

  Since your log file is located at `/usr/local/api/logs/system-monitor.log`, you can monitor it with:

  ```bash
  tail -f /usr/local/api/logs/system-monitor.log
  ```

Each time the timer triggers, systemd will run your Python script using the system’s Python 3 interpreter. The script will use the `SYSTEM_MONITOR_LOG_DIR` environment variable (set in the service unit file) to determine where to log its output.

---

### Summary

- **Environment Variables in systemd:**  
  As you suspected, if your script requires certain environment variables (like the log directory), you need to add them to the service unit file (using the `Environment` directive) or reference an external file using `EnvironmentFile`.
  
- **Custom Paths:**  
  We updated the script and unit file to use `/usr/local/dotmobi-tools/scripts/system-monitor.py` for the script location and `/usr/local/api/logs` for logging.

This setup allows you to easily adjust the log destination by changing the `Environment=SYSTEM_MONITOR_LOG_DIR=...` line in the service file, and it demonstrates how to integrate a Python script with systemd without using a virtual environment.
