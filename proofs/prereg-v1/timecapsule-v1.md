# Time-capsule scoring rules v1 (DRAFT for stamping, 18 Nov 2026)

**What this is.** These are the rules for scoring the first paid time-capsule run, written before any AI call has been made and before any result has been seen.
- **Spiro's OK to stamp them:** `decisions/2026-11-18-timecapsule-scoring.md`.
- **How it becomes final:** the lead copies this file to `prereg/timecapsule-v1.md`, and `research/timecapsule_scoring_rules_v1.json` to `prereg/timecapsule-v1.json`, byte for byte. Both are stamped with OpenTimestamps; the two `.ots` proofs (`prereg/timecapsule-v1.json.ots`, `prereg/timecapsule-v1.md.ots`) are committed next to them; publication follows `decisions/2026-11-18-timecapsule-scoring.md` (the harness checks the commit and the proofs, not the publication).
- **No edits after stamping.** A change means a new version (v2).
- **What the harness enforces.** The run and the probe refuse to start unless:
  - `prereg/timecapsule-v1.json`, `prereg/timecapsule-v1.md` and both `.ots` proofs exist, are committed, and are unchanged since the last commit;
  - the `.md` is the rules file the `.json` names (by its sha256);
  - every pinned value below matches.
  - There is no fallback to this research draft.
- **Background:** `research/timecapsule_testbed_v1.md`.

**In one line for Spiro.** We ask our frozen AI forecaster about 326 questions that opened after its training cutoff, using only news from before each question's test date. We then check whether its submitted numbers beat the crowd's on the 129 questions whose deadlines had clearly passed. "Inconclusive" is the most likely honest answer, because the submitted number is 80% crowd by design.

---

## a. The sample is pinned

**Pinned inputs** (sha256, unless noted). These are also in `research/timecapsule_scoring_rules_v1.json`, which becomes the file the harness checks.

