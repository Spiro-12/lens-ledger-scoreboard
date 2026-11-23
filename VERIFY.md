# How to verify

(Copy this file over `VERIFY.md` in the public scoreboard repo. It is kept here so the founder can review it first.)

## 1. Check a Bitcoin timestamp

1. Install the OpenTimestamps client: `pip install opentimestamps-client`.
2. Run `ots verify <file>.ots`, for example `ots verify proofs/prereg-v1/v1.freeze.json.ots`. It reports the Bitcoin block that proves the file existed at that time. A brand-new stamp shows "pending" for a few hours until Bitcoin confirms it; we commit the upgraded proof once it does.
3. When the rules text is published, check that `sha256sum` of each file matches `files_sha256` in `v1.freeze.json`.

## 2. Check a ten-minute fingerprint (`proofs/<date>/intraday/<time>.txt`)

Every 10 minutes we write one of these files. It lists every decision-log file written so far that UTC day and, from 23 Nov 2026, every forecast file our forward tests (weather highs and lows, flu proving grounds) wrote that day (paths `forward/<test>/...`), one line each:

    <sha256>  <bytes>  <path>

The log files only ever grow at the end, so each line's hash covers exactly the first `<bytes>` bytes of the file, and the check still works later, after the file has grown. Our log files are private until the questions settle; once a file is shared with you:

1. Check the one file: `head -c <bytes> <file> | sha256sum` must equal that line's `<sha256>`.
2. Check the whole fingerprint: the line `rolling_sha256:` at the top must equal the SHA-256 of all the file lines (everything after the blank line), sorted by path, each followed by a newline. In a shell: `tail -n +8 <fingerprint>.txt | LC_ALL=C sort -k3 | sha256sum`. (The sort keeps lines ordered by path.)

On a Mac without `sha256sum`, use `shasum -a 256` in its place in both commands.
3. Check the time: `ots verify <fingerprint>.txt.ots` shows the Bitcoin block that proves the fingerprint existed by then. This is what proves we did not decide after seeing the results.

Notes
- Files named `next-day-HHMM.txt` were written just after midnight UTC for the day before. They cover the last minutes of that day.
- A stamp may stay "pending" for a few hours. Pending is normal.
- Daily files (`proofs/<date>/manifest.json`, `manifest.root.txt` and its `.ots`) fingerprint the files our collectors and logs wrote that UTC day in our private raw-data folder and, from 23 Nov 2026, our forward tests' forecast files written that day (`forward/<test>/...`). They do not cover everything we keep (research and backtest files are not included).
