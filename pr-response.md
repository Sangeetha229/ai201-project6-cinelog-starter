# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used AI as a support tool during development, mainly for understanding existing project patterns, reviewing my reasoning, and debugging test issues.

- For Comment 2 (Deduplication), I used AI to help understand how the existing `add_to_collection()` function handled duplicate entries. I reviewed the existing pattern and implemented the same approach in `add_to_watchlist()` by checking for an existing `WatchlistEntry` before inserting a new record.


## Comment 1 — Rename

**What I did:**

I renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's existing verb-to-noun naming convention.

I updated all call sites that referenced the old function name, including the watchlist route implementation in `routes/watchlist/watchlist.py`.

**How I verified:**

I used project-wide search/find-all-references for `save_to_watchlist` to confirm that no old references remained. After the rename, I ran the application tests to verify there were no import errors or broken function references.

---

## Comment 2 — Deduplication

**What I did:**

I added duplicate prevention logic to `add_to_watchlist()`.

Before creating a new `WatchlistEntry`, the function now checks whether an entry already exists for the same `user_id` and `film_id`.

If an existing entry is found, the function raises `AlreadyInWatchListError` instead of creating a duplicate watchlist record.

**How I verified:**

I tested adding the same film twice for the same user. The first request successfully creates the watchlist entry, while the second request raises the expected error.

I also verified that only one `WatchlistEntry` exists in the database after attempting the duplicate addition.

The implementation follows the same pattern used in `add_to_collection()` from `services/collection_service.py`, where an existing relationship is checked before inserting a new record.

---
