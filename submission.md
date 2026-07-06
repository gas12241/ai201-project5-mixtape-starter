# Submission

## Codebase Map

### Routes

#### feed.py

- Purpose: Exposes the "friends listening now" and activity feed endpoints. Registered under the `/feed` URL prefix.
- Endpoints:
  - `GET /feed/<user_id>/listening-now`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | user_id | Path | String | ID of the current user |
    - Output contract: `200` with `{"feed": [...], "count": <int>}`, where `feed` is the list returned by `get_friends_listening_now`; `404` with `{"error": <message>}` if the user does not exist.
  - `GET /feed/<user_id>/activity`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | user_id | Path | String | ID of the current user |
    - Output contract: `200` with `{"feed": [...], "count": <int>}`, where `feed` is the list returned by `get_activity_feed` (called with the service's default `limit` of 20; this route does not expose a way to override it); `404` with `{"error": <message>}` if the user does not exist.

#### playlists.py

- Purpose: Exposes playlist creation, retrieval, and song-adding endpoints. Registered under the `/playlists` URL prefix.
- Endpoints:
  - `POST /playlists/`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | name | Body | String | Name of the playlist (required) |
      | created_by | Body | String | ID of the user creating the playlist (required) |
      | is_collaborative | Body | Boolean | Whether the playlist allows collaboration (optional, defaults to `True`) |
    - Output contract: `201` with the created playlist as JSON on success; `400` with `{"error": "name and created_by are required"}` if either required field is missing; `400` with `{"error": <message>}` if `create_playlist` raises `ValueError` (e.g. the creator does not exist).
  - `GET /playlists/<playlist_id>`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | playlist_id | Path | String | ID of the playlist being retrieved |
    - Output contract: `200` with the playlist dict as JSON; `404` with `{"error": <message>}` if the playlist does not exist.
  - `GET /playlists/<playlist_id>/songs`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | playlist_id | Path | String | ID of the playlist whose songs are requested |
    - Output contract: `200` with `{"songs": [...], "count": <int>}`, where `songs` is the list returned by `get_playlist_songs`; `404` with `{"error": <message>}` if the playlist does not exist.
  - `POST /playlists/<playlist_id>/songs`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | playlist_id | Path | String | ID of the playlist receiving the song |
      | song_id | Body | String | ID of the song being added (required) |
      | added_by | Body | String | ID of the user adding the song (required) |
    - Output contract: `201` with `{"message": "Song added to playlist"}` on success; `400` with `{"error": "song_id and added_by are required"}` if either required field is missing; `400` with `{"error": <message>}` if `add_to_playlist` raises `ValueError` (e.g. the playlist, song, or user is missing).

#### songs.py

- Purpose: Exposes song search, lookup, rating, and listening endpoints. Registered under the `/songs` URL prefix.
- Endpoints:
  - `GET /songs/search`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | q | Query | String | Search text used to match song titles or artists (required) |
    - Output contract: `200` with `{"results": [...], "count": <int>}`, where `results` is the list returned by `search_songs`; `400` with `{"error": "Query parameter 'q' is required"}` if `q` is missing or empty.
  - `GET /songs/<song_id>`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | song_id | Path | String | ID of the song being retrieved |
    - Output contract: `200` with the song dict as JSON; `404` with `{"error": <message>}` if the song does not exist.
  - `POST /songs/<song_id>/rate`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | song_id | Path | String | ID of the song being rated |
      | user_id | Body | String | ID of the user submitting the rating (required) |
      | score | Body | Integer | Rating value from 1 to 5 (required) |
    - Output contract: `201` with the created or updated rating dict on success; `400` with `{"error": "user_id and score are required"}` if either required field is missing; `400` with `{"error": <message>}` if `rate_song` raises `ValueError` (e.g. an out-of-range score or a missing user/song).
  - `POST /songs/<song_id>/listen`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | song_id | Path | String | ID of the song that was listened to |
      | user_id | Body | String | ID of the user who listened (required) |
    - Output contract: `201` with the created listening event dict on success; `400` with `{"error": "user_id is required"}` if `user_id` is missing; `400` with `{"error": <message>}` if `record_listening_event` raises `ValueError` (e.g. the user does not exist).

#### users.py

- Purpose: Exposes user profile, streak, and notification endpoints. Registered under the `/users` URL prefix.
- Endpoints:
  - `GET /users/<user_id>`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | user_id | Path | String | ID of the user being retrieved |
    - Output contract: `200` with the user dict as JSON; `404` with `{"error": "User not found"}` if the user does not exist. (This route queries the database directly rather than going through a service function.)
  - `GET /users/<user_id>/streak`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | user_id | Path | String | ID of the user whose streak is requested |
    - Output contract: `200` with `{"user_id": <id>, "streak": <int>}`; `404` with `{"error": <message>}` if the user does not exist.
  - `GET /users/<user_id>/notifications`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | user_id | Path | String | ID of the user whose notifications are requested |
      | unread_only | Query | String (`"true"`/`"false"`) | Whether to return only unread notifications (optional, defaults to `"false"`) |
    - Output contract: `200` with `{"notifications": [...], "count": <int>}`, where `notifications` is the list returned by `get_notifications`; `404` with `{"error": <message>}` if the user does not exist.
  - `POST /users/notifications/<notification_id>/read`
    - Input contract:
      | Attribute | Location | Type | Description |
      | --- | --- | --- | --- |
      | notification_id | Path | String | ID of the notification to mark as read |
    - Output contract: `200` with `{"message": "Notification marked as read"}` on success; `404` with `{"error": <message>}` if the notification does not exist.

### Services

#### feed_service.py

- Purpose: Provides feed-related queries for recent friend activity and listening events.
- Methods:
  - `get_friends_listening_now(user_id: str) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the current user |
    - Output contract: Returns a list of dictionaries containing each friend’s profile, the song they listened to, and the timestamp; raises `ValueError` if the user does not exist.
  - `get_activity_feed(user_id: str, limit: int = 20) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the current user |
      | limit | Integer | Maximum number of activity items to return |
    - Output contract: Returns a list of recent activity dictionaries for the user’s friends; raises `ValueError` if the user does not exist.

#### notification_service.py

- Purpose: Creates, retrieves, and updates notifications for user activity.
- Methods:
  - `create_notification(user_id: str, notification_type: str, body: str) -> Notification`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user who should receive the notification |
      | notification_type | String | Short type label for the notification |
      | body | String | Human-readable notification message |
    - Output contract: Returns the newly created notification object.
  - `add_to_playlist(playlist_id: str, song_id: str, added_by_user_id: str) -> None`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | playlist_id | String | ID of the playlist receiving the song |
      | song_id | String | ID of the song being added |
      | added_by_user_id | String | ID of the user adding the song |
    - Output contract: Adds the song to the playlist if needed and creates a notification for the song’s sharer when appropriate; raises `ValueError` if the playlist, song, or user is missing.
  - `rate_song(user_id: str, song_id: str, score: int) -> Rating`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user submitting the rating |
      | song_id | String | ID of the song being rated |
      | score | Integer | Rating value from 1 to 5 |
    - Output contract: Returns the created or updated rating object; raises `ValueError` for invalid scores or missing entities.
  - `get_notifications(user_id: str, unread_only: bool = False) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user whose notifications are requested |
      | unread_only | Boolean | Whether to return only unread notifications |
    - Output contract: Returns a list of notification dictionaries ordered by recency.
  - `mark_as_read(notification_id: str) -> None`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | notification_id | String | ID of the notification to mark as read |
    - Output contract: Marks the notification as read; raises `ValueError` if the notification does not exist.

#### playlist_service.py

- Purpose: Handles playlist creation and retrieval logic.
- Methods:
  - `create_playlist(name: str, created_by_user_id: str, is_collaborative: bool = True) -> Playlist`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | name | String | Name of the playlist |
      | created_by_user_id | String | ID of the user creating the playlist |
      | is_collaborative | Boolean | Whether the playlist allows collaboration |
    - Output contract: Returns the newly created playlist object; raises `ValueError` if the creator user does not exist.
  - `get_playlist_songs(playlist_id: str) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | playlist_id | String | ID of the playlist whose songs are requested |
    - Output contract: Returns the songs in the playlist in their stored order as dictionaries; raises `ValueError` if the playlist does not exist.
  - `get_playlist(playlist_id: str) -> dict`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | playlist_id | String | ID of the playlist being retrieved |
    - Output contract: Returns the playlist metadata as a dictionary; raises `ValueError` if the playlist does not exist.
  - `get_user_playlists(user_id: str) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user whose playlists are requested |
    - Output contract: Returns a list of playlists created by that user as dictionaries.

#### search_service.py

- Purpose: Supports song search and lookup operations.
- Methods:
  - `search_songs(query: str) -> list[dict]`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | query | String | Search text used to match song titles or artists |
    - Output contract: Returns matching songs as dictionaries, including their tags.
  - `get_song(song_id: str) -> dict`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | song_id | String | ID of the song being retrieved |
    - Output contract: Returns the requested song as a dictionary; raises `ValueError` if the song does not exist.

#### streak_service.py

- Purpose: Tracks user listening streaks and records listening activity.
- Methods:
  - `record_listening_event(user_id: str, song_id: str) -> ListeningEvent`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user who listened |
      | song_id | String | ID of the song that was listened to |
    - Output contract: Records a listening event, updates the user’s streak, and returns the created event object; raises `ValueError` if the user does not exist.
  - `update_listening_streak(user: User, now: datetime) -> None`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user | User | The user object whose streak is being updated |
      | now | DateTime | The current timestamp used for streak evaluation |
    - Output contract: Updates the user’s streak and last-listened timestamp in place, following these rules: if the user has never listened before, the streak starts at 1; if the user already listened today, no change is made; if the user listened yesterday, the streak increments by 1; if more than one day has passed since the last listen, the streak resets to 1.
  - `get_streak(user_id: str) -> int`
    - Input contract:
      | Attribute | Type | Description |
      | --- | --- | --- |
      | user_id | String | ID of the user whose streak is requested |
    - Output contract: Returns the user’s current streak as an integer; raises `ValueError` if the user does not exist.

### Data Flow Example

Data flow — user adds a song to a playlist: `POST /playlists/<playlist_id>/songs` in `routes/playlists.py` reads `song_id` and `added_by` from the JSON body and, after validating both are present, calls `notification_service.add_to_playlist(playlist_id, song_id, added_by)`. That function looks up the `Song`, `User`, and `Playlist` records (raising `ValueError` if any are missing), then appends the song to `playlist.songs` — a many-to-many relationship backed by the `playlist_entries` association table — but only if the song isn't already in the playlist. It then checks `song.shared_by` against the adder: if the person adding the song isn't the same person who originally shared it, it calls `create_notification()`, which inserts a new `Notification` row (`user_id`, `notification_type="song_added_to_playlist"`, `body`) for the original sharer. Notably, this notification check runs independent of whether the song was actually newly added to the playlist — calling the endpoint again for a song that's already in the playlist still creates another notification as long as the adder isn't the original sharer.

### app.py

**Purpose:** Serves as the Flask application factory and database initialization hub.

**Key Responsibilities:**

- Creates and configures the Flask application with the `create_app()` factory function
- Initializes SQLAlchemy database instance and connects it to the Flask app
- Configures database URI (SQLite by default, or from DATABASE_URL environment variable)
- Registers all route blueprints (songs, playlists, users, feed) with URL prefixes
- Initializes database tables on startup

**Entry Point:** When run directly, starts a development server with debug mode enabled.

### models.py

#### User

Model Name: User

Model Attributes:

| Attribute        | Type     | Attribute Description                               |
| ---------------- | -------- | --------------------------------------------------- |
| id               | String   | Unique identifier for the user                      |
| username         | String   | Display name used for the account                   |
| email            | String   | Email address for the account                       |
| listening_streak | Integer  | Current streak of listening activity                |
| last_listened_at | DateTime | Timestamp of the user’s most recent listening event |
| created_at       | DateTime | Timestamp when the account was created              |

Model Relationships: Has many playlists, ratings, listening events, notifications, and shared songs; also has many friends through a self-referential relationship.

Model Associations: Connected to other users through the friendships association table and to playlists through ownership/creation.

#### Tag

Model Name: Tag

Model Attributes:

| Attribute | Type   | Attribute Description                  |
| --------- | ------ | -------------------------------------- |
| id        | String | Unique identifier for the tag          |
| name      | String | Name of the tag used to describe songs |

Model Relationships: Represents a label that can be attached to multiple songs.

Model Associations: Linked to songs through the song_tags association table.

#### Song

Model Name: Song

Model Attributes:

| Attribute  | Type     | Attribute Description               |
| ---------- | -------- | ----------------------------------- |
| id         | String   | Unique identifier for the song      |
| title      | String   | Song title                          |
| artist     | String   | Song artist                         |
| album      | String   | Album name, if available            |
| genre      | String   | Genre category                      |
| shared_by  | String   | ID of the user who shared the song  |
| shared_at  | DateTime | Timestamp when the song was shared  |
| share_note | Text     | Optional note attached to the share |

Model Relationships: Has many ratings and listening events; can have many tags.

Model Associations: Associated with users through sharing, with tags through song_tags, and with playlists through playlist_entries.

#### ListeningEvent

Model Name: ListeningEvent

Model Attributes:

| Attribute   | Type     | Attribute Description                     |
| ----------- | -------- | ----------------------------------------- |
| id          | String   | Unique identifier for the listening event |
| user_id     | String   | ID of the user who listened               |
| song_id     | String   | ID of the song that was listened to       |
| listened_at | DateTime | Timestamp of the listening event          |

Model Relationships: Tracks the interaction between a user and a song.

Model Associations: Connects a user to a song through foreign keys.

#### Rating

Model Name: Rating

Model Attributes:

| Attribute | Type     | Attribute Description                   |
| --------- | -------- | --------------------------------------- |
| id        | String   | Unique identifier for the rating        |
| user_id   | String   | ID of the user who submitted the rating |
| song_id   | String   | ID of the rated song                    |
| score     | Integer  | Numeric rating value                    |
| rated_at  | DateTime | Timestamp when the rating was created   |

Model Relationships: Represents a user’s evaluation of a specific song.

Model Associations: Links a user and a song through a unique rating entry.

#### Playlist

Model Name: Playlist

Model Attributes:

| Attribute        | Type     | Attribute Description                     |
| ---------------- | -------- | ----------------------------------------- |
| id               | String   | Unique identifier for the playlist        |
| name             | String   | Name of the playlist                      |
| created_by       | String   | ID of the user who created it             |
| created_at       | DateTime | Timestamp when the playlist was created   |
| is_collaborative | Boolean  | Whether the playlist allows collaboration |

Model Relationships: Contains many songs and is owned by a creator user.

Model Associations: Uses the playlist_entries association table to connect songs to playlists with ordering information.

#### Notification

Model Name: Notification

Model Attributes:

| Attribute         | Type     | Attribute Description                       |
| ----------------- | -------- | ------------------------------------------- |
| id                | String   | Unique identifier for the notification      |
| user_id           | String   | ID of the recipient user                    |
| notification_type | String   | Type of notification                        |
| body              | Text     | Notification message content                |
| created_at        | DateTime | Timestamp when the notification was created |
| read              | Boolean  | Whether the notification has been read      |

Model Relationships: Represents messages sent to a specific user.

Model Associations: Connects a notification to its recipient through the user_id foreign key.

### Patterns

Pattern I noticed: every service function that can fail signals it the same way — a plain `ValueError` — and every route wraps its service call in `except ValueError as e: return jsonify({"error": str(e)}), <code>`. That exception type is the entire seam between the two layers: services never know about HTTP status codes, and routes never contain business rules. Interestingly, the specific status code isn't derived from the exception itself — each route decides it independently. `POST /playlists/` returns `400` when `create_playlist` raises `ValueError` because the creator doesn't exist, while `GET /playlists/<playlist_id>` returns `404` when `get_playlist` raises the exact same exception type because the playlist doesn't exist. So "not found" vs. "bad request" is a per-route judgment call, not something the exception type encodes. (The one place this pattern breaks down is `GET /users/<user_id>` in `routes/users.py`, which queries the `User` model directly instead of going through a service function, so it hardcodes its own 404 response instead of raising and catching `ValueError`.)

## Root Cause Analysis

The fixes described below are proposed, not yet applied to the codebase — each entry states what the change would be and how to confirm it, to be carried out and committed separately per issue.

#### Issue 1: My listening streak keeps resetting

- **How I reproduced it:** The streak keeps resetting because there is a conditional when checking to see if music has been listened to on the 6th day of the week (would be sunday because index 0 is counted) that makes the streak equal to 1, instead of incrementing the streak by one. I was able to reproduce this by updating a User's listening streak, and instead of it being incremented by one, it got resetted back to 1. The user is Darius in the Seed Data.

- **How I found the root cause:** I took a look at streak_service.py because that's what was recommended in the bug description found in the README. I went through each of the methods and realized that one of the conditionals stuck out like a sore thumb. There is a conditional that makes reference to a specific day of the week that isn't mentioned in the docstring. The docstring really mentions starting the streak at 1, incrementing by 1, or resetting back to 1, but no mention of the 7th weekday (is stated as 6th because index 0 is counted).

- **The root cause:** In `update_listening_streak()` ([streak_service.py:73](services/streak_service.py#L73)), the branch that should increment the streak when `days_since_last == 1` also requires `today.weekday() != 6`. Python's `date.weekday()` returns `6` for Sunday, so this silently disables the increment specifically when the _current_ day is a Sunday — even though the user listened on consecutive days — and falls through to the `else` branch, which resets the streak to `1`.

- **My fix and side-effect check:** Remove the `and today.weekday() != 6` clause, leaving `elif days_since_last == 1:`, matching the docstring's rule exactly. To confirm: re-run `pytest tests/test_streaks.py` and expect all 5 tests to pass. Check that the untouched branches still behave correctly — same-day repeat listens still no-op (`days_since_last == 0`), and skipping a day still resets to 1 (`else` branch) — since neither references `weekday()`. Finally, repeat the live reproduction (reseed, hit `/songs/<id>/listen` for darius) and confirm the streak becomes `4`, not `1`.

#### Issue 2: Friends Listening Now shows people from yesterday

- **How I reproduced it:** No test file covers `feed_service.py`, so I reproduced this live. Seeded the database, then queried the raw `ListeningEvent` rows and found kenji's friend nova has exactly one event within the service's lookback window: one from roughly 2.3 hours before the request. Calling `GET /feed/<kenji_id>/listening-now` returned nova in the feed with that 2+ hour-old `listened_at` timestamp, presented identically to a genuinely live listen.

- **How I found the root cause:** In the README, the issue notes the feed_service.py file as the affected service to check for issue 2. Considering the second issue mentions "Friends listening now, which is similar to the aptly-named "get_friends_listening_now" method, I ruled out that the "get_activity_feed" method would be the source of the issue. After re-reading the "get_friends_listening_now" and getting an idea of the issue, I asked Claude if my hunch was right (the window of listening events - named RECENT_THRESHOLD - was too big).

- **The root cause:** `get_friends_listening_now()` treats any friend listening event with a timestamp within the last 24 hours as "currently listening." A 24-hour window is far too wide for a real-time presence feature — it includes listens from many hours ago, which, depending on time of day, land on the previous calendar day, matching the "shows people from yesterday" complaint.

- **My fix and side-effect check:** Changed `RECENT_THRESHOLD` from `timedelta(hours=24)` to `timedelta(minutes=30)` ([feed_service.py:13](services/feed_service.py#L13)). I initially considered 15 minutes, but checked that against `seed_data.py`'s own "recent events" — they're seeded at 10, 15, and 20 minutes old and are commented as ones that "should appear in 'listening now'" — so 15 minutes would have cut off that third event. 30 minutes is the value the seed data's own comment names, and it clears all three recent events while still excluding the earliest "older" event (2 hours old). To confirm: `get_activity_feed()` doesn't use `RECENT_THRESHOLD` at all (it just takes the most recent N events, unfiltered by recency), so its output is unaffected. Reseeded and re-ran the live reproduction: `GET /feed/<kenji_id>/listening-now` no longer returns nova (her event is 2 hours old), and `GET /feed/<nova_id>/listening-now` still returns darius, simone, and kenji (their 10/15/20-minute-old events). Full test suite re-run afterward: 11 passing, 2 failing — both pre-existing failures from Issue 5's unfixed `songs[:-1]` bug, unrelated to this change.

#### Issue 3: The same song keeps showing up twice in search

- **How I reproduced it:**
  If you use raw SQL, like so:

SELECT song.id, song.title FROM song
LEFT OUTER JOIN song_tags ON song.id = song_tags.song_id
WHERE song.title LIKE '%Crown Heights%'

You will get the same song back 3 times because of the three different tags (rap, hip-hop, boom bap). This doesn't pop up when you use the app itself because under the hood, `db.session.query(Song)` calls unique() which stops a song from showing up more than once (thanks to SQLAlchemy).

- **How I found the root cause:** Traced `routes/songs.py`'s `search()` into `services/search_service.search_songs()`, and noticed the `.outerjoin(song_tags, Song.id == song_tags.c.song_id)` with no `.distinct()` or `.group_by()` afterward — the classic one-to-many join fan-out shape. Confirmed the fan-out at the SQL level as described above, which is what made me confident this is the intended defect rather than a red herring, despite it not reproducing symptomatically right now.

- **The root cause:** `search_songs()` joins `Song` to `song_tags` to make tag data available, but never deduplicates the resulting `Song` rows. A `LEFT OUTER JOIN` against a table with a one-to-many relationship (one song → multiple tag rows) produces one result row per matching tag row, so a song with 3 tags produces 3 rows for every place it matches the search filter. It's currently invisible because SQLAlchemy's legacy `Query` object happens to auto-apply `.unique()` for this exact join shape — a behavior the code doesn't rely on explicitly, so it would resurface immediately if the query were ever rewritten with the newer `session.execute(select(...))` style, which does not auto-dedupe.

- **My fix and side-effect check:** Add `.distinct()` to the query in `search_songs()`, so correctness doesn't depend on undocumented ORM behavior. To confirm: re-run `pytest tests/test_search.py` — should still be 5/5 passing, but now for a structural reason rather than an implementation-detail coincidence. Verify each song's `tags` list in the response is still complete (`.distinct()` dedupes `Song` rows, not the joined tag columns, so `to_dict()`'s own tag lookup is unaffected). Confirm 0-tag and 1-tag songs are unchanged (already covered by the existing tests).

#### Issue 4: I got notified when a friend added my song to a playlist but not when they rated it

- **How I reproduced it:** No test file covers this path. Live test: recorded the notification count (1, from seed data), then had darius (not the song's sharer) call `POST /songs/<song_id>/rate` on a song nova shared. The rating succeeded (`201`), but the notification count stayed at 1 — no new notification was created for nova.
- **How I found the root cause:** Traced `routes/songs.py`'s `rate()` into `services/notification_service.rate_song()`. Since `add_to_playlist()` in the same file is the working example of "notify the sharer when a friend interacts with their song," I compared the two functions side by side. The confidence moment: `add_to_playlist()` ends with an explicit `if song.shared_by != added_by_user_id: create_notification(...)` block; `rate_song()` has no equivalent block anywhere — it saves the `Rating` and returns.
- **The root cause:** `rate_song()` is missing the notification step entirely. It validates the score, creates or updates the `Rating` row, commits, and returns — there's no call to `create_notification()` for the song's sharer, unlike the otherwise-analogous `add_to_playlist()`.
- **My fix and side-effect check:** After saving the rating in `rate_song()`, add a `create_notification(...)` call mirroring `add_to_playlist()`'s pattern — only firing when `song.shared_by != user_id`, with a body describing the rating (e.g. including the score). To confirm: verify a friend rating a shared song now creates a notification for the sharer; verify the sharer rating their own song still creates none (matching the self-add exemption already in `add_to_playlist()`); and decide (a product question, not just a mechanical one) whether re-rating the same song on a later call should notify again each time or only on the first rating, since the `existing` vs. new-`Rating` branch currently treats both cases identically.

#### Issue 5: The last song in a playlist never shows up

- **How I reproduced it:** `pytest tests/test_playlists.py -v` fails two tests: `test_playlist_returns_all_songs` (a 5-song playlist returns 4) and `test_playlist_returns_songs_in_order` (the last title, "Track 5", is missing). Live test: a seeded playlist with 7 songs in the `playlist_entries` table returns only 6 from `GET /playlists/<playlist_id>/songs` — the 7th, last-positioned song is missing every time.
- **How I found the root cause:** Traced `routes/playlists.py`'s `get_songs()` into `services/playlist_service.get_playlist_songs()`. Read the function's own docstring, which explicitly states "Note: This function returns all songs in the playlist," then read the implementation, which ends with `return [song.to_dict() for song in songs[:-1]]`. The confidence moment was seeing the docstring's own claim directly contradicted by the last line of the function it documents — `songs[:-1]` slices off the final element of an already-correctly-ordered, fully-fetched list.
- **The root cause:** After querying all of a playlist's songs in position order, `get_playlist_songs()` returns `songs[:-1]` instead of `songs`, unconditionally dropping the last song in the playlist regardless of playlist size.
- **My fix and side-effect check:** Change `return [song.to_dict() for song in songs[:-1]]` to `return [song.to_dict() for song in songs]`. To confirm: re-run `pytest tests/test_playlists.py` — both previously-failing tests should pass, and `test_empty_playlist_returns_empty_list` should still pass (an empty list has no last element to worry about, but worth confirming `songs[:-1]` on `[]` — which is `[]` — wasn't accidentally the only thing keeping that test green). Also check a single-song playlist specifically: with the current `[:-1]` bug, a 1-song playlist would silently return `[]` — a sharper version of the same defect worth spot-checking isn't hiding a second issue. Reseed and re-`curl` `GET /playlists/<id>/songs` to confirm all 7 songs return in position order.
