# LogSentinel Pro

A Python SIEM mini. It turns raw auth log lines into events, detects brute force inside a time window, flags a successful login after brute force, and saves alerts to `alerts.json`.

Built to learn real detection logic: parse first, detect by time, return data, save, then print.

## What It Does

1. Parses auth log lines into structured events
2. Detects brute force by counting fails inside a sliding time window, not just totals
3. Detects a successful login that follows brute force, flagged as CRITICAL
4. Saves all alerts to `alerts.json`, one JSON object per line
5. Handles missing files and bad lines without crashing

## Functions

### `parse_auth_log(file_path)`
- Input: `file_path` - string, e.g. `"sample_auth2.log"`
- Output: list of dicts, each dict has `time`, `status`, `user`, `ip`, all strings
- Uses regex with named groups to pull `time`, `status`, `user`, `ip` from each line
- Skips lines with no match
- Returns `[]` if file is not found

### `detect_brute_force(events, threshold, window_minutes)`
- Input: `events` from `parse_auth_log`, `threshold` int, `window_minutes` int
- Output: list of alert dicts with `time`, `ip`, `severity`, `reason`
- Groups fails by IP, sorts timestamps, counts fails inside the window
- Severity: `HIGH` if fails > threshold x 2, else `MEDIUM`
- Returns 1 alert per IP

### `detect_success_after_brute_force(events)`
- Input: `events` from `parse_auth_log`
- Output: list of alert dicts, all `CRITICAL`
- Groups events by IP, sorts by time
- Looks for 5 or more fails, then 1 success within 10 minutes
- Reason: `Possible successful brute force: 5 failures followed by a successful login within 10 minutes`
- Returns 1 alert per IP

### `save_alerts(alerts)`
- Input: `alerts` - list of dicts
- Output: None, writes file
- Opens `alerts.json` for writing, writes one JSON object per line
- Handles `OSError` / `IOError` with `try/except` and prints the error

### `main()`
- Takes log file path from `sys.argv`
- Calls `parse_auth_log`, `detect_brute_force`, `detect_success_after_brute_force`
- Combines both lists into `all_alerts`
- Calls `save_alerts`, then prints each alert

## Quick Start

```bash
git clone https://github.com/YOUR-USERNAME/LogSentinel-Pro.git
cd LogSentinel-Pro
