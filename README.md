# replyneed — Which texts need a person, in reading order

**Paid tool: $49 once (plus sales tax where it applies), then a private GitHub repository you're invited to, with every update. Buy: https://dominares.org/tools/replyneed**

Commercial license v1.3 — see https://dominares.org/license · Support: support@dominares.org

replyneed decides which inbound texts really need a human reply (a thumbs-up, a shared link and a laughing emoji do not) and ranks the rest as said yes, upset, money or a question, with no model call.

## Get it

Buy replyneed at https://dominares.org/tools/replyneed: one payment of $49, plus sales tax where it applies, and every future update. After checkout you are invited to a private GitHub repository with the source and every version published to it. The repository includes the Python package and the replyneed command (check, rank, triage), an invented sample inbox, and the full test suite.

Requirements: Python 3.10 or newer, standard library only.

This page documents version 1.0.1. Install commands and file paths in the documentation below run inside the private repository you are invited to; the source is not in this repository.

## Documentation

### What it does

- **Decides whether anyone is waiting** (`needs_reply`). A message can be filed away only for a shape the package can name:
  - `reaction`: a phone reaction to one of your messages (twelve spellings are tested, among them `Liked "..."`, `Reacted 😂 to "..."` and the emoji form), or a message with no letter or digit in it at all, such as a bare emoji;
  - `link`: a shared link with fewer than four letters or digits around it;
  - `no_question`: your own reader (a person, or a language model) found nothing asked or requested. **You pass this verdict in; the package never produces it.** Counting words cannot tell "thanks, got it" from "k, 2pm?".
- **A yes can never be filed away.** "Yes", "Ok", "Sure", "Im in", "Count me in", "send me the details", "I'd like two" and a few dozen phrases like them veto every rule above, because a yes looks exactly like noise. A refusal ("no thanks", "not interested", "remove me") is checked first, so it is not mistaken for one. It leans wide on purpose: a false "this needs a person" costs a glance, a missed yes loses a customer.
- **Ranks what is left** (`priority_of`, `rank_of`, `classify`), most urgent first: said yes, upset with you, about money, asked a question, told you something. The order is a plain tuple (`PRIORITIES`), and a rank is a sort order, never a filter: nothing is hidden by it. Each priority carries a label and a one-line reason that can be shown on the row.
- **Reads the sender's words only.** A reaction collapses to an empty string, so your own "sign up" or "$50" in the quoted copy is never ranked as theirs. Shared links are taken out before words are counted, so the `?` in a video link's address is not a question and a bare link is ranked `Told us something`.
- **Keeps anger and refunds honest.** Anger counts as "upset with you" only when the message also says you, your, us, our, we, or a name you supply, so someone furious about traffic is not badged as a complaint about you. A refund, charge, receipt or dispute outranks an apparent yes, so "i want a refund for my $50 deposit" is not labelled "Said yes".
- **Sorts an inbox in one call** (`rank_messages`): needs-reply first, then by urgency, ties in arrival order, nothing dropped, your own fields (such as an id) carried through. The result writes `needs_reply`, `skip_reason`, `skip_label`, `priority`, `label`, `why` and `rank` into each message, so a field of yours with one of those names is overwritten. `no_question` counts only as the real `true`; the string `"false"` is no verdict.
- **Keeps a folder per thread** (`TriageStore`): Needs a reply, Ignored, Replied, in one JSON file. A reply sent moves a thread to Replied; a new text reopens it; a thread a person ignored stays ignored. Absence means Needs a reply, an unreadable file reads as Needs a reply, and a write to an unreadable file (or into a folder that cannot be written) raises `StoreError` and leaves the file as it was.

### The command

    replyneed check "Im in"                      one text: does it need a reply, how urgent
    replyneed check - < text.txt                 the text from standard input
    replyneed check "Thanks, got it" --no-question
    replyneed rank examples/inbox.jsonl          a whole inbox in reading order (--json for JSON)
    replyneed triage store.json set "t-100|a" ignored
    replyneed triage store.json sent "t-100|a"   a reply went out
    replyneed triage store.json inbound "t-100|a"  a new text arrived
    replyneed triage store.json show "t-100|a" "t-100|"   one conversation, two spellings
    replyneed triage store.json list

