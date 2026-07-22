## What is a Cron Job?

A **cron job** is a scheduled task on Unix-like systems (Linux, macOS, BSD) that runs automatically at specified times or intervals. The cron daemon (`crond`) reads a configuration file called a **crontab** and executes commands on schedule.

Cron is available natively on Linux and macOS. On Windows, the equivalent is **Task Scheduler**, but cron can also be used via WSL (Windows Subsystem for Linux).

---

## The Crontab File
Each user has their own crontab. There is also a system-wide crontab.
### Managing Crontabs

```bash
crontab -e        # Open your crontab in the default editor
crontab -l        # List your current cron jobs
crontab -r        # Remove (delete) your entire crontab — use with care
crontab -u alice -e  # Edit another user's crontab (requires root)
```

To set your preferred editor before opening:

```bash
export EDITOR=nano
crontab -e
```

---

## Cron Syntax

Every cron entry follows this format:

```
* * * * *  command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week  (0–7, where 0 and 7 = Sunday)
│ │ │ └──── Month        (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour         (0–23)
└────────── Minute       (0–59)
```

### Field Values

|Field|Allowed Values|Special Characters|
|---|---|---|
|Minute|0–59|`*` `,` `-` `/`|
|Hour|0–23|`*` `,` `-` `/`|
|Day of month|1–31|`*` `,` `-` `/`|
|Month|1–12|`*` `,` `-` `/`|
|Day of week|0–7|`*` `,` `-` `/`|

### Special Characters

|Character|Meaning|Example|
|---|---|---|
|`*`|Every possible value|`* * * * *` = every minute|
|`,`|List of values|`1,15,30` = at minute 1, 15, and 30|
|`-`|Range of values|`1-5` = Monday through Friday|
|`/`|Step/interval|`*/10` = every 10 units|

---

## Examples

### Basic Schedules

```bash
# Every minute
* * * * * /path/to/script.sh

# Every hour, at the top of the hour
0 * * * * /path/to/script.sh

# Every day at midnight
0 0 * * * /path/to/script.sh

# Every day at 3:30 AM
30 3 * * * /path/to/script.sh

# Every Monday at 9:00 AM
0 9 * * 1 /path/to/script.sh

# Every weekday (Mon–Fri) at 8:00 AM
0 8 * * 1-5 /path/to/script.sh

# First day of every month at noon
0 12 1 * * /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# Every 6 hours
0 */6 * * * /path/to/script.sh

# At 9 AM and 5 PM every weekday
0 9,17 * * 1-5 /path/to/script.sh

# Every Sunday at 2:00 AM
0 2 * * 0 /path/to/backup.sh
```

### Shorthand Strings (Supported in Most Cron Implementations)

```bash
@reboot    # Run once at system startup
@yearly    # Same as: 0 0 1 1 *
@monthly   # Same as: 0 0 1 * *
@weekly    # Same as: 0 0 * * 0
@daily     # Same as: 0 0 * * *
@hourly    # Same as: 0 * * * *
```

```bash
@daily /home/user/scripts/cleanup.sh
@reboot /home/user/scripts/start_server.sh
```