| What | Value |
|---|---|
| `archive_v1.jsonl` (326 questions, 1,015 as-of points) | `235922044c5e91a430a9ec08be4e4957a02088f5a279b4bca87b3bb6d7d7620d` |
| `pool_v1.jsonl` (base-rate pool: 775 pre-cutoff + 343 post) | `0f3da738f4870fb5cec8ff775623415d3c31ff615388afb186d1375ae46597af` |
| Archive build | Reference time 2026-11-18T04:03:59Z (the first build's, so the windows and the 14-day lag are unchanged); rebuilt 2026-11-18T08:21:13Z from commit `39eb2102cf81d8a15f6b895d66dfef0e62ba8a27` (clean tree), after the purges and the manual reviews in §b. Building twice gave the same bytes. |
| `config/timecapsule_v1.yaml` (filters, groups, the manual-review list) | `7c1b1b362170cecf8819e99808e6e12c38516df98b3d87ebff924bab4dac5395` |
| `config/models.yaml` | `2bc6700a0ee778550f50f30a84cca8d9ef2f9db450461a2087321feffc5b555b` |
| The `forecast` model it names | `claude-sonnet-5-5` |
| `config/budget.yaml` (the spend caps and prices) | `ae1f218e881b58bfb323ab9c15412c272fefc2fa5adaf631f14766dc68983b30` |
| `src/firm/budget/spend.py` (the spend guard that enforces the caps) | `dd11c80c538008847aafb0855635f6c32b4eb70f24a18ff99612933594187d9b` |
| `src/firm/forecast/pipeline.py` | `6acb171608996ee29743ee01b4d7a02fdd4ace54e22440b2a42eab799b2e86da` (= prereg addendum 2) |
| Prompts (`forecast/prompts/*.md`) | `dc5e99344b6d78c8410986e8d556af90ae82ec7c7c5a0c7185e132b077d6b396` (= prereg addendum 2) |
| `src/firm/forecast/blend.py` | `ec98e9356c29f0bd198d9056b7df126e1ee76d89053fe539f524625115a1a28a` |
| `src/firm/forecast/llm.py` | `d404f94a69f3019c1bad903fa7ca891a7dcaa178a42d661ff779b64b065c1c98` |
| `src/firm/forecast/news.py` (cleans the news queries) | `ca6cbb4abcd679bbd2ff154e7086d97a24b115121b58367a1eae1a6a07142bce` |
| `src/firm/lab/timecapsule.py` (the harness and scorer) | `f41c4101d87455961e5ab179064af56421b148ee9903e0657d532920adfe234a` |
| `src/firm/lab/timecapsule_archive.py` | `8d381a621030a422729c4d01104cdb2b1ad05a19bf1a4ca1f639b7d9c43d3b1f` |
| `src/firm/lab/timecapsule_news.py` (the news time-lock) | `90e90398630a5f2c37a2e235ab87becb3a05cd9de7f2bf716619a7c28f489b77` |
| `src/firm/lab/timecapsule_baserates.py` | `0139e2ee20caa6045af9c73be07c976b2c2f7a0ed603dbfe37bd10a75e4526bd` |

The harness refuses to run if any pinned value differs or is missing, or if the git tree is dirty (a git failure counts as dirty). Pinning `config/models.yaml` and the model id means the `forecast` role can't be switched to another model after stamping; pinning `forecast/news.py`, `config/budget.yaml` and `budget/spend.py` means the news-query cleaning and the spend caps can't change either. **No rebuild of the archive after stamping.**

**The run:**
- Seed **7**. All 326 archive questions: `--max-questions 400` takes them all.
- **Earliest as-of date only**: 1 day after the question opened.
- One question each, in the seeded order written to `tasks.jsonl`.
- Model: Sonnet 5.5 through the frozen pipeline.
- Cap: `--max-usd 12`, exactly, in the `research` category. Expected cost is about $4.67.
- **Budget room first.** The harness refuses to start unless at least $12 is left this budget month (months run from the 10th, `config/budget.yaml`) both in `research` and overall (after the unspent reserves of other categories).
- **The probe first.** The harness refuses the pre-registered run unless a **finished** answer-key probe (§g) is on record. It counts only if its record carries the sha256 of `prereg/timecapsule-v1.json`; a probe recorded under another or no pins file does not count.
- **The key first.** Before the run (or the probe) writes anything, the harness checks the Anthropic key: it must be present (environment or Keychain) and accepted by Anthropic's model lookup (`GET /v1/models/{id}`), which is free: no completion is made and nothing is billed. A missing or rejected key stops it there, with nothing written and nothing used up. Credit or billing problems only show on the first real call; see "aborted before start" below.
- **News first:** `prefetch-news` saves the run's time-locked headlines before the paid run, with no AI (§c). The harness refuses the run unless saved headlines cover at least 90% of its tasks; a task counts as covered when the fetch succeeded (Google answered), even with zero headlines, and running `prefetch-news` again retries the others. The manifest records how many tasks had saved headlines.
- **One run.** The harness refuses a second pre-registered run, and refuses any exploratory run on this archive before it.
  - It looks for an earlier one in `runs/`, in an append-only run log (`~/lens-ledger-data/state/timecapsule_v1_runs.jsonl`) and among answered calls in the spend ledger (calls named `timecapsule-v1-prereg:*`; rows recorded at worst-case cost for a call that timed out or lost its connection are not answers). Any one is enough. A run that started and has no end record counts. This holds even if it received no AI answer: a crash can't show that nothing was answered.
  - **An invalid run still uses up the run.** Its numbers are reported, labelled invalid. A re-run would need a new stamped version (v2).
  - **Aborted before start.** A run that ended with its end record and in which no AI call returned an answer of any kind (no forecast, refusal, `max_tokens` stop or unusable answer) has shown nothing about the model. It is recorded as "aborted before start" (manifest, run log) and reported, and it does **not** use up the run. Examples: a credit or billing error on the first call, the spend guard refusing the first call, or an outage that outlasts the retries on every task until the run stops.
- **If anything stops the run early after its first answer** (a budget cap, an error retrying can't fix, 5 AI exceptions in a row (§c), or an interruption with Ctrl-C, which ends the run with its end record like any other stop):
  - the run is **invalid**;
  - the completed tasks are still scored and reported, labelled invalid: they are the first *N* in seeded order, and the report gives *N* and the total.
- **If the run crashes** (the process dies, so the manifest has no finish time), it is invalid, and it is scored on the tasks it actually recorded: *N* is the last task with a forecast, record or skip on file, in seeded order (never all tasks), and the report gives *N* and the total.
- **Sequence:** (1) copy and stamp; commit both files and both `.ots` proofs; (2) `prefetch-news` with the stamped code; (3) `probe` until one finishes; (4) `run --max-questions 400 --max-usd 12`, under `caffeinate -i` and detached from the Terminal; (5) the run writes `score.json`; (6) the hand review of §g.2 before reporting.

## b. Exclusions (settled before stamping; applied by the build pinned above)

**The exact patterns are the code.** The filters are the regular expressions and word lists in the pinned `timecapsule_archive.py` (`title_exclusion`, `rules_exclusion`, `non_english`, `excluded_group` and the patterns they use), the group list and the manual-review list in the pinned config. The prose below is a summary; where it and the code differ, the pinned code decides.

| Step | Questions |
|---|---|
| Manifold markets listed (created since 1 Jan 2026) | 26,366 |
| Removed: not a binary play-money market | 10,377 |
| Removed: not resolved exactly YES or NO | 8,169 |
| Removed: illiquid (< 15 unique traders or < M$1,000 traded, counted after resolution) | 5,533 |
| Removed: outside both windows (opened before 31 Jul 2026 and settled after 1 Jul 2026) | 472 |
| Removed: personal, meta or chance-device title ("Will I …", "will i …", "@user", "this market", "Do you like N% odds?", coin flips), the creator's own life in the rules text ("my girlfriend wants to know …"), or (manual review) a question decided by another Manifold market or poll, or by the creator's own actions | 431 |
| Removed: copy of a Kalshi or Polymarket market (45 by title, 20 by rules text: "resolves same as …", venue links) | 65 |
| Removed: a person's death, health or injury (title, rules text or manual review; see below) | 72 |
| Removed: anyone's private life or a sensitive trait (romance, sex, bodily functions, appearance, whereabouts, private bets, private tests, ethnicity, sexuality) | 18 |
| Removed: about a private individual (manual review) | 14 |
| Removed: about a minor or a school class | 15 |
| Removed: title not in English (no filter can read it) | 4 |
| Removed: about Manifold users: their money in the rules (e.g. a bot account repaying a loan "to me"), or (manual review) a user named or credited, or a user's own puzzle | 10 |
| Removed: excluded Manifold groups (personal, fun, meta, gambling, will-i, relationships, sex-and-love, romance, dating, commitment-devices …; list in the config) | 39 |
| Kept as stored Manifold markets | 1,147 |
| Duplicates: the same market reached two ways (API and ForecastBench), or the same words with the same numbers and an overlapping open period (keep one) | 55 |
| ForecastBench rows (Manifold / INFER) removed: 474 unresolved, 46 original close not yet passed, 46 because the Manifold copy was removed, 8 about a person's death or health, 5 personal/meta, 3 venue copies | 582 |
| Post-cutoff with no as-of date (settled within about 36 h of opening) | 13 |
| Post-cutoff whose rules, after update notes are cut, still describe their own resolution | 4 |

Every count of a "Removed" row is a reason code in `manifold_rejected.json`; a market is counted once, under the first check that removed it (title checks, then the manual review, then groups and rules text).

**A person's death, health or injury** (people-data rule: health is a sensitive trait, and a person's life is private):
- **Title** (`HEALTH_RX`): dying, death, passing ("her passing", "passed away"), being alive, living until a date, surviving, outliving, being killed or a killing ("kill", "kills", "killing"), being shot, a shooting or murder victim, poisoning, stabbing, "sprayed with", assassination, murder, suicide, execution, hospital, illness and named conditions, surgery, injury ("injure", "injury"), pregnancy, and a public appearance used as a stand-in for health.
- **Who counts.** A single person counts when named, or as one member of a small named group (a legislature, a crew), or as the one victim of a killing or injury ("Will agents kill another citizen by March?"). Counts of people do not.
- **What cancels a match, for its own words only** (`COUNT_SPANS_RX`): counts of people (a number with a people noun, "at least 100 people killed", "kill at least 5 more soldiers", death tolls, "deaths", "casualties", "fatalities", "measles cases", outbreaks, infections, vaccinations); public-health and job titles ("public health", "health secretary", "Secretary of State for Health", "health care", Medicare, Medicaid, the World Health Organization); "medical school / students / devices / research / billing / journal / examiner / marijuana / cannabis"; and fixed phrases: "injury time" and "stoppage time", "dead heat", "Die Linke / Grünen / Partei / Basis / Welt / Zeit / Hard", "pregnant women / people / workers", surviving a bend, drop, stress, crash or durability test, "kill the bill / deal / filibuster / switch / plan / program / project / feature / app / product / game / company / startup / rally / momentum / chances / vibe", and "poison pill". "Killed" on its own does not cancel.
- **Rules text** (the full description, before update notes are cut):
  - anywhere in it, **named health phrases** (`RULES_HEALTH_RX`): "his / her / <Name>'s health, illness, medical, diagnosis, mental", "health condition / status / issues / problems / scare / crisis / concerns / decline", "medical condition / emergency / leave / procedure / treatment / records", "diagnosed with", having surgery, "hospitalized", "admitted to hospital", "in hospital", "hospice", "in a coma", "intensive care", a suicide attempt, "took his/her own life", dementia, Alzheimer's, Parkinson's, a heart attack, cognitive decline, "terminally ill". Other health words in background text (for example a named player's injury in a sports question's background) are not matched by this check; the one-time review (below), which also read every description with keyword help, removed questions naming a person's injury or illness.
  - in **resolution sentences** only (`RESOLUTION_SENTENCE_RX`: a sentence with "resolve…", YES or NO in capitals, "qualif…" or "count(s)"), a person's life or death deciding the question (`RULES_DEATH_RX`): alive, proof or signs of life, "has died", "dies before / on / by / in office …", "passed away", "his/her death / passing / survival", "<Name>'s death / passing", being assassinated, executed, killed, murdered, shot, poisoned or incapacitated, an assassination, public appearances, life support, surgery, recovering from.
  - **Known gap:** this sentence test can miss a definitional sentence or a bullet with no resolution word in it (for example "That includes resignation, removal by the board, retirement, death …", or "incapacitation" in a list). That is acceptable for office-tenure questions, where death is one of several ways to leave office and not the subject.
- **Private life, minors, private individuals, language** (titles): romance, sex, relationships, bodily functions, appearance ("wear glasses / a dress / a jumpsuit"), a private test ("driving test", "sleep schedule") and private bets (`PRIVATE_LIFE_RX`, `PRIVATE_BETS_RX`); minors and school classes ("homework", "problem set", "field trip", "Spanish class", "Science Bowl", qualifying for USA(J)MO, USAPhO and similar, `MINOR_SCHOOL_RX`); titles where over a fifth of the letters are not Latin, or with at least two common words of another language and fewer English ones (`non_english`). Accents are folded before the word lists are applied ("Cöin Flip" reads as "Coin Flip").
- **One-time manual text reviews**, both before any AI call, both shown only the id, the title and the description: never the outcome, resolution, crowd value or close date. When unsure, the question went out.
  - **Titles (third review):** every stored title was read once (1,359 questions across the raw files, the archive and the pool), with the description where the title was unclear. It marked **101** questions (`named_person_health` 18, `private_individual` 24, `private_life` 11, `minor_or_school` 11, `non_english` 4, `personal_or_meta` 32, `manifold_users` 1); **80** of them are caught by the automatic filters above (some under another code, such as `excluded_group`), and the other **21** are listed in the config.
  - **Descriptions (fourth review):** every kept description (1,211 questions across the raw files, the archive and the pool) was searched with the keyword help in `src/firm/lab/timecapsule_review.py` (first-person life words, school, olympiad and minor words, social and stream links, "credit to", names before a redacted handle, "<Name>'s death / passing", injuries, links to other Manifold markets, and any first-person word left once the usual rules phrases are masked), and every hit was read. Text after an update note was shown only as snippets with result words masked, and a test checks that the helper reads no outcome, crowd, time or liquidity field. It added **32** questions: `personal_or_meta` 14, `private_individual` 5, `manifold_users` 5, `named_person_health` 4, `minor_or_school` 3, `private_life` 1. One of them is a ForecastBench-only row, dropped at build (counted in the ForecastBench "5 personal/meta").
  - **Every title and description (fifth review):** all 1,130 built rows were read in full, before any AI call, never with outcomes or crowd values. It added **12**: `private_life` 5, `manifold_users` 3, `private_individual` 2, `named_person_health` 1, `personal_or_meta` 1.
  - All **65** are listed in `config/timecapsule_v1.yaml` under `manual_review` as question id and reason code only: `personal_or_meta` 15, `named_person_health` 14, `private_individual` 14, `private_life` 9, `manifold_users` 8, `minor_or_school` 5.
  - **Kept on purpose:** questions about a public writer's posts, decided by his reaction to public AI news (`ZC6gdANScC`, `hh62cpzClz` in the cohort; `EQ688gUnZ5`, `zAIuQQcuyZ` outside it), even if he created them; 'the creator's own actions' means a private creator's own life or doings.
  - **Also kept:** pre-cutoff pool rows that mention a head of state's public killing in background text: public news about states, not a health record.
- **What is kept of every removed Manifold market:** only its id and reason code (`manifold_rejected.json`), plus an id-and-code marker in `manifold_details.jsonl` when it was removed by its groups or rules text, so it is never fetched again. Titles, texts and crowd histories of removed markets were purged from the stored files on 18 Nov:
  - after the third review: `manifold_markets.jsonl` 1,337 → 1,190; `manifold_details.jsonl` 1,341 → 1,257 (67 of them id-and-code markers); `manifold_crowd_v2.jsonl` 369 → 358; `manifold_rejected.json` 25,029 → 25,176 entries;
  - after the fourth review: `manifold_markets.jsonl` 1,190 → 1,159; `manifold_details.jsonl` 1,257 → 1,226 (still 67 markers); `manifold_crowd_v2.jsonl` 358 → 352; `manifold_rejected.json` 25,176 → 25,207 entries;
  - after the fifth review: `manifold_markets.jsonl` 1,159 → 1,147; `manifold_details.jsonl` 1,226 → 1,214 (still 67 markers); `manifold_crowd_v2.jsonl` 352 → 348; `manifold_rejected.json` 25,207 → 25,219 entries;
  - `manifold_open_close_times.jsonl` stayed at 13,791 rows (ids and times only; none of the removed ids is there).

**What survives:** **326 time-capsule questions**, all opened on or after **2026-07-31** (cutoff end 2026-07-01 plus 30 days), all Manifold.

**Also settled:**
- **Text shown to the model:**
  - user names that follow a pattern become `[user]`: `@handle` → `@[user]`; a display name next to a redacted X handle ("Name (@[user]) on X") → `[user]`; `@/<user>/<market>` → `@/[user]/<market>`; `manifold.markets/<user>/…` and `manifold.markets/embed/<user>/…` → `[user]`; referral codes `?r=…` → `r=[user]`. Names in plain prose can't be caught by a pattern; the manual review removed the questions where it found one;
  - share-tracking query parameters (`utm_*`, `si`, X's `s` and `t`, click ids, news gift and share codes) are dropped from links;
  - the build counts rows with any of these patterns left (0 user names and 0 tracking parameters in the pinned pool), and a test scans the built archive and pool;
  - descriptions are cut at the first update / edit / "AI summary of creator comment" note;
  - the close date is never shown for Manifold questions.
- **Sources left out:** Metaculus, both direct and via ForecastBench (no written permission for AI evaluation), and Kalshi / Polymarket (no data at all).
- **ForecastBench rows:** 20 are in the pool (13 INFER, 7 Manifold questions via ForecastBench), all settled before the cutoff, so they feed the base rates only. None is a time-capsule question: the 6 that opened after the margin and settled are duplicates of Manifold API rows (kept once, from the API), 51 straddle the cutoff (not used), and of the other 188 pre-cutoff rows, 38 are duplicate Manifold API rows (kept once, from the API) and 150 were open longer than any post-cutoff question could have been.

## c. The primary comparison

- **Measure:** the Brier score of `blend` minus the Brier score of `crowd`, averaged so that each question counts once.
  - `blend` is the probability the frozen pipeline would submit.
  - `crowd` is the Manifold market's chance at the as-of time.
- **Subset: the frozen deadline cohort**, i.e. archive rows with `deadline_passed = true`.
  - These are questions whose deadline, read from their **title** by the rules in `timecapsule_archive.title_deadline`, ended at least **14 days before the build's reference time**. In practice that means a stated deadline of 3 November 2026 or earlier, which must have ended by 2026-11-04 04:03 UTC.
  - How a deadline is read: a full date counts to the end of that day; "in October 2026" to the end of October; "in / by 2026" to the end of 2026; "before 2027" to 1 Jan 2027. The latest date in the title counts.
  - **129 questions.** Membership depends on the title, not the outcome.
- **The AI step** (thresholds also in the `.json`, under `validity`):
  - **Retried:** 429 (rate limit), 529 (overloaded), other 5xx, 408, timeouts and connection errors. Each wait is the server's `retry-after` (or `retry-after-ms`) when it sends one, kept within 1–300 seconds; otherwise 15, 30, 60, 120, 240 and then 300 seconds. Retrying stops when the next wait would take the waiting for that task past **30 minutes** (without `retry-after`: 9 retries, about 28 minutes). These errors are not billed, except a timeout or a connection lost mid-request, which is recorded in the spend ledger at its worst-case cost (and counts toward the $12 cap).
  - **Not retried; the run stops and is invalid:** authentication or permission errors (401, 403), billing, credit or spending-limit errors (402, or a billing, credit or spending-limit message), a missing key, or another error raised by the Anthropic client library itself rather than by an HTTP status (for example a response it can't validate). If this happens before the first answer, the run is "aborted before start" (§a) instead.
  - **AI exception:** a task whose AI call still failed after the retries, or failed with an error that is neither retried nor fatal: any other 4xx (400 included, which may be about one question's text) or anything else, counted at once, without retries. That question is forecast market-only, as the pipeline would submit it (blend = crowd), and kept. Five in a row still stop the run (validity rule 3), so a systematic error can't run on.
  - **Refusals, `max_tokens` stops and answers the pipeline can't use** (no Yes number, or one outside 0–1): blend = crowd, kept. Each kind is counted separately in the manifest and in `score.json`, with the misread-guard drops. They do not make the run invalid.
- **News** (`prefetch-news`, before the run, no AI): the time-locked headlines of the run's tasks are fetched and saved under `news_cache/`, one file per task, only when Google answered. The run reads the saved file, re-applies the date rule (time-zone-aware publish time strictly before the as-of time) and recomputes the leak flags each time. The manifest records how many tasks had saved headlines and one sha256 over the saved files (`news_cache_at_start`). The harness refuses the run unless saved headlines cover at least 90% of its tasks (§a). A task with nothing saved is fetched once more during the run. If Google fails but GDELT gives usable headlines, the task runs with those (they are not saved). If Google fails and GDELT gives no usable headline (it failed or found nothing), the task is skipped without an AI call and listed by id in `score.json`. If Google answered, the task runs with whatever passed the date rule, possibly none (the pipeline sees "none found"), as the live forecaster would.
- **Scoring details.** Every variant is clipped to [0.005, 0.995] (the pipeline's `CLIP`) before scoring; the crowd shown to the model is clipped to [0.001, 0.999]. The pinned `score` command is the reference implementation.
- **Validity rules.** The run is **invalid** if any of these holds:
  1. **Dropped share above 10%.** Dropped share = 1 − (scored cohort points ÷ cohort tasks attempted). One cohort question has no crowd value at day 1 (an incomplete trade history), so the minimum is 1/129 (0.8%).
  2. **AI exceptions on more than 5% of the cohort tasks attempted.**
  3. **The run stopped early after its first answer:** 5 AI exceptions in a row (any tasks), an error that retrying can't fix, an interruption (Ctrl-C), or **any budget cap** (the $12 run cap, the $40 `research` month or the $90 overall month).
  4. **The run did not finish:** the manifest has no finish time (the process crashed or was killed).
  - An invalid run gets the verdict "invalid". Its numbers are still reported, and it still uses up the one run. A run "aborted before start" is reported as such and is not counted.

## d. Intervals

**95% intervals on the primary difference**, resampling whole clusters of questions, from one `random.Random(0)` stream: 2,000 resamples for the template interval, then 2,000 for the linked one; each interval runs from the 51st to the 1,950th of the sorted resampled means, rounded to 4 decimals before it is compared with 0:
1. **Template clusters.** Questions that differ only in numbers or dates resample together (`template_key`), e.g. a ladder of gas-price thresholds. The cohort has 92.
2. **Linked-event clusters** (`linked_cluster`), **as stored in the pinned archive's `linked` field**:
   - **One cluster for 2026 US elections** (`us_election_2026`): US-election questions resolving between 1 Aug and 31 Dec 2026.
   - **Everything else:** category × ISO week of resolution.
   - The cohort has 53 linked clusters, 13 of its questions in the US-election cluster.

**"Beats the crowd"** is claimed only if **both** intervals lie wholly below 0.

## e. The verdict (three ways)

| Verdict | Condition |
|---|---|
| **beats** | Both intervals wholly below 0 |
| **worse** | Both intervals wholly above 0 |
| **inconclusive** | Anything else |
| **invalid** | Any validity rule in §c fails (overrides the others) |

If the answer-key probe was suspect (§g), the verdict is reported with the label "possibly contaminated (probe suspect)" and is not a test of forecasting (`score.json`: `probe_suspect`, `verdict_label`).

**Why "inconclusive" is likely, and honest:**
- `blend` is 20% AI and 80% crowd in log-odds, so it can only move a little away from the crowd.
- For scale (no AI involved): on this cohort the crowd beats always-0.5 by 0.024 Brier (128 questions with a crowd value; 91 template and 53 linked clusters), yet even that gap is not significant. The intervals are −0.0167 to +0.0586 (template) and −0.0052 to +0.0511 (linked).
  - These are scored exactly like the primary comparison and reproduce exactly: `uv run python -m firm.lab.timecapsule baselines`, `deadline_cohort` → `always_half-crowd`.
- So a blend that improves on the crowd by a few thousandths will very likely read "inconclusive". We will report it that way and not call it a win.

## f. Secondary results (exploratory, not part of the verdict)

**Comparisons**, each on the points where both of its variants exist, with the number of dropped points:
- `ai − crowd`: the AI's own number;
- `base_rate − crowd`: the pre-cutoff category × shape rate;
- `blend − always_half`.

**Views:**
- on all points;
- on the **no-leak-flag** subset: no flagged headline, and no cut or flagged description.

**Other measures:**
- Log loss is reported for every variant, but is **not** used for any verdict.
- Descriptive only (chosen after the outcome, or small):
  - settled early versus at close;
  - by question shape;
  - crowd between 3% and 97%.

## g. Validity checks (before anything is reported)

1. **Answer-key probe**, before the main run: `uv run python -m firm.lab.timecapsule probe`, with exactly `--n 20 --seed 11 --max-usd 1`.
   - **What it does:** runs 20 deadline-cohort questions through the frozen pipeline with **no headlines and no crowd value**. It costs about $0.29. It needs no news, so `prefetch-news` does not cover it.
   - **Pinned:** the harness refuses other settings and any probe after the pre-registered run. The probe needs the stamped copy, its `.ots` proofs, the key check and $1 of budget room, and uses the same AI-step retries as the run.
   - **Its status** (in `probe.json`, the run log and the run's `score.json`):
     - **finished:** every one of its tasks attempted, not stopped early, and usable AI numbers on at least 10 of its 20 tasks (half);
     - **incomplete:** at least one AI answer, but it stopped early, did not attempt every task, or got usable AI numbers on fewer than 10 tasks; a probe that started and has no end record (it crashed) is incomplete too;
     - **aborted before start:** no AI call returned an answer of any kind.
   - **Only a finished probe can pass, and the pre-registered run needs a finished probe** recorded under the stamped pins file (§a). A probe with fewer than 10 usable numbers is never a pass. Once a probe has finished, no further probe may run. An incomplete or aborted probe does not use up the probe: it may be run again with the same pinned settings (answers already received are reused from the answer cache, which is never cleared before the run, so only the missing tasks are paid for).
   - **At most 3 probes may receive any AI answer** (finished or incomplete; a probe that crashed counts as incomplete even with no answer). If none of them finished, only a new stamped version (v2) can go on. `score.json` lists, for each probe question, how many billed probe calls the spend ledger holds, so a cleared cache would show as a second paid call.
   - Every probe, of every status, is recorded and reported, including any recorded under another or no pins file (listed as not counting).
   - **Pass:** if the model doesn't know the answers, it should score near chance, i.e. a Brier score around 0.25 with about half its numbers on the right side of 50%.
   - **Fail:** a Brier score below 0.20, or 80% or more of the questions with an AI number on the right side of 50% (a number of exactly 50% counts as not right; the denominator includes it), marks the run **suspect**. Its results are then reported as possibly contaminated and do not count as a test of forecasting.
   - **Chance alarms.** With only 20 questions, a model that knows nothing can still be labelled suspect by luck. Computed for this probe sample (7 Yes, 13 No; the sample is fixed by the seed and the cohort): if the model's 20 numbers are drawn independently of the outcomes from a Beta distribution with a mean of 0.30 to 0.50 and a standard deviation of 0.05 to 0.25 (100,000 simulated probes per setting, seed 0), it is labelled suspect at most about 6.5% of the time (the maximum, at mean 0.35, sd 0.175, on a grid of means in steps of 0.05 and standard deviations in steps of 0.025), almost all through the Brier test; the right-side test alone is at most 1.7%. A model with real forecasting skill but no memory of the answers would be flagged more often. So "suspect" means "contamination can't be ruled out", not "contaminated". It still applies as written: the results are reported as possibly contaminated, the probe's numbers are published with them, and telling luck from memory would need a larger probe in a new version.
2. **Manual review.** `score.json` lists every point where the AI's own number beat the crowd's by more than 40 points.
   - Each one is read by hand (headlines, rules text, dates) before reporting.
   - A point found to have leaked information from after its as-of time is reported, together with results **with and without** it. Nothing is removed silently.

## h. Labels and publication

- Every result is labelled **simulated, retrospective, exploratory**, with the crowd = **Manifold play-money markets**.
- These are research about our own forecaster. They never count as trading results.
- **Losses are published.** So are "worse", "inconclusive" and "invalid" verdicts, and runs or probes "aborted before start".
- **Publication follows `decisions/2026-11-18-manifold-data.md`:**
  - summary results only, crediting Manifold;
  - never Manifold market data (no prices, no question-level crowd values);
  - no Kalshi or Polymarket data anywhere.
- **Not for real-money decisions.** Using these results to decide real-money trading would be commercial use: under the Manifold decision we would first stop and ask data@manifold.markets for a licence.