An inbox file holds one message per line. A line that starts with `{` is a JSON object with
`body` and, optionally, `id`, `no_question` (true when your reader found nothing asked) and
`reason` (the reason you stored last time); any other line is the text itself. A file that
starts with `[` is one JSON array of the same. See `examples/inbox.jsonl`.

`check` and `rank` also take `--yes-phrase`, `--decline-phrase`, `--money-word` and
`--name`, each repeatable. Exit status is 0, or 2 when the input cannot be read.

### From Python

```python
from replyneed import assess, rank_messages, needs_reply, Rules

print(needs_reply("\U0001F44D"))                       # False: nobody is waiting
print(needs_reply("Thanks, got it", no_question=True))  # False: your reader said so
print(needs_reply("Im in", no_question=True))           # True: a yes is never filed away

result = assess("i want a refund for my $50 deposit")
print(result["priority"], result["rank"], result["needs_reply"])   # money 2 True

inbox = [{"id": "a", "body": "what time do you open"}, {"id": "b", "body": "Sure"}]
print([m["id"] for m in rank_messages(inbox)])          # ['b', 'a']
```

```python
from replyneed import Rules, TriageStore, effective_folder

store = TriageStore("triage.json")
store.set("t-100|a", "ignored", actor="sam")   # a person's call
store.note_inbound("t-100|a")                   # still ignored: that is a judgement about the sender
store.note_sent("t-100|a")                      # a reply went out: Replied
store.note_inbound("t-100|a")                   # they wrote back: reopened
print(store.get("t-100|a"))                     # needs_reply

# What to SHOW: a folder a person chose wins; otherwise the rules decide. Nothing is written.
print(effective_folder(store.folders(), "t-200|a", "\U0001F44D"))   # ignored
```

### Options

Every list ships with generic English defaults. `Rules` adds to them and replaces nothing:

| Field | Adds |
| --- | --- |
| `yes_phrases` | phrases counted as saying yes anywhere in a message ("book it") |
| `decline_phrases` | phrases counted as a refusal, checked before any yes |
| `money_words` | words counted as being about money ("tab", "deposit") |
| `names` | what an angry sender calls you, so anger that uses it ranks as upset |

A phrase you add is matched on whole words after lower-casing and removing punctuation and
apostrophes. Pass the rules as `rules=` to `needs_reply`, `skip_reason`, `assess`,
`rank_messages`, `priority_of`, `classify`, `is_affirmative` and the rest.

### Limits

- **English only.** The word lists are English. Other languages need `Rules`, and the reaction
  formats are the English and emoji spellings listed above.
- **The "nothing was asked" verdict is yours.** With none supplied, every message that is not a
  reaction, a bare link or a bare emoji needs a reply, including "no thanks" and "thanks!".
- **It is word matching, not understanding.** "yes?" and "ready?" are read as a yes. "you would
  not believe how ridiculous the traffic was" is read as upset, because it says "you" next to
  an angry word. "what a mess" and "Can't make it, sorry" are read as questions, and "I want to cancel" is read as a yes (through "i want"). Each costs a row sorted one place
  wrong, never a hidden message.
- **The ranking order is a choice**: a yes first suits customers, invitees and buyers. If your
  inbox is mostly complaints, sort on `priority_of` yourself.
- **Only the first 1,200 characters** of a message are checked for a yes, a refusal or a phrase you add. A yes that starts later in a very long message is not seen.
- **The default phrases match as plain text**, so "I need directions" counts as a yes through "i need". Leaning wide is deliberate; add `decline_phrases` for the cases that hurt.
- **Links** are `http://`, `https://` and `www.` addresses only.
- **The triage file** is for one machine: writes are atomic and use a lock file on systems with
  `fcntl`, but it is not a shared database.
- It does not send, receive or store texts, and it does not call any service.

---

Docs mirror of the private repository Dominares-Tools/replyneed; updated with each release.
