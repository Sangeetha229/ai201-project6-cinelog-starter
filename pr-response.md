# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used AI as a support tool during development, mainly for understanding existing project patterns, reviewing my reasoning, and debugging test issues.

- For Comment 2 (Deduplication), I used AI to help understand how the existing `add_to_collection()` function handled duplicate entries. I reviewed the existing pattern and implemented the same approach in `add_to_watchlist()` by checking for an existing `WatchlistEntry` before inserting a new record.

- For Comments 4 and 5 (Default Visibility and Sort Order), I used AI as a devil's advocate to stress-test my design decisions. I provided my draft reasoning and asked:
  
  > "What counterargument would a careful code reviewer raise against this position? What tradeoff am I not acknowledging?"

  The feedback helped me make my explanations more complete by acknowledging the privacy tradeoff for public visibility and the usability tradeoff between alphabetical sorting and date-added sorting. The final decisions and reasoning are my own and are based on CineLog user behavior.


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

## Comment 3 — Missing Test

**What I did:**

I created `tests/test_watchlist.py` and added a test for adding a nonexistent film to a user's watchlist.

The test was modeled after `test_add_to_collection_nonexistent_film_raises` from `tests/test_collection.py`.

**How I verified:**

The test creates a fake UUID that does not exist in the database and confirms that calling `add_to_watchlist()` raises `FilmNotFoundError`.

I ran:

```bash
pytest tests/test_watchlist.py -v
```

and verified that the test passes.

---

## Comment 4 — Default Visibility

**My position:**

The default visibility should be `public=True`.

**Reasoning:**

CineLog is designed around discovering and sharing film interests. A public-by-default watchlist allows users to share upcoming movies they are interested in watching, making it easier for friends and other users to discover similar interests and recommend films.

A user adding a movie to a CineLog watchlist is usually expressing interest rather than storing sensitive personal information. Making watchlists visible by default supports the social discovery aspect of the platform.

**Tradeoff acknowledged:**

A private-by-default watchlist would better support users who use their watchlist only as a personal tracking tool and do not want their viewing plans visible.

However, choosing public as the default provides more community value for CineLog while still allowing future functionality where users can change visibility settings if they prefer privacy.

---

## Comment 5 — Sort Order

**My position:**

I implemented watchlist sorting by `date_added` with the newest films displayed first.

**Reasoning:**

Users typically add movies to a watchlist when they discover something they want to watch soon. Showing recently added films first matches how users interact with their watchlist because the newest additions usually represent their current interests.

This also helps users quickly find the movies they recently saved without needing to search through the entire list.

**Engagement with reviewer's point:**

I agree with the maintainer's point that many users want to see what they added recently.

Alphabetical sorting can help when managing a very large collection, but a watchlist represents changing intentions rather than a permanent library. For CineLog's use case, chronological ordering provides more meaningful information about user behavior.

---

# Stretch Features

## remove_from_watchlist()

**What I did:**

I added `remove_from_watchlist(user_id, film_id)` following the same pattern used by `remove_from_collection()`.

The function searches for the matching `WatchlistEntry`. If the entry exists, it deletes it and commits the transaction.

If the film is not present in the user's watchlist, it raises `NotInWatchlistError` instead of silently failing.

**How I verified:**

I added tests covering:
- Successful removal of an existing watchlist entry
- Attempting to remove a film that is not in the watchlist

These tests confirm that the function handles both normal and error scenarios.

---