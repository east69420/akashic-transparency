# Akashic access-log public record

Every person who uses Akashic has an access log: a list, kept for them, of every time someone who runs
Akashic reached their instance (with the reason they gave) and every time they confirmed an action with
their passkey. Each entry carries a hash of the one before it, so an entry cannot be changed or removed
without the list showing it.

Once a day, at 00:10 UTC, the Akashic Hub writes one file here, named by its date. It holds one line
("leaf") per person, made from:

- a random salt that only that person's account holds, so nobody can tell whose leaf is whose or compare
  people;
- the number of their newest entry and that entry's hash.

A person with an empty log still has a leaf, so nothing can be learned from how many leaves there are.
The file holds no names, no counts per person and no contents of anyone's log.

## What it gives you

Once a day's file is here, nobody, including the people who run the Hub, can change an entry in your log
from before that day without the check below failing. Akashic does not control this repository's history:
GitHub keeps it.

## What it does not give you

It does not stop an access that was never written down, or a false entry written on the day itself. It
records the state of each log once a day, not every entry as it happens.

## How to check yours

Your Akashic app does this for you under Settings, Privacy, "Who has accessed your record", and shows
"Matches the public record of <date>". That app is served by the Hub, so for a check that does not rely
on Akashic at all, open "Check it yourself" there and run, on your own computer:

    python3 verify.py --salt <your salt> --seq <number> --hash <hash> --date <date>

`verify.py` is a single file with no other dependency:
https://akashicdesktop.com/transparency/verify.py

## The format

    leaf = sha256("akashic-access-anchor-v1" + 0x00 + salt (32 bytes) + seq (8 bytes, big-endian) + hash (32 bytes))
    root = sha256("akashic-access-root-v1" + 0x00 + previous root (32 bytes) + every leaf, sorted ascending)

Each file names the previous day's root (`prev_root`), so the files form a chain; the first file's
previous root is 32 zero bytes. `operator_log_sha256` is the hash of the Hub's own operator access log at
that moment.
