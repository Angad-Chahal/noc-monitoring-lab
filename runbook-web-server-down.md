# Runbook: Web Server Down (Nginx)

**Owner:** Angad Chahal  
**Last updated:** 2026-10-27  
**Environment:** Ubuntu Server VM, Nginx, monitored with Uptime Kuma

## 1. Symptom
- Uptime Kuma alert: **"Nginx web server is Down"** (Discord notification).
- Visiting `http://10.0.0.244` in a browser times out or shows an error.

## 2. Severity
- Web service unavailable to users: **High**. Start work immediately.

## 3. Checks (in order)
1. **Confirm the alert is real.** Open `http://10.0.0.244` in a browser. If it loads, the alert may be a short blip. Check the Uptime Kuma monitor history.
2. **Log in to the server** (`ssh vboxuser@10.0.0.244`) and check the service:
   ```
   systemctl status nginx
   ```
   - `active (running)`: Nginx is up. Check the network and firewall (step 5).
   - `inactive` or `failed`: continue.
3. **Read the recent logs for the cause:**
   ```
   sudo journalctl -u nginx -n 50
   ```
4. **Check disk space** (a full disk can stop services):
   ```
   df -h
   ```
   If any filesystem is at 100%, free space first (delete old logs, or expand the disk).
5. **Check the network path:**
   ```
   ip a
   ping -c 3 10.0.0.1
   curl http://localhost
   ```
   - `curl` works locally but not from the browser: network or firewall issue.

## 4. Fix
1. **Test the Nginx configuration for errors:**
   ```
   sudo nginx -t
   ```
   If it reports an error, edit the file it names, fix the mistake, and run the test again.
2. **Start (or restart) the service:**
   ```
   sudo systemctl start nginx
   ```
   or, if it is running but misbehaving:
   ```
   sudo systemctl restart nginx
   ```
3. **Verify the fix:**
   ```
   systemctl status nginx
   curl http://localhost
   ```
   Confirm the Uptime Kuma monitor turns **green (Up)** and the Discord "Up" message arrives.

## 5. Escalation
Escalate if any of these are true:
- Nginx will not start after `nginx -t` passes and a restart.
- The disk is full and space cannot be freed.
- The server is unreachable (no ping, no SSH).

**Contact:** on-call IT/server lead (name, phone, email). Include: time of the alert, checks already done, and the error text from `journalctl`.

## 6. After the incident
- Record the root cause, the fix, and how long the service was down.
- Update this runbook if any step was missing or unclear.

## Revision history
| Date | Change | Author |
|---|---|---|
| 2026-10-27 | First version | Angad Chahal |
