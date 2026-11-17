# How to verify

1. Install the OpenTimestamps client: `pip install opentimestamps-client`.
2. Check a stamp: `ots verify proofs/prereg-v1/v1.freeze.json.ots`. It reports the Bitcoin block that proves the file existed at that time. A brand-new stamp shows "pending" for a few hours until Bitcoin confirms it; we commit the upgraded proof once it does.
3. When the rules text is published, check that `sha256sum` of each file matches `files_sha256` in `v1.freeze.json`.
