# Change Date & Time — K2

The K2 syncs its clock via NTP (Network Time Protocol) when connected to the internet. If the date and time are incorrect it can cause SSL certificate errors when downloading the Helper Script or its components.

---

## Check Current Date and Time

```bash
date
```

---

## Sync Time via NTP (Recommended)

If connected to the internet, restart the NTP service:

```bash
/etc/init.d/S98sysntpd restart
sleep 5
date
```

---

## Set Time Manually

If the printer is not connected to the internet:

```bash
# Format: MMDDhhmm[[CC]YY][.ss]
# Example: June 4, 2026, 14:30
date 060414302026
```

---

## Set Timezone

The K2 timezone is configured in `system_config.json`. To change it:

```bash
# View current timezone setting
python3 -c "import json; d=json.load(open('/mnt/UDISK/creality/userdata/config/system_config.json')); print(d['user_info']['time_zone'])"

# The timezone is set by the Creality UI — change it from
# Settings → System → Time Zone on the touchscreen
```

---

## Fix SSL Errors When Downloading

A wrong system clock makes HTTPS certificate checks fail — for example when downloading the helper script with Python, or when `git` (installed via Entware) clones a repository for OctoEverywhere, Mobileraker Companion, OctoApp or Git Backup.

```bash
# Sync time first
/etc/init.d/S98sysntpd restart
sleep 10
date
```

Then run the download or installer again. Avoid disabling certificate verification (`git config --global http.sslVerify false`) — fix the clock instead.
