# Plan: recognise Plex NFO series IDs (upstream issue #3879)

## What Sean asked for

Fix Kometa-Team/Kometa#3879, "Support Plex NFO Series agent GUIDs", the first job for the Kometa team.

## What we found

- Plex's new NFO agent labels each show with an ID such as `tv.plex.agents.nfo.series://show/tvdb_113561`. Kometa doesn't understand this label, so it can't tell which show it is. It logs `Mapping Error | tv.plex.agents.nfo.series://show/... | Agent series not supported` and skips that show in collections and overlays.
- The same problem for movies was fixed two weeks ago in upstream PR #3843. That fix taught one function, `scan_guid` in `modules/convert.py`, to read the TMDb or IMDb number out of the movie label, and added tests in `tests/test_convert.py` and a line in `CHANGELOG.md`. Shows need the same treatment. The rest of the mapping code already handles TVDb, TMDb and IMDb numbers for shows.
- Plex's own forum posts show show labels with a TVDb number (`show/tvdb_78874`). Seasons and episodes use the same prefix with `season/` or `episode/`, but Kometa only looks at show labels, so those stay as they are. The NFO agent can also use TMDb or IMDb numbers. We found no real sample of a show label with a TMDb number, but it is the same format as the movie one.
- Nobody else has an open PR or branch for this. The issue is assigned to a Kometa maintainer but has no comments yet.

## Contribution rules (from the repo's guidelines and CONTRIBUTING)

- The fix PR targets `nightly`. PRs to `master` or `develop` are refused automatically.
- One logical change per PR, kept small and focused, and rebased on `nightly`.
- Add tests where feasible, and keep black, isort, flake8, pyright (baseline check) and the test suite passing.
- Add a `CHANGELOG.md` entry under `## [Unreleased]` → `### Fixed`, one user-facing sentence ending with the issue number.
- Fill in every section of the PR template.
- Avoid force-pushing a PR branch once it has had review comments.
- No CLA or sign-off rule upstream. Our commits are signed off anyway.

## What we'll change

1. **`modules/convert.py`, function `scan_guid`:** add a block next to the movie one. For labels that start `tv.plex.agents.nfo.series://show/`, read a `tvdb_<number>`, `tmdb_<number>` or `imdb_tt<number>` ID and hand it on as a TVDb, TMDb or IMDb ID. TVDb and TMDb numbers must be above zero. IMDb IDs must look like `tt` followed by digits, the same rule the movie fix uses. Anything else falls through to today's behaviour, so nothing that works now changes.
   *Checked by:* the new tests below.
2. **`tests/test_convert.py`:** add tests in the same style as the movie ones:
   - each of the three ID types is read correctly;
   - bad or local IDs (`tvdb_0`, `tvdb_bad`, `tvdb_123/extra`, `imdb_bad`, `local_123`, empty) are not accepted;
   - season and episode labels are left alone;
   - three mapping tests, one per ID type, for a show in a TV library: the label is turned into the right show IDs and saved to Kometa's cache. As in the movie fix, the outside sites (TVDb, TMDb, IMDb) are simulated in these tests, not contacted. The IMDb test also covers a show where no TVDb number can be found, which Kometa already accepts for other labels.
   *Checked by:* the full test suite (4,930 tests pass on `nightly` today).
3. **`CHANGELOG.md`:** one `### Fixed` line, e.g. "Recognize TVDb, TMDb and IMDb identifiers embedded in Plex NFO Series agent GUIDs when mapping TV and anime libraries. #3879".
   *Checked by:* review.
4. **The upstream PR text:** a filled-in PR template for Sean to post, written so he can explain the change.

Here we run the checks Kometa's CI runs: black, isort, flake8, the full test suite with its coverage minimum, the pyright baseline check, bandit, and the `kometa.py --help` smoke check, on Python 3.12 and 3.13 (Linux). Kometa's CI also runs the tests on Windows. That only happens on GitHub once the upstream PR is open, and the work isn't finished until those checks pass too.

## Delivery

After you approve the finished work on this PR, the team puts the fix commits (without this plan and the notes) on a branch of your fork, `fix-nfo-series-guids-3879`, based on the latest `nightly`. Before you're told it's ready, the team checks that branch holds exactly the approved change and re-runs the tests and checks on it, in case `nightly` has moved. You then open the PR to Kometa-Team/Kometa `nightly` with the text we give you. This PR on your fork is only for your two approvals. It's never merged, and Kometa never sees it.

## Who's on it

Researcher (done), Engineer, QA, Security Officer, Auditor, plan check and plain-English check.

## Risks

- A Kometa maintainer is assigned to the issue and might fix it first. If that happens before we're done, we stop.
- We can't test against a real Plex server with the NFO agent. We rely on Plex's published examples and the mocked tests, the same as the movie fix.
- The TMDb case for shows goes slightly beyond the issue text. It's three lines and covered by tests. If the maintainers would rather keep it to TVDb and IMDb, it's easy to drop.

## Out of scope

- Season and episode labels: Kometa doesn't look these up by ID.
- NFO labels with a bare local number (no `tvdb_`/`tmdb_`/`imdb_`), which can't be mapped to any outside site. They stay unsupported, as for movies.
- Any tidy-up of the movie code from #3843.