> Use [crontab.guru](https://crontab.guru/) to visually verify your cron expressions.

---

## Output and Logging

By default, cron emails output to the user. In most server environments there is no mail configured, so output is silently lost.

### Redirecting Output

```bash
# Discard all output (stdout and stderr)
* * * * * /path/to/script.sh > /dev/null 2>&1

# Log stdout only
* * * * * /path/to/script.sh >> /var/log/myjob.log

# Log both stdout and stderr
* * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Log stderr separately
* * * * * /path/to/script.sh >> /var/log/myjob.log 2>> /var/log/myjob_errors.log

# Append with timestamp using bash -c
* * * * * bash -c 'echo "$(date): Starting job" >> /var/log/myjob.log && /path/to/script.sh >> /var/log/myjob.log 2>&1'
```

---

## Environment in Cron

Cron runs with a **minimal environment** — it does not source your `~/.bashrc` or `~/.profile`. This is the most common source of cron failures.

### What's Missing by Default

- Your `$PATH` is stripped down (usually just `/usr/bin:/bin`)
- Environment variables like `HOME`, `USER`, `JAVA_HOME`, `NVM_DIR` are not set
- Shell aliases and functions are not available

### Fixes

**1. Set PATH at the top of the crontab:**

```
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

**2. Source your profile inside the command:**

```bash
* * * * * source /home/user/.bashrc && /path/to/script.sh
```

**3. Use absolute paths everywhere:**

```bash
# Bad — 'python3' may not resolve in cron's PATH
* * * * * python3 /home/user/script.py

# Good — full path used
* * * * * /usr/bin/python3 /home/user/script.py
```

Find the full path of any command:

```bash
which python3     # → /usr/bin/python3
which node        # → /usr/local/bin/node
```

**4. Set environment variables in the crontab:**

```
JAVA_HOME=/usr/lib/jvm/java-17-openjdk
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=""

0 2 * * * $JAVA_HOME/bin/java -jar /app/job.jar
```

---

## System-Wide Cron

System cron jobs live in `/etc/crontab` and `/etc/cron.d/`. These have an extra field for the **user** to run as:

```
# /etc/crontab format
* * * * * root /path/to/command
* * * * * www-data /var/www/scripts/cleanup.sh
```

Drop-in directories also exist for convenience:

|Directory|Runs|
|---|---|
|`/etc/cron.hourly/`|Every hour|
|`/etc/cron.daily/`|Every day|
|`/etc/cron.weekly/`|Every week|
|`/etc/cron.monthly/`|Every month|

Place executable scripts (no extension) in those folders and they run on schedule without editing any crontab.

---

## Practical Patterns

### Running a Python Script

```bash
# Install dependencies in a virtualenv and run
0 3 * * * /home/user/myenv/bin/python /home/user/scripts/process.py >> /var/log/process.log 2>&1
```

### Running a Node.js Script

```bash
0 6 * * * /usr/local/bin/node /home/user/app/job.js >> /var/log/node_job.log 2>&1
```

### Database Backup

```bash
# MySQL dump every night at 1 AM
0 1 * * * /usr/bin/mysqldump -u root -pPASSWORD mydb > /backups/mydb_$(date +\%Y\%m\%d).sql
```

> **Note:** Escape `%` as `\%` inside crontab files — unescaped `%` is treated as a newline by cron.

### Clearing Old Log Files

```bash
# Delete files older than 30 days in /tmp every Sunday at 3 AM
0 3 * * 0 /usr/bin/find /tmp -type f -mtime +30 -delete
```

### Sending a Health Check Ping

```bash
# Ping an uptime monitor every 5 minutes
*/5 * * * * /usr/bin/curl -fsS https://hc-ping.com/your-uuid > /dev/null 2>&1
```

### Chaining Commands

```bash
# Run command only if previous succeeds (&&)
0 2 * * * /scripts/backup.sh && /scripts/notify.sh

# Run second command regardless of first (;)
0 2 * * * /scripts/backup.sh ; /scripts/cleanup.sh
```

---

## Locking — Preventing Overlapping Runs

If a job takes longer than its interval, two instances can run simultaneously and cause conflicts.

```bash
# Use flock to prevent overlapping
*/5 * * * * /usr/bin/flock -n /tmp/myjob.lock /path/to/script.sh
```

`-n` means "non-blocking" — if the lock is already held, the new instance simply exits rather than waiting.

---

## Cron on Different Environments

### macOS

macOS uses `launchd` as its primary scheduler, but `cron` is still available:

```bash
crontab -e   # Works the same as on Linux
```

For persistent background tasks on macOS, `launchd` with a `.plist` file is preferred, but cron works for scripts.

### Docker / Containers

Cron doesn't run automatically in containers. You need to start the cron daemon explicitly:

```dockerfile
# In your Dockerfile
RUN apt-get install -y cron
COPY mycrontab /etc/cron.d/mycrontab
RUN chmod 0644 /etc/cron.d/mycrontab && crontab /etc/cron.d/mycrontab

CMD ["cron", "-f"]
```

Alternatively, use a process manager like **supervisord** to run both your app and cron.

### CI/CD and Cloud Environments

|Platform|Equivalent|
|---|---|
|GitHub Actions|`on: schedule` with cron syntax|
|GitLab CI|`rules: - if: $CI_PIPELINE_SOURCE == "schedule"`|
|AWS|EventBridge Scheduler / CloudWatch Events|
|Google Cloud|Cloud Scheduler|
|Kubernetes|`CronJob` resource|

**GitHub Actions example:**

```yaml
on:
  schedule:
    - cron: '0 3 * * *'   # Every day at 3 AM UTC
jobs:
  nightly:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: python scripts/nightly_job.py
```

**Kubernetes CronJob:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 1 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: myapp:latest
            command: ["/bin/sh", "-c", "python backup.py"]
          restartPolicy: OnFailure
```

---

## Debugging Cron Jobs

When a cron job silently fails, work through this checklist:

**1. Check if cron is running:**

```bash
systemctl status cron       # Debian/Ubuntu
systemctl status crond      # RHEL/CentOS
```

**2. Check system logs for cron activity:**

```bash
grep CRON /var/log/syslog          # Ubuntu/Debian
grep CRON /var/log/cron            # RHEL/CentOS
journalctl -u cron --since today   # systemd systems
```

**3. Test the command manually first — as the cron user:**

```bash
sudo -u www-data /path/to/script.sh
```

**4. Simulate the minimal cron environment:**

```bash
env -i HOME=/home/user PATH=/usr/bin:/bin /path/to/script.sh
```

**5. Capture all output to a log file:**

```bash
* * * * * /path/to/script.sh >> /tmp/debug.log 2>&1
```

**6. Check file permissions:**

```bash
chmod +x /path/to/script.sh   # Script must be executable
```

**7. Verify the shebang line in scripts:**

```bash
#!/bin/bash    # or #!/usr/bin/env bash
```

---

## Security Considerations

- **Restrict crontab access:** `/etc/cron.allow` and `/etc/cron.deny` control which users can use crontab.
- **Never put passwords in crontab** — use config files with restricted permissions (`chmod 600`) or a secrets manager.
- **Avoid running cron jobs as root** unless strictly necessary; use a dedicated service user.
- **Validate inputs** in scripts that accept external data to prevent injection attacks.
- **Log everything** your cron jobs do so you have an audit trail.

---

## Quick Reference Card

```
┌───────────── minute (0–59)
│ ┌─────────── hour (0–23)
│ │ ┌───────── day of month (1–31)
│ │ │ ┌─────── month (1–12)
│ │ │ │ ┌───── day of week (0–7, 0 and 7 = Sun)
│ │ │ │ │
* * * * *  command

Every minute:        * * * * *
Every hour:          0 * * * *
Every day midnight:  0 0 * * *
Every Monday 9 AM:   0 9 * * 1
Every 15 minutes:    */15 * * * *
Weekdays 8 AM:       0 8 * * 1-5
1st of month noon:   0 12 1 * *
```