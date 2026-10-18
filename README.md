# System-Temp-Monitor

A small Bash script for Linux that records the system temperature with a timestamp, so you can check how hot a machine runs over hours or days. Run it from cron every 10 minutes and it builds up a CSV log that opens in any spreadsheet.

It is designed for the **Raspberry Pi**, where it uses `vcgencmd`. On any other Linux machine it reads the kernel's thermal sensor (`/sys/class/thermal`) instead.

```
timestamp,temp_c
2026-10-18 09:40:00,48.3
2026-10-18 09:50:00,49.1
2026-10-18 10:00:00,51.6
```

> **On a Mac?** This script is Linux-only. See [launch-watch](https://github.com/PrivSter/launch-watch), which includes a System Check app for macOS with live temperature graphs.

## Install

```bash
git clone https://github.com/PrivSter/System-Temp-Monitor.git ~/System-Temp-Monitor
chmod +x ~/System-Temp-Monitor/Temprogram
```

To check that it works, run it by hand:

```bash
~/System-Temp-Monitor/Temprogram
# 48.3°C  (logged to /home/pi/temp-readings.csv)
```

## Record automatically every 10 minutes

Open your crontab:

```bash
crontab -e
```

Add this line at the bottom, then save and exit:

```
*/10 * * * * $HOME/System-Temp-Monitor/Temprogram
```

To change how often it records, change `*/10`. For example, `*/5` records every 5 minutes, and `0 *` records once an hour.

Readings go to `~/temp-readings.csv`. To use a different file, set `TEMP_LOG` in the crontab line:

```
*/10 * * * * TEMP_LOG=$HOME/logs/pi-temp.csv $HOME/System-Temp-Monitor/Temprogram
```

## Viewing the log

```bash
tail ~/temp-readings.csv                                  # latest readings
sort -t, -k2 -n ~/temp-readings.csv | tail -1             # hottest reading
```

You can also open the CSV in Excel, Numbers or Google Sheets and insert a line chart.

On a Raspberry Pi, the CPU starts slowing itself down at about **80°C**. Readings that stay above 70°C under normal use suggest the Pi needs a heatsink or fan.

## Troubleshooting

**Nothing is being logged.**
- Check cron is running with `systemctl status cron`.
- Run `grep CRON /var/log/syslog` to see whether the job fired.
- Make sure the script is executable: `chmod +x ~/System-Temp-Monitor/Temprogram`.

**"no temperature sensor found".** The machine exposes no thermal sensor that the script recognises. Virtual machines and containers usually don't have one.

**Do I need sudo?** No. Earlier versions called `sudo vcgencmd`, which stopped cron from working, because cron can't enter a password. The current script runs as your normal user.
