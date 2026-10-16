# Changelog

## 1.0.1

Fixes.

- A shared link's address is no longer ranked: the `?` that starts a query string is not a
  question, and a word in the address is not about money. A bare link is `Told us something`.
- A triage store file that is not UTF-8 text reads as Needs a reply and refuses writes with
  `StoreError`, as any other unreadable file does (it used to raise `UnicodeDecodeError`).
- A write into a folder that cannot be written raises `StoreError` instead of `PermissionError`.
- `note_inbound` reads and reopens under one lock, so an ignore filed by another process in
  between is no longer erased.
- `rank_messages` takes a message's `no_question` only when it is the real boolean `True`; a
  string such as `"false"` no longer files a question away.
- The README names the keys `rank_messages` writes, and two more limits.
- The README tests also pass on a README with a header and footer added around it.

## 1.0.0

First release.

- `needs_reply`, `skip_reason`, `cheap_reason`, `reason_for_decline`: decide whether a person is
  waiting on an inbound text. Phone reactions, bare links and bare emoji are filed away by rule; a
  yes can never be; the "nothing was asked" verdict is passed in by the caller.
- `priority_of`, `classify`, `rank_of`, `describe`: rank what needs a reply (said yes, upset,
  money, question, told you something) with a label and a one-line reason.
- `assess` and `rank_messages`: one call for one message or a whole inbox.
- `TriageStore`: a folder per thread (needs a reply, ignored, replied) in one JSON file, with
  the two automatic moves, multi-key threads, atomic writes and a refusal to write over an
  unreadable file. `effective_folder` shows what the rules alone would say.
- `Rules`: add yes phrases, refusal phrases, money words and your own name.
- The `replyneed` command (`check`, `rank`, `triage`) and `python -m replyneed`.
