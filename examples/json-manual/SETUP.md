# Manual JSON Export

Export Intervals.icu data locally for different time ranges.

## Prerequisites

- Python 3.8+
- `requests` library: `pip install requests`

## First-Time Setup

```bash
python sync.py --setup
```

Enter your Intervals.icu credentials when prompted. Config saves to `.sync_config.json`.

**Finding your credentials:**
- **Athlete ID**: Intervals.icu → Settings → bottom of page (e.g., `i123456`)
- **API Key**: Intervals.icu → Settings → Developer Settings → API Key

## Usage

### Export to local file

```bash
# Last 7 days (default)
python sync.py --output latest.json

# Last 14 days
python sync.py --days 14 --output 14days.json

# Last 90 days
python sync.py --days 90 --output 90days.json
```

### Common time ranges

| Days | Use case |
|------|----------|
| 7 | Weekly review |
| 14 | Two-week block |
| 42 | 6-week training block |
| 90 | Quarterly / build cycle |
| 180 | Season review |

### Push to GitHub (optional)

If you configured GitHub credentials during setup:

```bash
python sync.py --days 14
```

Pushes to your configured GitHub repo.

## Generated Files

The script creates/maintains these files:

| File | Purpose | When created |
|------|---------|--------------|
| `latest.json` | Training data export | Every run with `--output` |
| `history.json` | Longitudinal data — daily (90d), weekly (180d), monthly (3y) | First run, regenerates when outdated |
| `ftp_history.json` | FTP progression tracking | Automatically on first run |
| `.sync_config.json` | Your credentials + preferences (local only) | After `--setup` |

### FTP History

`ftp_history.json` tracks indoor and outdoor FTP changes over time:

```json
{
  "indoor": {"2026-01-01": 270, "2026-02-01": 275},
  "outdoor": {"2026-01-01": 280, "2026-02-01": 287}
}
```

- Updated automatically when FTP changes
- Used to calculate **Benchmark Index** (8-week FTP progression)
- Keep this file if you want continuous tracking

### History Data

`history.json` provides longitudinal context with tiered granularity:

| Tier | Granularity | Range |
|------|-------------|-------|
| `daily_90d` | Day-by-day | Last 90 days |
| `weekly_180d` | Week-by-week | Last 180 days |
| `monthly_1y/2y/3y` | Month-by-month | Up to 3 years |

Also includes period summaries, FTP timeline, and data gap detection. Generated automatically on first run and regenerated when outdated.

## What's Included

The export includes pre-calculated **derived metrics** for Section 11 compliance:

| Metric | Description |
|--------|-------------|
| CTL / ATL / TSB | Fitness, fatigue, form (decay-corrected) |
| Ramp Rate | Training load trend (smart: excludes uncompleted planned workouts) |
| ACWR | Acute: Chronic Workload Ratio |
| Recovery Index | HRV/RHR composite |
| Monotony / Strain | Training variability metrics |
| Grey Zone % | Z3 time (to minimize) |
| Quality Intensity % | Z4+ time (target ~20%) |
| Polarisation Index | Easy time ratio (target ~80%) |
| Benchmark Index | 8-week FTP progression |
| Phase Detected | Auto-detected training phase |

## Use with AI

**Option 1: Upload files**
Upload both `latest.json` and `history.json` to your AI platform for a complete analysis with longitudinal context.

**Option 2: Push to GitHub + configure AI**
Push to a GitHub repo (private recommended), then follow the instructions in the main [README](../README.md#quick-start). Provide both JSON URLs to your AI coach.

**Option 3: Use with desktop agents**
Claude Cowork, OpenAI Codex CLI, and local OpenClaw can read files directly from your filesystem — no GitHub needed. Point the agent at the folder containing your exported JSON files.

---

## Options Reference

| Flag | Description | Default |
|------|-------------|---------|
| `--setup` | Run setup wizard | - |
| `--days N` | Days of data to export | 7 |
| `--output FILE` | Save to local file | - |
| `--week-start DAY` | Training week start day (mon/tue/wed/thu/fri/sat/sun) | mon |
| `--debug` | Show API field debug info | off |
| `--anonymize` | Remove identifying info | on |

**Note:** `--week-start` can also be set in `.sync_config.json` (`"week_start": "sun"`) or via `WEEK_START` environment variable. Config file setting persists across runs — no need to pass the flag every time.

**Note:** Anonymization is enabled by default. Activity names, athlete ID, and location data are redacted in the output.

---

## Troubleshooting

### "Config not found" error
Run `python sync.py --setup` first.

### Empty or missing data
- Check your API key is valid
- Verify you have activities in the requested date range
- Run with `--debug` to see API responses

### FTP history not updating
FTP history only adds entries when FTP **changes**. If your FTP is the same, no new entry is added.
