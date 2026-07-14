# PR Response Doc — CineLog Watchlist Feature

## Comment 1 — Rename

**What I did:**

I renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's existing verb-to-noun naming convention.

I updated all call sites that referenced the old function name, including the watchlist route implementation in `routes/watchlist/watchlist.py`.

**How I verified:**

I used project-wide search/find-all-references for `save_to_watchlist` to confirm that no old references remained. After the rename, I ran the application tests to verify there were no import errors or broken function references.

---